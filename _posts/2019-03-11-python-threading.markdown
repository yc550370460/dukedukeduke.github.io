---
layout: post
title:  "python 多线程threading模块"
date:   2019-03-11 19:17:02 +0800
comments: true
tags:
- python
- threading
- 多线程
---
## 参考
https://python-parallel-programmning-cookbook.readthedocs.io/zh_CN/latest/chapter2/03_how_to_define_a_thread.html

## 定义一个线程
### threading.Thread
```ruby
class threading.Thread(group=None,
                       target=None,
                       name=None,
                       args=(),
                       kwargs={})
```
- group: 一般设置为 None ，这是为以后的一些特性预留的
- target: 当线程启动的时候要执行的函数
- name: 线程的名字，默认会分配一个唯一名字 Thread-N
- args: 传递给 target 的参数，要试用tuple类型
- kwargs: 同上，试用字段类型dict
### samples
```ruby
import threading

def function(i):
    print ("function called by thread %i\n" % i)
    return

threads = []

for i in range(5):
    # 类似于multiprocessing的Process
    t = threading.Thread(target=function , args=(i, ))
    threads.append(t)
    t.start()
    t.join()
```
线程被创建之后并不会马上运行，需要手动调用 start() ， join() 让调用它的线程一直等待直到执行结束（即阻塞调用它的主线程， t 线程执行结束，主线程才会继续执行）。

