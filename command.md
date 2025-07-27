 部署 
## 0. miniforge / conda

自行配置，用于后续安装kbqa服务所需库。

## 0.1 clash for linux
为了后续部署能进行外网访问，需安装clash for linux并挂上自己的梯子，教程可参考`https://blog.iswiftai.com/posts/clash-linux/`
## 0.2 git & github
自行配置，用于后续拉取AgentKBQA代码库，一定要用Kiong2002这个用户，只有该用户有权限拉取。
## 1. ES引擎

### 部署

```Bash

mkdir ~/opt/

# JDK

wget -c https://download.oracle.com/java/19/archive/jdk-19.0.2_linux-x64_bin.tar.gz

tar -zxvf jdk-19.0.2_linux-x64_bin.tar.gz -C ~/opt/

export ES_JAVA_HOME=~/opt/jdk-19.0.2


# ES

ES_version=8.7.1

wget "https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-${ES_version}-linux-x86_64.tar.gz"

tar -zxvf elasticsearch-${ES_version}-linux-x86_64.tar.gz

mv elasticsearch-${ES_version} ~/opt/

cd ~/opt/elasticsearch-${ES_version}

# start ES

./bin/elasticsearch


如果出现报错(仅供参考，不一定出错)：

```Bash

bin/elasticsearch

maxvirtualmemoryareasvm.max_map_count [65530] is too low, increase to at least [262144]

```

解决方案：切换root用户修改配置文件并激活

[ES启动异常：max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]-CSDN博客](https://blog.csdn.net/lizz861109/article/details/112512422)


## 2. 图查询工具virtuoso

### 安装virtuoso

以下网页中下载包 `virtuoso-opensource.x86_64-generic_glibc25-linux-gnu.tar.gz`

[OpenLink Virtuoso (Open-Source Edition) download | SourceForge.net](https://sourceforge.net/projects/virtuoso/)

下载后解压，然后这个.tar.gz文件可以删了

```Bash

tar -xvzf virtuoso-opensource.x86_64-generic_glibc25-linux-gnu.tar.gz


# 添加环境路径至~/.bashrc
export VIRTUOSO_HOME=/mnt/bn/maliva-gen-ai-v2/lishilong/projects/Agent-R2/virtuoso-opensource

export PATH=.:${VIRTUOSO_HOME}/bin:$PATH


# 应用当前环境路径 或 重启终端
source ~/.bashrc

```
virtuoso部署完成。
## 3. 部署AgentKBQA项目

### 3.1 clone项目并安装环境
默认已经完成了git以及github的各种配置。一定要指定分支为`dev/kbqa`
```Bash
git clone https://github.com/czhuo9/AgentKBQA.git -b dev/kbqa
```
下载所需pip包
```Bash
# 没有环境先创建：conda create -n agent python=3.10
conda activate agent
pip install -r requierment_server.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```
### 3.2 database setup (for freebase)
推荐服务器配置如下，主要是Memory至少需要完全空闲的80GB。
```
OS: Ubuntu 22.04.2 LTS
CPU: Intel(R) Xeon(R) Silver 4210R CPU @ 2.40GHz
Memory: 128GB
ElasticSearch: 8.7.1 (JDK 19.0.2)
Virtuoso: 07.20.3238 Single Server Edition
ChromaDB: 0.4.22
Python 3.10.10
Torch: 2.1.2+cu118
transformers: 4.37.2
DeepSpeed: 0.13.1
vLLM: 0.3.0
```
database配置过程是为了将知识库索引到virtuoso.db，对于不同的知识库需要分别处理。这部分介绍freebase数据集的索引流程，其他可以参考`https://github.com/JimXiongGM/Interactive-KBQA/blob/main/doc/db_and_tool_setup.md`.

首先下载freebase知识库文件

```Bash
pip install gdown
# 下载fb_filter_eng_fix_literal.gz（记得开启梯子）
gdown "https://drive.google.com/uc?id=1-chTQ-8UzNQOrnsvONAJvPYk5TZ674X0"
```

然后把处理好的文件加载到virtuoso当中。
```Bash

cd ~/AgentKBQA
mkdir database

```

将fb_filter_eng_fix_literal.gz文件移动到AgentKBQA文件夹下（path/to修改为自己保存的路径）

`mv path/to/fb_filter_eng_fix_literal.gz AgentKBQA/fb_filter_eng_fix_literal.gz`

启动virtuoso(最好在tmux内进行)。
注意这里的virtuoso_for_indexing.ini文件并不一定适合其他服务器，在索引GZIP时，建议使用64 GB的内存和SSD。建议根据服务器的规格修改numberOfbuffers和maxdirtybuffers，以使用尽可能多的资源，防止服务被kill。

```Bash

virtuoso-t -df -c tool_prepare/virtuoso_for_indexing.ini

```

切换另一个终端，来到AgentKBQA目录下，执行以下命令：

```Bash

isql 1111 dba dba

```

可能会出现报错：

> isql: error while loading shared libraries: libncurses.so.5: cannot open shared object file: No such file or directory

此时安装对应库即可，然后重新运行上述命令

```Bash

sudo apt install libncurses5

```

此时进入了SQL模式，直接粘贴下面的命令：

