# LeetCode高频SQL50题

## 查询

### 1. 可回收且低脂的产品

`Products`表：

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| product_id  | int     |
| low_fats    | enum    |
| recyclable  | enum    |
+-------------+---------+
product_id 是该表的主键（具有唯一值的列）。
low_fats 是枚举类型，取值为以下两种 ('Y', 'N')，其中 'Y' 表示该产品是低脂产品，'N' 表示不是低脂产品。
recyclable 是枚举类型，取值为以下两种 ('Y', 'N')，其中 'Y' 表示该产品可回收，而 'N' 表示不可回收。
```

编写解决方案找出既是低脂又是可回收的产品编号。

返回结果 **无顺序要求** 。

题解：

```sql
select product_id from Products
where low_fats = 'Y' and recyclable = 'Y'
```

> [!important]
>
> MySQL的枚举类型，enum
>
> 实际上是字符串类型，如果插入数据时选择的数据没有在enum定义的范围内，会存储为''（空字符串）

### 2. 寻找用户推荐人

 `Customer`表:

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| name        | varchar |
| referee_id  | int     |
+-------------+---------+
在 SQL 中，id 是该表的主键列。
该表的每一行表示一个客户的 id、姓名以及推荐他们的客户的 id。
```

找出那些 **没有被** `id = 2` 的客户 **推荐** 的客户的姓名。

以 **任意顺序** 返回结果表。

题解：

```sql
select name from Customer
where IFNULL(referee_id, -1) != 2
```

> [!important]
>
> 数值的默认值为NULL的情况下，使用不等于判断不会带出数值为NULL的数据行

### 3. 大的国家

`World` 表

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| name        | varchar |
| continent   | varchar |
| area        | int     |
| population  | int     |
| gdp         | bigint  |
+-------------+---------+
name 是该表的主键（具有唯一值的列）。
这张表的每一行提供：国家名称、所属大陆、面积、人口和 GDP 值。
```

如果一个国家满足下述两个条件之一，则认为该国是 **大国** ：

- 面积至少为 300 万平方公里（即，`3000000 km2`），或者
- 人口至少为 2500 万（即 `25000000`）

编写解决方案找出 **大国** 的国家名称、人口和面积。

题解：

```sql
select name,population,`area` from World
where `area` >= 3000000 or population >= 25000000
```
> [!important]
>
> 条件关系 OR 

### 4. 文章浏览 I

`Views` 表：

```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| article_id    | int     |
| author_id     | int     |
| viewer_id     | int     |
| view_date     | date    |
+---------------+---------+
此表可能会存在重复行。（换句话说，在 SQL 中这个表没有主键）
此表的每一行都表示某人在某天浏览了某位作者的某篇文章。
请注意，同一人的 author_id 和 viewer_id 是相同的。
```

请查询出所有浏览过自己文章的作者

结果按照 `id` 升序排列。

题解：

```sql
select distinct author_id as id from Views
where author_id = viewer_id
order by id asc
```

> [!important]
>
> order by 排序 asc 升序 desc 降序

### 5. 无效的推文

`Tweets`表：

```
+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| tweet_id       | int     |
| content        | varchar |
+----------------+---------+
在 SQL 中，tweet_id 是这个表的主键。
这个表包含某社交媒体 App 中所有的推文。
```

查询所有无效推文的编号（ID）。当推文内容中的字符数**严格大于** `15` 时，该推文是无效的。

以**任意顺序**返回结果表。

题解：

```sql
select tweet_id from Tweets where LENGTH(content) > 15
```

> [!IMPORTANT]
>
> LENGTH函数可以获取varchar字段的长度

## 表连接

### 6. 使用唯一标识码替换员工ID

`Employees` 表：

```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| name          | varchar |
+---------------+---------+
在 SQL 中，id 是这张表的主键。
这张表的每一行分别代表了某公司其中一位员工的名字和 ID 。
```

`EmployeeUNI` 表：

```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| unique_id     | int     |
+---------------+---------+
在 SQL 中，(id, unique_id) 是这张表的主键。
这张表的每一行包含了该公司某位员工的 ID 和他的唯一标识码（unique ID）。
```

展示每位用户的 **唯一标识码（unique ID ）**；如果某位员工没有唯一标识码，使用 null 填充即可。

你可以以 **任意** 顺序返回结果表。

题解：

```sql
select unique_id,name from Employees left join EmployeeUNI
on Employees.id = EmployeeUNI.id
```

> [!important]
>
> 左外连接：返回左表所有以及右表匹配的字段；A left join B  返回A表全部和B表中关联的
>
> 右外连接：返回右表所有以及左表匹配的字段；A right join B  返回B表全部和A表中关联的
>
> 内连接：两个表中都存在的行才展示
>
> 全连接：两个表都不限制，任意一个存在就展示

### 7. 产品销售分析 I

销售表 `Sales`：

```
+-------------+-------+
| Column Name | Type  |
+-------------+-------+
| sale_id     | int   |
| product_id  | int   |
| year        | int   |
| quantity    | int   |
| price       | int   |
+-------------+-------+
(sale_id, year) 是销售表 Sales 的主键（具有唯一值的列的组合）。
product_id 是关联到产品表 Product 的外键（reference 列）。
该表的每一行显示 product_id 在某一年的销售情况。
注意: price 表示每单位价格。
```

产品表 `Product`：

```
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| product_id   | int     |
| product_name | varchar |
+--------------+---------+
product_id 是表的主键（具有唯一值的列）。
该表的每一行表示每种产品的产品名称。
```

编写解决方案，以获取 `Sales` 表中所有 `sale_id` 对应的 `product_name` 以及该产品的所有 `year` 和 `price` 。

返回结果表 **无顺序要求** 。

题解：

```sql
select p.product_name,s.year,s.price from Sales as s
left join Product as p on p.product_id = s.product_id
```

> [!important]
>
> 用内连接(inner join)或左连接(left join)都可以：因为两张表之间存在外键约束。外键约束了两张表数据的完整性，所以不会出现null的情况。

### 8. 进店却未进行过交易的顾客

`Visits`表：

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| visit_id    | int     |
| customer_id | int     |
+-------------+---------+
visit_id 是该表中具有唯一值的列。
该表包含有关光临过购物中心的顾客的信息。
```

`Transactions`表：

```
+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| transaction_id | int     |
| visit_id       | int     |
| amount         | int     |
+----------------+---------+
transaction_id 是该表中具有唯一值的列。
此表包含 visit_id 期间进行的交易的信息。
```

