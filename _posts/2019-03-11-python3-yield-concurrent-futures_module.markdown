---
layout: post
title:  "python3_concurrent.futures 模块"
date:   2019-03-11 10:27:02 +0800
comments: true
tags:
- python
- python3
- concurrent
---
https://python-parallel-programmning-cookbook.readthedocs.io/zh_CN/latest/chapter4/02_Using_the_concurrent.futures_Python_modules.html#

## 介绍
Python3.2带来了 concurrent.futures 模块，这个模块具有线程池和进程池、管理并行编程任务、处理非确定性的执行流程、进程/线程同步等功能。

此模块由以下部分组成：

- concurrent.futures.Executor: 这是一个虚拟基类，提供了异步执行的方法。
- submit(function, argument): 调度函数（可调用的对象）的执行，将 argument 作为参数传入。
- map(function, argument): 将 argument 作为参数执行函数，以 异步 的方式。
- shutdown(Wait=True): 发出让执行者释放所有资源的信号。
- concurrent.futures.Future: 其中包括函数的异步执行。Future对象是submit任务（即带有参数的functions）到executor的实例。

Executor是抽象类，可以通过子类访问，即线程或进程的 ExecutorPools 。因为，线程或进程的实例是依赖于资源的任务，所以最好以“池”的形式将他们组织在一起，作为可以重用的launcher或executor。

## 使用线程池和进程池
线程池或进程池是用于在程序中优化和简化线程/进程的使用。通过池，你可以提交任务给executor。池由两部分组成，一部分是内部的队列，存放着待执行的任务；另一部分是一系列的进程或线程，用于执行这些任务。池的概念主要目的是为了重用：让线程或进程在生命周期内可以多次使用。它减少了创建创建线程和进程的开销，提高了程序性能。重用不是必须的规则，但它是程序员在应用中使用池的主要原因。

current.Futures 模块提供了两种 Executor 的子类，各自独立操作一个线程池和一个进程池。这两个子类分别是：

- concurrent.futures.ThreadPoolExecutor(max_workers)
- concurrent.futures.ProcessPoolExecutor(max_workers)

max_workers 参数表示最多有多少个worker并行执行任务。

## 举例

下面的示例代码展示了线程池和进程池的功能。这里的任务是，给一个list number_list ，包含1到10。对list中的每一个数字，乘以1+2+3…+10000000的和（这个任务只是为了消耗时间）。

下面的代码分别测试了：
- 顺序执行
- 通过有5个worker的线程池执行
- 通过有5个worker的进程池执行
```
import concurrent.futures
import time
number_list = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]


def evaluate_item(x):
        # 计算总和，这里只是为了消耗时间
        result_item = count(x)
        # 打印输入和输出结果
        return result_item

def  count(number) :
        i = 0
        for i in range(0, 10000000):
                i=i+1
        return i * number

if __name__ == "__main__":
        # 顺序执行
        start_time = time.time()
        for item in number_list:
                print(evaluate_item(item))
        print("Sequential execution in " + str(time.time() - start_time), "seconds")
        
        # 线程池执行
        start_time_1 = time.time()
        with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
                futures = [executor.submit(evaluate_item, item) for item in number_list]
                for future in concurrent.futures.as_completed(futures):
                        print(future.result())
        print ("Thread pool execution in " + str(time.time() - start_time_1), "seconds")
        
        # 进程池
        start_time_2 = time.time()
        with concurrent.futures.ProcessPoolExecutor(max_workers=5) as executor:
                futures = [executor.submit(evaluate_item, item) for item in number_list]
                for future in concurrent.futures.as_completed(futures):
                        print(future.result())
        print ("Process pool execution in " + str(time.time() - start_time_2), "seconds")
```

> ThreadPoolExecutor 使用线程池中的一个线程执行给定的任务。池中一共有5个线程，每一个线程从池中取得一个任务然后执行它。当任务执行完成，再从池中拿到另一个任务

> ProcessPoolExecutor 是一个executor，使用一个线程池来并行执行任务。然而，和 ThreadPoolExecutor 不同的是， ProcessPoolExecutor 使用了多核处理的模块，让我们可以不受GIL的限制，大大缩短执行时间。