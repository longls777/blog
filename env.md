``````bash
# 添加环境路径至~/.bashrc
export VIRTUOSO_HOME=/mnt/bn/maliva-gen-ai-v2/lishilong/projects/Agent-R2/virtuoso-opensource
export PATH=.:${VIRTUOSO_HOME}/bin:$PATH
# 应用当前环境路径 或 重启终端
source ~/.bashrc

cd /mnt/bn/maliva-gen-ai-v2/lishilong/projects/Agent-R2/AgentKBQA
virtuoso-t -df -c tool_prepare/virtuoso_for_indexing.ini

pip install -r requierment_server.txt 


unset http_proxy
unset https_proxy
unset ftp_proxy
unset no_proxy
unset HTTP_PROXY
unset HTTPS_PROXY
unset FTP_PROXY
unset NO_PROXY
python api/api_dbtool_server.py --db fb

``````



`````bash
cd verl
pip3 install -e .

# Install the latest stable version of vLLM
pip3 install vllm

# Install flash-attn
pip3 install flash-attn --no-build-isolation
`````