有一些顾客可能光顾了购物中心但没有进行交易。请你编写一个解决方案，来查找这些顾客的 ID ，以及他们只光顾不交易的次数。

返回以 **任何顺序** 排序的结果表。

题解：

```sql
select v.customer_id, count(1) as count_no_trans from Visits as v 
left join Transactions as t on v.visit_id = t.visit_id
where t.transaction_id is null
group by v.customer_id
```

> [!important]
>
> 查询只光顾不交易，通过`visit_id`关联交易表，并且`tracsaction_id`为`null`的行

### 9. 上升的温度

表： `Weather`

```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| recordDate    | date    |
| temperature   | int     |
+---------------+---------+
id 是该表具有唯一值的列。
没有具有相同 recordDate 的不同行。
该表包含特定日期的温度信息 
```

编写解决方案，找出与之前（昨天的）日期相比温度更高的所有日期的 `id` 。

返回结果 **无顺序要求** 。

题解：

```sql
select a.id from Weather as a
cross join Weather as b on datediff(a.recordDate, b.recordDate) = 1
where a.temperature > b.temperature
```

> [!important]
>
> 时间函数
>
> 1. datediff(a, b) 计算date类型字段相差天数，a大于b则为正数
> 2. timestampdiff(day, a, b) 计算a与b相差天数，a大于b则为负数
>
> 表连接（cross jion）
>
> 两个表之间每个行获取笛卡尔积，与left join不同，默认可以没有



### 10. 每台机器的进程平均运行时间

 `Activity`表:

```
+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| machine_id     | int     |
| process_id     | int     |
| activity_type  | enum    |
| timestamp      | float   |
+----------------+---------+
该表展示了一家工厂网站的用户活动。
(machine_id, process_id, activity_type) 是当前表的主键（具有唯一值的列的组合）。
machine_id 是一台机器的ID号。
process_id 是运行在各机器上的进程ID号。
activity_type 是枚举类型 ('start', 'end')。
timestamp 是浮点类型,代表当前时间(以秒为单位)。
'start' 代表该进程在这台机器上的开始运行时间戳 , 'end' 代表该进程在这台机器上的终止运行时间戳。
同一台机器，同一个进程都有一对开始时间戳和结束时间戳，而且开始时间戳永远在结束时间戳前面。
```

现在有一个工厂网站由几台机器运行，每台机器上运行着 **相同数量的进程** 。编写解决方案，计算每台机器各自完成一个进程任务的平均耗时。

完成一个进程任务的时间指进程的`'end' 时间戳` 减去 `'start' 时间戳`。平均耗时通过计算每台机器上所有进程任务的总耗费时间除以机器上的总进程数量获得。

结果表必须包含`machine_id（机器ID）` 和对应的 **average time（平均耗时）** 别名 `processing_time`，且**四舍五入保留3位小数。**

以 **任意顺序** 返回表。

题解：

```sql
select a.machine_id, round(avg(b.timestamp - a.timestamp), 3)as processing_time from Activity as a
left join Activity as b on a.machine_id = b.machine_id and a.process_id = b.process_id and a. activity_type = 'start' and b.activity_type = 'end'
group by a.machine_id
```

> [!important]
>
> avg函数计算平均值
>
> round函数小数点保留位数

### 11. 员工奖金

表：`Employee` 

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| empId       | int     |
| name        | varchar |
| supervisor  | int     |
| salary      | int     |
+-------------+---------+
empId 是该表中具有唯一值的列。
该表的每一行都表示员工的姓名和 id，以及他们的工资和经理的 id。
```

表：`Bonus`

```
+-------------+------+
| Column Name | Type |
+-------------+------+
| empId       | int  |
| bonus       | int  |
+-------------+------+
empId 是该表具有唯一值的列。
empId 是 Employee 表中 empId 的外键(reference 列)。
该表的每一行都包含一个员工的 id 和他们各自的奖金。 
```

编写解决方案，报告每个奖金 **少于** `1000` 的员工的姓名和奖金数额。

以 **任意顺序** 返回结果表。

结果格式如下所示。

**示例：**

```
输入：
Employee table:
+-------+--------+------------+--------+
| empId | name   | supervisor | salary |
+-------+--------+------------+--------+
| 3     | Brad   | null       | 4000   |
| 1     | John   | 3          | 1000   |
| 2     | Dan    | 3          | 2000   |
| 4     | Thomas | 3          | 4000   |
+-------+--------+------------+--------+
Bonus table:
+-------+-------+
| empId | bonus |
+-------+-------+
| 2     | 500   |
| 4     | 2000  |
+-------+-------+
输出：
+------+-------+
| name | bonus |
+------+-------+
| Brad | null  |
| John | null  |
| Dan  | 500   |
+------+-------+
```

题解：

```sql
select a.name, b.bonus from Employee as a 
left join Bonus as b on a.empId = b.empId
where b.bonus < 1000 or b.bonus is null
```

### 12. 学生们参加各科测试的次数

学生表: `Students`

```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| student_id    | int     |
| student_name  | varchar |
+---------------+---------+
在 SQL 中，主键为 student_id（学生ID）。
该表内的每一行都记录有学校一名学生的信息。
```

科目表: `Subjects`

```
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| subject_name | varchar |
+--------------+---------+
在 SQL 中，主键为 subject_name（科目名称）。
每一行记录学校的一门科目名称。
```

考试表: `Examinations`

```
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| student_id   | int     |
| subject_name | varchar |
+--------------+---------+
这个表可能包含重复数据（换句话说，在 SQL 中，这个表没有主键）。
学生表里的一个学生修读科目表里的每一门科目。
这张考试表的每一行记录就表示学生表里的某个学生参加了一次科目表里某门科目的测试。
```

查询出每个学生参加每一门科目测试的次数，结果按 `student_id` 和 `subject_name` 排序。

查询结构格式如下所示。

**示例：**

```
输入：
Students table:
+------------+--------------+
| student_id | student_name |
+------------+--------------+
| 1          | Alice        |
| 2          | Bob          |
| 13         | John         |
| 6          | Alex         |
+------------+--------------+
Subjects table:
+--------------+
| subject_name |
+--------------+
| Math         |
| Physics      |
| Programming  |
+--------------+
Examinations table:
+------------+--------------+
| student_id | subject_name |
+------------+--------------+
| 1          | Math         |
| 1          | Physics      |
| 1          | Programming  |
| 2          | Programming  |
| 1          | Physics      |
| 1          | Math         |
| 13         | Math         |
| 13         | Programming  |
| 13         | Physics      |
| 2          | Math         |
| 1          | Math         |
+------------+--------------+
输出：
+------------+--------------+--------------+----------------+
| student_id | student_name | subject_name | attended_exams |
+------------+--------------+--------------+----------------+
| 1          | Alice        | Math         | 3              |
| 1          | Alice        | Physics      | 2              |
| 1          | Alice        | Programming  | 1              |
| 2          | Bob          | Math         | 1              |
| 2          | Bob          | Physics      | 0              |
| 2          | Bob          | Programming  | 1              |
| 6          | Alex         | Math         | 0              |
| 6          | Alex         | Physics      | 0              |
| 6          | Alex         | Programming  | 0              |
| 13         | John         | Math         | 1              |
| 13         | John         | Physics      | 1              |
| 13         | John         | Programming  | 1              |
+------------+--------------+--------------+----------------+
解释：
结果表需包含所有学生和所有科目（即便测试次数为0）：
Alice 参加了 3 次数学测试, 2 次物理测试，以及 1 次编程测试；
Bob 参加了 1 次数学测试, 1 次编程测试，没有参加物理测试；
Alex 啥测试都没参加；
John  参加了数学、物理、编程测试各 1 次。
```

题解：

```sql
select a.student_id, a.student_name,b.subject_name, (select count(1) from Examinations as c where c.student_id = a.student_id and c.subject_name = b.subject_name) as attended_exams from Students as a 
cross join Subjects as b
order by a.student_id, b.subject_name asc

