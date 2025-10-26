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

## 注意点
进程共享 dict 时，多进程方法中无法只对一个 key 进行操作，必须是先备份一份到进程空间后操作完，再覆盖共享 dict
```python
from multiprocessing import Manager

def process_func(shared_dict):
    ...
    cp_dict = shared_dict
    cp_dict['new_k'] = 'new_v'
    shared_dict = cp_dict
    ...

if __name__ == '__main__':
    manager = Manager()
    with manager:
        shared_dict = manager.dict()
```