```

DB.DBA.TTLP_MT(gz_file_open ('fb_filter_eng_fix_literal.gz'), '', 'http://freebase.com', 128);

checkpoint;

exit;

```

该界面不会有任何响应，切换到启动virtuoso的终端，会有一些输出。等待10~20小时后indexing结束，会有以下输出：

```

SQL> SPARQL select count(*) from <http://freebase.com> where {?s ?r ?o};

callret-0

INTEGER

_______________________________________________________________________________


954533166


1 Rows. -- 2430 msec

```
知识库索引完成。默认在端口9501上启动FreeBase数据库，可以在http:// localhost:9501/sparql 访问它

### 3.3 配置ES引擎(用于searchNodes调用)
注意，这部分是为了进行实体链接工具调用，如果不需要使用改工具，可以跳过。
安装部署好ES后，进行indexing。

```

# 首先修改indexing时的内存设置

# modify config/jvm.options:

# when indexing: -Xms12g -Xmx12g; when searching: -Xms4g -Xmx4g


# 然后运行项目脚本，进行Indexing，大概需要30分钟

python tool/searchnode_fb.py

```

这一过程可能会产生报错，无法连接ES引擎，应该是由于客户端的安全性限制，修改ES配置文件即可。

```

# 修改elasticsearch.yml配置文件，将xpack.security.enabled设置为false

xpack.security.enabled: false

```

随后重新启动ES引擎，并运行上述脚本即可。

### 3.4 tool setup (for freebase)
对于不同的知识库需要分别处理。这部分介绍freebase数据集的工具部署流程，其他可以参考`https://github.com/JimXiongGM/Interactive-KBQA/blob/main/doc/db_and_tool_setup.md`.

首先预处理实体名和关系谓词。
```Bash
# 必要！不要直接进入~/AgentKBQA/tool等子文件夹，会造成文件结构混乱，导致脚本无法执行
cd ~/AgentKBQA
python tool_prepare/fb_cache_entity_en.py
python tool_prepare/fb_cache_predicate.py 
```
上述命令会产生以下文件：
```
database/freebase-info/freebase_entity_name_en.txt
database/freebase-info/predicate_freq.json
database/freebase-info/cvt_predicate_onehop.jsonl
```
随后进行实体链接工具构建。
**注意，以下步骤是为了构建实体链接工具，如不需要，可跳过。**


1. 在`https://github.com/dki-lab/GrailQA/tree/main/entity_linker/data`下载facc文件，将文件移动至：`database/freebase-info/surface_map_file_freebase_complete_all_mention`
2. 运行脚本`python tool/searchnode_fb.py`，如果顺利，会出现以下输出：
```
['Southern Peninsula', 'The Southern Arabian Peninsula', 'Southern Peninsular Malaysian Hokkien', 'Southern Yorke Peninsula Christian College', 'Peninsula']
{'count': 22767150, '_shards': {'total': 1, 'successful': 1, 'skipped': 0, 'failed': 0}}
```
3. 将facc文件缓存，运行`python tool_prepare/facc1.py`，如果顺利，会出现以下输出：
```
['m.0fs04cs', 'm.0zjywz4', 'm.0mtgjq8', 'm.0dss9vb', 'm.0msygvy']
```

然后进行谓词缓存。首先打开`tool/openai.py`文件夹，把default_key换成你购买的embedding api。
```Bash
python tool_prepare/fb_vectorization.py
```
最后进行类型缓存。
```Bash
python tool_prepare/fb_vectorize_type.py
```

tool setup完成，使用以下命令启动工具服务：
```Bash
# 最好在tmux内进行
python api/api_dbtool_server.py --db fb
```
切换终端，使用以下命令测试：
```Bash
curl -X POST "http://localhost:9905/fb/query" \
    -H "Content-Type: application/x-www-form-urlencoded" \
    -d "sparql=SELECT (?x0 AS ?value) WHERE {\nSELECT DISTINCT ?x0  WHERE { \n?x0 :type.object.type :rail.railway . \nVALUES ?x1 { :m.0nqv1 } \n?x0 :rail.railway.terminuses ?x1 . \nFILTER ( ?x0 != ?x1  )\n}\n}" \
    -w "\nTotal time: %{time_total}s\n"
```
### 注意事项
* 如果出现file not exist错误，去对应脚本内检查路径是否正确，如`with gzip.open("database/fb_filter_eng_fix_literal.gz") as f1:`,你的文件可能在`./fb_filter_eng_fix_literal.gz`，移动文件或修改脚本内的路径都可以。
* 无论是debug还是配置，执行脚本时都应该在`~/AgentKBQA`下，进入子目录会导致缓存的向量数据库找不到，出现各种报错。
* 需要时刻关注磁盘空间，用尽可能导致vscode无法进行远程连接，此时可以ssh连接到服务器，`rm -rf`大文件腾出空间。（一定要看好再删）
# 知识库服务启动概览
完成上述步骤后，可以按照以下步骤顺序启动知识库服务。

```Bash

# 启动知识图谱数据库

cd ~/AgentKBQA

virtuoso-t -df -c tool_prepare/virtuoso_for_indexing.ini

# 启动elasticsearch

cd ~/opt/elasticsearch

./bin/elasticsearch

# 启动freebase服务

conda activate agent

cd ~/AgentKBQA

python api/api_dbtool_server.p --db fb

```