-- 写法2
SELECT 
    s.student_id, s.student_name, sub.subject_name, IFNULL(grouped.attended_exams, 0) AS attended_exams
FROM 
    Students s
CROSS JOIN 
    Subjects sub
LEFT JOIN (
    SELECT student_id, subject_name, COUNT(*) AS attended_exams
    FROM Examinations
    GROUP BY student_id, subject_name
) grouped 
ON s.student_id = grouped.student_id AND sub.subject_name = grouped.subject_name
ORDER BY s.student_id, sub.subject_name;
```

结果集中的数据基础是`Students`表和`Subject`表的笛卡尔积，根据考试表的数据来填充`attended_exams`字段。

> [!important]
>
> cross join
>
> group by
>
> order by

### 13. 至少有5名直接下属的经理

表: `Employee`

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| name        | varchar |
| department  | varchar |
| managerId   | int     |
+-------------+---------+
id 是此表的主键（具有唯一值的列）。
该表的每一行表示雇员的名字、他们的部门和他们的经理的id。
如果managerId为空，则该员工没有经理。
没有员工会成为自己的管理者。
```

编写一个解决方案，找出至少有**五个直接下属**的经理。

以 **任意顺序** 返回结果表。

查询结果格式如下所示。 

**示例 1:**

```
输入: 
Employee 表:
+-----+-------+------------+-----------+
| id  | name  | department | managerId |
+-----+-------+------------+-----------+
| 101 | John  | A          | Null      |
| 102 | Dan   | A          | 101       |
| 103 | James | A          | 101       |
| 104 | Amy   | A          | 101       |
| 105 | Anne  | A          | 101       |
| 106 | Ron   | B          | 101       |
+-----+-------+------------+-----------+
输出: 
+------+
| name |
+------+
| John |
+------+
```

题解：

```sql
select Manager.Name as Name
from
Employee as Manager join Employee as Report
on Manager.Id = Report.ManagerId
group by Manager.Id
having count(Report.Id) >= 5
```

> [!important]
>
> group by having 子句

### 14. 确认率

表: `Signups`

```
+----------------+----------+
| Column Name    | Type     |
+----------------+----------+
| user_id        | int      |
| time_stamp     | datetime |
+----------------+----------+
User_id是该表的主键。
每一行都包含ID为user_id的用户的注册时间信息。
```

表: `Confirmations`

```
+----------------+----------+
| Column Name    | Type     |
+----------------+----------+
| user_id        | int      |
| time_stamp     | datetime |
| action         | ENUM     |
+----------------+----------+
(user_id, time_stamp)是该表的主键。
user_id是一个引用到注册表的外键。
action是类型为('confirmed'， 'timeout')的ENUM
该表的每一行都表示ID为user_id的用户在time_stamp请求了一条确认消息，该确认消息要么被确认('confirmed')，要么被过期('timeout')。
```

用户的 **确认率** 是 `'confirmed'` 消息的数量除以请求的确认消息的总数。没有请求任何确认消息的用户的确认率为 `0` 。确认率四舍五入到 **小数点后两位** 。

编写一个SQL查询来查找每个用户的 确认率 。

以 任意顺序 返回结果表。

查询结果格式如下所示。

**示例:**

```
输入：
Signups 表:
+---------+---------------------+
| user_id | time_stamp          |
+---------+---------------------+
| 3       | 2020-03-21 10:16:13 |
| 7       | 2020-01-04 13:57:59 |
| 2       | 2020-07-29 23:09:44 |
| 6       | 2020-12-09 10:39:37 |
+---------+---------------------+
Confirmations 表:
+---------+---------------------+-----------+
| user_id | time_stamp          | action    |
+---------+---------------------+-----------+
| 3       | 2021-01-06 03:30:46 | timeout   |
| 3       | 2021-07-14 14:00:00 | timeout   |
| 7       | 2021-06-12 11:57:29 | confirmed |
| 7       | 2021-06-13 12:58:28 | confirmed |
| 7       | 2021-06-14 13:59:27 | confirmed |
| 2       | 2021-01-22 00:00:00 | confirmed |
| 2       | 2021-02-28 23:59:59 | timeout   |
+---------+---------------------+-----------+
输出: 
+---------+-------------------+
| user_id | confirmation_rate |
+---------+-------------------+
| 6       | 0.00              |
| 3       | 0.00              |
| 7       | 1.00              |
| 2       | 0.50              |
+---------+-------------------+
解释:
用户 6 没有请求任何确认消息。确认率为 0。
用户 3 进行了 2 次请求，都超时了。确认率为 0。
用户 7 提出了 3 个请求，所有请求都得到了确认。确认率为 1。
用户 2 做了 2 个请求，其中一个被确认，另一个超时。确认率为 1 / 2 = 0.5。
```

题解：

```sql
SELECT
    s.user_id,
    ROUND(IFNULL(AVG(action = 'confirmed'),0),2) confirmation_rate
FROM Signups s
LEFT JOIN Confirmations c 
ON s.user_id = c.user_id
GROUP BY s.user_id
```

> [!important]
>
> avg(epr) 函数，除了计算字段的平均值，还可以计算符合条件的记录数占比
>
> 根据`user_id`分组后满足条件表达式`action = 'confirmed'`的行数占总行数占比



