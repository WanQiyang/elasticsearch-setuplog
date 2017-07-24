# Elasticsearch 数据的批量导入
## 准备工作
### 环境
操作系统：Ubuntu 16.04 64bit<br><br>
编译环境：Elasticsearch 5.5.0 | Python 2.7.12

### 工具安装
请在管理员权限下执行下面的命令，安装pyelasticsearch：
```sh
pip install pyelasticsearch
```
为了导入示例中csv格式的数据，我们用下面的命令安装pandas：
```sh
pip install pandas
```

## 示例数据
### 数据下载
上证A股日线数据，1999.12.09 至 2016.06.08，前复权，1095支股票；<br><br>
[下载地址](http://dataju.cn/Dataju/web/datasetInstanceDetail/37)

### 数据格式
解压后为1095个文件名为\######.SH.CSV的数据文件（其中\######为股票数字代码），每个文件记录一支股票的日线数据；
每行各列以逗号分隔，分别表示
股票代码、简称、日期、前收盘价(元)、开盘价(元)、最高价(元)、最低价(元)、收盘价(元)、成交量(股)、成交金额(元)、涨跌(元)、涨跌幅(%)、均价(元)、
换手率(%)、A股流通市值(元)、B股流通市值(元)、总市值(元)、A股流通股本(股)、B股流通股本(股)、总股本(股)、市盈率、市净率、市销率、市现率。<br><br>
数据包含中文，采用GB18030编码。<br><br>

## 批量导入
以下代码默认数据位于目录./A/中。保持Elasticsearch正常运行，新建文件import.py：
```python
import pandas as pd
from pyelasticsearch import ElasticSearch
import os

CHUNKSIZE = 512  # 块大小

index_name = "sh"  # 索引名
doc_type = "stock"  # 类型名

def index_data(es, data_path, chunksize, index_name, doc_type):
  f = open(data_path)
  csvfile = pd.read_csv(f, encoding='GB18030', iterator=True, chunksize=chunksize) 
  for i, df in enumerate(csvfile): 
    records = df.where(pd.notnull(df), None).T.to_dict()
    list_records = [records[it] for it in records]
    try :
      es.bulk_index(index_name, doc_type, list_records)  # 使用bulk接口批量导入数据
      # print(list_records)
    except :
      print("error!, skiping chunk!")
      pass

def GetFileList(dir, fileList):  # 枚举文件
  newDir = dir
  if os.path.isfile(dir):
    fileList.append(dir.decode('gbk'))
  elif os.path.isdir(dir):  
    for s in os.listdir(dir):
      newDir=os.path.join(dir,s)
      GetFileList(newDir, fileList)  
  return fileList

if __name__ == '__main__':
  es = ElasticSearch('http://localhost:9200/')  # 若目标为远程服务器上的Elasticsearch，则修改为相应地址
  try :
    es.delete_index(index_name)  # 删除同名索引
  except :
    pass
  es.create_index(index_name)  # 创建索引

  list = GetFileList('./A', [])
  for e in list:
    print e
    index_data(es, e, CHUNKSIZE, index_name, doc_type)

```
执行命令
```sh
python import.py
```
即可完成数据的批量导入。<br><br>
全部导入后使用浏览器访问页面 http://localhost:9200/sh/stock/_search （将地址改为Elasticsearch的地址）可验证数据已导入成功，显示共有3612326条记录。

