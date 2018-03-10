# 响应

使用 `sanic.response` 模块中的函数可以创建响应。

## 纯文本

```python
from sanic import response

@app.route('/text')
def handle_request(request):
    return response.text('Hello world!')
```

## HTML

```python
from sanic import response

@app.route('/html')
def handle_request(request):
    return response.html('<p>Hello world!</p>')
```

## JSON

```python
from sanic import response

@app.route('/json')
def handle_request(request):
    return response.json({'message': 'Hello world!'})
```

## 文件

```python
from sanic import response

@app.route('/file')
async def handle_request(request):
    return await response.file('/srv/www/whatever.png')
```

## 流

```python
from sanic import response

@app.route("/streaming")
async def index(request):
    async def streaming_fn(response):
        response.write('foo')
        response.write('bar')
    return response.stream(streaming_fn, content_type='text/plain')
```

## 文件流

结合了上面的文件和流来处理大的文件

```python
from sanic import response

@app.route('/big_file.png')
async def handle_request(request):
    return await response.file_stream('/srv/www/whatever.png')
```

## 重定向

```python
from sanic import response

@app.route('/redirect')
def handle_request(request):
    return response.redirect('/json')
```

## Raw

未编码的响应体

```python
from sanic import response

@app.route('/raw')
def handle_request(request):
    return response.raw(b'raw data')
```

## 修改响应头部和状态码

向那些函数中传入 `headers` 或 `status` 参数可以修改响应头部或状态码：

```python
from sanic import response

@app.route('/json')
def handle_request(request):
    return response.json(
        {'message': 'Hello world!'},
        headers={'X-Served-By': 'sanic'},
        status=200
    )
```