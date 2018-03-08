# 蓝图

Blueprints are objects that can be used for sub-routing within an application.
Instead of adding routes to the application instance, blueprints define similar
methods for adding routes, which are then registered with the application in a
flexible and pluggable manner.

蓝图对于大型的应用程序来说十分有用，你可以将应用程序的逻辑可以拆分成几个组或是职责分区。

## 我的第一个蓝图

The following shows a very simple blueprint that registers a handler-function at
the root `/` of your application.

假如你把这个文件保存为 `my_blueprint.py`，之后你就可以把它导入到你的主应用程序中了。

```python
from sanic.response import json
from sanic import Blueprint

bp = Blueprint('my_blueprint')

@bp.route('/')
async def bp_root(request):
    return json({'my': 'blueprint'})
```

## 注册蓝图

蓝图必须在应用程序中注册后才能使用。

```python
from sanic import Sanic
from my_blueprint import bp

app = Sanic(__name__)
app.blueprint(bp)

app.run(host='0.0.0.0', port=8000, debug=True)
```

This will add the blueprint to the application and register any routes defined
by that blueprint. In this example, the registered routes in the `app.router`
will look like:

```python
[Route(handler=<function bp_root at 0x7f908382f9d8>, methods=None, pattern=re.compile('^/$'), parameters=[])]
```

## 蓝图的分组和嵌套

Blueprints may also be registered as part of a list or tuple, where the registrar will recursively cycle through any sub-sequences of blueprints and register them accordingly. The `Blueprint.group` method is provided to simplify this process, allowing a 'mock' backend directory structure mimicking what's seen from the front end. Consider this (quite contrived) example:

```
api/
├──content/
│  ├──authors.py
│  ├──static.py
│  └──__init__.py
├──info.py
└──__init__.py
app.py
```

Initialization of this app's blueprint hierarchy could go as follows:

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

And registering these blueprints in `app.py` can now be done like so:

```python
# app.py
from sanic import Sanic

from .api import api

app = Sanic(__name__)

app.blueprint(api)
```

## 使用蓝图

蓝图有许多和应用实例相同的功能。

### WebSocket 路由

WebSocket handlers can be registered on a blueprint using the `@bp.websocket`
decorator or `bp.add_websocket_route` method.

### 中间件

Using blueprints allows you to also register middleware globally.

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

Exceptions can be applied exclusively to blueprints globally.

```python
@bp.exception(NotFound)
def ignore_404s(request, exception):
    return text("Yep, I totally found the page: {}".format(request.url))
```

### 静态文件

Static files can be served globally, under the blueprint prefix.

```python

# suppose bp.name == 'bp'

bp.static('/web/path', '/folder/to/serve')
# also you can pass name parameter to it for url_for
bp.static('/web/path', '/folder/to/server', name='uploads')
app.url_for('static', name='bp.uploads', filename='file.txt') == '/bp/web/path/file.txt'
```

## 启动和停止

Blueprints can run functions during the start and stop process of the server.
If running in multiprocessor mode (more than 1 worker), these are triggered
after the workers fork.

可用的事件有：

- `before_server_start`: Executed before the server begins to accept connections
- `after_server_start`: Executed after the server begins to accept connections
- `before_server_stop`: Executed before the server stops accepting connections
- `after_server_stop`: Executed after the server is stopped and all requests are complete

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

## 用例：API 的版本控制

Blueprints can be very useful for API versioning, where one blueprint may point
at `/v1/<routes>`, and another pointing at `/v2/<routes>`.

When a blueprint is initialised, it can take an optional `url_prefix` argument,
which will be prepended to all routes defined on the blueprint. This feature
can be used to implement our API versioning scheme.

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

When we register our blueprints on the app, the routes `/v1` and `/v2` will now
point to the individual blueprints, which allows the creation of *sub-sites*
for each API version.

```python
# main.py
from sanic import Sanic
from blueprints import blueprint_v1, blueprint_v2

app = Sanic(__name__)
app.blueprint(blueprint_v1, url_prefix='/v1')
app.blueprint(blueprint_v2, url_prefix='/v2')

app.run(host='0.0.0.0', port=8000, debug=True)
```

## 使用 `url_for` 构建 URL

如果你希望在蓝图中为一个路由生成一个 URL，remember that the endpoint name
takes the format `<blueprint_name>.<handler_name>`. For example:

```python
@blueprint_v1.route('/')
async def root(request):
    url = request.app.url_for('v1.post_handler', post_id=5) # --> '/v1/post/5'
    return redirect(url)


@blueprint_v1.route('/post/<post_id>')
async def post_handler(request, post_id):
    return text('Post {} in Blueprint V1'.format(post_id))
```