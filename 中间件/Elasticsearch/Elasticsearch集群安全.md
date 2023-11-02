# Elasticsearch集群安全

## X_PACK

### X_PACK功能信息

```bash
curl -X GET "http://elastic:MhxzKhl2021@172.16.1.10:9292/_xpack?pretty"

```

### 认证

```bash
curl -X GET "http://elastic:MhxzKhl2021@172.16.1.10:9292/_security/_authenticate?pretty"
```


## 设置用户角色访问Elasticsearch的控制

```bash
# 角色
curl -XPOST -u elastic 'localhost:9200/_security/role/events_admin' -H "Content-Type: application/json" -d '{
  "indices" : [
    {
      "names" : [ "events*" ],
      "privileges" : [ "all" ]
    },
    {
      "names" : [ ".kibana*" ],
      "privileges" : [ "manage", "read", "index" ]
    }
  ]
}'

# 用户
curl -XPOST -u elastic 'localhost:9200/_security/user/johndoe' -H "Content-Type: application/json" -d '{
  "password" : "userpassword",
  "full_name" : "John Doe",
  "email" : "john.doe@anony.mous",
  "roles" : [ "events_admin" ]
}'
```