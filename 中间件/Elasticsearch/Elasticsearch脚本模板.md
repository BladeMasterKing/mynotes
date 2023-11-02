# Elasticsearch脚本

## 7.x版本

### 索引API
```bash
curl -X GET "http://elastic:MhxzKhl2021@172.16.1.10:9292/_all/_settings?pretty"
```


### 计算集群中的文档数量

### 设置索引mapping


### 根据查询更新
```bash
curl -X POST "http://host:port{index}/_update_by_query"
{
  "script": {
    "source": "ctx._source['field_name'] = 'field_value'",
    "lang": "painless"
  },
  "query": {
    "term": {
      "field_name": "field_value"
    }
  }
}
```

### 删除字段
```shell
POST {index}/_update_by_query
{
  "script": {
    "source": "ctx._source.remove('field_name')",
    "lang": "painless"
  },
  "query": {
    "term": {
      "field_name": "field_value"
    }
  }
}
```

### 虚拟列聚合
```shell
POST {index}/_search
{
  "aggs": {
    "genres": {
      "terms": {
        "script": {
          "source": "doc['sip'].value + '@'+doc['dip'].value",
          "lang": "painless"
        }
      }
    }
  }
}
```
