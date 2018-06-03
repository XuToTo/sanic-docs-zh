# 蓝图

在一个应用内，蓝图是种可以被用作于子路由的对象。除了直接向应用实例中添加路由外，蓝图也定义了类似的方法来添加路由，但是蓝图可以以一种灵活的可拔插的方式在应用内注册。蓝图对于大型的应用程序来说十分有用，你可以将应用程序的逻辑可以拆分成几个组或是职责分区。

## 第一个蓝图

下面展示了一个非常简单的蓝图，它在应用的 `/` 根路由上注册了一个处理函数。

假设你把这个文件保存为 `my_blueprint.py`，之后你就可以把它导入到你的主应用程序中了。

```python
from sanic.response import json
from sanic import Blueprint

bp = Blueprint('my_blueprint')

@bp.route('/')
async def bp_root(request):
    return json({'my': 'blueprint'})
```

---

## 注册蓝图

蓝图必须在应用程序中注册后才能使用。

```python
from sanic import Sanic
from my_blueprint import bp

app = Sanic(__name__)
app.blueprint(bp)

app.run(host='0.0.0.0', port=8000, debug=True)
```

这会将一个蓝图添加到应用中，并且注册了所有蓝图所定义的路由。在这个例子中，注册到 `app.router` 中的路由看上去是下面这个样子的：

```python
[Route(handler=<function bp_root at 0x7f908382f9d8>, methods=None, pattern=re.compile('^/$'), parameters=[])]
```

---

## 蓝图的分组和嵌套

蓝图还可以注册为一个列表或是元组的一部分，注册器会循环递归蓝图序列中的每个子序列，然后依次注册它们。`Blueprint.group` 可以用于简化这个过程，它允许以一种模仿后端目录结构的形式来模拟从前端所看到的内容。

看一下下面这个（十分刻意的）例子：

```text
api/
├──content/
│  ├──authors.py
│  ├──static.py
│  └──__init__.py
├──info.py
└──__init__.py
app.py
```

可以像下面这样初始化应用的蓝图层级机制：

```python
# api/content/authors.py
from sanic import Blueprint

authors = Blueprint('content_authors', url_prefix='/authors')
```

```python
# api/content/static.py
from sanic import Blueprint

static = Blueprint('content_static', url_prefix='/static')
```

```python
# api/content/__init__.py
from sanic import Blueprint

from .static import static
from .authors import authors

content = Blueprint.group(assets, authors, url_prefix='/content')
```

```python
# api/info.py
from sanic import Blueprint

info = Blueprint('info', url_prefix='/info')
```

```python
# api/__init__.py
from sanic import Blueprint

from .content import content
from .info import info

api = Blueprint.group(content, info, url_prefix='/api')
```

可以像下面这样在 `app.py` 中完成这些蓝图的注册：

```python
# app.py
from sanic import Sanic

from .api import api

app = Sanic(__name__)

app.blueprint(api)
```

---

## 使用蓝图

蓝图有许多和应用实例相同的功能。

### WebSocket 路由

WebSocket 的处理函数可以通过 `@bp.websocket` 装饰器或是 `bp.add_websocket_route` 方法注册到蓝图上。

### 中间件

还可以为对应的整个蓝图注册中间件。

```python
@bp.middleware
async def print_on_request(request):
    print("I am a spy")

@bp.middleware('request')
async def halt_request(request):
    return text('I halted the request')

@bp.middleware('response')
async def halt_response(request, response):
    return text('I halted the response')
```

### 异常

异常处理可以应用于对应的整个蓝图。

```python
@bp.exception(NotFound)
def ignore_404s(request, exception):
    return text("Yep, I totally found the page: {}".format(request.url))
```

### 静态文件

可以通过添加蓝图的前缀在整个应用内使用静态文件服务。

```python
# suppose bp.name == 'bp'

bp.static('/web/path', '/folder/to/serve')
# also you can pass name parameter to it for url_for
bp.static('/web/path', '/folder/to/server', name='uploads')
app.url_for('static', name='bp.uploads', filename='file.txt') == '/bp/web/path/file.txt'
```

---

## 启动和停止

蓝图可以在服务进程启动和停止时执行函数。如果是运行在多进程模式下（超过 1 个 worker）的话，这些函数将会在 worker fork 完成后被触发。

可用的事件有：

- `before_server_start`: 在服务开始接受连接之前执行
- `after_server_start`: 在服务开始接受连接之后执行
- `before_server_stop`: 在服务停止接受连接之前执行
- `after_server_stop`: 在服务停止并且所有请求都已完成之后执行

```python
bp = Blueprint('my_blueprint')

@bp.listener('before_server_start')
async def setup_connection(app, loop):
    global database
    database = mysql.connect(host='127.0.0.1'...)

@bp.listener('after_server_stop')
async def close_connection(app, loop):
    await database.close()
```

---

## 用例：API 的版本控制

蓝图对于 API 的版本控制来说非常有用，可以让一个蓝图指向 `/v1/<routes>`，然后让另一个指向 `/v2/<routes>`。

在初始化一个蓝图时，可以为其添加一个可选的 `url_prefix` 参数，这样可以为所有在蓝图中定义的路由添加一个前缀。这个特性可以作为模式来实现我们的 API 的版本控制。

```python
# blueprints.py
from sanic.response import text
from sanic import Blueprint

blueprint_v1 = Blueprint('v1', url_prefix='/v1')
blueprint_v2 = Blueprint('v2', url_prefix='/v2')

@blueprint_v1.route('/')
async def api_v1_root(request):
    return text('Welcome to version 1 of our documentation')

@blueprint_v2.route('/')
async def api_v2_root(request):
    return text('Welcome to version 2 of our documentation')
```

当我们把蓝图注册到 `app` 后，路由 `/v1` 和 `/v2` 就会指向各自的蓝图，这样就可以为每一个版本的 API 创建各自子站点。

```python
# main.py
from sanic import Sanic
from blueprints import blueprint_v1, blueprint_v2

app = Sanic(__name__)
app.blueprint(blueprint_v1, url_prefix='/v1')
app.blueprint(blueprint_v2, url_prefix='/v2')

app.run(host='0.0.0.0', port=8000, debug=True)
```

---

## 使用 `url_for` 构建 URL

如果你希望为一个蓝图中的路由生成 URL，记住端点名称要使用这样的格式 `<blueprint_name>.<handler_name>`。例如：

```python
@blueprint_v1.route('/')
async def root(request):
    url = request.app.url_for('v1.post_handler', post_id=5) # --> '/v1/post/5'
    return redirect(url)


@blueprint_v1.route('/post/<post_id>')
async def post_handler(request, post_id):
    return text('Post {} in Blueprint V1'.format(post_id))
```
