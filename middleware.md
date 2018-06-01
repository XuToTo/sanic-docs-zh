# 中间件和监听器

中间件是可以在请求服务前后执行的函数。它们可以用来修改发送到用户定义的处理函数中的请求或是来自其中的响应。

此外，Sanic 还提供了监听器，允许你在应用生命周期的各个点执行代码。

## 中间件

中间件的类型有两种： request 和 response。两者都使用 `@app.middleware` 装饰器进行声明，并接受一个代表类型的字符串参数：`'request'` 或 `'response'`。其中，响应中间件接收请求和响应作为参数。

最简单的中间件就是根本不去修改 `request` 或者 `response`：

```python
@app.middleware('request')
async def print_on_request(request):
    print("I print when a request is received by the server")

@app.middleware('response')
async def print_on_response(request, response):
    print("I print when a response is returned by the server")
```

---

## 修改请求和响应

中间件可以修改传入的请求或是响应参数，*只要不返回它*。以下示例展示了部分用例：

```python
app = Sanic(__name__)

@app.middleware('response')
async def custom_banner(request, response):
    response.headers["Server"] = "Fake-Server"

@app.middleware('response')
async def prevent_xss(request, response):
    response.headers["x-xss-protection"] = "1; mode=block"

app.run(host="0.0.0.0", port=8000)
```

上面的代码将会按顺序应用两个中间件。第一个 **custom_banner** 中间件将会把 HTTP 响应头部中的 *Server* 改成 *Fake-Server*，第二个中间件 **prevent_xss** 会添加用来阻止跨站脚本（XSS）攻击的 HTTP 头部。这两个函数都会在用户函数返回响应后被调用（译注：即这两个 response 类型的中间件是在用户定义的处理函数之后生效的）。

---

## 提前响应

如果中间件返回一个 `HTTPResponse` 对象，那么将会停止处理请求，然后返回响应。如果返回发生在请求到达相关用户路由处理函数之前，那么这个处理函数就永远不会被调用。

返回响应将会阻断任何在此之后执行的中间件。

```python
@app.middleware('request')
async def halt_request(request):
    return text('I halted the request')

@app.middleware('response')
async def halt_response(request, response):
    return text('I halted the response')
```

---

## 监听器

如果你想在服务启动或关闭时执行启动/终止代码，你可以使用下面这些监听器：

- `before_server_start`
- `after_server_start`
- `before_server_stop`
- `after_server_stop`

这些监听器以装饰器的形式应用在函数上，它们接受 `app` 对象还有 `asyncio` 事件循环对象。

例如：

```python
@app.listener('before_server_start')
async def setup_db(app, loop):
    app.db = await db_setup()

@app.listener('after_server_start')
async def notify_server_started(app, loop):
    print('Server successfully started!')

@app.listener('before_server_stop')
async def notify_server_stopping(app, loop):
    print('Server shutting down!')

@app.listener('after_server_stop')
async def close_db(app, loop):
    await app.db.close()
```

还可以使用 `register_listener` 方法注册监听器。

如果你在实例化应用之外的模块中定义了监听器，那么这个方法就会很有用了。

```python
app = Sanic()

async def setup_db(app, loop):
    app.db = await db_setup()

app.register_listener(setup_db, 'before_server_start')
```

如果你想要在循环开始后计划执行一个后台任务，可以通过 Sanic 提供的 `add_task` 方法轻松实现。

```python
async def notify_server_started_after_five_seconds():
    await asyncio.sleep(5)
    print('Server successfully started!')

app.add_task(notify_server_started_after_five_seconds())
```

Sanic 会试图自动地注入 `app`，将其作为一个参数传入任务函数中：

```python
async def notify_server_started_after_five_seconds(app):
    await asyncio.sleep(5)
    print(app.name)

app.add_task(notify_server_started_after_five_seconds)
```

或者你可以显示地传递 `app`，效果是一样的：

```python
async def notify_server_started_after_five_seconds(app):
    await asyncio.sleep(5)
    print(app.name)

app.add_task(notify_server_started_after_five_seconds(app))
```