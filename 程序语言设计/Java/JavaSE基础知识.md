# JavaSE基础知识

## 1.基本数据类型包装类
### equals方法
```java
// Long类型需要数值比较
Long long;
long.equals(0);
```
此时 `equals` 返回值为 `false` ，因为 `Long` 是长整型，要写`equals(0L)` 才正确
这是因为在 `Long` 包装类的 `equals` 方法中比较的是对象，`equals(0)` 在比较时进行自动装箱操作，将 `0` 转换成 `Integer` 类型，`equals` 方法中使用 `instance of` 判断了比较的对象类型，所以会返回 `false`

```java
// equals 方法的源码
public boolean equals(Object obj) {
    if (obj instanceof Long) {
        return value == ((Long)obj).longValue();
    }
    return false;
}
```

### valueOf 方法
`valueOf()` 方法和直接指定数值 `0L` 的方式有什么区别？
没有区别，`valueOf()` 方法编译后就是直接数值
```java
// 编译前的源码
Long a = 0L;
Long b = Long.valueOf(0L);

// 编译后的 class
Long a = 0L;
Long b = 0L;
```
不过通过查看 `valueOf` 源码发现一个新问题， 数值在 `-128` 到 `127`间的 `Long` 相同数值的对象地址是相同的
```java
// valueOf 方法源码
public static Long valueOf(long l) {
    final int offset = 128;
    if (l >= -128 && l <= 127) { // will cache
        return LongCache.cache[(int)l + offset];
    }
    return new Long(l);
}

//  因为都是在 cache 中获取，所以 == 判断对象地址也是相同的
Long a = 0L;
Long b = Long.valueOf(0L);

a == b  true
a.equals(b) true

Long c = 128L;
Long d = Long.valueOf(128L);

c == d false
c.equals(d) true
```

## 2.String类
### format方法的参数设置
- %d ：正常输出十进制数 。
- %Yd：十进制数，输出 Y 位。如果本身大于 Y 位，正常输出。
- %XYd：十进制数，输出 Y 位，不足 Y 位就补 X 。如果本身大于 Y 位，正常输出。

例如： %d，%2d， %02d 为例，
- %d：十进制数正常输出 。
- %2d：十进制数，输出 2 位。如果本身大于 2 位，正常输出。
- %02d ：十进制数，输出 2 位，不足 2 位就补 0 。如果本身大于 2 位，正常输出。

### DecimalFormat 格式化

以下是一些常用的 DecimalFormat 模式字符串字符以及它们的含义：

0：表示数字的占位符，如果数字不足此位数，将用零填充。例如，"###.00" 表示小数部分总是显示两位数字，整数部分至少显示三位数字。
#：与 0 类似，但如果数字不足此位数，则不会用零填充。例如，"#.##" 表示小数部分最多显示两位数字，整数部分最多显示一个数字。
.：小数点字符。在模式字符串中的小数点将用于分隔整数部分和小数部分。
,：千位分隔符。它将数字分隔成三位一组，通常用于整数部分。例如，"#,###" 将数字使用千位分隔符格式化。
-：减号字符。可以用于指示负数的位置。
%：百分比符号。将数字乘以100，并在末尾添加百分号符号。
E 或 e：科学计数法符号。用于显示科学计数法表示的数字。
'：撇号字符。用于在模式字符串中转义特殊字符。
以下是一些示例：

模式字符串 "0.00" 表示小数部分总是显示两位数字。
模式字符串 "#.##" 表示小数部分最多显示两位数字。
模式字符串 "#,###" 表示使用千位分隔符格式化整数部分。
模式字符串 "0.00%" 表示将数字乘以100，并以百分比形式显示，小数部分总是显示两位数字。
模式字符串 "0.###E0" 表示以科学计数法表示数字，并最多显示三位小数位。


## Java Builder模式

Effective Java 中遇到多个构造器参数时考虑构建器模式，与重叠构建器模式、JavaBeans模式相比，Builder模式实现的对象更利于使用。

## 三种模式

**重叠构造器**
```java
public class Person{
    private int id;
    private String name;
    private int age;
    private String sex;
    private String phone;
    
    Person(int id, String name){
        this.id = id;
        this.name = name;
    }
    
    Person(int id,String name, int age){
        this.id = id;
        this.name = name;
        this.age = age;
    }
    
    Person(int id, String name, int age, String sex){
        this.id = id;
        this.name = name;
        this.age = age;
        this.sex = sex;
    }
    
    Person(int id, String name, int age, String sex, String phone){
        this.id = id;
        this.name = name;
        this.age = age;
        this.sex = sex;
        this.phone = phone;
    }
}
```

**JavaBeans**

```java
public class Person{
    private int id;
    private String name;
    private int age;
    private String sex;
    private String phone;
    
    public void setId(int id){
        this.id = id;
    }
    
    public int getId(){
        return this.id;
    }
    
    public void setName(String name){
        this.name = name;
    }
    
    public String getName(){
        return this.name;
    }
    
    public void setAge(int age){
        this.age = age;
    }
    
    public int getAge(){
        return this.age;
    }
    
    public void setSex(String sex){
        this.sex = sex;
    }
    
    public String getSex(){
        return this.sex;
    }
    
    public void setPhone(String phone){
        this.phone = phone;
    }
    
    public String getPhone(String phone){
        return this.phone;
    }
}
```

