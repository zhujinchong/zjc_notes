# 并发/并行/线程/协程

## 并发和并行？

并发：表面上同时发生，实际上串行（交替执行）。

并行：物理、实际上的同时发生。

IO密集型任务，即，读写磁盘很多，如，读写文件，一般并发解决。理解：CPU空闲了，CPU可以多维护一些任务，所以可以设置线程池数多一些。

CPU密集型任务，即，复杂计算很多，如，深度学习，一般并行解决。理解：CPU很忙，IO空着也用不起来，所以设置线程池数=CPU核心数。

## 异步IO解决什么问题？

在一个线程中，CPU执行代码的速度极快，然而，一旦遇到IO操作，如读写文件、发送网络数据时，就需要等待IO操作完成，才能继续进行下一步操作。这种情况称为同步IO。

因为一个IO操作就阻塞了当前线程，导致其他代码无法执行，所以我们必须使用多线程或者多进程来并发执行代码，为多个用户服务。每个用户都会分配一个线程，如果遇到IO导致线程被挂起，其他用户的线程不受影响。

多线程和多进程的模型虽然解决了并发问题，但是系统不能无上限地增加线程。由于系统切换线程的开销也很大，所以，一旦线程数量过多，CPU的时间就花在线程切换上了，真正运行代码的时间就少了，结果导致性能严重下降。

**所以：多线程和多进程只是解决IO阻塞的一种方法（并行方式），另一种解决IO问题的方法是异步IO，也叫协程（并发方式）。**

当代码需要执行一个耗时的IO操作时，它只发出IO指令，并不等待IO结果，然后就去执行其他代码了。一段时间后，当IO返回结果时，再通知CPU进行处理。

python里的异步也叫协程：

* **协程** ：协程在单线程中通过控制流切换实现并发，没有线程切换开销，但资源占用相对较少，适合I/O等待任务。
* 所有使用async和awit关键字的方法就是协程/异步方法。

## 协程为什么比线程还快？

IO密集任务，多线程之间的调度，会导致调度时间占比更高。

协程怎么解决：

1、协程的特点在于是一个线程执行

2、由程序自身控制，相当于写成一个程序。


# Python中如何实现协程

## asyncio对异步IO的支持

`asyncio`是Python 3.4版本引入的标准库，直接内置了对异步IO的支持。

`asyncio`的编程模型就是一个消息循环。我们从 `asyncio`模块中直接获取一个 `EventLoop`的引用，然后把需要执行的协程扔到 `EventLoop`中执行，就实现了异步IO。

```
import threading
import asyncio

@asyncio.coroutine
def hello():
    print('Hello world! (%s)' % threading.currentThread())
    yield from asyncio.sleep(1)
    print('Hello again! (%s)' % threading.currentThread())

loop = asyncio.get_event_loop()
tasks = [hello(), hello()]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()
```

同时，执行上面代码发现，他们用的是同一个线程，在python里这叫**协程**。

## asyncio中的关键字async

为了简化并更好地标识异步IO，从Python 3.5开始引入了新的语法 `async`和 `await`，可以让coroutine的代码更简洁易读。

```
async def hello():
    print("Hello world!")
    r = await asyncio.sleep(1)
    print("Hello again!")
```



# faskapi接口什么时候用async？

官方说，**你是否使用async，FastAPI都将异步工作，以达到Fast的运行速度。**

实际上这里的异步工作是指：**FastAPI既支持异步并发，也支持多线程并行。**

实际分析：

```
from fastapi import APIRouter
import time
import asyncio
import uvicorn

app = APIRouter()

@app.get("/a")
async def a():
    time.sleep(1)
    return {"message": "异步模式，内部没有await调用异步方法，最后是串行执行"}

@app.get("/b")
async def b():
    loop = asyncio.get_event_loop()
    await loop.run_in_executor(None, time.sleep, 1)
    return {"message": "异步模式，放到一个event loop中去运行，函数内部用到了await异步方法"}

@app.get("/c")
async def c():
    await asyncio.sleep(1)
    return {"message": "异步模式，包含异步方法，所以很快"}

@app.get("/d")
def d():
    time.sleep(1)
    return {"message": "同步模式，但是FastAPI会放在线程池中运行，所以很快"}

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

我们并发100个请求分别测试这4个接口。
结果：
/a接口：100秒
/b接口：1秒
/c接口：1秒
/d接口：3秒

**就像官方所说，如果你不清楚你函数里的调用是否异步，那就定义为普通函数。因为它可以采用多线程的方式解决。反之，定义了async函数，里面却是同步的调用（在一个线程中串行执行），那么这将慢的是灾难！**
