线程池使用示例：

```
private static final ThreadPoolExecutor thread = new ThreadPoolExecutor(5, 5, 1L, TimeUnit.MINUTES, new LinkedBlockingQueue<>(10), new ThreadPoolExecutor.CallerRunsPolicy());

thread.execute(() -> {
 crmImportService.insertCRMByBatchList(tempList);
});
```

参数（按顺序）：

```
corePoolSize 核心线程数
maximumPoolSize 最大线程数
keepAliveTime 空闲线程存活时间
TimeUnit  时间单位
workQueue  阻塞任务队列：如果线程都在用，就会将任务放入阻塞队列
threadFactory 新建线程工厂：如果线程池没有线程，使用该方法创建线程。该参数一般省略，使用默认的方法就可以。
RejectedExecutionHandler  拒绝机制：当任务数大于maximumPoolSize + wordQueue，就会执行该机制。
```

阿里编码规范，不推荐使用Executors工具类，如

`public ExecutorService threadPool = Executors.newFixedThreadPool(threadPoolCount);`

因为他们的`workQueue`是无界的，如果数据量大，任务都进入`workQueue`，容易OOM。

拒绝机制：

```
AbortPolicy 默认抛出异常。
CallerRunsPolicy：提交给该任务的线程来执行，任务不会丢。
DiscardPolicy：直接丢弃被拒绝的任务。
DiscardOldestPolicy: 丢弃队列首部的任务，提交这个新的任务。
```