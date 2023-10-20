# Date代替方案

- 转载自公众号 https://mp.weixin.qq.com/s/v-Va_GuSUGr9HVAW84kloQ[CodeSheep的文章"什么？你项目还在用Date表示时间？！"]


## 替换Date工具类的原因
Date用法的例子：

* 新建当前日期

```java
Date rightNow = new Date();
System.out.println("当前时刻：" + rightNow);
System.out.println("当前年份：" + rightNow.getYear());
System.out.println("当前月份：" + rightNow.getMonth());

输出结果为：
// 当前时刻：Fri Dec 13 21:46:34 CST 2019
// 当前年份：119
// 当前月份：11
```

可读性太差

* 构造指定日期

```java
Date beforeDate = new Date(2019,12,12);
```

方法过时了，而且很多方法都过时了。

## LocalDateTime
自 Java8开始， JDK中其实就增加了一系列表示日期和时间的新类，最典型的就是 LocalDateTime。直言不讳，这玩意的出现就是为了干掉之前 JDK版本中的 Date老哥！
同样，我们也先来感受一下用法！

### 获取当前此刻的时间

```java
LocalDateTime rightNow = LocalDateTime.now();
System.out.println("当前时刻：" + rightNow);
System.out.println("当前年份：" + rightNow.getYear());
System.out.println("当前月份：" + rightNow.getMonth());
System.out.println("当前日份：" + rightNow.getDayOfMonth());
System.out.println("当前时：" + rightNow.getHour());
System.out.println("当前分：" + rightNow.getMinute());
System.out.println("当前秒：" + rightNow.getSecond());

// 输出结果：
当前时刻：2019-12-13T22:05:26.779
当前年份：2019
当前月份：DECEMBER
当前日份：13
当前时：22
当前分：5
当前秒：26
```

### 构造指定时间
比如，想构造：2019年10月12月12日9时21分32秒

```java
LocalDateTime beforeDate = LocalDateTime.of(2019, Month.DECEMBER, 12, 9, 21, 32);
```

### 修改时间

```java
LocalDateTime rightNow = LocalDateTime.now();
rightNow = rightNow.minusYears(2); // 减少 2 年
rightNow = rightNow.plusMonths(3); // 增加 3 个月
rightNow = rightNow.withYear(2008); // 直接修改年份到2008年
rightNow = rightNow.withHour(13); // 直接修改小时到13时
```

### 格式化日期

```java
LocalDateTime rightNow = LocalDateTime.now();
String result1 = rightNow.format(DateTimeFormatter.ISO_DATE);
String result2 = rightNow.format(DateTimeFormatter.BASIC_ISO_DATE);
String result3 = rightNow.format(DateTimeFormatter.ofPattern("yyyy/MM/dd"));

System.out.println("格式化后的日期(基本样式一举例)：" + result1);
System.out.println("格式化后的日期(基本样式二举例)：" + result2);
System.out.println("格式化后的日期(自定义样式举例)：" + result3);

// 输出结果：
格式化后的日期(基本样式一举例)：2019-12-13
格式化后的日期(基本样式二举例)：20191213
格式化后的日期(自定义样式举例)：2019/12/13
```

### 时间反解析
给你一个陌生的字符串，你可以按照你需要的格式把时间给反解出来

```java
LocalDateTime time = LocalDateTime.parse("2002--01--02 11:21", DateTimeFormatter.ofPattern("yyyy--MM--dd HH:mm"));
System.out.println("字符串反解析后的时间为：" + time);

// 输出：
字符串反解析后的时间为：2002-01-02T11:21
```

## 线程安全问题
其实上面讲来讲去只讲了两者在用法上的差别，这其实倒还好，并不致命，可是接下来要讨论的线程安全性问题才是致命的！
其实以前我们惯用的 Date时间类是可变类，这就意味着在多线程环境下对共享 Date变量进行操作时，必须由程序员自己来保证线程安全！否则极有可能翻车。
而自 Java8开始推出的 LocalDateTime却是线程安全的，开发人员不用再考虑并发问题，这点我们从 LocalDateTime的官方源码中即可看出:

```java
This class is immutable and thread-safe.（不可变、线程安全！）
```

## 日期格式化选择
大家除了惯用 Date来表示时间之外，还有一个用于和 Date连用的 SimpleDateFormat 时间格式化类大家可能也戒不掉了!
SimpleDateFormat最主要的致命问题也是在于它本身并不线程安全，这在它的源码注释里已然告知过了：

```java
Date dormats are not synchronized.
If multiple threads access a format concurrently, it must be synchronized externally.
```

那取而代之，我们现在改用什么呢？其实在前文已经用到啦，那就是了 DateTimeFormatter了，他也是线程安全的。