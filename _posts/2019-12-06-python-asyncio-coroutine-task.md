---
layout: post
title:  "Python 异步模块 asyncio 中的协程与任务"
date:   2019-12-06 23:13:00+0800
categories: python asyncio
---
协程（Coroutine）是允许执行被**挂起**、**恢复**、以及**取消**的程序。Python 3 中最初是使用 `@asyncio.coroutine` 装饰器和 `yield from` 关键字组合来实现协程。单词 `yield` 在这里并非在生成器（generator）中所表示的“**产出**”，而是和交通标志中所表达的“**让步**”之意。其实在生成器中也包含“让步”的意思，即把执行权交给调用者，生成器暂缓执行，等待调用者对生成的结果处理完成，再恢复生成器的执行。在异步程序中，`yield` 则是把当前的执行权交给事件循环中的其它协程。Python 3.5 开始 async/await 被引入，Python 3.7 开始成为保留关键字，让协程的使用更加方便和直观。本文使用 **Python 3.8**。

使用 `async def` 定义一个协程：

```python
async def main():
    print('hello coroutine')
```

协程具体可以分为使用 `async def` 定义的**协程函数**和调用协程函数返回的**协程对象**。调用一个协程函数并不会执行协程中的程序，而是只返回一个协程对象。

`asyncio` 提供了三种方式来执行协程：

**1. 使用 `asyncio.run()`**

`run()` 函数接收一个协程对象，在执行时，总会创建一个新的事件循环，并在结束后关闭循环。理想情况下，`run()` 函数应当被作为程序的总入口，并且只会被调用一次。如果同一线程中还有其它事件循环在运行，则此方法不能被调用。

```python
async def main():
    print('hello coroutine')

asyncio.run(main())
```

**2. 使用 `await` 等待一个协程**

`await` 的作用和 `yield from` 相同，即让出当前的执行权，等待的对象有结果返回时，再重新获得可以被继续执行的权利。

只有可等待对象（Awaitable object）才能被 `await`。除**协程**（`Coroutine`）外，`asyncio` 还提供了两种可等待对象：**任务**（`Task`）和**期货**（`Future`）。

```python
async def main():
    print('hello')
    await asyncio.sleep(1)
    await asyncio.sleep(2)
    print('coroutine')
```

上述程序在打印出 *hello* 后会等待 3 秒再打印出 *coroutine*。存在多个 `await` 时，会**依次顺序**执行，因为有两条 `sleep()` 语句，分别等待 1 秒和 2 秒，所以一共用时 3 秒。

**3. 使用 `async.create_task()` 创建 `Task`**

`create_task()` 会把一个协程打包成一个**任务**（`Task`），并立即排入日程准备执行，函数返回值是打包完成的 `Task` 对象。

```python
async def foo(n):
    await asyncio.sleep(n)

async def main():
    task1 = asyncio.create_task(foo(1))
    task2 = asyncio.create_task(foo(2))

    print('hello')
    await task1
    await task2
    print('coroutine')
```

当使用 `create_task()` 时，创建的任务立即被加入到事件循环中，并不会阻塞当前的程序，所以上述程序在打印出 *hello* 后只需等待 2 秒就打印出 *coroutine*。

如上面所介绍，使用 `create_task()` 可以并发执行程序。`asyncio` 同时提供了几个函数用于方便地实现多任务并发执行：

**1. `asyncio.gather(*aws, return_exceptions=False)`**

`gather()` 函数接受传入多个可等待对象 *aws*，如果某个可等待对象是协程，则会被自动打包成 `Task`。`gather()` 返回结果是和 *aws* 传入顺序一致的列表。`gather()` 同时可以传入参数 *return_exceptions* 来处理异常，默认值为 `False`。如果为 `False` 时，执行过程中引发的首个异常会立即返回给等待 `gather()` 的任务，`await` 会直接结束等待并抛出异常，但是其它正常执行的 `Task` 不会被取消，这种情况适用于确保任务尽可能被执行完成，但是不关心返回结果，因为如果有任何一个任务出现异常，返回结果列表就不会顺利生成。如果为 *return_exceptions* 设为 `True`，异常会和正常结果一同被聚合进最终结果列表，适用于对结果有需求应用场景。

