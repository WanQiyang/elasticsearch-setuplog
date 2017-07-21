# Elasticsearch 安装记录

## Elasticsearch 介绍
Elasticsearch是一个基于Java的实时分布式搜索和分析引擎，能够对数据进行实时处理、分析。<br>
本次安装的操作系统环境为**Ubuntu 16.04 64bit**，将按照[《Elasticsearch权威指南》](https://github.com/looly/elasticsearch-definitive-guide-cn)中的流程进行。

## Java环境
在终端输入
```sh
sudo apt-get install default-jre
```
安装最新版本的Java运行环境。安装结束后可输入
```sh
java -version
```
检查是否安装成功。如果安装成功，会返回Java的版本号，如1.8.0_131。<br><br>
*注：需要注意的是，如果Ubuntu的更新源选择不恰当，极有可能安装失败。aliyun的源实测可用，建议在安装前换源。*

## 运行Elasticsearch
### 方式一：通过下载安装包
从官方网站 https://www.elastic.co/downloads/elasticsearch 下载最新版本的Elasticsearch。<br><br>
拷贝tar包至工作目录后，输入
```sh
tar -xvf elasticsearch-$VERSION.tar.gz
```
解压文件，其中将$VERSION替换为当前版本号。<br><br>
输入
```sh
./elasticsearch-$VERSION/bin/elasticsearch
```
运行Elasticsearch。<br><br>
*注：如果需要在后台运行，添加命令行参数-d即可。*<br><br>
稍等片刻，运行成功会显示
```sh
[0] indices into cluster_state
```

### 方式二：通过Debian Package
部分操作与后续安装Kibana相同，执行一次即可。<br><br>
默认安装目录为 /usr/share/elasticsearch/。<br><br>
Import the Elasticsearch PGP Key
```sh
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```
Installing from the APT repository
```sh
sudo apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-5.x.list
```
Install the Elasticsearch Debian package
```sh
sudo apt-get update && sudo apt-get install elasticsearch
```

需要注意的是，此时不能以root权限直接运行可执行文件，而应通过服务启动：
```sh
service elasticsearch start
```

### 验证安装
此时另开启一个终端，输入
```sh
curl 'http://localhost:9200'
```
检验Elasticsearch是否运行，若正在运行则会返回（文本可能由于版本差异不同）
```json
{
    "name" : "RcmBSMl",
    "cluster_name" : "elasticsearch",
    "cluster_uuid" : "M6u8eyc8QF6cIAWOh0RkMQ",
    "version" : {
        "number" : "5.5.0",
        "build_bash" : "260387d",
        "build_date" : "2017-06-30T23:16:05.735Z",
        "build_snapshot" : false,
        "lucene_version" : "6.6.0"
    },
    "tagline" : "You Know, for Search"
}
```

## 运行Demo
Elasticsearch数据库可使用HTML的GET、PUT等方法访问和修改。

### PUT添加数据
```sh
curl -T $JSON_FILE "http://localhost:9200/$INDEX/$TYPE/$ID"
```
其中$JSON_FILE是json数据文件，$INDEX是索引名，$TYPE是类型名，$ID是记录编号。<br><br>
另外，若相同编号的记录已存在，PUT操作则会更新之。

### GET访问数据
```sh
curl -XGET "http://localhost:9200/$INDEX/$TYPE/$ID"
```

### GET无条件查询数据
```sh
curl -XGET "http://localhost:9200/$INDEX/$TYPE/_search"
```
该操作会列出同索引同类型的所有记录。

### GET条件查询数据
```sh
curl -XGET "http://localhost:9200/$INDEX/$TYPE/_search?q=$COND:$VALUE"
```
其中$COND是条件名，$VALUE是条件值，该操作会列出同索引同类型符合条件的记录。

### DELETE删除数据
```sh
curl -XDELETE "http://localhost:9200/$INDEX/$TYPE/$ID"
```
<br>
更多方法详见 [官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html) 。
