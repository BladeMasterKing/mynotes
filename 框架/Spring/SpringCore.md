# SpringCore

2.1.1  IoC是什么
Ioc—Inversion of Control，即“控制反转”，不是什么技术，而是一种设计思想。在Java开发中，Ioc意味着将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制。如何理解好Ioc呢？理解好Ioc的关键是要明确“谁控制谁，控制什么，为何是反转（有反转就应该有正转了），哪些方面反转了”，那我们来深入分析一下：
 
●谁控制谁，控制什么：传统Java SE程序设计，我们直接在对象内部通过new进行创建对象，是程序主动去创建依赖对象；而IoC是有专门一个容器来创建这些对象，即由Ioc容器来控制对象的创建；谁控制谁？当然是IoC 容器控制了对象；控制什么？那就是主要控制了外部资源获取（不只是对象包括比如文件等）。
●为何是反转，哪些方面反转了：有反转就有正转，传统应用程序是由我们自己在对象中主动控制去直接获取依赖对象，也就是正转；而反转则是由容器来帮忙创建及注入依赖对象；为何是反转？因为由容器帮我们查找及注入依赖对象，对象只是被动的接受依赖对象，所以是反转；哪些方面反转了？依赖对象的获取被反转了。
用图例说明一下，传统程序设计如图2-1，都是主动去创建相关对象然后再组合起来：


 
图2-1 传统应用程序示意图
当有了IoC/DI的容器后，在客户端类中不再主动去创建这些对象了，如图2-2所示:
 ![spring__2](media/16885421599405/spring__2.jpeg)


图2-2有IoC/DI容器后程序结构示意图
 
1.1.2  IoC能做什么
IoC不是一种技术，只是一种思想，一个重要的面向对象编程的法则，它能指导我们如何设计出松耦合、更优良的程序。传统应用程序都是由我们在类内部主动创建依赖对象，从而导致类与类之间高耦合，难于测试；有了IoC容器后，把创建和查找依赖对象的控制权交给了容器，由容器进行注入组合对象，所以对象与对象之间是松散耦合，这样也方便测试，利于功能复用，更重要的是使得程序的整个体系结构变得非常灵活。
其实IoC对编程带来的最大改变不是从代码上，而是从思想上，发生了“主从换位”的变化。应用程序原本是老大，要获取什么资源都是主动出击，但是在IoC/DI思想中，应用程序就变成被动的了，被动的等待IoC容器来创建并注入它所需要的资源了。
IoC很好的体现了面向对象设计法则之一—— 好莱坞法则：“别找我们，我们找你”；即由IoC容器帮对象找相应的依赖对象并注入，而不是由对象主动去找。
 
2.1.3  IoC和DI
DI—Dependency Injection，即“依赖注入”：是组件之间依赖关系由容器在运行期决定，形象的说，即由容器动态的将某个依赖关系注入到组件之中。依赖注入的目的并非为软件系统带来更多功能，而是为了提升组件重用的频率，并为系统搭建一个灵活、可扩展的平台。通过依赖注入机制，我们只需要通过简单的配置，而无需任何代码就可指定目标需要的资源，完成自身的业务逻辑，而不需要关心具体的资源来自何处，由谁实现。
 
理解DI的关键是：“谁依赖谁，为什么需要依赖，谁注入谁，注入了什么”，那我们来深入分析一下：
 
●谁依赖于谁：当然是应用程序依赖于IoC容器；
●为什么需要依赖：应用程序需要IoC容器来提供对象需要的外部资源；
●谁注入谁：很明显是IoC容器注入应用程序某个对象，应用程序依赖的对象；
●注入了什么：就是注入某个对象所需要的外部资源（包括对象、资源、常量数据）。
 
