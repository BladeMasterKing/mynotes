# CURL执行Elasticsearch操作


## 修改ES数据只读

ES服务器磁盘满了之后会自动开启只读模式，需要手动将模式改回正常,按下面方式执行就能解决。

```shell
PUT {index}/_settings

{
    "index":{
        "blocks":{
            "read_only_allow_delete":"false"
        }
    }
}
```


由于给用户讲解，使用可视化工具给小白讲相当费劲，所以想到了使用CURL直接使用命令行ctrl+c ctrl+v 的方式。

```shell
curl -H "Content-Type: application/json" -XPUT http://localhost:9200/*/_settings?pretty -d '{"index":{"blocks":{"read_only_allow_delete":"false"}}}'
```

一键执行即可成功搞定。

