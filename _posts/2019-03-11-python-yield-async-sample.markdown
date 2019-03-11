---
layout: post
title:  "Python yield 异步编程"
date:   2019-03-11 09:27:02 +0800
comments: true
tags:
- python
- yield
---
python 代码遇到yield就会暂停于此， 可在yield之前部署请求代码，yield等待期间用于等待耗IO（可进一步优化成selector事件响应）。
如下， 先让所有函数（或者叫做执行对象），暂停在yield处， 然后记录下原生成器对象，最后统一调度，让其继续执行，如下实例：
```
global_list = [1,2,3]


def calc():
    result = 0
    for i in range(10000000):
        result += 1
    return result


class Execute:
    def __init__(self, id):
        self.id = id
        self.value = None

    def method(self):
        try:
            print("start id:%s" %self.id)
            calc()
            index = yield self.id
            print("get index:%s" %index)
            calc()
            print("end id:%s" % self.id)
        except StopIteration:
            pass

class Manager:
    def __init__(self, e):
        self.f = e
        exe = Execute(0)
        self.call(exe)

    def call(self, future):
        f = self.f.method()
        exe_list.append(f)
        f.send(future.value)


if __name__ == "__main__":
    exe_list = list()
    for item in global_list:
        e = Execute(item)
        # exe_list.append({"index": item, "execute":e})
        m = Manager(e)

    for index, item in enumerate(exe_list):
        try:
            item.send(index)
        except StopIteration:
            pass
```
output:
```
start id:1
start id:2
start id:3
get index:0
end id:1
get index:1
end id:2
get index:2
end id:3
```