IoC和DI由什么关系呢？其实它们是同一个概念的不同角度描述，由于控制反转概念比较含糊（可能只是理解为容器控制对象这一个层面，很难让人想到谁来维护对象关系），所以2004年大师级人物Martin Fowler又给出了一个新的名字：“依赖注入”，相对IoC 而言，“依赖注入”明确描述了“被注入对象依赖IoC容器配置依赖对象”。
注：如果想要更加深入的了解IoC和DI，请参考大师级人物Martin Fowler的一篇经典文章《Inversion of Control Containers and the Dependency Injection pattern》，原文地址：http://www.martinfowler.com/articles/injection.html。
2.2.1  IoC容器的概念
IoC容器就是具有依赖注入功能的容器，IoC容器负责实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。应用程序无需直接在代码中new相关的对象，应用程序由IoC容器进行组装。在Spring中BeanFactory是IoC容器的实际代表者。
Spring IoC容器如何知道哪些是它管理的对象呢？这就需要配置文件，Spring IoC容器通过读取配置文件中的配置元数据，通过元数据对应用中的各个对象进行实例化及装配。一般使用基于xml配置文件进行配置元数据，而且Spring与配置文件完全解耦的，可以使用其他任何可能的方式进行配置元数据，比如注解、基于java文件的、基于属性文件的配置都可以。
那Spring IoC容器管理的对象叫什么呢？
2.2.2  Bean的概念
由IoC容器管理的那些组成你应用程序的对象我们就叫它Bean， Bean就是由Spring容器初始化、装配及管理的对象，除此之外，bean就与应用程序中的其他对象没有什么区别了。那IoC怎样确定如何实例化Bean、管理Bean之间的依赖关系以及管理Bean呢？这就需要配置元数据，在Spring中由BeanDefinition代表，后边会详细介绍，配置元数据指定如何实例化Bean、如何组装Bean等。概念知道的差不多了，让我们来做个简单的例子。
2.2.3  Hello World
一、配置环境：
l       JDK安装：安装最新的JDK，至少需要Java 1.5及以上环境；
l       开发工具：SpringSource Tool Suite，简称STS，是个基于Eclipse的开发环境，用以构建Spring应用，其最新版开始支持Spring 3.0及OSGi开发工具，但由于其太庞大，很多功能不是我们所必需的所以我们选择Eclipse+ SpringSource Tool插件进行Spring应用开发；到eclipse官网下载最新的Eclipse，注意我们使用的是Eclipse IDE for Java EE Developers（eclipse-jee-helios-SR1）；
安装插件：启动Eclipse，选择Help->Install New Software，如图2-3所示
 

 

图2-3 安装
2、首先安装SpringSource Tool Suite插件依赖，如图2-4:
Name为：SpringSource Tool Suite Dependencies
Location为：http://dist.springsource.com/release/TOOLS/composite/e3.6
 

图2-4 安装
3、安装SpringSource Tool Suite插件，只需安装如图2-5所选中的就可以：
Name为：SpringSource Tool Suite
Location为：http://dist.springsource.com/release/TOOLS/update/e3.6
 

图2-4 安装
4、安装完毕，开始项目搭建吧。
Spring 依赖：本书使用spring-framework-3.0.5.RELEASE
                    spring-framework-3.0.5.RELEASE-with-docs.zip表示此压缩包带有文档的；
                    spring-framework-3.0.5.RELEASE-dependencies.zip表示此压缩包中是spring的依赖jar包，所以需要什么依赖从这里找就好了；
                   下载地址：http://www.springsource.org/download
 
二、开始Spring Hello World之旅
1、准备需要的jar包
  核心jar包：从下载的spring-framework-3.0.5.RELEASE-with-docs.zip中dist目录查找如下jar包
org.springframework.asm-3.0.5.RELEASE.jar
org.springframework.core-3.0.5.RELEASE.jar
org.springframework.beans-3.0.5.RELEASE.jar
org.springframework.context-3.0.5.RELEASE.jar
org.springframework.expression-3.0.5.RELEASE.jar
   依赖的jar包：从下载的spring-framework-3.0.5.RELEASE-dependencies.zip中查找如下依赖jar包
com.springsource.org.apache.log4j-1.2.15.jar
com.springsource.org.apache.commons.logging-1.1.1.jar
com.springsource.org.apache.commons.collections-3.2.1.jar
2、创建标准Java工程：
（1）选择“window”—> “Show View” —>“Package Explorer”，使用包结构视图；
 

图2-5 包结构视图
（2）创建标准Java项目，选择“File”—>“New”—>“Other”；然后在弹出来的对话框中选择“Java Project”创建标准Java项目；
 

图2-6 创建Java项目

 
图2-7 创建Java项目
 

图2-8 创建Java项目
       （3）配置项目依赖库文件，右击项目选择“Properties”；然后在弹出的对话框中点击“Add JARS”在弹出的对话框中选择“lib”目录下的jar包；然后再点击“Add Library”，然后在弹出的对话框中选择“Junit”，选择“Junit4”；
 

图2-9 配置项目依赖库文件
 

