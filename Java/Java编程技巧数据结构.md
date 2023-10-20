# Java编程技巧之数据结构

## 使用HashSet判断主键是否存在

HashSet 实现 Set 接口，由哈希表（实际上是 HashMap ）实现，但不保证 set  的迭代顺序，并允许使用 null 元素。HashSet 的时间复杂度跟 HashMap 一致，如果没有哈希冲突则时间复杂度为 O(1) ，如果存在哈希冲突则时间复杂度不超过 O(n) 。所以，在日常编码中，可以使用 HashSet 判断主键是否存在。
案例：给定一个字符串(不一定全为字母)，请返回第一个重复出现的字符。

```java
/** 查找第一个重复字符 */
public static char findFirstRepeatedChar(String string) {
   // 检查空字符串
   if (Objects.isNull(string) || string.isEmpty()) {
       return null;
  }

   // 查找重复字符
   char[] charArray = string.toCharArray();
   Set charSet = new HashSet<>(charArray.length);
   for (char ch : charArray) {
       if (charSet.contains(ch)) {
           return ch;
      }
       charSet.add(ch);
  }

   // 默认返回为空
   return null;
}
```

其中，由于 Set 的 add 函数有个特性——如果添加的元素已经再集合中存在，则会返回 false 。可以简化代码为：

```java
if (!charSet.add(ch)) {
   return ch;
}
```

## 使用HashMap存取键值映射关系

简单来说，HashMap 由数组和链表组成的，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的。如果定位到的数组位置不含链表，那么查找、添加等操作很快，仅需一次寻址即可，其时间复杂度为 O(1) ；如果定位到的数组包含链表，对于添加操作，其时间复杂度为 O(n) ——首先遍历链表，存在即覆盖，不存在则新增；对于查找操作来讲，仍需要遍历链表，然后通过key对象的 equals 方法逐一对比查找。从性能上考虑， HashMap 中的链表出现越少，即哈希冲突越少，性能也就越好。所以，在日常编码中，可以使用 HashMap 存取键值映射关系。
案例：给定菜单记录列表，每条菜单记录中包含父菜单标识（根菜单的父菜单标识为 null ），构建出整个菜单树。

```java
/** 菜单DO类 */
@Setter
@Getter
@ToString
public static class MenuDO {
   /** 菜单标识 */
   private Long id;
   /** 菜单父标识 */
   private Long parentId;
   /** 菜单名称 */
   private String name;
   /** 菜单链接 */
   private String url;
}

/** 菜单VO类 */
@Setter
@Getter
@ToString
public static class MenuVO {
   /** 菜单标识 */
   private Long id;
   /** 菜单名称 */
   private String name;
   /** 菜单链接 */
   private String url;
   /** 子菜单列表 */
   private List<MenuVO> childList;
}

/** 构建菜单树函数 */
public static List<MenuVO> buildMenuTree(List<MenuDO> menuList) {
   // 检查列表为空
   if (CollectionUtils.isEmpty(menuList)) {
       return Collections.emptyList();
  }

   // 依次处理菜单
   int menuSize = menuList.size();
   List<MenuVO> rootList = new ArrayList<>(menuSize);
   Map<Long, MenuVO> menuMap = new HashMap<>(menuSize);
   for (MenuDO menuDO : menuList) {
       // 赋值菜单对象
       Long menuId = menuDO.getId();
       MenuVO menu = menuMap.get(menuId);
       if (Objects.isNull(menu)) {
           menu = new MenuVO();
           menu.setChildList(new ArrayList<>());
           menuMap.put(menuId, menu);
      }
       menu.setId(menuDO.getId());
       menu.setName(menuDO.getName());
       menu.setUrl(menuDO.getUrl());

       // 根据父标识处理
       Long parentId = menuDO.getParentId();
       if (Objects.nonNull(parentId)) {
           // 构建父菜单对象
           MenuVO parentMenu = menuMap.get(parentId);
           if (Objects.isNull(parentMenu)) {
               parentMenu = new MenuVO();
               parentMenu.setId(parentId);
               parentMenu.setChildList(new ArrayList<>());
               menuMap.put(parentId, parentMenu);
          }
           
           // 添加子菜单对象
           parentMenu.getChildList().add(menu);
      } else {
           // 添加根菜单对象
           rootList.add(menu);
      }
  }

   // 返回根菜单列表
   return rootList;
}
```


