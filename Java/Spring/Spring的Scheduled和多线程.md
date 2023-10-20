# Spring的Scheduled和多线程

在我们日常的开发中，经常用到数据同步的更新，这时我们采用的是spring的定时任务和java的多线程进行数据的更新，进行时实的服务调用。
二.实现思路

1. 创建线程类
2. 创建ExecutorService线程连接池
3. 调用线程池操作
4. spring的Scheduled（定时任务）配置 
5. 
三.创建线程类

```java
    public class ArchiveTask implements Runnable {
      
    /**
     * log.
     */
    private static Logger logger = LoggerFactory.getLogger(ArchiveTask.class);
      
    //注：由于spring事务配置采用的注解的方式来配置传播性配置，此类没有在spring传播配置里面，所以不能用自动装配的方式来配置
    private static FaxRecvArchiveCtrl faxRecvArchiveCtrl = AppContextAware.getBean(FaxRecvArchiveCtrl.class);
  
    private FaxRecvArchiveDTO faxRecvArchiveDTO;
      
    /**
     *  CountDownLatch是一个同步辅助类，在一组线程在执行操作之前，允许一个或多线程处于等待
     *  主要方法
     *          public CountDownLatch(int count);
     *          public void countDown();
     *          public void await() throws InterruptedException
     *
     *          构造方法参数指定了计数的次数
     *          countDown方法，当前线程调用此方法，则计数减一
     *          awaint方法，调用此方法会一直阻塞当前线程，直到计时器的值为0
     *
     */
    private CountDownLatch latch;
      
       
    public ArchiveTask(FaxRecvArchiveDTO faxRecvArchiveDTO, CountDownLatch latch) {
        this.faxRecvArchiveDTO = faxRecvArchiveDTO;
        this.latch = latch;
    }
  
    @Override
    public void run() {
        try {
            faxRecvArchiveCtrl.updateArchiveFileFlag(ServiceFactory.getCurrentSessionId(), faxRecvArchiveDTO);
        } catch (Exception ex) {
            logger.debug("["+this.faxRecvArchiveDTO.getFaxKey()+"]线程执行异常，错误信息为："+ex.getMessage());
        } finally{
            logger.debug("线程执行完："+this.faxRecvArchiveDTO.getFaxKey());
            latch.countDown();
        }
    }
}
```