**Builder模式**

```java
public class Person{
    private int id;
    private String name;
    private int age;
    private String sex;
    private String phone;
    
    private Person(Builder builder){
        this.id = builder.id;
        this.name = builder.name;
        this.age = builder.age;
        this.sex = builder.sex;
        this.phone = phone;
    }
    
    public static class Builder{
        private int id;
        private String name;
        private int age;
        private String sex;
        private String phone;
        
        public Builder id(int id){
            this.id = id;
        }
        
        public Builder name(String name){
            this.name = name;
        }
        
        public Builder age(int age){
            this.age = age;
        }
        
        public Builder sex(String sex){
            this.sex = sex;
        }
        
        public Builder phone(String phone){
            this.phone = phone;
        }
        
        public Person build(){
            return new Person(this);
        }
    }
}
```

## lombok 使用Builder模式

`@Builder` 注解可以为类生成复杂的构造器
`@Builder` 注解的作用域可以是类、构造函数和普通方法，在类和构造函数上最常见

### @Builder 做了什么

1. 创建了一个名为 `ThisClassBuilder` 的内部静态类，具有和实体相同的属性
2. 目标类中所有的属性和未初始化的 `final` 字段，都会在 `Builder` 中创建对应属性
3. `Builder` 类中创建设置参数的参数名同名方法，便于链式调用
4. `Builder` 创建一个 `build()` 方法，用于创建实体对象
5. 在目标类中创建一个 `builder()` 方法，用来创建 `Builder` 类实体

```java
@Builder
public class User {
    private final Integer code = 200;
    private String username;
    private String password;
}

// 编译后
public class User {
    private String username;
    private String password;
    User(String username, String password){
        this.username = username; this.password = password;
    }
    
    public static User.UserBuilder builder(){
        return new User.UserBuilder();
    }
    
    public static class UserBuilder {
        private String username;
        private String password;
        UserBuilder(){}
        
        public User.UserBuilder username(String username){
            this.username = username;
            return this;
        }
        
        public User.UserBuilder password(String password){
            this.password = password;
            return this;
        }
        
        public User build(){
            return new User(this.username, this.password);
        }
        
        public String toString(){
            return "User.UserBuilder(username="+this.username+",password="+this.password+")";
        }
    }
}
```

### 集合元素

使用 `@Singular` 注解集合类型的字段，`lombok` 会为集合字段生成两个 `add` 方法：一个向集合添加单个元素，一个向集合添加所有元素

`@Singular` 只能应用于 `lombok` 已知的集合类型，目前支持的类型有：

- java.util:
    - Iterable、Collection、List
    - Set、SortedSet、NavigableSet
    - Map、SortedMap、NavigableMap
- Guava
    - ImmutableCollection、ImmutableList
    - ImmutableSet、ImmutableSortedSet
    - ImmutableMap、ImmutableBiMap、ImmutableSortedMap
    - ImmutableTable

```java
@Builder
public class User {
    private final Integer id;
    private String username;
    private String password;
    @Singular
    private List<String> hobbies;
}

// 编译后
public class User {
    private final Integer id;
    private String username;
    private String password;
    private List<String> hobbies;
    User(Integer id, String username, String password, List<String> hobbies){
        this.id = id;this.username = username;
        this.password = password;this.hobbies = hobbies;
    }
    
    public static User.UserBuilder builder(){
        return new User.UserBuilder();
    }
    
    public static class UserBuilder {
        private Integer id;
        private String username;
        private String password;
        private ArrayList<String> hobbies;
        
        UserBuilder(){}
        public User.UserBuilder id(Integer id){this.id = id; return this;}
        public User.UserBuilder username(String username){
            this.username = username;
            return this;
        }
        
        public User.UserBuilder password(String password){
            this.password = password;
            return this;
        }
        
        public User.UserBuilder hobby(String hobby){
            if(this.hobbies == null){
                this.hobbies = new ArrayList();
            }
            this.hobbies.add(hobby);
            return this;
        }
        
        public User.UserBuilder hobbies(Collection<? extends String> hobbies) {
            if(this.hobbies == null){
                this.hobbies = new ArrayList();
            }
            this.hobbies.addAll(hobbies);
            return this;
        }
        
        public User.UserBuilder clearHobbies(){
            if(this.hobbies != null){
                this.hobbies.clear();
            }
            return this;
        }
        
        public User build(){
            List hobbies;
            switch(this.hobbies == null ? 0 : this.hobbies.size()){
                case 0:
                    hobbies = Collections.emptyList();
                    break;
                case 1:
                    hobbies = Collections.singletonList(this.hobbies.get(0));
                    break;
                default:
                    hobbies = Collection.unmodifiableList(new ArrayList(this.hobbies));
            }
            return new User(this.username, this.password,hobbies);
        }
        
        public String toString(){
            return "User.UserBuilder(username="+this.username+",password="+this.password+",hobbies="+this.hobbies+")";
        }
    }
}
```