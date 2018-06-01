# 处理函数装饰器

由于 Sanic 的处理器函数只是简单的 Python 函数，所以你可以以一种类似于 Flask 的方式对它们应用装饰器。一个典型的用例就是当你想在处理函数代码执行之前先执行一些代码。

## 认证装饰器

假设你想要检查一个用户是否授权访问一个特殊的端点。你可以创建一个装饰器装饰处理函数，从而检查请求判断客户端是否授权访问资源，并且返回适当的响应。

```python
from functools import wraps
from sanic.response import json

def authorized():
    def decorator(f):
        @wraps(f)
        async def decorated_function(request, *args, **kwargs):
            # 执行一些检查请求的客户端授权状态的方法
            is_authorized = check_request_for_authorization_status(request)

            if is_authorized:
                # 用户已授权
                # 执行处理函数并返回响应
                response = await f(request, *args, **kwargs)
                return response
            else:
                # 用户没有授权
                return json({'status': 'not_authorized'}, 403)
        return decorated_function
    return decorator

@app.route("/")
@authorized()
async def test(request):
    return json({status: 'authorized'})
```