output:
```ruby
>>>function called by thread 0
>>>function called by thread 1
>>>function called by thread 2
>>>function called by thread 3
>>>function called by thread 4
```
t.join() ，这意味着，t线程结束之前并不会看到后续的线程，换句话说，主线程会调用t线程，然后等待t线程完成再执行for循环开启下一个t线程，事实上，这段代码是顺序运行的，实际运行顺序永远是01234顺序出现。
要纠正这种顺序， 需要将join单独拿出来，如下：
```ruby
import threading

def function(i):
    print ("function called by thread %i\n" % i)
    return

threads = []

for i in range(5):
    t = threading.Thread(target=function , args=(i, ))
    threads.append(t)
    t.start()
    
for t in threads:
    t.join()
```
## 确定当前的线程
每一个 Thread 实例创建的时候都有一个带默认值的名字，并且可以修改。
```ruby
import threading
import time

def first_function():
    print(threading.currentThread().getName() + str(' is Starting '))
    time.sleep(2)
    print (threading.currentThread().getName() + str(' is Exiting '))
    return

def second_function():
    print(threading.currentThread().getName() + str(' is Starting '))
    time.sleep(2)
    print (threading.currentThread().getName() + str(' is Exiting '))
    return

def third_function():
    print(threading.currentThread().getName() + str(' is Starting '))
    time.sleep(2)
    print(threading.currentThread().getName() + str(' is Exiting '))
    return

if __name__ == "__main__":
    t1 = threading.Thread(name='first_function', target=first_function)
    t2 = threading.Thread(name='second_function', target=second_function)
    t3 = threading.Thread(name='third_function', target=third_function)
    t1.start()
    t2.start()
    t3.start()
    t1.join()
    t2.join()
    t3.join()
```
output:
```ruby
first_function is Starting 
second_function is Starting 
third_function is Starting 

first_function is Exiting 
second_function is Exiting 
third_function is Exiting 
```
## 用类实现线程
- 定义一个 Thread 类的子类
- 覆盖 __init__(self [,args]) 方法，可以添加额外的参数
- 最后，需要覆盖 run(self, [,args]) 方法来实现线程要做的事情
创建了新的 Thread 子类的时候，你可以实例化这个类，调用 start() 方法来启动它。线程启动之后将会执行 run() 方法。
```ruby
import threading
import time

exitFlag = 0

class myThread (threading.Thread):
    def __init__(self, threadID, name, counter):
        threading.Thread.__init__(self)
        self.threadID = threadID
        self.name = name
        self.counter = counter

    def run(self):
        print("Starting " + self.name)
        print_time(self.name, self.counter, 5)
        print("Exiting " + self.name)

def print_time(threadName, delay, counter):
    while counter:
        if exitFlag:
            # 译者注：原书中使用的thread，但是Python3中已经不能使用thread，以_thread取代，因此应该
            # import _thread
            # _thread.exit()
            import thread
            thread.exit() #或者直接return
        time.sleep(delay)
        print("%s: %s" % (threadName, time.ctime(time.time())))
        counter -= 1

thread1 = myThread(1, "Thread-1", 1)
thread2 = myThread(2, "Thread-2", 2)

thread1.start()
thread2.start()

thread1.join()
thread2.join()
print("Exiting Main Thread")
```
## 线程同步
为了简化问题，我们设有两个并发的线程（线程A和线程B)，需要资源1和资源2 .假设线程A需要资源1，线程B需要资源2 .在这种情况下，两个线程都使用各自的锁，目前为止没有冲突。现在假设，在双方释放锁之前，线程A 需要 资源2的锁，线程B 需要 资源1 的锁，没有资源线程不会继续执行。鉴于目前两个资源的锁都是被占用的，而且在对方的锁释放之前都处于等待且不释放锁的状态。这是死锁的典型情况。所以如上所说，使用锁来解决同步问题是一个可行却存在潜在问题的方案。
### Lock
只能在lock release之后才能acquire成功(一次acquire对应一次release， 严格按照release之后才能acquire的规则， 包括重复获取同一个lock)， 否则一致阻塞在acquire这里。
```ruby
# -*- coding: utf-8 -*-

import threading

shared_resource_with_lock = 0
shared_resource_with_no_lock = 0
COUNT = 100000
shared_resource_lock = threading.Lock()

# 有锁的情况
def increment_with_lock():
    global shared_resource_with_lock
    for i in range(COUNT):
        shared_resource_lock.acquire()
        shared_resource_with_lock += 1
        shared_resource_lock.release()

def decrement_with_lock():
    global shared_resource_with_lock
    for i in range(COUNT):
        shared_resource_lock.acquire()
        shared_resource_with_lock -= 1
        shared_resource_lock.release()

# 没有锁的情况
def increment_without_lock():
    global shared_resource_with_no_lock
    for i in range(COUNT):
        shared_resource_with_no_lock += 1

def decrement_without_lock():
    global shared_resource_with_no_lock
    for i in range(COUNT):
        shared_resource_with_no_lock -= 1

if __name__ == "__main__":
    t1 = threading.Thread(target=increment_with_lock)
    t2 = threading.Thread(target=decrement_with_lock)
    t3 = threading.Thread(target=increment_without_lock)
    t4 = threading.Thread(target=decrement_without_lock)
    t1.start()
    t2.start()
    t3.start()
    t4.start()
    t1.join()
    t2.join()
    t3.join()
    t4.join()
    print ("the value of shared variable with lock management is %s" % shared_resource_with_lock)
    print ("the value of shared variable with race condition is %s" % shared_resource_with_no_lock)
```
lock.acquire()和lock.release()可用with lock上下文来代替， 更为简洁。
### RLock
对于同一个lock, 重复获取不会阻塞（acquire之后并未release， 当前线程再去获取同一lock， 不会导致阻塞）。acquire多少次就必须release多少次，只有最后一次release才能改变RLock的状态为unlocked
如下会导致阻塞：
```ruby
import threading
import time


class MyThread(threading.Thread):
    def __init__(self, threadID):
        threading.Thread.__init__(self)
        self.id = threadID

    def run(self):
        global num
        print("start thread:%s" %self.id)
        time.sleep(1)
        if mutex.acquire(1):
            print("get lock success:%s" % self.id)
            num = num+1
            msg = self.name+' set num to '+str(num)
            print(msg)
            mutex.acquire()
            mutex.release()
            mutex.release()
            print("release lock success:%s" % self.id)
num = 0
mutex = threading.Lock()

if __name__ == '__main__':
    for i in range(5):
        t = MyThread(i)
        t.start()
        t.join()
```
而下面不会阻塞：
```ruby
import threading
import time


class MyThread(threading.Thread):
    def __init__(self, threadID):
        threading.Thread.__init__(self)
        self.id = threadID

    def run(self):
        global num
        print("start thread:%s" %self.id)
        time.sleep(1)
        if mutex.acquire(1):
            print("get lock success:%s" % self.id)
            num = num+1
            msg = self.name+' set num to '+str(num)
            print(msg)
            mutex.acquire()
            mutex.release()
            mutex.release()
            print("release lock success:%s" % self.id)
num = 0
mutex = threading.RLock()

if __name__ == '__main__':
    for i in range(5):
        t = MyThread(i)
        t.start()
        t.join()
```
同样，lock.acquire()和lock.release()可用with lock上下文来代替， 更为简洁。
### 信号量
每当调用acquire()时，内置计数器-1,直到为0的时候阻塞
每当调用release()时，内置计数器+1，并让某个线程的acquire()从阻塞变为不阻塞。

