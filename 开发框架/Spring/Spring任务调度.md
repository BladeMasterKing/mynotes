# Spring任务调度

Spring Framework分别通过**TaskExecutor**和**TaskScheduler**接口提供了异步执行和任务调度的抽象。Spring还提供了那些接口的实现，这些接口在应用程序服务器环境中支持线程池或委托给CommonJ。最终，在公共接口后面使用这些实现可以抽象化Java SE 5，Java SE 6和Java EE环境之间的差异。

Spring还具有集成类，以支持Timer （从1.3开始成为JDK的一部分）和 [Quartz Scheduler](<https://www.quartz-scheduler.org/) 进行调度。您可以分别使用FactoryBean带有可选引用> Timer或Trigger实例的方式来设置这两个调度程序。此外，Timer还提供了Quartz Scheduler和的便利类，可用于调用现有目标对象的方法（类似于正常MethodInvokingFactoryBean 操作）。

#### 1. spring TaskExecutor

Executors是线程池概念的JDK名称。“执行程序”的命名是由于无法保证基础实现实际上是一个池。执行程序可能是单线程的，甚至是同步的。Spring的抽象隐藏了Java SE和Java EE环境之间的实现细节。

Spring的TaskExecutor接口与该java.util.concurrent.Executor 接口相同。实际上，最初，其存在的主要原因是在使用线程池时抽象出对Java 5的需求。该接口具有一个方法（execute(Runnable task)），该方法根据线程池的语义和配置接受要执行的任务。

在TaskExecutor最初创建在需要时给其他Spring组件的一个抽象的线程池。比如ApplicationEventMulticaster，JMS的AbstractMessageListenerContainer，和Quartz的整合都使用了 TaskExecutor抽象线程池。但是，如果您的bean需要线程池行为，则也可以根据自己的需要使用此抽象。

##### TaskExecutor 种类

Spring包含的许多预构建实现TaskExecutor。您极有可能无需实现自己的方法。Spring提供的变体如下：
* **SyncTaskExecutor**：此实现不会异步执行调用。而是，每个调用都在调用线程中进行。它主要用于不需要多线程的情况下，例如在简单的测试案例中。
* **SimpleAsyncTaskExecutor**注意：此实现不重用任何线程。而是，它为每次调用启动一个新线程。但是，它确实支持并发限制，该限制会阻止超出限制的所有调用，直到释放插槽为止。如果您正在寻找真正的池，请参阅ThreadPoolTaskExecutor此列表后面的。
* **ConcurrentTaskExecutor**：此实现是java.util.concurrent.Executor实例的适配器。有一个替代方法（ThreadPoolTaskExecutor）将Executor 配置参数公开为bean属性。很少需要ConcurrentTaskExecutor直接使用 。但是，如果ThreadPoolTaskExecutor不能满足您的需求，ConcurrentTaskExecutor则可以选择。
* **ThreadPoolTaskExecutor**：此实现是最常用的。它公开了用于配置的bean属性，java.util.concurrent.ThreadPoolExecutor并将其包装在中TaskExecutor。如果您需要适应其他类型的java.util.concurrent.Executor，我们建议您ConcurrentTaskExecutor改用。
* **WorkManagerTaskExecutor**：此实现使用CommonJ WorkManager作为其支持服务提供者，并且是在Spring应用程序上下文中的WebLogic或WebSphere上设置基于CommonJ的线程池集成的中心便利类。
* **DefaultManagedTaskExecutor**：此实现使用ManagedExecutorService在JSR-236兼容的运行时环境（例如Java EE 7+应用程序服务器）中获得的JNDI，为此替换了CommonJ WorkManager。

##### 使用TaskExecutor

Spring的TaskExecutor的实现被用作简单JavaBean，下面的实例中我们使用ThreadPoolTaskExecutor同步打印一些消息。

```java
import org.springframework.core.task.TaskExecutor;
public class TaskExecutorExample {
    private class MessagePrinterTask implements Runnable {
        private String message;
        public MessagePrinterTask(String message) {
            this.message = message;
        }
        public void run() {
            System.out.println(message);
        }
    }
    private TaskExecutor taskExecutor;
    public TaskExecutorExample(TaskExecutor taskExecutor) {
        this.taskExecutor = taskExecutor;
    }
    public void printMessages() {
        for(int i = 0; i < 25; i++) {
            taskExecutor.execute(new MessagePrinterTask("Message" + i));
        }
    }
}
```

不是自己从线程池检索线程并执行，而是用户自己将runnable添加到队列，然后TaskExecutor根据规则选择合适来执行。

```xml
<bean id="taskExecutor" class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
    <property name="corePoolSize" value="5"/>
    <property name="maxPoolSize" value="10"/>
    <property name="queueCapacity" value="25"/>
</bean>

<bean id="taskExecutorExample" class="TaskExecutorExample">
    <constructor-arg ref="taskExecutor"/>
</bean>
```

#### 2. Spring TaskScheduler

除了TaskExecutor抽象之外，Spring 3.0还引入了TaskScheduler 带有多种方法的调度任务在将来某个时刻运行的方法。以下清单显示了TaskScheduler接口定义：

```java
public interface TaskScheduler {
    ScheduledFuture schedule(Runnable task, Trigger trigger);
    ScheduledFuture schedule(Runnable task, Instant startTime);
    ScheduledFuture schedule(Runnable task, Date startTime);
    ScheduledFuture scheduleAtFixedRate(Runnable task, Instant startTime, Duration period);
    ScheduledFuture scheduleAtFixedRate(Runnable task, Date startTime, long period);
    ScheduledFuture scheduleAtFixedRate(Runnable task, Duration period);
    ScheduledFuture scheduleAtFixedRate(Runnable task, long period);
    ScheduledFuture scheduleWithFixedDelay(Runnable task, Instant startTime, Duration delay);
    ScheduledFuture scheduleWithFixedDelay(Runnable task, Date startTime, long delay);
    ScheduledFuture scheduleWithFixedDelay(Runnable task, Duration delay);
    ScheduledFuture scheduleWithFixedDelay(Runnable task, long delay);
}
```

最简单的schedule方法是只包含一个Runnable和一个Date。这将导致任务在指定时间后运行一次。其他所有方法都可以安排任务重复运行。固定速率和固定延迟方法用于简单的定期执行，但是接受一个Trigger的方法则更加灵活。

##### Trigger接口

Trigger接口实质上受到JSR-236的启发，JSR-236从Spring 3.0开始尚未正式实现，Trigger的基本思想是，执行时间也许是基于之前的执行结果或者是任意条件来决定的。如果之前的执行结果确实纳入这些决定条件，那么在TriggerContext的信息是可用的。trigger接口相当的简单：

```java
public interface Trigger {
    Date nextExecutionTime(TriggerContext triggerContext);
}
```

TriggerContext 是最重要的部分，，它囊括了所有的相关数据并且在将来必要时开放扩展，TriggerContext 是一个接口，SimpleTriggerContext是默认使用的实现:
```java
public interface TriggerContext {
    Date lastScheduledExecutionTime();
    Date lastActualExecutionTime();
    Date lastCompletionTime();
}
```

##### Trigger实现

Spring提供了Trigger接口的两个实现，最有意思的一个是**CronTrigger**，它能够基于Cron表达式调度任务。例如，下面的任务在每个小时的15分钟时运行，仅在每个工作日的9点到5点工作时间:

```java
    scheduler.schedule(task, new CronTrigger("0 15 9-17 * * MON-FRI"));
```
另一个实现是**PeriodicTrigger**，它接受一个固定的周期，一个初始的延时值，和一个boolean来指示该周期是应解释为固定速率还是固定延迟。TaskScheduler 接口已经定义了以固定速率或固定延迟调度任务的方法，因此应尽可能直接使用这些方法。PeriodicTrigger 的价值是可以在依赖Trigger抽象的组件中使用它。

##### PeriodicTrigger 实现

与Spring的TaskExecutor一样，TaskScheduler的主要优点在于应用程序的Scheduler与部署环境解耦。在部署到不应由应用程序本身直接创建线程的服务器环境时，抽象级别特别重要。

#### 3. 支持调度和异步执行的注解

##### 3.1 开启调度注解

```java
    @Configuration
    @EnableAsync
    @EnableScheduling
    public class AppConfig {
    }
```

这些注解是可选择的，例如仅需要支持@Scheduled，则可以忽略@EnableAsync注解。更细粒度的控制，可以实现SchedulingConfigurer接口，AsyncConfigurer 接口。\
如果偏爱XML控制：

```xml
<task:annotation-driven executor="myExecutor" scheduler="myScheduler"/>
<task:executor id="myExecutor" pool-size="5"/>
<task:scheduler id="myScheduler" pool-size="10"/>
```

##### 3.2 @Scheduled注解

带有trigger的元数据添加@Scheduled注解到一个方法。例如，下面的方法每隔5秒就以固定的延时调用一次，意味着周期地从之前一次的完成时间开始计算的:

```java
@Scheduled(fixedDelay=5000)
public void doSomething() {
    // something that should execute periodically
}
```

如果需要以固定的速率执行，例如每5秒执行一次：

```
@Scheduled(fixedRate=5000)
public void doSomething() {
    // something that should execute periodically
}
```

对于固定延时和固定速率的任务，可以通过指定第一次执行方法之前要等待的毫秒数来指定初始延时:

```java
@Scheduled(initialDelay=1000, fixedRate=5000)
public void doSomething() {
    // something that should execute periodically
}
```

cron表达式

```java
@Scheduled(cron="*/5 * * * * MON-FRI")
public void doSomething() {
    // something that should execute on weekdays only
}
```

**调度方法必须是空返回值**，
@Scheduled不能注解同一个类的多个实例
@Scheduled注解与@Configurable注解不要在同一个类上使用，否则将会初始化两次导致每个@Scheduled方法调用两次。

##### 3.3 @Async注解

@Async注解用于方法，是方法的调用为异步，调用方法后立即返回，而方法实际交给了TaskExecutor来执行。最简单例如空返回值：

```java
@Async
void doSomething() {
    // this will be executed asynchronously
}
```

异步方法的返回值，必须是Future类型

```java
@Async
Future<String> returnSomething(int i) {
    // this will be executed asynchronously
}
```

@Async不能与@PostConstruct类似的生命周期回调混合使用，需要单独使用：

```java
public class SampleBeanImpl implements SampleBean {
    @Async
    void doSomething() {
        // ...
    }
}

public class SampleBeanInitializer {
    private final SampleBean bean;
    public SampleBeanInitializer(SampleBean bean) {
        this.bean = bean;
    }
    @PostConstruct
    public void initialize() {
        bean.doSomething();
    }
}
```

##### 3.4 @Async执行条件

默认情况下，在@Async指定一个方法异步时，使用的executor是开启异步支持配置的那个也就是注解驱动的元素，也可以使用XML或者AsyncConfigurer 的实现，指定默认方法以外的执行器

```java
@Async("otherExecutor")
void doSomething(String s) {
    // this will be executed asynchronously by "otherExecutor"
}
```

这种情况下"otherExecutor"可以是spring容器中任何Bean的名称，也可以是容器相关的限定符。

##### 3.5 @Async异常管理

当@Async注解的方法有Future返回值，是易于管理在方法执行过程抛出的异常，可以调用Future的get方法拿到返回值。对于void类型的，异常未被捕获，可以提供AsyncUncaughtExceptionHandler来处理异常：

```java
public class MyAsyncUncaughtExceptionHandler implements AsyncUncaughtExceptionHandler {
    @Override
    public void handleUncaughtException(Throwable ex, Method method, Object... params) {
        // handle exception
    }
}
```

#### 4. Task名称空间

##### 4.1 scheduler元素

创建指定线程池大小的ThreadPoolTaskScheduler：

```xml
<task:scheduler id="scheduler" pool-size="10"/>
```

id是池中线程名称的前缀，不设置pool-size属性，默认线程池中只有一个线程，没有其他选项

##### 4.2 executor元素

创建ThreadPoolTaskExecutor实例：

```xml
<task:executor id="executor" pool-size="10"/>
```

id与pool-size属性一样，ThreadPoolTaskExecutor更具有可配置性，pool-size属性如果设置单个值，则线程池大小固定，也可以设置min-max的形式：

```xml
<task:executor
        id="executorWithPoolSizeRange"
        pool-size="5-25"
        queue-capacity="100"/>
```

queue-capacity设置的值，根据executor的队列容量考虑线程池的配置。提交任务时，如果活跃的线程数小于核心线程数，executor首先尝试使用空闲线程；若达到核心线程大小，未达到容量上限，则将任务添加到队列中；如果超过最大容量，则拒绝该任务。\
默认情况下，队列是无界的，可能导致OutOfMemory。因为队列是无界的，线程池大小的设置完全无效，executor在创建超过核心线程的新线程之前会尝试队列，队列必须具有有限的容量。\
使用拒绝策略(AbortPolicy)会引发异常，高负载下跳过某些任务，可以选择DiscardPolicy或DiscardOldestPolicy；重负载下限制提交任务最好选择CallerRunsPolicy，不会引发异常或放弃任务。

```xml
<task:executor
        id="executorWithCallerRunsPolicy"
        pool-size="5-25"
        queue-capacity="100"
        rejection-policy="CALLER_RUNS"/>
```

设置keep-alive确定线程在终止之前可以保持空闲状态的时间限制。线程池中超过核心数的线程，在不处理任务等待超过keep-alive的时间后，将会被终止。

```xml
<task:executor
        id="executorWithKeepAlive"
        pool-size="5-25"
        keep-alive="120"/>
```

##### 4.3 scheduled-tasks

spring task最强大的功能是支持在Spring Application Context中配置调度的任务。ref属性指向spring管理的bean，method属性提供对象上调用的方法：

```xml
<task:scheduled-tasks scheduler="myScheduler">
    <task:scheduled ref="beanA" method="methodA" fixed-delay="5000"/>
</task:scheduled-tasks>

<task:scheduler id="myScheduler" pool-size="10"/>
```

调度程序中每个单独的任务都包括触发元数据配置，元数据定义了具有fixed-delay(固定延迟)的周期性触发器，fixed-rate指定了方法的执行频率，与@Scheduled注解配置相同：

```xml
    <task:scheduled-tasks scheduler="myScheduler">
        <task:scheduled ref="beanA" method="methodA" fixed-delay="5000" initial-delay="1000"/>
        <task:scheduled ref="beanB" method="methodB" fixed-rate="5000"/>
        <task:scheduled ref="beanC" method="methodC" cron="*/5 * * * * MON-FRI"/>
    </task:scheduled-tasks>

    <task:scheduler id="myScheduler" pool-size="10"/>
```

#### 5. Quartz Scheduler

Quartz使用Trigger，Job和JobDetail实现任务调度，spring提供了两个类，简化spring中使用Quartz的过程。

##### 5.1 JobDetailFactoryBean

Quartz的JobDetail中包含运行作业所需的所有信息，JobDetailFactoryBean提供了XML配置：

```xml
    <bean name="exampleJob" class="org.springframework.scheduling.quartz.JobDetailFactoryBean">
        <property name="jobClass" value="example.ExampleJob"/>
        <property name="jobDataAsMap">
            <map>
                <entry key="timeout" value="5"/>
            </map>
        </property>
    </bean>
```

```java
package example;
public class ExampleJob extends QuartzJobBean {
    private int timeout;
    /**
     * Setter called after the ExampleJob is instantiated
     * with the value from the JobDetailFactoryBean (5)
     */
    public void setTimeout(int timeout) {
        this.timeout = timeout;
    }
    protected void executeInternal(JobExecutionContext ctx) throws JobExecutionException {
        // do the actual work
    }
}
```

##### 5.2 MethodInvokingJobDetailFactoryBean

通常只需要在特定对象上调用方法：

```xml
<bean id="jobDetail" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
    <property name="targetObject" ref="exampleBusinessObject"/>
    <property name="targetMethod" value="doIt"/>
</bean>
```

这样会直接调用doIt方法：

```java
    public class ExampleBusinessObject {
        // properties and collaborators
        public void doIt() {
            // do the actual work
        }
    }
```
```xml
    <bean id="exampleBusinessObject" class="examples.ExampleBusinessObject"/>
```
如果为同一个JobDetail指定两个触发器，可能在第一个任务完成之前启动第二个任务，如果JobDetail类实现了Stateful接口，不会发生这个问题。concurrent设置为false，将会使MethodInvokingJobDetailFactoryBean将并发产生的作业会变为非并发。:

```xml
<bean id="jobDetail" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
    <property name="targetObject" ref="exampleBusinessObject"/>
    <property name="targetMethod" value="doIt"/>
    <property name="concurrent" value="false"/>
</bean>
```

##### 5.3 SchedulerFactoryBean

通过使用trigger和SchedulerFactoryBean来安排调度工作，spring提供了两个便捷的Quartz的FactoryBean默认实现：CronTriggerFactoryBean 和 SimpleTriggerFactoryBean。\
trigger需要被调度，spring提供一个**SchedulerFactoryBean**，将trigger作为属性暴露出来，SchedulerFactoryBean通过这些trigger调度实际的作业。

```xml
<bean id="simpleTrigger" class="org.springframework.scheduling.quartz.SimpleTriggerFactoryBean">
    <!-- see the example of method invoking job above -->
    <property name="jobDetail" ref="jobDetail"/>
    <!-- 10 seconds -->
    <property name="startDelay" value="10000"/>
    <!-- repeat every 50 seconds -->
    <property name="repeatInterval" value="50000"/>
</bean>

<bean id="cronTrigger" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
    <property name="jobDetail" ref="exampleJob"/>
    <!-- run every morning at 6 AM -->
    <property name="cronExpression" value="0 0 6 * * ?"/>
</bean>
```

要完成最后的调度工作，需要设置SchedulerFactoryBean：

```xml
<bean class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
    <property name="triggers">
        <list>
            <ref bean="cronTrigger"/>
            <ref bean="simpleTrigger"/>
        </list>
    </property>
</bean>
```