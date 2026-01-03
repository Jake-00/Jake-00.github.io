---
layout:       post
title:        "multiprocessing in Python"
author:       "Jake"
header-style: text
catalog:      true
tags:
    - Python
    - concurrent
---

# 前言
工作中遇到过处理 n 个超多行文件，单个文件需要逐行读取并正则匹配，单个文件解析下来要10分钟。采取单并发方式显然消耗过长时间，需要用到多进程去加速处理。

## 多进程方式
1. 通过 Manager 方式
2. 通过 Pool 方式

## 动手实操
### 1. 实现 square 函数，计算输入数字的二次方并记录到一个字典中。使用 Manager 类的共享字典来跨进程更新，并用进程池来管理进程
```python
from multiprocessing import Manager, Pool

def square(n, shared_dict):
    shared_dict[n] = n ** 2  # 直接赋值，不需要复制

if __name__ == '__main__':
    with Manager() as manager, Pool() as pool:
        shared_dict = manager.dict()
        pool.starmap(square, [(i, shared_dict) for i in range(1, 6)])
        print(dict(shared_dict))
```
> 小技巧：使用多进程加速的函数参数多于1个时，采用 Pool.starmap() ，否则直接使用 Pool.map()
### 2. 进一步，客户想要把平方和相等的数字存放成一个数组，设计到数组元素增加
#### 2.1 直觉上的写法，错误示范
```python
from multiprocessing import Manager, Pool

def square(n, shared_dict):
    shared_dict[n] = n ** 2  # 直接赋值，不需要复制

if __name__ == '__main__':
    with Manager() as manager, Pool() as pool:
        shared_dict = manager.dict()
        pool.starmap(square, [(i, shared_dict) for i in range(-2, 6)])
        print(dict(shared_dict))
"""
{4: [-2], 1: [-1], 0: [0], 9: [3], 25: [5], 16: [4]}
"""
```
元素 1 和 2 没有存在于分布式字典中，也不报错！
#### 2.2 进程共享 dict 时，多进程方法中不同进程对一个 key 的操作不可见，必须使用进程锁，再备份一份到进程空间后操作完，再覆盖共享 dict
```python
from multiprocessing import Manager, Pool

def square(n, shared_dict, lock_):
    res = n ** 2  # 直接赋值，不需要复制
    with lock_:
        if res in shared_dict:
            tmp_list = shared_dict[res]
            tmp_list.append(n)
            shared_dict[res] = tmp_list
        else:
            shared_dict[res] = [n]  # 使用共享列表

if __name__ == '__main__':
    with Manager() as manager, Pool() as pool:
        shared_dict = manager.dict()
        lock_ = manager.Lock()
        pool.starmap(square, [(i, shared_dict, lock_) for i in range(-2, 6)])
        print(dict(shared_dict))
```
#### 2.3 *[Better Practice]*不在多进程函数中保存状态，仅返回计算后结果，在主进程进行合并操作
```python
from multiprocessing import Pool
from collections import defaultdict

def square(n):
    return (n ** 2, n)  # 返回元组，避免共享状态

if __name__ == '__main__':
    with Pool() as pool:
        # 并行计算，结果收集
        results = pool.map(square, range(-2, 6))
        
        # 在主进程合并结果
        merged = defaultdict(list)
        for res, n in results:
            merged[res].append(n)
        
        print(dict(merged))
```