图2-10 配置项目依赖库文件
 


图2-11 配置项目依赖库文件
（4）项目目录结构如下图所示，其中“src”用于存放java文件；“lib”用于存放jar文件；“resources”用于存放配置文件；
  

图2-12 项目目录结构
 
3、项目搭建好了，让我们来开发接口，此处我们只需实现打印“Hello World!”，所以我们定义一个“sayHello”接口，代码如下：
package cn.javass.spring.chapter2.helloworld;  
public interface HelloApi {  
       public void sayHello();  
}  
 
4、接口开发好了，让我们来通过实现接口来完成打印“Hello World!”功能；
package cn.javass.spring.chapter2.helloworld;  
public class HelloImpl implements HelloApi {  
              @Override  
              public void sayHello() {  
                     System.out.println("Hello World!");  
              }  
}  
 
5、接口和实现都开发好了，那如何使用Spring IoC容器来管理它们呢？这就需要配置文件，让IoC容器知道要管理哪些对象。让我们来看下配置文件chapter2/helloworld.xml（放到resources目录下）：
<?xml version="1.0" encoding="UTF-8"?>  
<beans  
xmlns="http://www.springframework.org/schema/beans"  
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
xmlns:context="http://www.springframework.org/schema/context"  
xsi:schemaLocation="  
http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
http://www.springframework.org/schema/context                http://www.springframework.org/schema/context/spring-context-3.0.xsd">  
  <!-- id 表示你这个组件的名字，class表示组件类 -->  
<bean id="hello" class="cn.javass.spring.chapter2.helloworld.HelloImpl"></bean>  
</beans>  
 
6、现在万一具备，那如何获取IoC容器并完成我们需要的功能呢？首先应该实例化一个IoC容器，然后从容器中获取需要的对象，然后调用接口完成我们需要的功能，代码示例如下：
package cn.javass.spring.chapter2.helloworld;  
import org.junit.Test;  
import org.springframework.context.ApplicationContext;  
import org.springframework.context.support.ClassPathXmlApplicationContext;  
public class HelloTest {  
       @Test  
       public void testHelloWorld() {  
             //1、读取配置文件实例化一个IoC容器  
             ApplicationContext context = new ClassPathXmlApplicationContext("helloworld.xml");  
             //2、从容器中获取Bean，注意此处完全“面向接口编程，而不是面向实现”  
              HelloApi helloApi = context.getBean("hello", HelloApi.class);  
              //3、执行业务逻辑  
              helloApi.sayHello();  
       }  
}  
   
 
7、自此一个完整的Spring Hello World已完成，是不是很简单，让我们深入理解下容器和Bean吧。
2.2.4  详解IoC容器
在Spring Ioc容器的代表就是org.springframework.beans包中的BeanFactory接口，BeanFactory接口提供了IoC容器最基本功能；而org.springframework.context包下的ApplicationContext接口扩展了BeanFactory，还提供了与Spring AOP集成、国际化处理、事件传播及提供不同层次的context实现 (如针对web应用的WebApplicationContext)。简单说， BeanFactory提供了IoC容器最基本功能，而 ApplicationContext 则增加了更多支持企业级功能支持。ApplicationContext完全继承BeanFactory，因而BeanFactory所具有的语义也适用于ApplicationContext。
容器实现一览：
• XmlBeanFactory：BeanFactory实现，提供基本的IoC容器功能，可以从classpath或文件系统等获取资源；
  （1）  File file = new File("fileSystemConfig.xml");
           Resource resource = new FileSystemResource(file);
           BeanFactory beanFactory = new XmlBeanFactory(resource);
  （2）
          Resource resource = new ClassPathResource("classpath.xml");                 
          BeanFactory beanFactory = new XmlBeanFactory(resource);
 
• ClassPathXmlApplicationContext：ApplicationContext实现，从classpath获取配置文件；
         BeanFactory beanFactory = new ClassPathXmlApplicationContext("classpath.xml");
• FileSystemXmlApplicationContext：ApplicationContext实现，从文件系统获取配置文件。
         BeanFactory beanFactory = new FileSystemXmlApplicationContext("fileSystemConfig.xml");
 
具体代码请参考cn.javass.spring.chapter2.InstantiatingContainerTest.java。
 