## 聚合函数



### 15. 有趣的电影

表：`cinema`

```
+----------------+----------+
| Column Name    | Type     |
+----------------+----------+
| id             | int      |
| movie          | varchar  |
| description    | varchar  |
| rating         | float    |
+----------------+----------+
id 是该表的主键(具有唯一值的列)。
每行包含有关电影名称、类型和评级的信息。
评级为 [0,10] 范围内的小数点后 2 位浮点数。
```

编写解决方案，找出所有影片描述为 **非** `boring` (不无聊) 的并且 **id 为奇数** 的影片。

返回结果按 `rating` **降序排列**。

结果格式如下示例。

**示例：**

```
输入：
+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   1     | War       |   great 3D   |   8.9     |
|   2     | Science   |   fiction    |   8.5     |
|   3     | irish     |   boring     |   6.2     |
|   4     | Ice song  |   Fantacy    |   8.6     |
|   5     | House card|   Interesting|   9.1     |
+---------+-----------+--------------+-----------+
输出：
+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   5     | House card|   Interesting|   9.1     |
|   1     | War       |   great 3D   |   8.9     |
+---------+-----------+--------------+-----------+
解释：
我们有三部电影，它们的 id 是奇数:1、3 和 5。id = 3 的电影是 boring 的，所以我们不把它包括在答案中。
```

题解：

```sql
select * from cinema
where description != 'boring' and mod(id, 2) = 1
order by rating desc
```

> [!important]
>
> MySQL中判断数字为奇数的方法
>
> 1. `mod(x, 2) = 1` mod函数判断除以2的余数是否等于1
> 2. `power(-1, x) = -1` 幂函数计算-1的x次方是否等于-1
> 3. `x % 2 = 1` 取模运算，与第一种相似
> 4. `x & 1 = 1` 位运算，计算与1按位与的结果是否为1
> 5. `x regexp '[1,3,5,7,9]$' = 1` 正则判断数字是否以1、3、5、7、9结尾
> 6. `x>>1<<1 != x` 位运算符右移一位再左移一位，判断是否与原值相等

### 16. 平均售价

表：`Prices`

```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product_id    | int     |
| start_date    | date    |
| end_date      | date    |
| price         | int     |
+---------------+---------+
(product_id，start_date，end_date) 是 prices 表的主键（具有唯一值的列的组合）。
prices 表的每一行表示的是某个产品在一段时期内的价格。
每个产品的对应时间段是不会重叠的，这也意味着同一个产品的价格时段不会出现交叉。
```

表：`UnitsSold`

```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product_id    | int     |
| purchase_date | date    |
| units         | int     |
+---------------+---------+
该表可能包含重复数据。
该表的每一行表示的是每种产品的出售日期，单位和产品 id。
```

编写解决方案以查找每种产品的平均售价。`average_price` 应该 **四舍五入到小数点后两位**。

返回结果表 **无顺序要求** 。

结果格式如下例所示。

**示例 1：**

```
输入：
Prices table:
+------------+------------+------------+--------+
| product_id | start_date | end_date   | price  |
+------------+------------+------------+--------+
| 1          | 2019-02-17 | 2019-02-28 | 5      |
| 1          | 2019-03-01 | 2019-03-22 | 20     |
| 2          | 2019-02-01 | 2019-02-20 | 15     |
| 2          | 2019-02-21 | 2019-03-31 | 30     |
+------------+------------+------------+--------+
UnitsSold table:
+------------+---------------+-------+
| product_id | purchase_date | units |
+------------+---------------+-------+
| 1          | 2019-02-25    | 100   |
| 1          | 2019-03-01    | 15    |
| 2          | 2019-02-10    | 200   |
| 2          | 2019-03-22    | 30    |
+------------+---------------+-------+
输出：
+------------+---------------+
| product_id | average_price |
+------------+---------------+
| 1          | 6.96          |
| 2          | 16.96         |
+------------+---------------+
解释：
平均售价 = 产品总价 / 销售的产品数量。
产品 1 的平均售价 = ((100 * 5)+(15 * 20) )/ 115 = 6.96
产品 2 的平均售价 = ((200 * 15)+(30 * 30) )/ 230 = 16.96
```

题解：

```sql
SELECT
    product_id,
    IFNULL(Round(SUM(sales) / SUM(units), 2), 0) AS average_price
FROM (
    SELECT
        Prices.product_id AS product_id,
        Prices.price * UnitsSold.units AS sales,
        UnitsSold.units AS units
    FROM Prices 
    LEFT JOIN UnitsSold ON Prices.product_id = UnitsSold.product_id
    AND (UnitsSold.purchase_date BETWEEN Prices.start_date AND Prices.end_date)
) T
GROUP BY product_id
```

> [!important]
>
> sum() 求和
>
> round() 保留小数位数

### 17. 平均员工 I

