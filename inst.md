## KBQA服务部分
1. 确定virtuoso是否健在：执行好以下命令，tmux终端内始终没有出现exit等情况就可以。
```bash
# 添加环境路径至~/.bashrc
export VIRTUOSO_HOME=/workspace/home/czhuo/virtuoso-opensource
export PATH=.:${VIRTUOSO_HOME}/bin:$PATH
source ~/.bashrc
# 新建tmux终端
cd path/to/AgentKBQA
virtuoso-t -df -c tool_prepare/virtuoso_for_indexing.ini
```
2. 启动fb服务，执行最后一条命令正常返回一些带有education字符的数组，可确认启动成功。
```bash
cd path/to/AgentKBQA
conda create -n agent python=3.10
pip install -r requirements_server.txt
# 新建tmux终端
python api/api_db_server.py --db fb
# 切换到任意终端
curl -X POST "http://localhost:9901/fb/SearchTypes"     -H "Content-Type: application/x-www-form-urlencoded"     -d "query=education"     -w "\nTotal time: %{time_total}s\n"
```

## SFT部分
1. cd到LLaMA-Factory， 接受邮件邀请、关联上远程仓库、拉取main分支：https://github.com/czhuo9/KBQA-LLaMA-Factory;
2. 下载数据集：https://drive.google.com/file/d/1D0bDZEuvEF1qJnf2zDGv8X8lXHGl4v1w/view?usp=sharing，移动到path/to/LLaMA-Factory/data文件夹下的cold_start文件夹。
3. 修改path/to/LLaMA-Factory/examples/train_lora/qwen25_lora_sft.yaml文件中的model_name_or_path为Qwen2.5-Coder-7B模型路径。
4. 配置环境并运行：
```bash
cd path/to/LLaMA-Factory
conda create -n sft python=3.10
conda activate sft
pip install -e ".[torch,metrics]" --no-build-isolation
# 若有多张卡，修改scripts/train.sh中：export CUDA_VISIBLE_DEVICES="7"为多卡，去掉export FORCE_TORCHRUN=1这一行
bash scripts/train.sh
```
5. （可选）评测ckpt：今晚（7.24）先不评，没时间了

