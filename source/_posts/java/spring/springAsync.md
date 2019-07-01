---
title: spring异步调用
date: 2019-06-28 11:08:54
tags: [java,spring]
---

### 1.快速上手

###### 1.添加@EnableAsync
```
@Configuration
@EnableAsync
public class AsyncConfig {
}

```
###### 2.异步方法
```
/**
 * 要执行的异步任务
 */
@Component
public class AsyncDoThings {

    public void doThing1() throws Exception{
        System.out.println("[*] doThing1 Start : " + System.currentTimeMillis());
        Thread.sleep(10000);
        System.out.println("[*] doThing1 end: " + System.currentTimeMillis());
    }

    @Async
    public void AsyncDoThing1() throws Exception{
        System.out.println("[*] AsyncDoThing1 Start : " + System.currentTimeMillis());
        Thread.sleep(10000);
        System.out.println("[*] AsyncDoThing1 end: " + System.currentTimeMillis());
    }
```
###### 3.执行
```
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootDemoApplicationTests {

    @Autowired
    AsyncDoThings asyncDoThings;

    @Test
    public void contextLoads() throws Exception{
        asyncDoThings.doThing1();
        asyncDoThings.AsyncDoThing1();
        asyncDoThings.doThing1();
    }

}
```
###### 4.结果
可以看到  异步任务 开始后，下一个任务立即执行，未发生阻塞等待<br>
![Async1](http://67.216.218.49:8000/file/blogs/java/spring/Async1.png)

### 2.稍微深入

###### 1.使用自己的线程池

实现AsyncConfiguer接口<br>
```
/**
 * 异步任务配置类
 */
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    /**
     * 使用自己的线程池
     * @return
     */
    @Override
    public Executor getAsyncExecutor(){
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(200);
        executor.setKeepAliveSeconds(60*30);
        //等待任务在关机时完成--表明等待所有线程执行完
        executor.setWaitForTasksToCompleteOnShutdown(true);
        //线程名前缀
        executor.setThreadNamePrefix("MyAsync-");
        executor.initialize();
        return executor;
    }

    /**
     * 处理异步任务中发生的异常
     * @return
     */
    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler(){
        //return new AsyncExecuteExceptionHandler();
        return new SimpleAsyncUncaughtExceptionHandler();
    }

    class AsyncExecuteExceptionHandler implements AsyncUncaughtExceptionHandler{
        @Override
        public void handleUncaughtException(Throwable ex, Method method, Object... params){
            System.out.println("[*]: 异步任务发生异常 " + ex.getMessage());
        }

    }

}

```

###### 2.获取异步任务的返回结果

```
/**
 * 要执行的异步任务
 */
@Component
public class AsyncDoThings {

    public void doThing1() throws Exception{
        System.out.println("[*] doThing1 Start : " + System.currentTimeMillis());
        System.out.println("[*] threadname " + Thread.currentThread().getName() );
        Thread.sleep(1000);
        System.out.println("[*] doThing1 end: " + System.currentTimeMillis());
    }

    @Async
    public void AsyncDoThing1() throws Exception{
        System.out.println("[*] AsyncDoThing1 Start : " + System.currentTimeMillis());
        System.out.println("[*] threadname " + Thread.currentThread().getName() );
        Thread.sleep(3000);
        System.out.println("[*] AsyncDoThing1 end: " + System.currentTimeMillis());
    }

    //获取返回结果
    @Async
    public Future<String> AsyncWithResponse(String tips) throws Exception{
        System.out.println("[*] AsyncWithResponse Start : " + System.currentTimeMillis());
        System.out.println("[*] threadname " + Thread.currentThread().getName() );
        Thread.sleep(3000);
        System.out.println("[*] AsyncWithResponse end: " + System.currentTimeMillis());
        return new AsyncResult<>(tips + " : finished! ");
    }
}

//测试类中
@Test
public void contextLoads() throws Exception{
    asyncDoThings.doThing1();
    asyncDoThings.AsyncDoThing1();
    asyncDoThings.doThing1();

    Future future = asyncDoThings.AsyncWithResponse("AsyncWithRes ");
    System.out.println("[*]: "  + Thread.currentThread().getName() + " " + future.get());
}
```

###### 3.结果
![Async2](http://67.216.218.49:8000/file/blogs/java/spring/Async2.png)