项目表 `Project`： 

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| project_id  | int     |
| employee_id | int     |
+-------------+---------+
主键为 (project_id, employee_id)。
employee_id 是员工表 Employee 表的外键。
这张表的每一行表示 employee_id 的员工正在 project_id 的项目上工作。
```

员工表 `Employee`：

```
+------------------+---------+
| Column Name      | Type    |
+------------------+---------+
| employee_id      | int     |
| name             | varchar |
| experience_years | int     |
+------------------+---------+
主键是 employee_id。数据保证 experience_years 非空。
这张表的每一行包含一个员工的信息。
```

请写一个 SQL 语句，查询每一个项目中员工的 **平均** 工作年限，**精确到小数点后两位**。

以 **任意** 顺序返回结果表。

查询结果的格式如下。

**示例 1:**

```
输入：
Project 表：
+-------------+-------------+
| project_id  | employee_id |
+-------------+-------------+
| 1           | 1           |
| 1           | 2           |
| 1           | 3           |
| 2           | 1           |
| 2           | 4           |
+-------------+-------------+
Employee 表：
+-------------+--------+------------------+
| employee_id | name   | experience_years |
+-------------+--------+------------------+
| 1           | Khaled | 3                |
| 2           | Ali    | 2                |
| 3           | John   | 1                |
| 4           | Doe    | 2                |
+-------------+--------+------------------+
输出：
+-------------+---------------+
| project_id  | average_years |
+-------------+---------------+
| 1           | 2.00          |
| 2           | 2.50          |
+-------------+---------------+
解释：第一个项目中，员工的平均工作年限是 (3 + 2 + 1) / 3 = 2.00；第二个项目中，员工的平均工作年限是 (3 + 2) / 2 = 2.50
```

题解：

```sql
select a.project_id, round(avg(b.experience_years),2) as average_years from Project as a 
left join Employee as b on a.employee_id = b.employee_id
group by a.project_id
```

### 18. 各赛事的用户注册率

用户表： `Users`

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| user_id     | int     |
| user_name   | varchar |
+-------------+---------+
user_id 是该表的主键(具有唯一值的列)。
该表中的每行包括用户 ID 和用户名。
```

 注册表： `Register`

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| contest_id  | int     |
| user_id     | int     |
+-------------+---------+
(contest_id, user_id) 是该表的主键(具有唯一值的列的组合)。
该表中的每行包含用户的 ID 和他们注册的赛事。
```

编写解决方案统计出各赛事的用户注册百分率，保留两位小数。

返回的结果表按 `percentage` 的 **降序** 排序，若相同则按 `contest_id` 的 **升序** 排序。

返回结果如下示例所示。 

**示例 1：**

```
输入：
Users 表：
+---------+-----------+
| user_id | user_name |
+---------+-----------+
| 6       | Alice     |
| 2       | Bob       |
| 7       | Alex      |
+---------+-----------+
Register 表：
+------------+---------+
| contest_id | user_id |
+------------+---------+
| 215        | 6       |
| 209        | 2       |
| 208        | 2       |
| 210        | 6       |
| 208        | 6       |
| 209        | 7       |
| 209        | 6       |
| 215        | 7       |
| 208        | 7       |
| 210        | 2       |
| 207        | 2       |
| 210        | 7       |
+------------+---------+
输出：
+------------+------------+
| contest_id | percentage |
+------------+------------+
| 208        | 100.0      |
| 209        | 100.0      |
| 210        | 100.0      |
| 215        | 66.67      |
| 207        | 33.33      |
+------------+------------+
解释：
所有用户都注册了 208、209 和 210 赛事，因此这些赛事的注册率为 100% ，我们按 contest_id 的降序排序加入结果表中。
Alice 和 Alex 注册了 215 赛事，注册率为 ((2/3) * 100) = 66.67%
Bob 注册了 207 赛事，注册率为 ((1/3) * 100) = 33.33%
```

题解：

```SQL
select contest_id, ROUND(100 * count(1)/ (select count(1) from Users), 2) as percentage from Register
group by contest_id
order by percentage desc, contest_id asc
```

### 19. 查询结果的质量和占比

`Queries` 表： 

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| query_name  | varchar |
| result      | varchar |
| position    | int     |
| rating      | int     |
+-------------+---------+
此表可能有重复的行。
此表包含了一些从数据库中收集的查询信息。
“位置”（position）列的值为 1 到 500 。
“评分”（rating）列的值为 1 到 5 。评分小于 3 的查询被定义为质量很差的查询。
```

将查询结果的质量 `quality` 定义为：

> 各查询结果的评分与其位置之间比率的平均值。

将劣质查询百分比 `poor_query_percentage` 为：

> 评分小于 3 的查询结果占全部查询结果的百分比。

编写解决方案，找出每次的 `query_name` 、 `quality` 和 `poor_query_percentage`。

`quality` 和 `poor_query_percentage` 都应 **四舍五入到小数点后两位** 。

以 **任意顺序** 返回结果表。

结果格式如下所示： 

**示例 1：**

```
输入：
Queries table:
+------------+-------------------+----------+--------+
| query_name | result            | position | rating |
+------------+-------------------+----------+--------+
| Dog        | Golden Retriever  | 1        | 5      |
| Dog        | German Shepherd   | 2        | 5      |
| Dog        | Mule              | 200      | 1      |
| Cat        | Shirazi           | 5        | 2      |
| Cat        | Siamese           | 3        | 3      |
| Cat        | Sphynx            | 7        | 4      |
+------------+-------------------+----------+--------+
输出：
+------------+---------+-----------------------+
| query_name | quality | poor_query_percentage |
+------------+---------+-----------------------+
| Dog        | 2.50    | 33.33                 |
| Cat        | 0.66    | 33.33                 |
+------------+---------+-----------------------+
解释：
Dog 查询结果的质量为 ((5 / 1) + (5 / 2) + (1 / 200)) / 3 = 2.50
Dog 查询结果的劣质查询百分比为 (1 / 3) * 100 = 33.33

Cat 查询结果的质量为 ((2 / 5) + (3 / 3) + (4 / 7)) / 3 = 0.66
Cat 查询结果的劣质查询百分比为 (1 / 3) * 100 = 33.33
```

题解：

```SQL
SELECT query_name, ROUND(AVG(rating/position),2) AS quality, ROUND(100 * AVG(rating < 3),2) AS poor_query_percentage
FROM Queries
WHERE query_name IS NOT NULL
GROUP BY query_name
```

> [!important]
>
> AVG()函数用法，表达输入条件，得出符合表达式记录数对总数的占比

### 20. 每月交易 I

表：`Transactions`

```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| country       | varchar |
| state         | enum    |
| amount        | int     |
| trans_date    | date    |
+---------------+---------+
id 是这个表的主键。
该表包含有关传入事务的信息。
state 列类型为 ["approved", "declined"] 之一。
```

编写一个 sql 查询来查找每个月和每个国家/地区的事务数及其总金额、已批准的事务数及其总金额。

以 **任意顺序** 返回结果表。

查询结果格式如下所示。

**示例 1:**

```
输入：
Transactions table:
+------+---------+----------+--------+------------+
| id   | country | state    | amount | trans_date |
+------+---------+----------+--------+------------+
| 121  | US      | approved | 1000   | 2018-12-18 |
| 122  | US      | declined | 2000   | 2018-12-19 |
| 123  | US      | approved | 2000   | 2019-01-01 |
| 124  | DE      | approved | 2000   | 2019-01-07 |
+------+---------+----------+--------+------------+
输出：
+----------+---------+-------------+----------------+--------------------+-----------------------+
| month    | country | trans_count | approved_count | trans_total_amount | approved_total_amount |
+----------+---------+-------------+----------------+--------------------+-----------------------+
| 2018-12  | US      | 2           | 1              | 3000               | 1000                  |
| 2019-01  | US      | 1           | 1              | 2000               | 2000                  |
| 2019-01  | DE      | 1           | 1              | 2000               | 2000                  |
+----------+---------+-------------+----------------+--------------------+-----------------------+
```

题解：

