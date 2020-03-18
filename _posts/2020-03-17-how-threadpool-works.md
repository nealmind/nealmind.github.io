---
layout: post
title: Java线程池工作原理
tags: 线程池
author: neal

---
* content
{:toc}

## Java中线程池工作原理解析

首先看下ThreadPoolExecutor构造函数：
```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler){
	/*******************省略部分代码********************/
}
```
主要看几个参数含义：
* `corePoolSize`:
核心线程数，当线程池中活跃线程达到核心线程数时，新添加的任务
会添加到阻塞队列中等待执行；
* `maximumPoolSize`:
最大线程数，当线程池中的工作线程数达到最大线程数时，执行拒绝策略；
* `keepAliveTime`:
空闲线程活跃时间，如果当前线程数超过了`corePoolSize`并且`allowCoreThreadTimeOut`配置为true，当空闲线程等待时间超过了
配置的`keepAliveTime`时，中断空闲的线程，否则一直等待新的任务提交；
* `workQueue`: 阻塞队列，用以保存任务；
* `threadFactory`: 创建线程工厂类
* `handler`: 拒绝策略；





## 线程池工作逻辑
当任务提交时：
1. 首先检查当前线程数是否超过`corePoolSize`，如果未超过则创建一个线程执行任务，
否则，添加任务到阻塞队列；
2. 如果阻塞队列已满，则判断是否达到`maximumPoolSize`，如果没达到，则创建线程执行任务，
否则执行拒绝策略；

首先看execute方法：
```java
    public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
       
        int c = ctl.get();
		//1. 当前工作线程小于corePoolSize，直接创建线程并执行
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
		//2.如果线程池正在运行并且添加任务到阻塞队列成功：
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
			//再次检查线程池状态，并根据结果选择是否移除任务并执行拒绝策略
            if (! isRunning(recheck) && remove(command))
                reject(command);
			//如果工作线程在reCheck期间都结束了，即当前工作线程空了，则添加一个新的工作线程
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false))
            reject(command);
    }
```
接下来看addWorker方法，其主要执行逻辑是新建线程执行任务

