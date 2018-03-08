# 路由

路由允许用户为不同的 URL 端点指定处理函数。

下面是一个基础的路由，`app` 是一个 `Sanic` 类的实例：

```python
from sanic.response import json

@app.route("/")
async def test(request):
    return json({ "hello": "world" })
```

当访问地址 `http://server.url/` （服务的根地址）时，最后的那个 `/` 会被路由匹配到 `test` 处理函数，这个函数返回了一个 JSON 对象。

Sanic 处理函数**必须**使用 `async def` 语法定义，因为它们都是异步函数。

## 请求参数

Sanic 自带的基础路由支持请求参数。可以像 `<PARM>` 这样，通过尖括号来指定请求参数。请求参数会作为关键字参数传入路由处理函数。

```python
from sanic.response import text

@app.route('/tag/<tag>')
async def tag_handler(request, tag):
    return text('Tag - {}'.format(tag))
```

在尖括号中的参数名后添加 `:type` 可以指定参数的类型，

如果参数不符合指定的类型的话，Sanic 就会抛出一个 `NotFound` 异常，那么这个 URL 将会得到一个 `404: Page not found` 的错误结果。

```python
from sanic.response import text

@app.route('/number/<integer_arg:int>')
async def integer_handler(request, integer_arg):
    return text('Integer - {}'.format(integer_arg))

@app.route('/number/<number_arg:number>')
async def number_handler(request, number_arg):
    return text('Number - {}'.format(number_arg))

@app.route('/person/<name:[A-z]+>')
async def person_handler(request, name):
    return text('Person - {}'.format(name))

@app.route('/folder/<folder_id:[A-z0-9]{0,4}>')
async def folder_handler(request, folder_id):
    return text('Folder - {}'.format(folder_id))
```

## HTTP 请求类型

默认情况下，一个 URL 的路由仅仅只对 GET 请求有效。

不过，`@app.route` 装饰器可以接受一个 `methods` 可选参数，这个参数允许处理函数对列表中指定的 HTTP 方法生效。

```python
from sanic.response import text

@app.route('/post', methods=['POST'])
async def post_handler(request):
    return text('POST request - {}'.format(request.json))

@app.route('/get', methods=['GET'])
async def get_handler(request):
    return text('GET request - {}'.format(request.args))

```

此外，还有一个可选的 `host` 参数（可以是列表或者字符串）。这个参数限制了路由只能处理指定的主机。如果还有额外的一个没有指定 host 的路由，那么这个路由就会作为最后的默认路由。

```python
@app.route('/get', methods=['GET'], host='example.com')
async def get_handler(request):
    return text('GET request - {}'.format(request.args))

# if the host header doesn't match example.com, this route will be used
@app.route('/get', methods=['GET'])
async def get_handler(request):
    return text('GET request in default - {}'.format(request.args))
```

还可以使用对应的装饰器快捷方法:

```python
from sanic.response import text

@app.post('/post')
async def post_handler(request):
    return text('POST request - {}'.format(request.json))

@app.get('/get')
async def get_handler(request):
    return text('GET request - {}'.format(request.args))
```

## `add_route` 方法

正如我们所见，路由通常是使用 `@app.route` 装饰器指定的。其实，这个装饰器只是 `app.add_route` 方法的一个封装，这个方法可以像下面这样使用：

```python
from sanic.response import text

# 定义处理函数
async def handler1(request):
    return text('OK')

async def handler2(request, name):
    return text('Folder - {}'.format(name))

async def person_handler2(request, name):
    return text('Person - {}'.format(name))

# 把每一个处理函数都添加为路由
app.add_route(handler1, '/test')
app.add_route(handler2, '/folder/<name>')
app.add_route(person_handler2, '/person/<name:[A-z]>', methods=['GET'])
```

## 使用 `url_for` 构建 URL

Sanic 提供了一个 `url_for` 方法来基于处理函数的名称生成 URL。如果你想在你的应用里避免硬编码 URL 路径的话，这会非常实用。你只需要引用处理函数的名称，就像下面这样：

```python
from sanic.response import redirect

@app.route('/')
async def index(request):
    # 为端点 `post_handler` 生成一个 URL
    url = app.url_for('post_handler', post_id=5)
    # the URL is `/posts/5`, redirect to it
    return redirect(url)

@app.route('/posts/<post_id>')
async def post_handler(request, post_id):
    return text('Post - {}'.format(post_id))
```

在使用 `url_for` 时还需要注意一些其它的事情：

- 如果传入 `url_for` 的关键字参数不是请求参数的话将会被包括在 URL 的查询字段中。例如：

```python
url = app.url_for('post_handler', post_id=5, arg_one='one', arg_two='two')
# /posts/5?arg_one=one&arg_two=two
```

- Multivalue argument can be passed to `url_for`. For example:

```python
url = app.url_for('post_handler', post_id=5, arg_one=['one', 'two'])
# /posts/5?arg_one=one&arg_one=two
```

- Also some special arguments (`_anchor`, `_external`, `_scheme`, `_method`, `_server`) passed to `url_for` will have special url building (`_method` is not support now and will be ignored). For example:

