---
layout:       post
title:        "How to convert numpy array to pyarrow array"
author:       "Jake"
header-style: text
catalog:      true
tags:
    - Python
    - Numpy
    - Pyarray
---

# 需求
假设有一个 npy 文件 存储 3 x 4 的 numpy array，我想要转成 pyarrow array ，要如何实现呢？

## 0.前置工作
```python
"""
前置工作：
import os, random
import numpy as np


random.seed(seed_num)
# generate pseudo random numbers    
arr = [float(random.random() * 10) for _ in range(6)]
data = np.array(arr).reshape((2, 3))
npy_path = './data.npy'
np.save(npy_path, data)
"""
```

试下 load 进来直接转型
```python
import numpy as np
import pyarrow as pa

data = np.load(npy_path)
data = pa.array(data)
```
会报错！

## 方法一：np.fromfile()
```python
import numpy as np
import pyarrow as pa


data = np.fromfile(npy_path)  # 按二进制读取
data_pyarrow = pa.array(data)
```
> 不会报错，进一步验证下数据准性
```python
data = np.fromfile(npy_path)  # 按二进制读取
print(data.shape)
"""
(22,)
"""
```
预料之外，居然是一维数组，元素个数 22 个也不对。查看 fromfile 函数签名，发现它就是读取二进制文件或存储1D数组的文件，不能用于读取存储 nd 数组文件。


## 方法二：np.load() 之后 pickle dump 成二进制文件
```python
import numpy as np
import pyarrow as pa
import pickle

data = np.load(npy_path)
data = pickle.dump(data)  # 序列化
data = pa.array(data)
```
不会报错，还能保留精度和类型！