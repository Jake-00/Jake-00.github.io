---
layout:       post
title:        "asyncio 学习笔记"
author:       "Jake"
header-style: text
catalog:      true
tags:
    - Python
    - asyncio
---

# asyncio 学习笔记
> 高天老师B站视频：https://www.bilibili.com/video/BV1oa411b7c9/?spm_id_from=333.788.video.desc.click&vd_source=721e18013412c1b59a2fd6a464d919fa

## 什么是 coroutine 和 task？
协程中，coroutine有2个概念：`coroutine function` 和 `coroutine object`
```py
# 由 async 修饰的函数就是 coroutine function
async def func():
    ...

# 直接调用 coroutine function 是不会执行的，他会返回一个 coroutine object
coro_obj = func()
```

## 如何调用呢？
event loop 是不会执行 coroutine ，它最小执行单位是 Task，就需要3个手段把他们转变成 Task
1. await 协程对象
```python
await coro_obj
```
2. create_task
当一个 coroutine function 内部有多个需要并发执行时，连续 await 多个 coroutine function 是同步的逻辑
```python
coro_obj1 = func()
coro_obj2 = func()

await coro_obj1
await coro_obj2
```
Q：如何实现在执行 coro_obj1 时遇到阻塞点直接执行 coro_obj2 呢？
A：通过 `asyncio.create_task()` 创建 Task ，注册到 Event Loop 后，让 await 去调用！
```python
import time
import asyncio


async def main():
    task1 = asyncio.create_task(
        say_after('hello', 1)
    )
    task2 = asyncio.create_task(
        say_after('world', 2)
    )
    print(f"start at {time.strftime('%X')}")
    await task1
    await task2
    
    print(f"end at {time.strftime('%X')}")


if __name__ == "__main__":
    asyncio.run(main())
```

3. 如果超过 2 个函数需要传入 create_task ，有快速方式吗？**`asyncio.gather()`**
它会按顺序返回结果并汇总成 list 对象
```python
import time
import asyncio

async def say_after(word: str, delay: int):
    await asyncio.sleep(delay)
    return f'{word} - {delay}'


async def main():
    print(f"start at {time.strftime('%X')}")
    ret = await asyncio.gather(
        say_after('hello', 1),
        say_after('world', 2)
    )
    
    print(ret)
    
    print(f"end at {time.strftime('%X')}")


if __name__ == "__main__":
    asyncio.run(main())
```