UrlProducer线程，爬取url，多个htmlSpider线程，爬取url对应的网页。如果直接开20个htmlSpider线程，20个线程是同时执行的，现在要限制同时执行能执行三个，就可以使用信号量来控制
```ruby
import threading
import time
class htmlSpider(threading.Thread):
    def __init__(self, url, sem):
        super().__init__()
        self.url = url
        self.sem = sem

    def run(self):
        time.sleep(2)
        print("got html text success")
        self.sem.release() # 内部维护的计数器加1，并通知内部维护的conditon通知acquire

class UrlProducer(threading.Thread):
    def __init__(self, sem):
        super().__init__()
        self.sem = sem

    def run(self):
        for i in range(20):
            self.sem.acquire() # 内部维护的计数器减1，到0就会阻塞
            html_thread = htmlSpider("http://baidu.com/{}".format(i), self.sem)
            html_thread.start()

if __name__ == "__main__":
    sem = threading.Semaphore(3) #设置同时最多3个
    url_producer = UrlProducer(sem)
    url_producer.start()
```

### Event
event.wait(timeout=None)：调用该方法的线程会被阻塞，如果设置了timeout参数，超时后，线程会停止阻塞继续执行；

event.set()：将event的标志设置为True，调用wait方法的所有线程将被唤醒；

event.clear()：将event的标志设置为False，调用wait方法的所有线程将被阻塞；

event.isSet()：判断event的标志是否为True。
```ruby
# encoding=utf8

import threading
from time import sleep


def test(n, event):
    while not event.isSet():
        print 'Thread %s is ready' % n
        sleep(1)
    event.wait()
    while event.isSet():
        print 'Thread %s is running' % n
        sleep(1)


def main():
    event = threading.Event()
    for i in xrange(0, 2):
        th = threading.Thread(target=test, args=(i, event))
        th.start()
    sleep(3)
    print '----- event is set -----'
    event.set()
    sleep(3)
    print '----- event is clear -----'
    event.clear()


if __name__ == '__main__':
    main()
```
### queue
```ruby
class Queue:
    """Create a queue object with a given maximum size.

    If maxsize is <= 0, the queue size is infinite.
    """
    def __init__(self, maxsize=0):
    ...
```
Queue常用的方法有以下四个：
- Queue.put(item [,block[, timeout]]): 往queue中放一个item, Queue是同步的，在插入数据之前内部有一个内置的锁机制, 如果 block 为 True ， timeout 为 None（这也是默认的选项，本例中使用默认选项），那么可能会阻塞掉，直到出现可用的位置。如果timeout是正整数，那么阻塞直到这个时间，就会抛出一个异常；如果block为False，如果队列有闲置那么会立即插入，否则就立即抛出异常（ timeout将会被忽略）。本例中，put()检查队列是否已满，然后调用 wait() 开始等待。
- Queue.get([block[,timeout]]):从queue删除一个item，并返回删除的这个item,queue内部也会经过锁的处理。如果队列为空，消费者阻塞。
- task_done(): 每次item被处理的时候需要调用这个方法
- join(): 所有item都被处理之前一直阻塞

```ruby
from threading import Thread, Event
from Queue import Queue
import time
import random


class Producer(Thread):
    def __init__(self, queue):
        Thread.__init__(self)
        self.queue = queue

    def run(self):
        for i in range(10):
            item = random.randint(0, 256)
            self.queue.put(item)
            print('Producer notify: item %d appended to queue by %s' % (item, self.name))
            time.sleep(1)


class Consumer(Thread):
    def __init__(self, queue):
        Thread.__init__(self)
        self.queue = queue

    def run(self):
        while True:
            item = self.queue.get()
            print('Consumer notify : %d popped from queue by %s' % (item, self.name))
            self.queue.task_done()

if __name__ == '__main__':
    queue = Queue()
    t1 = Producer(queue)
    t2 = Consumer(queue)
    t3 = Consumer(queue)
    t4 = Consumer(queue)
    t1.start()
    t2.start()
    t3.start()
    t4.start()
    t1.join()
    t2.join()
    t3.join()
    t4.join()
```
消费者从队列中取出整数然后用 task_done() 方法将其标为任务已处理。