ApplicationContext接口获取Bean方法简介：
• Object getBean(String name) 根据名称返回一个Bean，客户端需要自己进行类型转换；
• T getBean(String name, Class<T> requiredType) 根据名称和指定的类型返回一个Bean，客户端无需自己进行类型转换，如果类型转换失败，容器抛出异常；
• T getBean(Class<T> requiredType) 根据指定的类型返回一个Bean，客户端无需自己进行类型转换，如果没有或有多于一个Bean存在容器将抛出异常；
• Map<String, T> getBeansOfType(Class<T> type) 根据指定的类型返回一个键值为名字和值为Bean对象的 Map，如果没有Bean对象存在则返回空的Map。
 
让我们来看下IoC容器到底是如何工作。在此我们以xml配置方式来分析一下：
 
一、准备配置文件：就像前边Hello World配置文件一样，在配置文件中声明Bean定义也就是为Bean配置元数据。
二、由IoC容器进行解析元数据： IoC容器的Bean Reader读取并解析配置文件，根据定义生成BeanDefinition配置元数据对象，IoC容器根据BeanDefinition进行实例化、配置及组装Bean。
三、实例化IoC容器：由客户端实例化容器，获取需要的Bean。
 
整个过程是不是很简单，执行过程如图2-5，其实IoC容器很容易使用，主要是如何进行Bean定义。下一章我们详细介绍定义Bean。
 

图2-5 Spring Ioc容器
2.2.5  小结
除了测试程序的代码外，也就是程序入口，所有代码都没有出现Spring任何组件，而且所有我们写的代码没有实现框架拥有的接口，因而能非常容易的替换掉Spring，是不是非入侵。
客户端代码完全面向接口编程，无需知道实现类，可以通过修改配置文件来更换接口实现，客户端代码不需要任何修改。是不是低耦合。
如果在开发初期没有真正的实现，我们可以模拟一个实现来测试，不耦合代码，是不是很方便测试。
Bean之间几乎没有依赖关系，是不是很容易重用。

2.3.1  XML配置的结构
一般配置文件结构如下：
<beans>  
    <import resource=”resource1.xml”/>  
    <bean id=”bean1”class=””></bean>  
    <bean id=”bean2”class=””></bean>  
<bean name=”bean2”class=””></bean>  
    <alias alias="bean3" name="bean2"/>  
    <import resource=”resource2.xml”/>  
</beans>  
 
 
1、<bean>标签主要用来进行Bean定义；
2、alias用于定义Bean别名的；
3、import用于导入其他配置文件的Bean定义，这是为了加载多个配置文件，当然也可以把这些配置文件构造为一个数组（new String[] {“config1.xml”, config2.xml}）传给ApplicationContext实现进行加载多个配置文件，那一个更适合由用户决定；这两种方式都是通过调用Bean Definition Reader 读取Bean定义，内部实现没有任何区别。<import>标签可以放在<beans>下的任何位置，没有顺序关系。
 
2.3.2  Bean的配置
Spring IoC容器目的就是管理Bean，这些Bean将根据配置文件中的Bean定义进行创建，而Bean定义在容器内部由BeanDefinition对象表示，该定义主要包含以下信息：
●全限定类名（FQN）：用于定义Bean的实现类;
●Bean行为定义：这些定义了Bean在容器中的行为；包括作用域（单例、原型创建）、是否惰性初始化及生命周期等；
●Bean创建方式定义：说明是通过构造器还是工厂方法创建Bean；
●Bean之间关系定义：即对其他bean的引用，也就是依赖关系定义，这些引用bean也可以称之为同事bean 或依赖bean，也就是依赖注入。
Bean定义只有“全限定类名”在当使用构造器或静态工厂方法进行实例化bean时是必须的，其他都是可选的定义。难道Spring只能通过配置方式来创建Bean吗？回答当然不是，某些SingletonBeanRegistry接口实现类实现也允许将那些非BeanFactory创建的、已有的用户对象注册到容器中，这些对象必须是共享的，比如使用DefaultListableBeanFactory 的registerSingleton() 方法。不过建议采用元数据定义。
 
2.3.3    Bean的命名
       每个Bean可以有一个或多个id（或称之为标识符或名字），在这里我们把第一个id称为“标识符”，其余id叫做“别名”；这些id在IoC容器中必须唯一。如何为Bean指定id呢，有以下几种方式；
 
 
一、  不指定id，只配置必须的全限定类名，由IoC容器为其生成一个标识，客户端必须通过接口“T getBean(Class<T> requiredType)”获取Bean；