```python
async def foo():
    return 'foo'

async def bar():
    raise RuntimeError('fake runtime error')

async def main():
    task1 = asyncio.create_task(foo())
    task2 = asyncio.create_task(bar())

    # return_exceptions=True
    results = await asyncio.gather(task1, task2, return_exceptions=True)
    # 输出: ['foo', RuntimeError('fake runtime error')]
    print(results)
    # 返回结果的顺序和传参顺序一致
    assert isinstance(results[1], RuntimeError)

    # return_exceptions=False
    try:
        results = await asyncio.gather(task1, task2, return_exceptions=False)
        # 此处打印并不会被执行, results 也未被赋值
        print(results)
    except RuntimeError as runtime_err:
        # 捕获异常并打印: fake runtime error
        print(runtime_err)
```

**2. `asyncio.wait(aws, *, timeout=None, return_when=ALL_COMPLETED)`**

`wait()` 接受 *aws* 任务集合传入， 然后并发执行。参数 *return_when* 用来控制返回条件，当 *return_when=ALL_COMPLETED* 时，会在所有任务完成后返回结果；当 *return_when=FIRST_COMPLETED* 时，任务集合中有一个任务完成就立即返回结果。*timeout* 参数用来控制超时时间，可以是整数或浮点数，以**秒**为单位。`wait()` 的返回值是 *(done, pending)* 元组，*done* 中包含运行完成的任务，*pending* 中包含未完成被挂起的任务。

```python
async def foo():
    await asyncio.sleep(3)
    return 'foo'


async def bar():
    await asyncio.sleep(1)
    return 'bar'


async def main():
    # 有一个任务执行完成即返回, 总共耗时 1 秒
    done, pending = await asyncio.wait({foo(), bar()}, return_when=asyncio.FIRST_COMPLETED)
    # done 集合里包含打包成 Task 的 bar()
    print(f'done: {done}')
    # pendding 集合里包含打包成 Task 的 foo()
    print(f'pending: {pending}')

    # 所有任务执行完成后返回, 总共耗时 3 秒
    done, pending = await asyncio.wait({foo(), bar()}, return_when=asyncio.ALL_COMPLETED)
    # done 集合里包含被带打包成 Task 的 foo() 和 bar()
    print(f'done: {done}')
    # pending 集合为空
    print(f'pending: {pending}')

    # 所有任务执行完成, 但运行时间不能超 2 秒后返回, 总共耗时 2 秒
    done, pending = await asyncio.wait({foo(), bar()}, timeout=2, return_when=asyncio.ALL_COMPLETED)
    # done 集合里包含打包成 Task 的 bar()
    print(f'done: {done}')
    # pendding 集合里包含打包成 Task 的 foo()
    print(f'pending: {pending}')
```

**3. `asyncio.as_completed(aws)`**

`as_completed()` 接受 *aws* 集合，然后返回一个 `Future` 迭代器，遍历这个迭代器会依次遍历剩余可等待对象集合中**最早完成**的结果。

```python
async def foo():
    await asyncio.sleep(2)
    return 'foo'

async def bar():
    await asyncio.sleep(1)
    return 'bar'

async def main():
    for fut in asyncio.as_completed({foo(), bar()}):
        earliest_result = await fut
        # 会依次打印 bar 和 foo, 因为 bar() 会更早执行完毕
        print(earliest_result)
```

上面介绍多任务并发时引入了超时的概念，超时也可以被应用在单独的一个任务中，使用 `asyncio.wait_for(aw, timeout)` 函数，该函数接受一个任务 *aw* 和超时时间 *timeout*，如果在限制时间内完成，则会正常返回，否则会被取消并抛出 `asyncio.TimeoutError` 异常。

为了防止任务被取消，可以使用 `asyncio.shield(aw)` 进行保护。`shield()` 会屏蔽外部取消操作，如果外部任务被取消，其内部正在执行的任务不会被取消，在内部看来取消操作并没有发生，由于内部仍正常执行，执行完毕后会触发 `asyncio.CancelledError` 异常，如果确保程序能忽略异常继续执行，需要在外部使用 `try-except` 捕获异常。如果在任务内部取消，则会被成功取消。