```SQL
SELECT
DATE_FORMAT(trans_date, '%Y-%m') AS month,
country,
COUNT(1) AS trans_count,
SUM(IF(state = 'approved', 1, 0)) AS approved_count,
SUM(amount) AS trans_total_amount,
SUM(IF(state = 'approved', amount, 0)) AS approved_total_amount
FROM Transactions
GROUP BY month, country
```

> [!important]
>
> DATE_FORMAT() 日期格式化后分组

### 21. 即使食物配送 II

配送表: `Delivery`

```
+-----------------------------+---------+
| Column Name                 | Type    |
+-----------------------------+---------+
| delivery_id                 | int     |
| customer_id                 | int     |
| order_date                  | date    |
| customer_pref_delivery_date | date    |
+-----------------------------+---------+
delivery_id 是该表中具有唯一值的列。
该表保存着顾客的食物配送信息，顾客在某个日期下了订单，并指定了一个期望的配送日期（和下单日期相同或者在那之后）。
```

如果顾客期望的配送日期和下单日期相同，则该订单称为 「**即时订单**」，否则称为「**计划订单**」。

「**首次订单**」是顾客最早创建的订单。我们保证一个顾客只会有一个「首次订单」。

编写解决方案以获取即时订单在所有用户的首次订单中的比例。**保留两位小数。**

结果示例如下所示：

**示例 1：**

```
输入：
Delivery 表：
+-------------+-------------+------------+-----------------------------+
| delivery_id | customer_id | order_date | customer_pref_delivery_date |
+-------------+-------------+------------+-----------------------------+
| 1           | 1           | 2019-08-01 | 2019-08-02                  |
| 2           | 2           | 2019-08-02 | 2019-08-02                  |
| 3           | 1           | 2019-08-11 | 2019-08-12                  |
| 4           | 3           | 2019-08-24 | 2019-08-24                  |
| 5           | 3           | 2019-08-21 | 2019-08-22                  |
| 6           | 2           | 2019-08-11 | 2019-08-13                  |
| 7           | 4           | 2019-08-09 | 2019-08-09                  |
+-------------+-------------+------------+-----------------------------+
输出：
+----------------------+
| immediate_percentage |
+----------------------+
| 50.00                |
+----------------------+
解释：
1 号顾客的 1 号订单是首次订单，并且是计划订单。
2 号顾客的 2 号订单是首次订单，并且是即时订单。
3 号顾客的 5 号订单是首次订单，并且是计划订单。
4 号顾客的 7 号订单是首次订单，并且是即时订单。
因此，一半顾客的首次订单是即时的。
```

题解：

```SQL
SELECT ROUND(100 * AVG(a.order_date = a.customer_pref_delivery_date), 2) AS immediate_percentage FROM Delivery AS a
INNER JOIN
    (SELECT customer_id, MIN(order_date) AS order_date FROM Delivery GROUP BY customer_id) AS b ON a.order_date = b.order_date
AND a.customer_id = b.customer_id
```

> [!important]
>
> 确定用户首次订单，通过根据客户Id分组取每组下单时间最小值，再通过子查询，获取所有首次下单的🉐交易数据

### 22. 游戏玩法分析 IV

 `Activity`表:

```
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| player_id    | int     |
| device_id    | int     |
| event_date   | date    |
| games_played | int     |
+--------------+---------+
（player_id，event_date）是此表的主键（具有唯一值的列的组合）。
这张表显示了某些游戏的玩家的活动情况。
每一行是一个玩家的记录，他在某一天使用某个设备注销之前登录并玩了很多游戏（可能是 0）。
```

 编写解决方案，报告在首次登录的第二天再次登录的玩家的 **比率**，**四舍五入到小数点后两位**。换句话说，你需要计算从首次登录日期开始至少连续两天登录的玩家的数量，然后除以玩家总数。

结果格式如下所示：

 **示例 1：**

```
输入：
Activity table:
+-----------+-----------+------------+--------------+
| player_id | device_id | event_date | games_played |
+-----------+-----------+------------+--------------+
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-03-02 | 6            |
| 2         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-02 | 0            |
| 3         | 4         | 2018-07-03 | 5            |
+-----------+-----------+------------+--------------+
输出：
+-----------+
| fraction  |
+-----------+
| 0.33      |
+-----------+
解释：
只有 ID 为 1 的玩家在第一天登录后才重新登录，所以答案是 1/3 = 0.33
```

题解：

```SQL
SELECT
	ROUND( AVG( b.player_id IS NOT NULL ), 2 ) AS fraction 
FROM
	( SELECT player_id, MIN( event_date ) AS event_date FROM Activity GROUP BY player_id ) AS a
	LEFT JOIN Activity AS b ON a.player_id = b.player_id 
	AND DATEDIFF( b.event_date, a.event_date ) = 1
```



## 排序与分组



### 23.  每位教师所教授的科目种类的数量

表: `Teacher`

```
+-------------+------+
| Column Name | Type |
+-------------+------+
| teacher_id  | int  |
| subject_id  | int  |
| dept_id     | int  |
+-------------+------+
在 SQL 中，(subject_id, dept_id) 是该表的主键。
该表中的每一行都表示带有 teacher_id 的教师在系 dept_id 中教授科目 subject_id。
```

查询每位老师在大学里教授的科目种类的数量。

以 **任意顺序** 返回结果表。

查询结果格式示例如下。

**示例 1:**

```
输入: 
Teacher 表:
+------------+------------+---------+
| teacher_id | subject_id | dept_id |
+------------+------------+---------+
| 1          | 2          | 3       |
| 1          | 2          | 4       |
| 1          | 3          | 3       |
| 2          | 1          | 1       |
| 2          | 2          | 1       |
| 2          | 3          | 1       |
| 2          | 4          | 1       |
+------------+------------+---------+
输出:  
+------------+-----+
| teacher_id | cnt |
+------------+-----+
| 1          | 2   |
| 2          | 4   |
+------------+-----+
解释: 
教师 1:
  - 他在 3、4 系教科目 2。
  - 他在 3 系教科目 3。
教师 2:
  - 他在 1 系教科目 1。
  - 他在 1 系教科目 2。
  - 他在 1 系教科目 3。
  - 他在 1 系教科目 4。
```

题解：
```SQL
SELECT teacher_id, count(DISTINCT subject_id) AS cnt FROM Teacher
GROUP BY teacher_id
```

### 24. 查询近30天活跃用户数

表：`Activity`

```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| user_id       | int     |
| session_id    | int     |
| activity_date | date    |
| activity_type | enum    |
+---------------+---------+
该表没有包含重复数据。
activity_type 列是 ENUM(category) 类型， 从 ('open_session'， 'end_session'， 'scroll_down'， 'send_message') 取值。
该表记录社交媒体网站的用户活动。
注意，每个会话只属于一个用户。
```

