# Java 创建多线程的方式



> 来源: [微信公众号"编码之外"的文章-【无法接受！！！】一个Java多线程的问题，颠覆了我多年的认知！](https://mp.weixin.qq.com/s/xEgqmj8wmhtc9bnm3hP2hA)

## 碰见个奇怪的多线程问题

小白们也不用怕，今天的文章你们都能看得懂😁，最近的学习中，碰到这样的一个问题：Java创建多线程的方式有哪几种啊？ 你可能会说啦，这还不简单，不就是：

* 继承Thread类
* 实现Runnable接口

好像也是，如果你让我回答这个问题，我似乎也会这样回答，顶多我会再回答一个callable的方式，但是啊，最近看到这样的一个说法，让我陷入了深深的思考啊😂

**Java中创建多线程的方法有且仅有一种，那就是new Thread的方式**

嗯哼？这是怎么回事呢？这个就有点颠覆认知啊，我有点不敢相信了，那么这到底是怎么回事呢？看到这个回答我觉得我应该深入探讨下这个问题。

## 一般这问题都是怎么问的

关于上述说到的这个问题啊，并不是什么高深的问题，而且我们大多数人都能够回答上来，只不过可能回答的不全面，我这里带着大家去找找这个问题在面试题中是怎么出现的。
首先随便搜到了一套关于Java多线程的面试题，其中找到了关于本题的这种问法：

* 线程实现的方式有哪几种(4种)？

你可以思考下，这个问题让你回答，你会怎么回答，它说的四种，有哪四种？
这里我希望大家的着眼点应该是它怎么问的，它这里说的是线程的实现方式，记住，是实现方式，我们继续找找其他面试题：

* 线程创建有几种方式？

在这个版本中关于这个问题是这样问的，注意是**创建线程**，我们上面那一个说的是**实现线程**，是的，就是不同的说法，但是是一样的嘛？
如果我们在面试中被问到这样的问题，无论是问我们**创建线程的方式**还是**实现线程的方式**，我们的答案几乎一定是围绕着继承Thread类和实现Runnable接口这几个去说的，我相信应该不会有多少人上去就说：

* 创建线程的方式有且只有一种，那就是new Thread的方式

估计你这样的问答一定会被反问，为什么啊？是啊，为什么啊，其实看到这个回答，在我认真思考了之后我觉得这个说法是没有啥错误的。

## 难道我之前学的都是错的

我们一起来分析一下，在Java中啊，有这么个段子，就是没有女朋友的咋办，那就new一个啊，学习Java的都知道这是怎么回事，在Java中万物皆对象啊，创建对象一般就是new的方式了。
在Java中，Thread这个是线程类，按理说我们创建一个线程对象，那就应该是new Tread的方式啊，我们先来看我们平常都是怎么去创建一个线程的，一般的我们推荐实现接口的方式，这是源于Java的单继承多实现，我们来一起看下代码：

```java
class MyThread implements Runnable{
	@Override
	public void run(){
		System.out.println("实现Runnbale的方式……");
	}
}
```

当我们写完上述代码之后，我们就需要停下来思考以下了，这里我们创建了一个线程了嘛？我们这里貌似只是创建了一个实现了Runnbale接口的类，好像并没有哪里有体现我们创建线程了啊，我们来做个简单的测试：

```java
public class Test {
    public static void main(String[] args) {

        //获取线程数
        ThreadGroup threadGroup = Thread.currentThread().getThreadGroup();
        while(threadGroup.getParent() != null){
            threadGroup = threadGroup.getParent();
        }
        int totalThread = threadGroup.activeCount();
        System.out.println("当前线程数："+totalThread);
    }
}
```

我这里写了一段简单的程序，就是获取当前默认线程组中有几个线程，这段代码你不用去管他，只需要之道它有什么用，我们运行试一下：
这里是6，然后我们加上我们之前实现Runnbale那个类，一起来看下：

```java
public class Test {
    public static void main(String[] args) {

        //获取线程数
        ThreadGroup threadGroup = Thread.currentThread().getThreadGroup();
        while(threadGroup.getParent() != null){
            threadGroup = threadGroup.getParent();
        }
        int totalThread = threadGroup.activeCount();
        System.out.println("当前线程数："+totalThread);
    }
}

class MyThread implements Runnable{
    @Override
    public void run(){
        System.out.println("实现Runnbale的方式……");
    }
}
```

在main方法中并没有关于MyThread的体现，可想，目前线程数还是6，我们一般都是怎么使用这个MyThread的呢？是不是这样？

```java
public class Test {
    public static void main(String[] args) {
        
        Thread thread = new Thread(new MyThread());
        thread.start();

        //获取线程数
        ThreadGroup threadGroup = Thread.currentThread().getThreadGroup();
        while(threadGroup.getParent() != null){
            threadGroup = threadGroup.getParent();
        }
        int totalThread = threadGroup.activeCount();
        System.out.println("当前线程数："+totalThread);
    }
}
```

熟悉吧，我们一般都是这样操作的，这里想必大家也都知道，需要调用start才是真正的启用线程，我们再来运行下看看：
看吧，线程数增加了1，也打印出相关数据了，这才创建了一个线程，原因是我们写了这么些代码：

```java
Thread thread = new Thread(new MyThread());
        thread.start();
```

发现什么没，重点来了，就是这里的new Thread，我们接下来看看这样的代码：

```java
public class Test {
    public static void main(String[] args) {

        new Thread();

        //获取线程数
        ThreadGroup threadGroup = Thread.currentThread().getThreadGroup();
        while(threadGroup.getParent() != null){
            threadGroup = threadGroup.getParent();
        }
        int totalThread = threadGroup.activeCount();
        System.out.println("当前线程数："+totalThread);
    }
}
```

猜一下，现在的线程数是多少？
会不会有人说是7😂，知道为什么嘛，那是因为你没有调用start的啊，再来看：

```java
public class Test {
    public static void main(String[] args) {

        new Thread().start();1

        //获取线程数
        ThreadGroup threadGroup = Thread.currentThread().getThreadGroup();
        while(threadGroup.getParent() != null){
            threadGroup = threadGroup.getParent();
        }
        int totalThread = threadGroup.activeCount();
        System.out.println("当前线程数："+totalThread);
    }
}
```

这属于线程的基础知识了，题外话，你可知道为啥调用start而不是run嘛？
以上说明一个什么问题呢？真正的创建线程还是通过new Thread啊，然后调用start启动该线程，你看这个：

```java
Thread thread = new Thread(new MyThread());
        thread.start();
```

也是new Thread的方式，然后构造函数传入一个Runnbale，我们看看Thread的构造函数吧：
看到了吧，这里可以传入一个Runnable，我们继续往下思考。
创建线程干嘛

你想一下，我们创建线程干嘛，简单来说，是不是也是需要这个线程为我们干活啊，怎么干活嘞，简单来说是不是就是这个run方法啊：

```java
 @Override
    public void run(){
        System.out.println("实现Runnbale的方式……");
    }
```

我们在这个run方法中去执行一些任务，其实在Thread类中也有这个run方法，可以看一下：

```java
  @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
```

Thread类中的run方法没有具体的执行某些任务，而是去执行target中的run，这个target是啥：
**private Runnable target;**
是个Runnbale，你再看看我们实现Runnbale的MyThread的类：

```java
class MyThread implements Runnable{
    @Override
    public void run(){
        System.out.println("实现Runnbale的方式……");
    }
}
```

然后再看这个：

```java
Thread thread = new Thread(new MyThread());
        thread.start();
```

我想你应该明白了吧，这么一大圈就是为了去执行MyThread中的run方法，因为这是我们新建的这个线程要干的活啊。
可能我们以前真的错了

我们再看看长说的另一个方式，那就是继承Thread类的形式：

```java
class A extends Thread {
    @Override
    public void run() {
        System.out.println("继承Thread类的线程……");
    }
}
```

这个我们知道Thread类中有这个run方法并且上面也带大家看了，所以这里就是重写了run方法，而如果我们要启动这个线程则要这样：
new A().start();
这里的new A本质还是new Thread啊，不用解释吧，然后我们再看其他的方式，比如匿名内部类的方式：

```java
 new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("匿名内部类的方式创建线程");
            }
        }).start();
```
多么明显，还是new Thread啊，再继续看看实现callable的方式：

```java
class C implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        System.out.println("实现callable的形式创建的线程");
        return 1024;
    }
}
```

然后我们还需要这样：

```java
FutureTask<Integer> futureTask = new FutureTask<>(new C());
        Thread thread = new Thread(futureTask);
        thread.start();
        System.out.println(futureTask.get());
```

这里真的创建线程还是new Thread的方式。
所以经过上述的简单分析啊，我们之前的理解可能真的错了，我们经常说，创建线程的方式有什么继承Thread类的方式，还可以实现Runnable接口等等，但是现在看来，这似乎是错误的，正确的回答应该是：
创建线程的方式有且仅有一种，那就是new Thread()的方式
盘点之前的错误回答

说到这里我觉得有必要盘点一下我们之前的错误回答了，因为很多人即使按照之前的回答，要么回答的不全整，要么回答的不够好，首先，我们看看在之前我们最完整的回答应该包含以下几种方式：

* 继承Thread类
* 实现Runnable接口
* 匿名内部类
* 实现callable接口
* 使用线程池

以上五个回答是比较完整的了，一般啊，我们推荐实现接口的方式，这是源于java的单继承和多实现，另外实现callable和使用线程池在实际中应用的更多。
那么有些人可能会有疑惑了，你既然你说创建线程的方式有且仅有一种那就是new Thread的方式，那么上述这五种是干嘛的啊。

## 总结

是啊，那我们之前脱口而出的这些又是干嘛的呢？经过我们上面的分析，我想大家应该有看到，无论是继承Thread类还是实现Runnbale，又或者其实其他方式，好像目的就是为了去实现那个run方法（callable的不是），准确来说就是去执行我们真正要做的任务，也就是执行任务，也就是说啊，我们创建线程只有一种方式那就是new Thread的方式，但是你想啊，我们创建线程是让他干活的，那干啥活嘞，我们可以通过继承Thread类，然后重写run方法告诉线程该干嘛，又或者我们整一个Runnable，然后实现其中的run方法，然后把这个Runnable扔给Thread，告诉线程该干嘛，其他的也是同样的道理。
那么我们是不是可以理解为：
这些都是线程执行任务的方式，或者说是真正实现线程任务的方式，但是无论怎样，说是创建线程的方式，是不是有点不对呢？

```bash
#!bin/bash
#extracting command line options as parameters

.\bin\logstash -f .\conf\logstash_default.conf --path.data .\data\file-logstash-kafka-alarm -b 1000 -l .\logs\file-logstash-kafka-alarm --log.level error 2>&1 &
```