<bean class=” cn.javass.spring.chapter2.helloworld.HelloImpl”/>              （1）  
 
测试代码片段如下：
@Test  
public void test1() {  
BeanFactory beanFactory =  
   new ClassPathXmlApplicationContext("chapter2/namingbean1.xml");  
    //根据类型获取bean  
    HelloApi helloApi = beanFactory.getBean(HelloApi.class);  
    helloApi.sayHello();  
}  
 
 
 二、指定id，必须在Ioc容器中唯一；

<bean id=” bean” class=” cn.javass.spring.chapter2.helloworld.HelloImpl”/>    （2）  
 
测试代码片段如下：
@Test  
public void test2() {  
BeanFactory beanFactory =  
new ClassPathXmlApplicationContext("chapter2/namingbean2.xml");  
//根据id获取bean  
    HelloApi bean = beanFactory.getBean("bean", HelloApi.class);  
    bean.sayHello();  
}  
 
 
三、指定name，这样name就是“标识符”，必须在Ioc容器中唯一；

<bean name=” bean” class=” cn.javass.spring.chapter2.helloworld.HelloImpl”/> （3）  
 
测试代码片段如下：
@Test  
public void test3() {  
    BeanFactory beanFactory =  
new ClassPathXmlApplicationContext("chapter2/namingbean3.xml");  
    //根据name获取bean  
HelloApi bean = beanFactory.getBean("bean", HelloApi.class);  
bean.sayHello();  
}  
 
 
四、指定id和name，id就是标识符，而name就是别名，必须在Ioc容器中唯一；
<bean id=”bean1”name=”alias1”  
class=” cn.javass.spring.chapter2.helloworld.HelloImpl”/>  
<!-- 如果id和name一样，IoC容器能检测到，并消除冲突 -->  
<bean id="bean3" name="bean3" class="cn.javass.spring.chapter2.helloworld.HelloImpl"/>              （4）  
 
测试代码片段如下：
@Test  
public void test4() {  
BeanFactory beanFactory =  
new ClassPathXmlApplicationContext("chapter2/namingbean4.xml");  
    //根据id获取bean  
    HelloApi bean1 = beanFactory.getBean("bean1", HelloApi.class);  
    bean1.sayHello();  
    //根据别名获取bean  
    HelloApi bean2 = beanFactory.getBean("alias1", HelloApi.class);  
    bean2.sayHello();  
    //根据id获取bean  
    HelloApi bean3 = beanFactory.getBean("bean3", HelloApi.class);  
    bean3.sayHello();  
    String[] bean3Alias = beanFactory.getAliases("bean3");  
    //因此别名不能和id一样，如果一样则由IoC容器负责消除冲突  
    Assert.assertEquals(0, bean3Alias.length);  
}  
 
 
五、指定多个name，多个name用“，”、“；”、“ ”分割，第一个被用作标识符，其他的（alias1、alias2、alias3）是别名，所有标识符也必须在Ioc容器中唯一；

<bean name=” bean1;alias11,alias12;alias13 alias14”  
      class=” cn.javass.spring.chapter2.helloworld.HelloImpl”/>     
<!-- 当指定id时，name指定的标识符全部为别名 -->  
<bean id="bean2" name="alias21;alias22"  
class="cn.javass.spring.chapter2.helloworld.HelloImpl"/>              （5）  
 
  测试代码片段如下：
@Test  
public void test5() {  
BeanFactory beanFactory =  
new ClassPathXmlApplicationContext("chapter2/namingbean5.xml");  
    //1根据id获取bean  
    HelloApi bean1 = beanFactory.getBean("bean1", HelloApi.class);  
    bean1.sayHello();  
    //2根据别名获取bean  
    HelloApi alias11 = beanFactory.getBean("alias11", HelloApi.class);  
    alias11.sayHello();  
    //3验证确实是四个别名         
    String[] bean1Alias = beanFactory.getAliases("bean1");  
    System.out.println("=======namingbean5.xml bean1 别名========");  
    for(String alias : bean1Alias) {  
        System.out.println(alias);  
    }  
    Assert.assertEquals(4, bean1Alias.length);  
    //根据id获取bean  
    HelloApi bean2 = beanFactory.getBean("bean2", HelloApi.class);  
    bean2.sayHello();  
    //2根据别名获取bean  
    HelloApi alias21 = beanFactory.getBean("alias21", HelloApi.class);  
    alias21.sayHello();  
    //验证确实是两个别名  
    String[] bean2Alias = beanFactory.getAliases("bean2");  
    System.out.println("=======namingbean5.xml bean2 别名========");  
    for(String alias : bean2Alias) {  
        System.out.println(alias);  
    }     
    Assert.assertEquals(2, bean2Alias.length);     
}  
 
 
六、使用<alias>标签指定别名，别名也必须在IoC容器中唯一
<bean name="bean" class="cn.javass.spring.chapter2.helloworld.HelloImpl"/>  
<alias alias="alias1" name="bean"/>  
<alias alias="alias2" name="bean"/>                                  （6）  
 
 测试代码片段如下：
