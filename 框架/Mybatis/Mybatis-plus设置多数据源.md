# mybatis-plus设置多数据源

## 引入maven依赖

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.14</version>
        <relativePath/>
    </parent>

    <!-- MyBatis-plus 动态数据源-->
    <dependency>
      <groupId>com.baomidou</groupId>
      <artifactId>dynamic-datasource-spring-boot-starter</artifactId>
      <version>4.2.0</version>
    </dependency>
    <!-- MyBatis-plus 核心jar包-->
    <dependency>
      <groupId>com.baomidou</groupId>
      <artifactId>mybatis-plus-boot-starter</artifactId>
      <version>3.5.4</version>
    </dependency>

    <!-- sql-lite 数据库驱动-->
    <dependency>
      <groupId>org.xerial</groupId>
      <artifactId>sqlite-jdbc</artifactId>
      <version>3.43.0.0</version>
    </dependency>

```

## 设置启动配置文件

```yml
# 多数据源配置
spring:
  application:
    name: code-test
  datasource:
    dynamic:
      primary: sqlite-1 #主库必须配置
      datasource:
        sqlite-1:
          driver-class-name: org.sqlite.JDBC
          # url使用绝对路径，相对路径会在编译之后再target/class目录下生成一份，
          # 不建议使用相对路径，jar包部署后无法写入内容
          # url相对路径写法：jdbc:sqlite::resource:test.db 
          url: jdbc:sqlite:/Users/wangjiansheng/github/code-test/src/main/resources/db/test1.db
          username:
          password:
        sqlite-2:
          driver-class-name: org.sqlite.JDBC
          url: jdbc:sqlite:/Users/wangjiansheng/github/code-test/src/main/resources/db/test2.db
          username:
          password:
```

## 启动app上设置mapper包扫描

```java
@MapperScan("com.ding.qualification.mapper")
@SpringBootApplication
public class App {
    public static void main( String[] args )
    {
        SpringApplication.run(App.class, args);

    }

}
```

## 数据源切换

```java
//方法1
@DS("sqlite-1")
public List<QualificationsCate> getAll(){
    return dao.queryAll();
}
// 方法2
@Transactional
@DS("sqlite-2")
public int batchInsert(List<QualificationsCate> param){
    int size = 0;
    for (;size<param.size(); size++){
        dao.insert(param.get(size));
    }
    System.out.println(size);
    return size;
}
```

`mybatis-plus`的`@DS`注解可以自动切换数据源