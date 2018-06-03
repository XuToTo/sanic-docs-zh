# 版本控制

你可以向路由装饰器或是蓝图的初始函数中传入 `version` 关键字参数。这会为 URL 添加一段 `v{version}` 前缀，其中 `{version}` 是版本号。

## 每个路由的版本

你可以直接向路由中传入版本号。

```python
from sanic import response


@app.route('/text', version=1)
def handle_request(request):
    return response.text('Hello world! Version 1')

@app.route('/text', version=2)
def handle_request(request):
    return response.text('Hello world! Version 2')

app.run(port=80)
```

然后使用 `curl` 测试一下：

```bash
curl localhost/v1/text
curl localhost/v2/text
```

---

## 蓝图的全局版本

你还可以向蓝图中传入版本号，这会应用于蓝图中的所有路由。

```python
from sanic import response
from sanic.blueprints import Blueprint

bp = Blueprint('test', version=1)

@bp.route('/html')
def handle_request(request):
    return response.html('<p>Hello world!</p>')
```

然后使用 `curl` 测试一下：

```bash
curl localhost/v1/html
```