编写解决方案，统计截至 `2019-07-27`（包含2019-07-27），近 `30` 天的每日活跃用户数（当天只要有一条活动记录，即为活跃用户）。

以 **任意顺序** 返回结果表。

结果示例如下。

**示例 1:**

```
输入：
Activity table:
+---------+------------+---------------+---------------+
| user_id | session_id | activity_date | activity_type |
+---------+------------+---------------+---------------+
| 1       | 1          | 2019-07-20    | open_session  |
| 1       | 1          | 2019-07-20    | scroll_down   |
| 1       | 1          | 2019-07-20    | end_session   |
| 2       | 4          | 2019-07-20    | open_session  |
| 2       | 4          | 2019-07-21    | send_message  |
| 2       | 4          | 2019-07-21    | end_session   |
| 3       | 2          | 2019-07-21    | open_session  |
| 3       | 2          | 2019-07-21    | send_message  |
| 3       | 2          | 2019-07-21    | end_session   |
| 4       | 3          | 2019-06-25    | open_session  |
| 4       | 3          | 2019-06-25    | end_session   |
+---------+------------+---------------+---------------+
输出：
+------------+--------------+ 
| day        | active_users |
+------------+--------------+ 
| 2019-07-20 | 2            |
| 2019-07-21 | 2            |
+------------+--------------+ 
解释：注意非活跃用户的记录不需要展示。
```

题解：

```SQL
SELECT activity_date AS day, count(DISTINCT user_id) AS active_users FROM Activity
WHERE  DATEDIFF('2019-07-27', activity_date) BETWEEN 0 AND 29
GROUP BY activity_date
```

> [!important]
>
> COUNT(DISTINCT column) 字段去重计数
>
> DATEDIFF(date1, date2)  date1减date2的数值
>
> BETWEEN AND 用法

### 25. 销售分析III

表： `Product`

```
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| product_id   | int     |
| product_name | varchar |
| unit_price   | int     |
+--------------+---------+
product_id 是该表的主键（具有唯一值的列）。
该表的每一行显示每个产品的名称和价格。
```

表：`Sales`

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| seller_id   | int     |
| product_id  | int     |
| buyer_id    | int     |
| sale_date   | date    |
| quantity    | int     |
| price       | int     |
+------ ------+---------+
这个表可能有重复的行。
product_id 是 Product 表的外键（reference 列）。
该表的每一行包含关于一个销售的一些信息。 
```

编写解决方案，报告`2019年春季`才售出的产品。即**仅**在`**2019-01-01**`至`**2019-03-31**`（含）之间出售的商品。

以 **任意顺序** 返回结果表。

结果格式如下所示。

**示例 1:**

```
输入：
Product table:
+------------+--------------+------------+
| product_id | product_name | unit_price |
+------------+--------------+------------+
| 1          | S8           | 1000       |
| 2          | G4           | 800        |
| 3          | iPhone       | 1400       |
+------------+--------------+------------+
Sales table:
+-----------+------------+----------+------------+----------+-------+
| seller_id | product_id | buyer_id | sale_date  | quantity | price |
+-----------+------------+----------+------------+----------+-------+
| 1         | 1          | 1        | 2019-01-21 | 2        | 2000  |
| 1         | 2          | 2        | 2019-02-17 | 1        | 800   |
| 2         | 2          | 3        | 2019-06-02 | 1        | 800   |
| 3         | 3          | 4        | 2019-05-13 | 2        | 2800  |
+-----------+------------+----------+------------+----------+-------+
输出：
+-------------+--------------+
| product_id  | product_name |
+-------------+--------------+
| 1           | S8           |
+-------------+--------------+
解释:
id 为 1 的产品仅在 2019 年春季销售。
id 为 2 的产品在 2019 年春季销售，但也在 2019 年春季之后销售。
id 为 3 的产品在 2019 年春季之后销售。
我们只返回 id 为 1 的产品，因为它是 2019 年春季才销售的产品。
```

题解：

```SQL
SELECT product_id,product_name FROM Product
WHERE product_id NOT IN (
    SELECT DISTINCT product_id FROM Sales WHERE sale_date > '2019-03-31' OR sale_date < '2019-01-01'
) AND product_id IN (SELECT DISTINCT product_id FROM Sales WHERE sale_date BETWEEN '2019-01-01' AND '2019-03-31')

SELECT a.product_id,a.product_name FROM Product AS a
LEFT JOIN Sales AS b ON a.product_id = b.product_id
GROUP BY b.product_id
HAVING MIN(b.sale_date) >= '2019-01-01' AND MAX(b.sale_date) <= '2019-03-31'
```

> [!important]
>
> HAVING 用法

### 26. 超过5名学生的课

表: `Courses`

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| student     | varchar |
| class       | varchar |
+-------------+---------+
在 SQL 中，(student, class)是该表的主键列。
该表的每一行表示学生的名字和他们注册的班级。
```

查询 **至少有5个学生** 的所有班级。

以 **任意顺序** 返回结果表。

查询结果格式如下所示。

**示例 1:**

```
输入: 
Courses table:
+---------+----------+
| student | class    |
+---------+----------+
| A       | Math     |
| B       | English  |
| C       | Math     |
| D       | Biology  |
| E       | Math     |
| F       | Computer |
| G       | Math     |
| H       | Math     |
| I       | Math     |
+---------+----------+
输出: 
+---------+ 
| class   | 
+---------+ 
| Math    | 
+---------+
解释: 
-数学课有6个学生，所以我们包括它。
-英语课有1名学生，所以我们不包括它。
-生物课有1名学生，所以我们不包括它。
-计算机课有1个学生，所以我们不包括它。
```

题解：

```
SELECT class FROM Courses
GROUP BY class
HAVING COUNT(student) >= 5
```

> [!important]
>
> HAVING是在聚合操作（COUNT()、SUM()、AVG()、MAX()、MIN()等）完成之后对已经分组的数据进行筛选，WHERE在数据分组之前筛选记录

### 27. 求关注着的数量

表： `Followers`

```
+-------------+------+
| Column Name | Type |
+-------------+------+
| user_id     | int  |
| follower_id | int  |
+-------------+------+
(user_id, follower_id) 是这个表的主键（具有唯一值的列的组合）。
该表包含一个关注关系中关注者和用户的编号，其中关注者关注用户。
```

编写解决方案，对于每一个用户，返回该用户的关注者数量。