```java
/**
 * 主要通过新建Worker类并调用其run方法来实现线程的执行
 * Worker类是封装了执行线程和任务(可能为null)的类，实现了
 * Runnable和继承了AQS的封装类类
 * firstTask：线程启动时首先执行的任务，为null则去队列中获取；
 * core：是否用corePoolSize作为判断条件，否则用maximumPoolSize
 */
    private boolean addWorker(Runnable firstTask, boolean core) {
	//自旋更新工作线程数量
        retry:
        for (;;) {
            int c = ctl.get();
            int rs = runStateOf(c);

            // Check if queue empty only if necessary.
            if (rs >= SHUTDOWN &&
                ! (rs == SHUTDOWN &&
                   firstTask == null &&
                   ! workQueue.isEmpty()))
                return false;

            for (;;) {
                int wc = workerCountOf(c);
                if (wc >= CAPACITY ||
                    wc >= (core ? corePoolSize : maximumPoolSize))
                    return false;
                if (compareAndIncrementWorkerCount(c))
                    break retry;
                c = ctl.get();  // Re-read ctl
                if (runStateOf(c) != rs)
                    continue retry;
                // else CAS failed due to workerCount change; retry inner loop
            }
        }
				
        boolean workerStarted = false;
        boolean workerAdded = false;
        Worker w = null;
        try {
		//新建Worker实例并添加到workers集合中
            w = new Worker(firstTask);
            final Thread t = w.thread;
            if (t != null) {
                final ReentrantLock mainLock = this.mainLock;
                mainLock.lock();
                try {
                    // Recheck while holding lock.
                    // Back out on ThreadFactory failure or if
                    // shut down before lock acquired.
                    int rs = runStateOf(ctl.get());

                    if (rs < SHUTDOWN ||
                        (rs == SHUTDOWN && firstTask == null)) {
                        if (t.isAlive()) // precheck that t is startable
                            throw new IllegalThreadStateException();
                        workers.add(w);
                        int s = workers.size();
                        if (s > largestPoolSize)
                            largestPoolSize = s;
                        workerAdded = true;
                    }
                } finally {
                    mainLock.unlock();
                }
	    //如果添加到workers集合成功，则启动线程				
                if (workerAdded) {
                    t.start();
                    workerStarted = true;
                }
            }
        } finally {
            if (! workerStarted)
                addWorkerFailed(w);
        }
        return workerStarted;
    }
```
接下来看Worker类，这是个ThreadPoolExecutor的内部类
```java
    private final class Worker
        extends AbstractQueuedSynchronizer
        implements Runnable
    {
        /**
         * This class will never be serialized, but we provide a
         * serialVersionUID to suppress a javac warning.
         */
        private static final long serialVersionUID = 6138294804551838833L;

        /** Thread this worker is running in.  Null if factory fails. */
        final Thread thread;
        /** Initial task to run.  Possibly null. */
        Runnable firstTask;
        /** Per-thread task counter */
        volatile long completedTasks;

        /**
         * Creates with given first task and thread from ThreadFactory.
         * @param firstTask the first task (null if none)
         */
        Worker(Runnable firstTask) {
            setState(-1); // inhibit interrupts until runWorker
            this.firstTask = firstTask;
		//将this作为参数新建线程，则调用this.thread.start()方法时
		//会执行Worker的run方法
            this.thread = getThreadFactory().newThread(this);
        }

        /** Delegates main run loop to outer runWorker  */
	    /** 当启动worker封装的线程时，调用此方法，代理到runWorker方法 */
        public void run() {
            runWorker(this);
        }

		/*********省略部分代码*********/
    }
```
接下来看runWorker方法，其执行逻辑是首先执行创建Worker实例时传入的任务firstTask，也就是execute方法的入参command。
当firstTask为null时，调用getTask()方法从队列获取任务执行。其实线程池的意义就在于此：
新建一个线程(即Worker实例)，可以多次执行传入的任务(即提交的Runnable)，也就是起到了重复利用线程的目的。
```java
    final void runWorker(Worker w) {
        Thread wt = Thread.currentThread();
        Runnable task = w.firstTask;
        w.firstTask = null;
        w.unlock(); // allow interrupts
        boolean completedAbruptly = true;
        try {
	/**循环执行，首先执行firstTask任务，也就是在execute方法传入的command；
	 *如果firstTask为null，则调用getTask()去队列中获取任务执行 
	 */
            while (task != null || (task = getTask()) != null) {
                w.lock();
                // If pool is stopping, ensure thread is interrupted;
                // if not, ensure thread is not interrupted.  This
                // requires a recheck in second case to deal with
                // shutdownNow race while clearing interrupt
                if ((runStateAtLeast(ctl.get(), STOP) ||
                     (Thread.interrupted() &&
                      runStateAtLeast(ctl.get(), STOP))) &&
                    !wt.isInterrupted())
                    wt.interrupt();
                try {
                    beforeExecute(wt, task);
                    Throwable thrown = null;
                    try {
                        task.run();
                    } catch (RuntimeException x) {
                        thrown = x; throw x;
                    } catch (Error x) {
                        thrown = x; throw x;
                    } catch (Throwable x) {
                        thrown = x; throw new Error(x);
                    } finally {
                        afterExecute(task, thrown);
                    }
                } finally {
                    task = null;
                    w.completedTasks++;
                    w.unlock();
                }
            }
            completedAbruptly = false;
        } finally {
            processWorkerExit(w, completedAbruptly);
        }
    }
```
源码分析到此结束。

ps：
* 创建线程池时，考虑直接调用ThreadPoolExecutor构造函数来设置，做到灵活可控。
* 设置线程池线程数量时，需要考虑到每秒需要执行的任务数量，和每个任务执行的时间以及队列长度
当然，要根据实际cpu负载情况来定
一般来说对cpu密集型任务，设置N<sub>cpu</sub>+1 ,对于io密集型任务，设置2N<sub>cpu</sub>
