@Test  
public void test6() {  
BeanFactory beanFactory =  
new ClassPathXmlApplicationContext("chapter2/namingbean6.xml");  
    //根据id获取bean  
    HelloApi bean = beanFactory.getBean("bean", HelloApi.class);  
   bean.sayHello();  
    //根据别名获取bean  
    HelloApi alias1 = beanFactory.getBean("alias1", HelloApi.class);  
    alias1.sayHello();  
    HelloApi alias2 = beanFactory.getBean("alias2", HelloApi.class);  
    alias2.sayHello();  
    String[] beanAlias = beanFactory.getAliases("bean");  
    System.out.println("=======namingbean6.xml bean 别名========");  
    for(String alias : beanAlias) {  
        System.out.println(alias);  
   }  
   System.out.println("=======namingbean6.xml bean 别名========");  
    Assert.assertEquals(2, beanAlias.length);  
}  
 
以上测试代码在cn.javass.spring.chapter2.NamingBeanTest.java文件中。
 
从定义来看，name或id如果指定它们中的一个时都作为“标识符”，那为什么还要有id和name同时存在呢？这是因为当使用基于XML的配置元数据时，在XML中id是一个真正的XML id属性，因此当其他的定义来引用这个id时就体现出id的好处了，可以利用XML解析器来验证引用的这个id是否存在，从而更早的发现是否引用了一个不存在的bean，而使用name，则可能要在真正使用bean时才能发现引用一个不存在的bean。
 
 
 ●Bean命名约定：Bean的命名遵循XML命名规范，但最好符合Java命名规范，由“字母、数字、下划线组成“，而且应该养成一个良好的命名习惯， 比如采用“驼峰式”，即第一个单词首字母开始，从第二个单词开始首字母大写开始，这样可以增加可读性。
 
2.3.4  实例化Bean
Spring IoC容器如何实例化Bean呢？传统应用程序可以通过new和反射方式进行实例化Bean。而Spring IoC容器则需要根据Bean定义里的配置元数据使用反射机制来创建Bean。在Spring IoC容器中根据Bean定义创建Bean主要有以下几种方式：
一、使用构造器实例化Bean：这是最简单的方式，Spring IoC容器即能使用默认空构造器也能使用有参数构造器两种方式创建Bean，如以下方式指定要创建的Bean类型：
 
使用空构造器进行定义，使用此种方式，class属性指定的类必须有空构造器
 
<bean name="bean1" class="cn.javass.spring.chapter2.HelloImpl2"/>  
 

使用有参数构造器进行定义，使用此中方式，可以使用< constructor-arg >标签指定构造器参数值，其中index表示位置，value表示常量值，也可以指定引用，指定引用使用ref来引用另一个Bean定义，后边会详细介绍：
<bean name="bean2" class="cn.javass.spring.chapter2.HelloImpl2">  
<!-- 指定构造器参数 -->  
     <constructor-arg index="0" value="Hello Spring!"/>  
</bean>  
 
知道如何配置了，让我们做个例子的例子来实践一下吧：
（1）准备Bean class(HelloImpl2.java)，该类有一个空构造器和一个有参构造器：
package cn.javass.spring.chapter2;   
  public class HelloImpl2 implements HelloApi {  
           private String message;  
      public HelloImpl2() {  
                  this.message = "Hello World!";  
           }  
         Public HelloImpl2(String message) {  
                  this.message = message;  
           }  
           @Override  
           public void sayHello() {  
                  System.out.println(message);  
           }  
}  
   
 
（2）在配置文件(resources/chapter2/instantiatingBean.xml)配置Bean定义，如下所示：
<!--使用默认构造参数-->  
<bean name="bean1" class="cn.javass.spring.chapter2.HelloImpl2"/>  
    <!--使用有参数构造参数-->  
   
