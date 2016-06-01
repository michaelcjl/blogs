---
title: python多线程 VS 多进程
date: 
tags: python,多线程,多线程
categories: python

---
#  python多线程 VS 多进程

##  1.基本概念

### 1.1 线程是什么鬼?
***线程（有时被称为轻量级进程）跟进程有些相似，不同的是，所有的线程运行在同一个进程中，共享相同的运行环境。***它们可以想像成是在主进程或“主线程”中并行运行的“迷你进程”。***线程有开始，顺序执行和结束三部分。它有一个自己的指令指针，记录自己运行到什么地方。线程的运行可能被抢占（中断），或暂时的被挂起（也叫睡眠），让其它的线程运行，这叫做让步。一个进程中的各个线程之间共享同一片数据空间，所以线程之间可以比进程之间更方便地共享数据以及相互通讯。线程一般都是并发执行的，正是由于这种并行和数据共享的机制使得多个任务的合作变为可能。***实际上，在单CPU的系统中，真正的并发是不可能的，每个线程会被安排成每次只运行一小会，然后就把CPU让出来，让其它的线程去运行。在进程的整个运行过程中，每个线程都只做自己的事，在需要的时候跟其它的线程共享运行的结果。

### 1.2 进程又是什么鬼？
计算机程序只不过是磁盘中可执行的，二进制（或其它类型）的数据。它们只有在被读取到内存中，被操作系统调用的时候才开始它们的生命期。***进程（有时被称为重量级进程）是程序的一次执行。每个进程都有自己的地址空间，内存，数据栈以及其它记录其运行轨迹的辅助数据。***操作系统管理在其上运行的所有进程，并为这些进程公平地分配时间。**进程也可以通过 fork 和 spawn 操作来完成其它的任务。不过各个进程有自己的内存空间，数据栈等，所以只能使用进程间通讯（IPC），而不能直接共享信息。**

##  2.Python、线程和全局解释器锁
Python 代码的执行由 Python 虚拟机(也叫解释器主循环)来控制。Python 在设计之初就考虑到要在主循环中，同时只有一个线程在执行，就像单 CPU 的系统中运行多个进程那样，内存中可以存放多个程序，但任意时刻，只有一个程序在 CPU 中运行。同样地，***虽然 Python 解释器中可以“运行”多个线程，但在任意时刻，只有一个线程在解释器中运行。***对 Python 虚拟机的访问由全局解释器锁（GIL）来控制，正是这个锁能保证同一时刻只有一个线程在运行。在多线程环境中，Python 虚拟机按以下方式执行：
1. 设置 GIL
2. 切换到一个线程去运行
3. 运行：
     *指定数量的字节码指令，或者*
     *线程主动让出控制（可以调用 time.sleep(0)）*
4. 把线程设置为睡眠状态
5. 解锁 GIL
6. 再次重复以上所有步骤

##  3.实例
### 3.1 多线程实例
```
#!/usr/local/bin/python2.7
# encoding: utf-8
from time import ctime, sleep
import threading

class MyThread(threading.Thread):
    
    def __init__(self, func, args, name=''): 
        threading.Thread.__init__(self)
        self.name = name
        self.func = func
        self.args = args
    
    def run(self):
        print 'starting', self.name, 'at:', ctime()
        self.res = apply(self.func, self.args)
        print self.name, 'finished at:', ctime()


def fib(x):
    sleep(0.005)
    if x<2:return 1
    return (fib(x-2) + fib(x-1))

def fac(x):
    sleep(0.1)
    if x<2:return 1
    return (x*fac(x-1))

def sum_1(x):
    sleep(0.1)
    if x<2:return 1
    return (x+ sum_1(x-1))


funcs = [fib, fac, sum_1]

n = 12

def main():
    nfuncs = range(len(funcs))
    
    print '*** SINGLE THREAD'
    
    for i in nfuncs:
        print 'starting', funcs[i].__name__, 'at:', ctime()
        print 'result:', funcs[i](n)
        print funcs[i].__name__, 'finished at:', ctime()
        
    print '\n*** MULTIPLE THREADS'

    threads = []
    for i in nfuncs:
        t = MyThread(funcs[i], (n,), funcs[i].__name__)
        threads.append(t)
        
    for i in nfuncs:
        threads[i].start()
        
    for i in nfuncs:
        threads[i].join()
        print threads[i]
               
    print 'all Done'
        
if __name__ == "__main__":
    main()
```
### 3.2 多进程实例
这里使用了进程池
```
from multiprocessing import Pool
import os, time, random

def long_time_task(name):
    print 'Run task %s (%s)...' % (name, os.getpid())
    start = time.time()
    time.sleep(random.random() * 3)
    end = time.time()
    print 'Task %s runs %0.2f seconds.' % (name, (end - start))

if __name__=='__main__':
    print 'Parent process %s.' % os.getpid()
    p = Pool()
    for i in range(5):
        p.apply_async(long_time_task, args=(i,))
    print 'Waiting for all subprocesses done...'
    p.close()
    p.join()
    print 'All subprocesses done.'
```



