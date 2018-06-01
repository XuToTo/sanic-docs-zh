# 异常

异常可以从请求处理器内部抛出，并且能够被 Sanic 自动处理。异常将异常信息作为它们的第一个参数，同时还可以携带状态码传递回 HTTP 的响应中。

## 抛出一个异常

只要从 `sanic.exceptions` 模块中 `raise` 相关的异常就可以抛出一个异常。

```python
from sanic.exceptions import ServerError

@app.route('/killme')
async def i_am_ready_to_die(request):
    raise ServerError("Something bad happened", status_code=500)
```

你还可以使用带有合适状态码的 `abort` 函数：

```python
from sanic.exceptions import abort
from sanic.response import text

@app.route('/youshallnotpass')
async def no_no(request):
        abort(401)
        # this won't happen
        text("OK")
```

---

## 处理异常

可以通过 `@app.exception` 装饰器来重写 Sanic 默认的异常处理器。该装饰器接受一个需要处理异常的列表作为参数。你可以传入 `SanicException` 来捕获所有的异常。被装饰的异常处理函数必须接受 `Request` 和 `Exception` 对象作为参数。

```python
from sanic.response import text
from sanic.exceptions import NotFound

@app.exception(NotFound)
async def ignore_404s(request, exception):
    return text("Yep, I totally found the page: {}".format(request.url))
```

---

## 常用异常

下面是一些常用的异常：

- `NotFound`: 当没有为请求找到合适的路由时会被调用

- `ServerError`: 当服务内部出现错误时会被调用。这个异常的出现通常是因为在用户代码中有异常抛出。

参看 `sanic.exceptions` 模块获取所有可以用来抛出的异常列表。