* cd到path/to/Agent-R1/kbr1/,创建sft_evaluation.sh文件，粘贴以下内容：
```Bash
#!/bin/bash
# 基础模型路径 (合并时作为底模)
SFT_BASE_MODEL_PATH="/data3/chenzhuo/models/Qwen/Qwen2.5-Coder-7B"

# SFT检查点(lora)的根目录
SFT_CHECKPOINT_BASE_DIR="/data3/chenzhuo/LLaMA-Factory/saves/Qwen2.5-Coder-7B/cold_start_lora/cold_start/multi_turn_sft_data_1543_20250724-15_max_4k"


# 用于临时存放合并后模型的根目录 (脚本会自动创建和清理)
TEMP_MERGE_BASE_DIR="./tmp/auto_eval_merged_sft_models"

# 评测相关设置
DATASETS=("grailqa" "webqsp" "cwq")

EVAL_SCRIPT="evaluation_official.py"
GPU_DEVICE="5"
PORT="18001"
SAVE_FILE_PREFIX="test_0724_sft"


# --- 2. 脚本初始化与清理函数 ---

export CUDA_VISIBLE_DEVICES=${GPU_DEVICE}
mkdir -p ${TEMP_MERGE_BASE_DIR}

VLLM_PID=""
CURRENT_TEMP_DIR=""

# 定义清理函数，用于在脚本意外退出时执行
cleanup() {
    echo ""
    echo "---"
    echo "捕获到退出信号，正在执行紧急清理..."
    if [ -n "$VLLM_PID" ] && ps -p $VLLM_PID > /dev/null; then
        echo "正在关闭 vLLM 服务 (PID: $VLLM_PID)..."
        kill $VLLM_PID
        wait $VLLM_PID 2>/dev/null
    fi

    if [ -n "$CURRENT_TEMP_DIR" ] && [ -d "$CURRENT_TEMP_DIR" ]; then
        echo "正在删除临时模型目录: $CURRENT_TEMP_DIR"
        rm -rf "$CURRENT_TEMP_DIR"
    fi
    echo "清理完毕。"
    exit 1
}

# 设置陷阱(trap)，确保按 Ctrl+C 时也能调用 cleanup 函数
trap cleanup SIGINT SIGTERM


# --- 3. 主评测循环 ---

# 外层循环: 遍历所有检查点步骤
for i in $(seq 2400 800 7200)
do
    CKPT_NAME="ckpt${i}"
    LORA_PATH="${SFT_CHECKPOINT_BASE_DIR}/checkpoint-${i}"
    CURRENT_TEMP_DIR="${TEMP_MERGE_BASE_DIR}/${CKPT_NAME}"

    echo "============================================================"
    echo "开始处理检查点: ${CKPT_NAME}"
    echo "============================================================"

    # ==================== CRITICAL FIX ====================
    # 修复: 使用正确的变量 $LORA_PATH 检查目录，而不是未定义的 $RL_ACTOR_PATH
    if [ ! -d "$LORA_PATH" ]; then
        echo "错误: 找不到 LoRA 检查点目录: ${LORA_PATH}，跳过此检查点。"
        continue
    fi
    # ======================================================

    # 步骤 1: 合并模型 (每个检查点执行一次)
    echo "[步骤 1/4] 正在合并模型权重..."
    mkdir -p "${CURRENT_TEMP_DIR}"
    llamafactory-cli export  \
        --model_name_or_path ${SFT_BASE_MODEL_PATH} \
        --adapter_name_or_path ${LORA_PATH} \
        --template qwen \
        --trust_remote_code true \
        --export_dir ${CURRENT_TEMP_DIR} \
        --export_size 5 \
        --export_device cpu \
        --export_legacy_format false
    if [ $? -ne 0 ]; then
        echo "错误: 模型合并失败！请检查 merger 脚本的输出。"
        rm -rf "${CURRENT_TEMP_DIR}"
        continue
    fi
    echo "模型合并成功。"

    # 步骤 2: 启动 vLLM 服务 (每个检查点执行一次)
    echo "[步骤 2/4] 正在启动 vLLM 服务..."
    vllm serve ${CURRENT_TEMP_DIR} --enable-auto-tool-choice --tool-call-parser hermes --served-model-name agent --port ${PORT} --max-model-len 8192 --gpu-memory-utilization 0.7 > "vllm_server_${CKPT_NAME}.log" 2>&1 &
    VLLM_PID=$!
    echo "vLLM 服务已在后台启动，PID 为: ${VLLM_PID}"

    # 步骤 3: 等待 vLLM 服务就绪 (每个检查点执行一次)
    echo "[步骤 3/4] 正在等待 vLLM 服务就绪..."
    # (此处的等待逻辑是健壮且正确的)
    while ! curl -s --fail "http://localhost:${PORT}/health" > /dev/null; do
        if ! ps -p $VLLM_PID > /dev/null; then
            echo "错误: vLLM 服务启动失败！请查看日志 vllm_server_${CKPT_NAME}.log"
            rm -rf "${CURRENT_TEMP_DIR}"
            continue 2
        fi
        sleep 2
    done
    echo "vLLM 服务已就绪。"

    # --- 步骤 4: 运行评测 ---
    # !! 优化点：内层循环，遍历所有数据集，复用同一个vLLM服务 !!
    echo "[步骤 4/4] 开始对所有数据集进行评测..."
    EVAL_MODEL_NAME="qwen2.5_coder_7b_v3_cold_start${i}"
    
    for dataset in "${DATASETS[@]}"
    do
        echo "  -> 正在评测数据集: ${dataset}"
        # 动态生成包含数据集名称的保存文件名
        SAVE_FILE_NAME="${SAVE_FILE_PREFIX}_${CKPT_NAME}_${dataset}"
        
        python ${EVAL_SCRIPT} \
            -d ${dataset} \
            --model_name ${EVAL_MODEL_NAME} \
            -m 10 \
            --save_file ${SAVE_FILE_NAME} \
            --mode test
        
        echo "  -> 数据集 [${dataset}] 评测完成。"
    done
    echo "所有数据集评测完毕。"

    # 步骤 5: 清理当前检查点的资源
    echo "正在清理资源..."
    echo "  - 正在关闭 vLLM 服务 (PID: $VLLM_PID)..."
    kill $VLLM_PID
    wait $VLLM_PID 2>/dev/null
    unset VLLM_PID
    echo "  - 正在删除临时模型目录: ${CURRENT_TEMP_DIR}"
    rm -rf "${CURRENT_TEMP_DIR}"
    unset CURRENT_TEMP_DIR
    echo "清理完成。"
    
    sleep 5
done

# --- 最终清理 ---
echo "============================================================"
echo "所有检查点评测完毕！正在清理根临时目录..."
rm -rf "${TEMP_MERGE_BASE_DIR}"
echo "脚本执行结束。"
```
* 将上述脚本中的SFT_BASE_MODEL_PATH，SFT_CHECKPOINT_BASE_DIR，for i in $(seq 2400 800 7200)修改为想要评测的ckpt目录，和step范围。
* 运行评测,查看结果，确定最好的sft ckpt.
```bash
cd path/to/Agent-R1/kbr1/
bash sft_evaluation.sh
```
6. 合并训练好的模型

* 直接选择第7-8个epoch内的ckpt作为要合并的adapter。（总步数*0.7/0.8对应的step）

* 修改path/to/LLaMA-Factory/examples/merge_lora/qwen25_lora.yaml中的adapter_name_or_path， export_dir(应该是/mnt/bn/maliva-gen-ai-v2/lishilong/ckpts)为选定位置。
```bash
llamafactory-cli export examples/merge_lora/qwen25_lora.yaml
```
## RL部分
1. 配置环境
```bash
cd path/to/Agent-R1
conda create -n verl python==3.10
conda activate verl
cd verl
pip3 install -e .
# Install the latest stable version of vLLM
pip3 install vllm
# Install flash-attn
pip3 install flash-attn --no-build-isolation
```
2. 找到*从头运行而非resume*的脚本（可能是kbqa.sh）中的BASE_MODEL='/mnt/bn/maliva-gen-ai-v2/lishilong/ckpts/xxx'(刚合并的模型)，并运行该脚本。
```Bash
cd path/to/Agent-R1
bash examples/trainer/kbqa.sh
```