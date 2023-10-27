## 什么是Spring Shell ##

并非所有应用程序都需要精美的Web用户界面！ 有时，使用交互式终端与应用程序进行交互是完成工作的最合适方法。

Spring Shell允许人们轻松创建这样一个可运行的应用程序，用户将在其中输入文本命令，这些命令将被执行直到程序终止。 Spring Shell项目提供了创建此类REPL（读取，评估，打印循环）的基础结构，从而使开发人员可以使用熟悉的Spring编程模型来专注于命令实现。

诸如解析，TAB完成，输出着色，精美的ascii-art表显示，输入转换和验证等高级功能都是免费提供的，而开发人员仅需专注于核心命令逻辑即可。

## 使用 Spring Shell ##

### 入门 ###

要查看Spring Shell提供的功能，让我们编写一个简单的Shell应用程序，该应用程序具有一个简单的命令将两个数字加在一起。

#### 简单启动的应用 ####

从版本2开始，Spring Shell已从头开始进行了重写，并牢记了各种增强功能，其中之一是易于与Spring Boot集成，尽管这并不是很强的要求。 就本教程而言，让我们创建一个简单的Boot应用程序，例如使用[start.spring.io](https://start.spring.io/)。 这个最小的应用程序仅依赖spring-boot-starter并配置spring-boot-maven-plugin，生成可执行文件über-jar：

```xml
...
<dependencies>
	<dependency>
    	<groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
...
```

#### 添加Spring Shell依赖 ####

开始使用Spring Shell的最简单方法是依赖 _spring-shell-starter_ 工件。 这附带了使用Spring Shell所需的一切，并且可以很好地与Boot配合使用，仅根据需要配置必要的bean：

```xml
...
<dependency>
    <groupId>org.springframework.shell</groupId>
    <artifactId>spring-shell-starter</artifactId>
    <version>2.0.0.RELEASE</version>
</dependency>
...
```

鉴于Spring Shell将借助存在的依赖关系启动并启动REPL，您将需要在本教程中构建跳过测试（ **-DskipTests**）或删除由 [start.spring.io](https://start.spring.io/) 生成的示例集成测试。  如果您不这样做，那么集成测试将创建 Spring **ApplicationContext**，并且根据您的构建工具，它将停留在eval循环中或因 **NPE** 而崩溃。

#### 第一个命令 ####

是时候添加我们的第一个命令了。 创建一个新类（随便命名），并用注解@ShellComponent（@Component的一种变体，用于限制扫描候选命令的类集）。

然后，创建一个采用两个整数（a和b）并返回其总和的add方法。 用@ShellMethod对其进行注释，并在注释中提供命令的描述（唯一需要的信息）：

```java
package com.example.demo;

import org.springframework.shell.standard.ShellMethod;
import org.springframework.shell.standard.ShellComponent;

@ShellComponent
public class MyCommands {

    @ShellMethod("Add two integers together.")
    public int add(int a, int b) {
        return a + b;
    }
}
```

#### 项目启动 ####

像这样构建应用程序并运行生成的jar。

```shell
./mvnw clean install -DskipTests
[...]
java -jar target/demo-0.0.1-SNAPSHOT.jar
```

以下屏幕将为您打招呼（标语来自Spring Boot，并且可以[照常](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-banner)进行 自定义）：

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.6.RELEASE)

shell:>
```

下面是一个黄色的 `shell :>` 提示符，邀请您键入命令。 键入添加1 2，然后按Enter并欣赏魔术！

### 编写自己的命令 ###

Spring Shell 决定将方法转换为实际Shell命令的方式是完全可插入的（请参阅扩展Spring Shell），但是从Spring Shell 2.x开始，推荐的编写命令的方法是使用本节中描述的新API（ 所谓的标准API）。

使用标准API，Bean上的方法将变成可执行命令，前提是

* bean类带有 @ShellComponent 注解。 这用于限制所考虑的bean的集合。
* 该方法带有 @ShellMethod 注解。

@ShellComponent 是一个构造型注解，它本身带有 @Component 元注释。 这样，除了过滤机制外，还可以使用它来声明bean（例如使用@ComponentScan）。

可以使用注解的value属性自定义创建的bean的名称。

#### 都与文档相关 ####

@ShellMethod 注解的唯一必需属性是其value属性，该属性应用于编写简短的单句描述命令的功能。 这很重要，这样您的用户就可以在不离开外壳程序的情况下获得有关命令的一致帮助（请参阅带有help命令的Integrated Documentation）。

您的命令说明应该简短，只能是一两个句子。 为了获得更好的一致性，建议以大写字母开头，以点结束。

#### 自定义命令名称 ####

默认情况下，无需为您的命令指定键（即在shell中用于调用它的单词）。 方法的名称将用作命令键，将camelCase名称转换为虚线的gnu样式名称（即sayHello（）将变为say-hello）。

但是，可以使用注解的key属性来显式设置命令键，如下所示：

```java
        @ShellMethod(value = "Add numbers.", key = "sum")
        public int add(int a, int b) {
                return a + b;
        }
```