*使用 ThreadLocal 存储线程专有对象*
ThreadLocal 提供了线程专有对象，可以在整个线程生命周期中随时取用，极大地方便了一些逻辑的实现。
常见的 ThreadLocal 用法主要有两种:

. 保存线程上下文对象，避免多层级参数传递；
. 保存非线程安全对象，避免多线程并发调用。

*保存线程上下文对象，避免多层级参数传递*
这里，以 PageHelper 插件的源代码中的分页参数设置与使用为例说明

*设置分页参数代码:*

```java
/** 分页方法类 */
public abstract class PageMethod {
   /** 本地分页 */
   protected static final ThreadLocal<Page> LOCAL_PAGE = new ThreadLocal<Page>();

   /** 设置分页参数 */
   protected static void setLocalPage(Page page) {
       LOCAL_PAGE.set(page);
  }

   /** 获取分页参数 */
   public static <T> Page<T> getLocalPage() {
       return LOCAL_PAGE.get();
  }

   /** 开始分页 */
   public static <E> Page<E> startPage(int pageNum, int pageSize, boolean count, Boolean reasonable, Boolean pageSizeZero) {
       Page<E> page = new Page<E>(pageNum, pageSize, count);
       page.setReasonable(reasonable);
       page.setPageSizeZero(pageSizeZero);
       Page<E> oldPage = getLocalPage();
       if (oldPage != null && oldPage.isOrderByOnly()) {
           page.setOrderBy(oldPage.getOrderBy());
      }
       setLocalPage(page);
       return page;
  }
}
```

*使用分页参数代码:*

```java
/** 虚辅助方言类 */
public abstract class AbstractHelperDialect extends AbstractDialect implements Constant {
   /** 获取本地分页 */
   public <T> Page<T> getLocalPage() {
       return PageHelper.getLocalPage();
  }

   /** 获取分页SQL */
   @Override
   public String getPageSql(MappedStatement ms, BoundSql boundSql, Object parameterObject, RowBounds rowBounds, CacheKey pageKey) {
       String sql = boundSql.getSql();
       Page page = getLocalPage();
       String orderBy = page.getOrderBy();
       if (StringUtil.isNotEmpty(orderBy)) {
           pageKey.update(orderBy);
           sql = OrderByParser.converToOrderBySql(sql, orderBy);
      }
       if (page.isOrderByOnly()) {
           return sql;
      }
       return getPageSql(sql, page, pageKey);
  }
  ...
}
```

*使用分页插件代码:*

```java
/** 查询用户函数 */
public PageInfo<UserDO> queryUser(UserQuery userQuery, int pageNum, int pageSize) {
 PageHelper.startPage(pageNum, pageSize);
 List<UserDO> userList = userDAO.queryUser(userQuery);
 PageInfo<UserDO> pageInfo = new PageInfo<>(userList);
 return pageInfo;
}
```

如果要把分页参数通过函数参数逐级传给查询语句，除非修改 MyBatis 相关接口函数，否则是不可能实现的。
*保存非线程安全对象，避免多线程并发调用*
在写日期格式化工具函数时，首先想到的写法如下:

```java
/** 日期模式 */
private static final String DATE_PATTERN = "yyyy-MM-dd";

/** 格式化日期函数 */
public static String formatDate(Date date) {
   return new SimpleDateFormat(DATE_PATTERN).format(date);
}
```
其中，每次调用都要初始化 DateFormat 导致性能较低，把 DateFormat 定义成常量后的写法如下:

```java
/** 日期格式 */
private static final DateFormat DATE_FORMAT = new SimpleDateFormat("yyyy-MM-dd");

/** 格式化日期函数 */
public static String formatDate(Date date) {
   return DATE_FORMAT.format(date);
}
```

由于 SimpleDateFormat 是非线程安全的，当多线程同时调用 formatDate 函数时，会导致返回结果与预期不一致。如果采用 ThreadLocal 定义线程专有对象，优化后的代码如下:

