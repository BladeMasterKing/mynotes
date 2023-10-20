# Elasticsearch脚本

## 7.x版本
### 根据查询更新
```shell
POST {index}/_update_by_query
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
