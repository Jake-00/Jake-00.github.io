---
layout:       post
title:        "Easily register service class using decorator"
author:       "Jake"
header-style: text
catalog:      true
tags:
    - Python
    - decorator
---
## 前言
在日常工作中开发一个小型项目，想要注册不同的模型推理服务类到全局字典，让服务调用无感。实现方式有2种，一种是工厂模式，另外一种就是装饰器+`__init__.py`方案。

## 代码结构
```
demo
├── main.py
├── pipeline
│   └── service_register.py
└── utils
    ├── __init__.py
    └── services.py
```

## 装饰器设计与使用
1. 设计
我们把服务抽象成许多类 Service1、Service2 等等，设计一个装饰器，实现自动注册在字典中。我把装饰器函数放在 pipeline/service_register.py
```python
import logging

logger = logging.getLogger()
SERVICES = {}

def register(service_key):
    def inner(_cls):
        SERVICES[service_key] = _cls
        logger.info(f"Successfully registered {service_key}")
        return _cls
    return inner
```

2. 在 Service 类定义时注解，我这里偷懒把 Service1、Service2 都放进了 services.py 中
```python
from pipeline.service_register import register

@register('service1')
class Service1:
    pass

@register('service2')
class Service2:
    pass
```

## 精妙的 `__init__.py` 文件
当 import 某个模块时，其目录下的 `__init__.py` 会被自动加载，利用这个特性进行类加载，实现自动注册
```python
from .services import Service1
from .services import Service2
```
完成以上步骤后，在其他脚本 `import utils` 时就能实现预期功能，这里提供简单的 main.py
```python
from pipeline.service_register import SERVICES
import utils

print(SERVICES)  # {'service1': <class 'utils.services.Service1'>, 'service2': <class 'utils.services.Service2'>}
```

## 缺点
1. ​隐式注册机制降低可维护性​
   * 循环导入风险​
   * logging 日志不打印
   * 新类易遗漏​：新增服务类时，若忘记在__init__.py中添加导入语句，服务将无法注册（需手动维护导入列表）
2. 测试与调试困难​
   * ​注册时机关联​：服务类的注册发生在模块导入时，而非装饰器调用时，可能导致调试时难以追踪注册时机，表现为 register 装饰器不打印日志
   * ​全局状态污染​：SERVICES作为全局字典，可能被意外修改（需配合线程锁或不可变数据结构）

## 更好的方案（待定）
1. metaclass
2. 工厂类