```java
/** 本地日期格式 */
private static final ThreadLocal<DateFormat> LOCAL_DATE_FORMAT = new ThreadLocal<DateFormat>() {
   @Override
   protected DateFormat initialValue() {
       return new SimpleDateFormat("yyyy-MM-dd");
  }
};

/** 格式化日期函数 */
public static String formatDate(Date date) {
   return LOCAL_DATE_FORMAT.get().format(date);
}
```
这是在没有线程安全的日期格式化工具类之前的实现方法。在 JDK8 以后，建议使用 DateTimeFormatter 代替 SimpleDateFormat ，因为 SimpleDateFormat 是线程不安全的，而 DateTimeFormatter 是线程安全的。当然，也可以采用第三方提供的线程安全日期格式化函数，比如 apache 的 DateFormatUtils 工具类。
注意：ThreadLocal 有一定的内存泄露的风险，尽量在业务代码结束前调用 remove 函数进行数据清除。

## 使用 Pair 实现成对结果的返回
[%hardbreaks]
在 C/C++ 语言中， Pair （对）是将两个数据类型组成一个数据类型的容器，比如 std::pair 。
Pair 主要有两种用途:

. 把 key 和 value 放在一起成对处理，主要用于 Map 中返回名值对，比如 Map 中的 Entry 类；
. 当一个函数需要返回两个结果时，可以使用 Pair 来避免定义过多的数据模型类。

第一种用途比较常见，这里主要说明第二种用途。

*定义模型类实现成对结果的返回*

```java
/** 点和距离类 */
@Setter
@Getter
@ToString
@AllArgsConstructor
public static class PointAndDistance {
   /** 点 */
   private Point point;
   /** 距离 */
   private Double distance;
}

/** 获取最近点和距离 */
public static PointAndDistance getNearestPointAndDistance(Point point, Point[] points) {
   // 检查点数组为空
   if (ArrayUtils.isEmpty(points)) {
       return null;
  }

   // 获取最近点和距离
   Point nearestPoint = points[0];
   double nearestDistance = getDistance(point, points[0]);
   for (int i = 1; i < points.length; i++) {
       double distance = getDistance(point, point[i]);
       if (distance < nearestDistance) {
           nearestDistance = distance;
           nearestPoint = point[i];
      }
  }

   // 返回最近点和距离
   return new PointAndDistance(nearestPoint, nearestDistance);
}
```

*函数使用案例*

```java
Point point = ...;
Point[] points = ...;
PointAndDistance pointAndDistance = getNearestPointAndDistance(point, points);
if (Objects.nonNull(pointAndDistance)) {
   Point point = pointAndDistance.getPoint();
   Double distance = pointAndDistance.getDistance();
  ...
}
```


*使用 Pair 类实现成对结果的返回*
在 JDK 中，没有提供原生的 Pair 数据结构，也可以使用 Map::Entry 代替。不过， Apache 的 commons-lang3 包中的 Pair 类更为好用，下面便以 Pair 类进行举例说明。
*函数实现代码:*

```java
/** 获取最近点和距离 */
public static Pair<Point, Double> getNearestPointAndDistance(Point point, Point[] points) {
   // 检查点数组为空
   if (ArrayUtils.isEmpty(points)) {
       return null;
  }

   // 获取最近点和距离
   Point nearestPoint = points[0];
   double nearestDistance = getDistance(point, points[0]);
   for (int i = 1; i < points.length; i++) {
       double distance = getDistance(point, point[i]);
       if (distance < nearestDistance) {
           nearestDistance = distance;
           nearestPoint = point[i];
      }
  }

   // 返回最近点和距离
   return Pair.of(nearestPoint, nearestDistance);
}
```

*函数使用案例:*

```java
Point point = ...;
Point[] points = ...;
Pair<Point, Double> pair = getNearestPointAndDistance(point, points);
if (Objects.nonNull(pair)) {
   Point point = pair.getLeft();
   Double distance = pair.getRight();
  ...
}
```

## 定义 Enum 类实现取值和描述
