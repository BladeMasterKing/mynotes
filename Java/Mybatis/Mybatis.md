# Mybatis IN 查询

```java
// dao
List<ProductAgentDO> queryByGroupId(@Param("groupIds") List<Long> groupIds) throws Exception;
```

```xml
<select id="queryByGroupId" resultMap="base_result_map">
    select
    <include refid="base_column_list"/>
    from t_product_agent
    where group_id in
    <foreach collection="groupIds" item="groupId" index="index" open="(" close=")" separator=",">
        #{groupId}
    </foreach>
</select>
```

`collection` 中对应 `param` 参数名，`item` 对应从集合中取的值 `#{}`


> mysql 表关联查询

在MySQL中，可以使用JOIN语句进行表关联查询。JOIN语句允许你在查询中联合多个表，根据它们之间的关联条件获取相关的数据。

以下是一些常见的JOIN语句类型：

1.  内连接（INNER JOIN）：返回两个表中满足关联条件的行。
2.  左连接（LEFT JOIN）：返回左表中的所有行，以及满足关联条件的右表中的匹配行。
3.  右连接（RIGHT JOIN）：返回右表中的所有行，以及满足关联条件的左表中的匹配行。
4.  全连接（FULL JOIN）：返回左表和右表中的所有行，如果没有匹配的行，则填充NULL。

下面是一个示例，演示如何使用JOIN语句进行表关联查询：

假设我们有两个表：`Customers`和`Orders`。

**Customers表**：

```
| CustomerID | CustomerName |
| --- | --- |
| 1 | John |
| 2 | Alice |
```

**Orders表**：

```
| OrderID | CustomerID | OrderName |
| --- | --- | --- |
| 1 | 1 | Order 1 |
| 2 | 1 | Order 2 |
| 3 | 2 | Order 3 |
```

以下是一些常见的表关联查询的示例：

1.  内连接（INNER JOIN）：

sql

```sql
SELECT Customers.CustomerName, Orders.OrderName
FROM Customers
INNER JOIN Orders ON Customers.CustomerID = Orders.CustomerID;
```

这条查询语句将返回`Customers`表和`Orders`表中满足关联条件（`Customers.CustomerID = Orders.CustomerID`）的行。结果将包含`CustomerName`和`OrderName`两列。

2.  左连接（LEFT JOIN）：

sql

```sql
SELECT Customers.CustomerName, Orders.OrderName
FROM Customers
LEFT JOIN Orders ON Customers.CustomerID = Orders.CustomerID;
```

这条查询语句将返回`Customers`表中的所有行，以及满足关联条件（`Customers.CustomerID = Orders.CustomerID`）的`Orders`表中的匹配行。如果没有匹配的行，则`OrderName`列将填充为NULL。

3.  右连接（RIGHT JOIN）：

sql

```sql
SELECT Customers.CustomerName, Orders.OrderName
FROM Customers
RIGHT JOIN Orders ON Customers.CustomerID = Orders.CustomerID;
```

这条查询语句将返回`Orders`表中的所有行，以及满足关联条件（`Customers.CustomerID = Orders.CustomerID`）的`Customers`表中的匹配行。如果没有匹配的行，则`CustomerName`列将填充为NULL。

4.  全连接（FULL JOIN）：

MySQL不直接支持FULL JOIN语句。你可以使用LEFT JOIN和RIGHT JOIN的组合来模拟FULL JOIN。

sql

```sql
SELECT Customers.CustomerName, Orders.OrderName
FROM Customers
LEFT JOIN Orders ON Customers.CustomerID = Orders.CustomerID
UNION
SELECT Customers.CustomerName, Orders.OrderName
FROM Customers
RIGHT JOIN Orders ON Customers.CustomerID = Orders.CustomerID;
```

这条查询语句使用UNION操作符将LEFT JOIN和RIGHT JOIN的结果合并，得到一个包含`Customers`表和`Orders`表中的所有行的结果。

这只是一些基本的表关联查询示例。根据具体的数据模型和查询需求，你可以使用不同类型的JOIN语句和适当的关联条件来编写表关联查询。

> 左外连接、右外连接、内连接的区别示例

下面我将为你提供左外连接、右外连接和内连接的区别示例。

假设我们有两个表：`Customers`和`Orders`。

**Customers表**：

```
| CustomerID | CustomerName |
| --- | --- |
| 1 | John |
| 2 | Alice |
| 3 | Bob |
```

**Orders表**：

```
| OrderID | CustomerID | OrderName |
| --- | --- | --- |
| 1 | 1 | Order 1 |
| 2 | 1 | Order 2 |
| 3 | 2 | Order 3 |
```

1.  左外连接（LEFT OUTER JOIN）：

左外连接将返回左表的所有行，以及满足关联条件的右表的匹配行。如果没有匹配的行，右表的列将填充为NULL。

示例查询：

sql

```sql
SELECT Customers.CustomerName, Orders.OrderName
FROM Customers
LEFT JOIN Orders ON Customers.CustomerID = Orders.CustomerID;
```

结果：

```
| CustomerName | OrderName |
| --- | --- |
| John | Order 1 |
| John | Order 2 |
| Alice | Order 3 |
| Bob | NULL |
```

2.  右外连接（RIGHT OUTER JOIN）：

右外连接将返回右表的所有行，以及满足关联条件的左表的匹配行。如果没有匹配的行，左表的列将填充为NULL。

示例查询：

sql

```sql
SELECT Customers.CustomerName, Orders.OrderName
FROM Customers
RIGHT JOIN Orders ON Customers.CustomerID = Orders.CustomerID;
```

结果：

```
| CustomerName | OrderName |
| --- | --- |
| John | Order 1 |
| John | Order 2 |
| Alice | Order 3 |
| NULL | NULL |
```

3.  内连接（INNER JOIN）：

内连接将返回满足关联条件的左表和右表的匹配行，不包括任何未匹配的行。

示例查询：

sql

```sql
SELECT Customers.CustomerName, Orders.OrderName
FROM Customers
INNER JOIN Orders ON Customers.CustomerID = Orders.CustomerID;
```

结果：

```
| CustomerName | OrderName |
| --- | --- |
| John | Order 1 |
| John | Order 2 |
| Alice | Order 3 |
```

在上述示例中，左外连接返回了`Customers`表的所有行，并根据关联条件匹配了`Orders`表的行，未匹配的行填充为NULL。右外连接则返回了`Orders`表的所有行，并根据关联条件匹配了`Customers`表的行，未匹配的行填充为NULL。而内连接仅返回满足关联条件的匹配行。

这些不同类型的连接可以根据数据模型和查询需求来选择使用，以确保返回结果符合预期并满足查询要求。