<bean name="bean2" class="cn.javass.spring.chapter2.HelloImpl2">  
<!-- 指定构造器参数 -->  
    <constructor-arg index="0" value="Hello Spring!"/>  
</bean>  
 
 
（3）配置完了，让我们写段测试代码（InstantiatingContainerTest）来看下是否工作吧：
@Test  
public void testInstantiatingBeanByConstructor() {  
       //使用构造器  
     BeanFactory beanFactory =  
new ClassPathXmlApplicationContext("chapter2/instantiatingBean.xml");  
       HelloApi bean1 = beanFactory.getBean("bean1", HelloApi.class);  
       bean1.sayHello();  
       HelloApi bean2 = beanFactory.getBean("bean2", HelloApi.class);  
       bean2.sayHello();  
}  
 
 
二、使用静态工厂方式实例化Bean，使用这种方式除了指定必须的class属性，还要指定factory-method属性来指定实例化Bean的方法，而且使用静态工厂方法也允许指定方法参数，spring IoC容器将调用此属性指定的方法来获取Bean，配置如下所示：
（1）先来看看静态工厂类代码吧HelloApiStaticFactory：
public class HelloApiStaticFactory {  
    //工厂方法  
       public static HelloApi newInstance(String message) {  
              //返回需要的Bean实例  
           return new HelloImpl2(message);  
}  
}  
 
 
（2）静态工厂写完了，让我们在配置文件(resources/chapter2/instantiatingBean.xml)配置Bean定义：
<!-- 使用静态工厂方法 -->  
<bean id="bean3" class="cn.javass.spring.chapter2.HelloApiStaticFactory" factory-method="newInstance">  
     <constructor-arg index="0" value="Hello Spring!"/>  
</bean>  
 
 
（3）配置完了，写段测试代码来测试一下吧，InstantiatingBeanTest：
@Test  
public void testInstantiatingBeanByStaticFactory() {  
       //使用静态工厂方法  
       BeanFactory beanFactory =  
new ClassPathXmlApplicationContext("chaper2/instantiatingBean.xml");  
       HelloApi bean3 = beanFactory.getBean("bean3", HelloApi.class);  
       bean3.sayHello();  
}  
 
 
三、使用实例工厂方法实例化Bean，使用这种方式不能指定class属性，此时必须使用factory-bean属性来指定工厂Bean，factory-method属性指定实例化Bean的方法，而且使用实例工厂方法允许指定方法参数，方式和使用构造器方式一样，配置如下：
（1）实例工厂类代码（HelloApiInstanceFactory.java）如下：
package cn.javass.spring.chapter2;  
public class HelloApiInstanceFactory {  
public HelloApi newInstance(String message) {  
          return new HelloImpl2(message);  
   }  
}  
   
   
 
（2）让我们在配置文件(resources/chapter2/instantiatingBean.xml)配置Bean定义：
 

<!—1、定义实例工厂Bean -->  
<bean id="beanInstanceFactory"  
class="cn.javass.spring.chapter2.HelloApiInstanceFactory"/>  
<!—2、使用实例工厂Bean创建Bean -->  
<bean id="bean4"  
factory-bean="beanInstanceFactory"  
     factory-method="newInstance">  
 <constructor-arg index="0" value="Hello Spring!"></constructor-arg>  
</bean>  
 
（3）测试代码InstantiatingBeanTest：

@Test  
public void testInstantiatingBeanByInstanceFactory() {  
//使用实例工厂方法  
       BeanFactory beanFactory =  
new ClassPathXmlApplicationContext("chapter2/instantiatingBean.xml");  
       HelloApi bean4 = beanFactory.getBean("bean4", HelloApi.class);  
       bean4.sayHello();  
}  
 
       通过以上例子我们已经基本掌握了如何实例化Bean了，大家是否注意到？这三种方式只是配置不一样，从获取方式看完全一样，没有任何不同。这也是Spring IoC的魅力，Spring IoC帮你创建Bean，我们只管使用就可以了，是不是很简单。
 
2.3.5  小结
       到此我们已经讲完了Spring IoC基础部分，包括IoC容器概念，如何实例化容器，Bean配置、命名及实例化，Bean获取等等。不知大家是否注意到到目前为止，我们只能通过简单的实例化Bean，没有涉及Bean之间关系。接下来一章让我们进入配置Bean之间关系章节，也就是依赖注入。 
 