按 `user_id` 的顺序返回结果表。

查询结果的格式如下示例所示。

**示例 1：**

```
输入：
Followers 表：
+---------+-------------+
| user_id | follower_id |
+---------+-------------+
| 0       | 1           |
| 1       | 0           |
| 2       | 0           |
| 2       | 1           |
+---------+-------------+
输出：
+---------+----------------+
| user_id | followers_count|
+---------+----------------+
| 0       | 1              |
| 1       | 1              |
| 2       | 2              |
+---------+----------------+
解释：
0 的关注者有 {1}
1 的关注者有 {0}
2 的关注者有 {0,1}
```

题解：

```SQL
SELECT user_id, COUNT(1) AS followers_count FROM followers
GROUP BY user_id
ORDER BY user_id 
```

### 28. 只出现一次的最大数字

`MyNumbers` 表：

```
+-------------+------+
| Column Name | Type |
+-------------+------+
| num         | int  |
+-------------+------+
该表可能包含重复项（换句话说，在SQL中，该表没有主键）。
这张表的每一行都含有一个整数。
```

**单一数字** 是在 `MyNumbers` 表中只出现一次的数字。

找出最大的 **单一数字** 。如果不存在 **单一数字** ，则返回 `null` 。

查询结果如下例所示。

**示例 1：**

```
输入：
MyNumbers 表：
+-----+
| num |
+-----+
| 8   |
| 8   |
| 3   |
| 3   |
| 1   |
| 4   |
| 5   |
| 6   |
+-----+
输出：
+-----+
| num |
+-----+
| 6   |
+-----+
解释：单一数字有 1、4、5 和 6 。
6 是最大的单一数字，返回 6 。
```

**示例 2：**

```
输入：
MyNumbers table:
+-----+
| num |
+-----+
| 8   |
| 8   |
| 7   |
| 7   |
| 3   |
| 3   |
| 3   |
+-----+
输出：
+------+
| num  |
+------+
| null |
+------+
解释：输入的表中不存在单一数字，所以返回 null 。
```


```SQL
SELECT MAX(num) AS num FROM (SELECT num FROM MyNumbers
GROUP BY num 
HAVING COUNT(1) = 1) AS a 
```

### 29. 买下所有产品的客户

`Customer` 表：

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| customer_id | int     |
| product_key | int     |
+-------------+---------+
该表可能包含重复的行。
customer_id 不为 NULL。
product_key 是 Product 表的外键(reference 列)。
```

`Product` 表：

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| product_key | int     |
+-------------+---------+
product_key 是这张表的主键（具有唯一值的列）。
```

 编写解决方案，报告 `Customer` 表中购买了 `Product` 表中所有产品的客户的 id。

返回结果表 **无顺序要求** 。

返回结果格式如下所示。

**示例 1：**

```
输入：
Customer 表：
+-------------+-------------+
| customer_id | product_key |
+-------------+-------------+
| 1           | 5           |
| 2           | 6           |
| 3           | 5           |
| 3           | 6           |
| 1           | 6           |
+-------------+-------------+
Product 表：
+-------------+
| product_key |
+-------------+
| 5           |
| 6           |
+-------------+
输出：
+-------------+
| customer_id |
+-------------+
| 1           |
| 3           |
+-------------+
解释：
购买了所有产品（5 和 6）的客户的 id 是 1 和 3 。
```

题解：

```SQL
SELECT customer_id FROM Customer
GROUP BY customer_id
HAVING COUNT(DISTINCT product_key) = (SELECT COUNT(1) FROM Product)
```



## 高级查询和连接



### 30. 每位经理的下属员工数量

表：`Employees`

```
+-------------+----------+
| Column Name | Type     |
+-------------+----------+
| employee_id | int      |
| name        | varchar  |
| reports_to  | int      |
| age         | int      |
+-------------+----------+
employee_id 是这个表中具有不同值的列。
该表包含员工以及需要听取他们汇报的上级经理的 ID 的信息。 有些员工不需要向任何人汇报（reports_to 为空）。
```

对于此问题，我们将至少有一个其他员工需要向他汇报的员工，视为一个经理。

编写一个解决方案来返回需要听取汇报的所有经理的 ID、名称、直接向该经理汇报的员工人数，以及这些员工的平均年龄，其中该平均年龄需要四舍五入到最接近的整数。

返回的结果集需要按照 `employee_id` 进行排序。

结果的格式如下：

**示例 1:**

```
输入：
Employees 表：
+-------------+---------+------------+-----+
| employee_id | name    | reports_to | age |
+-------------+---------+------------+-----+
| 9           | Hercy   | null       | 43  |
| 6           | Alice   | 9          | 41  |
| 4           | Bob     | 9          | 36  |
| 2           | Winston | null       | 37  |
+-------------+---------+------------+-----+
输出：
+-------------+-------+---------------+-------------+
| employee_id | name  | reports_count | average_age |
+-------------+-------+---------------+-------------+
| 9           | Hercy | 2             | 39          |
+-------------+-------+---------------+-------------+
解释：
Hercy 有两个需要向他汇报的员工, 他们是 Alice and Bob. 他们的平均年龄是 (41+36)/2 = 38.5, 四舍五入的结果是 39.
```

**示例 2:**

```
输入： 
Employees 表：
+-------------+---------+------------+-----+ 
| employee_id | name    | reports_to | age |
|-------------|---------|------------|-----|
| 1           | Michael | null       | 45  |
| 2           | Alice   | 1          | 38  |
| 3           | Bob     | 1          | 42  |
| 4           | Charlie | 2          | 34  |
| 5           | David   | 2          | 40  |
| 6           | Eve     | 3          | 37  |
| 7           | Frank   | null       | 50  |
| 8           | Grace   | null       | 48  |
+-------------+---------+------------+-----+ 
输出： 
+-------------+---------+---------------+-------------+
| employee_id | name    | reports_count | average_age |
| ----------- | ------- | ------------- | ----------- |
| 1           | Michael | 2             | 40          |
| 2           | Alice   | 2             | 37          |
| 3           | Bob     | 1             | 37          |
+-------------+---------+---------------+-------------+
```

题解：

```SQL
SELECT a.reports_to AS employee_id, b.name, a.reports_count, a.average_age FROM
(SELECT reports_to,COUNT(1) AS reports_count, ROUND(AVG(age)) AS average_age FROM Employees GROUP BY reports_to HAVING reports_to IS NOT NULL) AS a 
LEFT JOIN Employees AS b ON a.reports_to = b.employee_id
ORDER BY employee_id
```

### 31. 