```python
url = app.url_for('post_handler', post_id=5, arg_one='one', _anchor='anchor')
# /posts/5?arg_one=one#anchor

url = app.url_for('post_handler', post_id=5, arg_one='one', _external=True)
# //server/posts/5?arg_one=one
# _external requires passed argument _server or SERVER_NAME in app.config or url will be same as no _external

url = app.url_for('post_handler', post_id=5, arg_one='one', _scheme='http', _external=True)
# http://server/posts/5?arg_one=one
# when specifying _scheme, _external must be True

# you can pass all special arguments one time
url = app.url_for('post_handler', post_id=5, arg_one=['one', 'two'], arg_two=2, _anchor='anchor', _scheme='http', _external=True, _server='another_server:8888')
# http://another_server:8888/posts/5?arg_one=one&arg_one=two&arg_two=2#anchor
```

- All valid parameters must be passed to `url_for` to build a URL. If a parameter is not supplied, or if a parameter does not match the specified type, a `URLBuildError` will be thrown.

## WebSocket 路由

WebSocket 协议的路由可以通过 `@app.websocket` 装饰器进行定义：

```python
@app.websocket('/feed')
async def feed(request, ws):
    while True:
        data = 'hello!'
        print('Sending: ' + data)
        await ws.send(data)
        data = await ws.recv()
        print('Received: ' + data)
```

或者使用 `app.add_websocket_route` 方法而不是装饰器：

```python
async def feed(request, ws):
    pass

app.add_websocket_route(my_websocket_handler, '/feed')
```

Handlers for a WebSocket route are passed the request as first argument, and a WebSocket protocol object as second argument. The protocol object has `send` and `recv` methods to send and receive data respectively.

WebSocket support requires the [websockets](https://github.com/aaugustin/websockets)
package by Aymeric Augustin.

## 关于 `strict_slashes`

你可以决定 `routes` 是否需要对尾部斜杠进行严格匹配，这是可以配置的。

```python
# 为所有的路由提供默认的 strict_slashes 值
app = Sanic('test_route_strict_slash', strict_slashes=True)

# 你还可以重写特定路由的 strict_slashes 的值
@app.get('/get', strict_slashes=False)
def handler(request):
    return text('OK')

# 对蓝图也管用
bp = Blueprint('test_bp_strict_slash', strict_slashes=True)

@bp.get('/bp/get', strict_slashes=False)
def handler(request):
    return text('OK')

app.blueprint(bp)
```

## 自定义路由名称

你可以向装饰器中传入 `name` 参数更改路由名称，从而避免使用默认名称（`handler.__name__`）。

```python
app = Sanic('test_named_route')

@app.get('/get', name='get_handler')
def handler(request):
    return text('OK')

# 接下来你需要使用 `app.url_for('get_handler')`
# 而不是 `app.url_for('handler')`

# 还可以用在蓝图上
bp = Blueprint('test_named_bp')

@bp.get('/bp/get', name='get_handler')
def handler(request):
    return text('OK')

app.blueprint(bp)

# 接下来你需要使用 `app.url_for('test_named_bp.get_handler')`
# 而不是 `app.url_for('test_named_bp.handler')`

# 可以为相同 URL 的不同方法上使用不同的名称

@app.get('/test', name='route_test')
def handler(request):
    return text('OK')

@app.post('/test', name='route_post')
def handler2(request):
    return text('OK POST')

@app.put('/test', name='route_put')
def handler3(request):
    return text('OK PUT')

# 下面的 URL 都是相同的，你可以使用它们其中任意一个
# '/test'
app.url_for('route_test')
# app.url_for('route_post')
# app.url_for('route_put')

# for same handler name with different methods
# you need specify the name (it's url_for issue)
@app.get('/get')
def handler(request):
    return text('OK')

@app.post('/post', name='post_handler')
def handler(request):
    return text('OK')

# 然后
# app.url_for('handler') == '/get'
# app.url_for('post_handler') == '/post'
```

## 为静态文件构建 URL

现在你可以使用 `url_for` 为静态文件构建 URL。如果是为文件目录构建 URL，那么 `filename` 参数是可以被忽略的。

```python
app = Sanic('test_static')
app.static('/static', './static')
app.static('/uploads', './uploads', name='uploads')
app.static('/the_best.png', '/home/ubuntu/test.png', name='best_png')

bp = Blueprint('bp', url_prefix='bp')
bp.static('/static', './static')
bp.static('/uploads', './uploads', name='uploads')
bp.static('/the_best.png', '/home/ubuntu/test.png', name='best_png')
app.blueprint(bp)

# then build the url
app.url_for('static', filename='file.txt') == '/static/file.txt'
app.url_for('static', name='static', filename='file.txt') == '/static/file.txt'
app.url_for('static', name='uploads', filename='file.txt') == '/uploads/file.txt'
app.url_for('static', name='best_png') == '/the_best.png'

# blueprint url building
app.url_for('static', name='bp.static', filename='file.txt') == '/bp/static/file.txt'
app.url_for('static', name='bp.uploads', filename='file.txt') == '/bp/uploads/file.txt'
app.url_for('static', name='bp.best_png') == '/bp/static/the_best.png'
```