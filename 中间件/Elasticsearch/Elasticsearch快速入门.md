# Elasticsearch快速入门

## 下载ES

```bash
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.13.4-darwin-x86_64.tar.gz
# 校验码校验
# wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.13.4-darwin-x86_64.tar.gz.sha512
# shasum -a 512 -c elasticsearch-7.13.4-darwin-x86_64.tar.gz.sha512 
tar -xzf elasticsearch-7.13.4-darwin-x86_64.tar.gz
cd elasticsearch-7.13.4/
```

## ES启动

### 命令行启动

```bash
./bin/elasticsearch
```
默认前台启动，关闭通过`ctrl+c`来关闭

### 作为守护进程启动

```bash
./bin/elasticsearch -d -p pid
```
在ES根目录生成pid文件，存储ES程序的进程ID，关闭时通过pid来关闭
通过`pkill`指定文件来关闭

```bash
pkill -F pid
```

## ES配置

### 配置文件

在 `$ES_HOME`目录中`config`目录下

- `elasticsearch.yml`用来配置ES
- `jvm.options`配置ES运行的虚拟机
- `log4j.properties`配置ES运行的日志

### 重要的ES配置

集群名称：cluster.name
节点名称：node.name
集群发现：discovery.seed_hosts

### 节点配置

节点角色

- master
- data

仅投票 `voting_only`

### 安装ik分词器

从[Github](https://github.com/medcl/elasticsearch-analysis-ik)上下载对应elasticsearch的版本的ik分词器，
解压后将文件夹放在`ES_Home`下名字为`plugin`的目录下
然后重启ES，执行命令查看插件是否安装成功
```bash
$ ./bin/elasticsearch-plugin list
ik
```

## 创建索引

### 显式映射创建索引

创建索引时设置索引
```bash
curl -X PUT "localhost:9200/test?pretty" -H 'Content-Type: application/json' -d'
{
  "settings": {
    "number_of_shards": 1
  },
  "mappings": {
    "properties": {
      "field1": { "type": "text" }
    }
  }
}
'
```

```bash
curl -X PUT "root:talent@localhost:9292/dingban_product?pretty" -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "properties": {
        "id":    { "type": "long" }, 
      "accountId":    { "type": "long" },  
      "groupId":  { "type": "long"  }, 
      "name":   { "type": "text", "analyzer" : "ik_syno_max"  }, 
      "brand":   { "type": "text", "analyzer" : "ik_syno_max"  },
      "canGive":    { "type": "integer" },
      "serviceTypeId":    { "type": "long" },
      "photoUrl":    { "type": "keyword" },
      "selectJson":    { "type": "keyword" },
      "hasTalents":    { "type": "integer" },
      "status":    { "type": "integer" },
    "addTime": {
        "type":   "date",
        "format": "yyyy-MM-dd HH:mm:ss"
      },
      "updateTime": {
        "type":   "date",
        "format": "yyyy-MM-dd HH:mm:ss"
      }
    }
  }
}
'
```