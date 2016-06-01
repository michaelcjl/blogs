---
title: python线程池
date: 
tags: python,线程池
categories: python

---
#  python线程池

##  1.为什么需要线程池 
　目前的大多数网络服务器，包括Web服务器、Email服务器以及数据库服务器等都具有一个共同点，就是单位时间内必须处理数目巨大的连接请求，但处理时间却相对较短。
　　传统多线程方案中我们采用的服务器模型则是一旦接受到请求之后，即创建一个新的线程，由该线程执行任务。任务执行完毕后，线程退出，这就是是“即时创建， 即时销毁”的策略。尽管与创建进程相比，创建线程的时间已经大大的缩短，但是如果提交给线程的任务是执行时间较短，而且执行次数极其频繁，那么服务器将处于不停的创建线程，销毁线程的状态。
　　我们将传统方案中的线程执行过程分为三个过程：T1、T2、T3：
　　T1：线程创建时间
　　T2：线程执行时间，包括线程的同步等时间
　　T3：线程销毁时间

　　那么我们可以看出，线程本身的开销所占的比例为(T1+T3) / (T1+T2+T3)。如果线程执行的时间很短的话，这比开销可能占到20%-50%左右。如果任务执行时间很频繁的话，这笔开销将是不可忽略的。
　　除此之外，线程池能够减少创建的线程个数。通常线程池所允许的并发线程是有上界的，如果同时需要并发的线程数超过上界，那么一部分线程将会等待。而传统方案中，如果同时请求数目为2000，那么最坏情况下，系统可能需要产生2000个线程。尽管这不是一个很大的数目，但是也有部分机器可能达不到这种要求。
　　**因此线程池的出现正是着眼于减少线程池本身带来的开销。**线程池采用预创建的技术，在应用程序启动之后，将立即创建一定数量的线程(N1)，放入空闲队列 中。这些线程都是处于阻塞（Suspended）状态，不消耗CPU，但占用较小的内存空间。当任务到来后，缓冲池选择一个空闲线程，把任务传入此线程中运行。当N1个线程都在处理任务后，缓冲池自动创建一定数量的新线程，用于处理更多的任务。在任务执行完毕后线程也不退出，而是继续保持在池中等待下一次的任务。当系统比较空闲时，大部分线程都一直处于暂停状态，线程池自动销毁一部分线程，回收系统资源。
　　基于这种预创建技术，线程池将线程创建和销毁本身所带来的开销分摊到了各个具体的任务上，执行次数越多，每个任务所分担到的线程本身开销则越小，不过我们另外可能需要考虑进去线程之间同步所带来的开销。
## 2.构建线程池框架 
一般线程池都必须具备下面几个组成部分：

 -  线程池管理器:用于创建并管理线程池
 -  工作线程: 线程池中实际执行的线程
 -  任务接口: 尽管线程池大多数情况下是用来支持网络服务器，但是我们将线程执行的任务抽象出来，形成任务接口，从而是的线程池与具体的任务无关。
 -  任务队列:线程池的概念具体到实现则可能是队列，链表之类的数据结构，其中保存执行线程。
　　**我们把任务放进队列中去，然后开N个线程，每个线程都去队列中取一个任务，执行完了之后告诉系统说我执行完了，然后接着去队列中取下一个任务，直至队列中所有任务取空，退出线程。**
　　
## 3.线程池的python实现
```
# !/usr/bin/env python
# -*- coding:utf-8 -*-
# ref_blog:http://www.open-open.com/home/space-5679-do-blog-id-3247.html

import Queue
import threading
import time

class WorkManager(object):
    def __init__(self, work_num=1000,thread_num=2):
        self.work_queue = Queue.Queue()
        self.threads = []
        self.__init_work_queue(work_num)
        self.__init_thread_pool(thread_num)

    """
        初始化线程
    """
    def __init_thread_pool(self,thread_num):
        for i in range(thread_num):
            self.threads.append(Work(self.work_queue))

    """
        初始化工作队列
    """
    def __init_work_queue(self, jobs_num):
        for i in range(jobs_num):
            self.add_job(do_job, i)

    """
        添加一项工作入队
    """
    def add_job(self, func, *args):
        self.work_queue.put((func, list(args)))#任务入队，Queue内部实现了同步机制
    """
        检查剩余队列任务
    """
    def check_queue(self):
        return self.work_queue.qsize()

    """
        等待所有线程运行完毕
    """   
    def wait_allcomplete(self):
        for item in self.threads:
            if item.isAlive():item.join()

class Work(threading.Thread):
    def __init__(self, work_queue):
        threading.Thread.__init__(self)
        self.work_queue = work_queue
        self.start()

    def run(self):
        #死循环，从而让创建的线程在一定条件下关闭退出
        while True:
            try:
                do, args = self.work_queue.get(block=False)#任务异步出队，Queue内部实现了同步机制
                do(args)
                self.work_queue.task_done()#通知系统任务完成
            except Exception,e:
                print str(e)
                break

#具体要做的任务
def do_job(args):
    print args
    time.sleep(0.1)#模拟处理时间
    print threading.current_thread(), list(args)

if __name__ == '__main__':
    start = time.time()
    work_manager =  WorkManager(10, 2)#或者work_manager =  WorkManager(10000, 20)
    work_manager.wait_allcomplete()
    end = time.time()
    print "cost all time: %s" % (end-start)
```
Work类是一个Python线程池，不断地从workQueue队列中获取需要执行的任务，执行之，并将结果写入到resultQueue中。这里的workQueue和resultQueue都是线程安全的，其内部对各个线程的操作做了互斥。当从workQueue中获取任务超时，则线程结束。
　　WorkerManager负责初始化Python线程池，提供将任务加入队列和获取结果的接口，并能等待所有任务完成。
　　在 Python 中使用线程时，这个模式是一种很常见的并且推荐使用的方式。具体工作步骤描述如下：

    1.创建一个 Queue.Queue() 的实例，然后使用数据对它进行填充。
    2.将经过填充数据的实例传递给线程类，后者是通过继承 threading.Thread 的方式创建的。
    3.生成守护线程池。
    4.每次从队列中取出一个项目，并使用该线程中的数据和 run 方法以执行相应的工作。
    5.在完成这项工作之后，使用 queue.task_done() 函数向任务已经完成的队列发送一个信号。
    6.对队列执行 join 操作，实际上意味着等到队列为空，再退出主程序。

　　在使用这个模式时需要注意一点：通过将守护线程设置为 true，将允许主线程或者程序仅在守护线程处于活动状态时才能够退出。这种方式创建了一种简单的方式以控制程序流程，因为在退出之前，您可以对队列执行 join 操作、或者等到队列为空。

    join()保持阻塞状态，直到处理了队列中的所有项目为止。在将一个项目添加到该队列时，未完成的任务的总数就会增加。当使用者线程调用 task_done() 以表示检索了该项目、并完成了所有的工作时，那么未完成的任务的总数就会减少。当未完成的任务的总数减少到零时，join() 就会结束阻塞状态。

