

## 字段默认值为null的情况查询

```
Customer 表:
+----+------+------------+
| id | name | referee_id |
+----+------+------------+
| 1  | Will | null       |
| 2  | Jane | null       |
| 3  | Alex | 2          |
| 4  | Bill | null       |
| 5  | Zack | 1          |
| 6  | Mark | 2          |
+----+------+------------+
```



1. 使用推荐人不等于2过滤名称


 ```sql
 select name from Customer where referee_id != 2
 
 | name |
 | ---- |
 | Zack |
 ```

   过滤结果只有`referee_id`字段值为1的一行数据，值为null的数据并没有查询到

原因：mysql中认为null为没有值，所以使用值不等于查询会自动过滤掉

解决：可以在sql中给null值的字段赋一个业务无关的值来查询

```sql
select name from Customer where IFNULL(referee_id, -1) != 2

| name |
| ---- |
| Will |
| Jane |
| Bill |
| Zack |
```

查询结果就包含值为null的数据行了

所以业务中避免出现字段默认值为null的情况