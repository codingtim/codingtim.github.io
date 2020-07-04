---
title: "Configure the Spring ThreadPoolTaskExecutor."
tags:
  - java
  - spring
---

Exploring the configuration options of the Spring ThreadPoolTaskExecutor.

## Configuration

The default configuration of the Spring ThreadPoolTaskExecutor is described pretty well in the [javadoc](https://github.com/spring-projects/spring-framework/blob/master/spring-context/src/main/java/org/springframework/scheduling/concurrent/ThreadPoolTaskExecutor.java).

```
The default configuration is a core pool size of 1, with unlimited max pool size
and unlimited queue capacity. This is roughly equivalent to
java.util.concurrent.Executors#newSingleThreadExecutor(), sharing a single
thread for all tasks.
```

At first glance this seems weird. 
The max pool size and queue capacity are both unlimited so why are all the tasks executed in a single thread?
Internally a ThreadPoolTaskExecutor uses a native Java ThreadPoolExecutor. 
Snippet from the ThreadPoolExecutor [javadoc](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ThreadPoolExecutor.html):

```
When a new task is submitted in method execute(Runnable),
and fewer than corePoolSize threads are running, a new thread is
created to handle the request, even if other worker threads are
idle.  If there are more than corePoolSize but less than
maximumPoolSize threads running, a new thread will be created only
if the queue is full.
```

The ThreadPoolExecutor will only create a new thread once the queue is full. 
If we look back at the default configuration for the ThreadPoolTaskExecutor, the queue capacity is unlimited.
In such a configuration new tasks will always be queued and no new threads will ever be started. 

So suppose we change the configuration and use other values: 

```
corePoolSize=8
maxPoolSize=16
queueCapacity=200
```

This would mean the first 8 tasks would be started on threads, after which all tasks will be queued untill the capacity of 200 is reached.
Only then will new threads be started for newly submitted tasks, which hopefully leads to a decrease of tasks in the queue. 

In the event of a full queue and the max pool size has been reached. 
The next task submitted for execution will be rejected according to the configured RejectedExecutionHandler. 
By default this is the AbortPolicy which throws an error. 
Other pre-existing handers include CallerRunsPolicy, DiscardPolicy and DiscardOldestPolicy.

Let's compare this with the default Spring Boot configuration: 

```
corePoolSize=8
maxPoolSize=Integer.MAX_VALUE
queueCapacity=Integer.MAX_VALUE
```

Spring Boot will use 8 threads and queue everything after the first 8 tasks are running. 

Let's say we don't want to queue anything, how do we configure that? 
We can achieve this by setting the queue size to 0 and max pool size to unlimited: 

```
corePoolSize=1
maxPoolSize=Integer.MAX_VALUE
queueCapacity=0
```

This approach can lead to a high amount of threads being spawned and fighting over resources. 
This configuration is very similar to java.util.concurrent.Executors#newCachedThreadPool().

Another approach would be to first start threads untill the max pool size is reached and then start queueing. 
This approach is not supported out of the box but a solution can be found on [stackoverflow](https://stackoverflow.com/questions/19528304/how-to-get-the-threadpoolexecutor-to-increase-threads-to-max-before-queueing).

Other interesting configuration options include: 
- keepAliveSeconds: specifies how long a thread will be kept alive before being removed when idle (default 60)
- allowCoreThreadTimeOut: specifies whether threads in the core pool will be removed when idle  (default false)

## Shutdown

Now that we know how threads are started let's take a look at the shutdown behaviour of the ThreadPoolTaskExecutor.
The shutdown behaviour is located in the superclass ExecutorConfigurationSupport.
This class implements the DisposableBean interface.
When the ApplicationContext closes Spring will call the destroy() method which shuts down the native executor.

The default configuration is to interrupt all ongoing tasks and clear the queue. 
To  alter the shutdown behaviour there are 2 options:
- waitForTasksToCompleteOnShutdown: sets whether to wait for tasks to finish when shutting down (default false)
- awaitTerminationMillis: determines how long to wait for tasks to finish (default 0)

The javadoc on the set methods for these fields explains the behaviour very well.

When using long running tasks it might be interesting to periodically check whether the current Thread is interrupted. 
Once the ThreadPoolExecutor shutdown method is called `Thread.interrupted()` will return true.
This gives running tasks the possibility to stop cleanly before being killed.
