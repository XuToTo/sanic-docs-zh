# 请求数据

当端点接收到 HTTP 请求时，`Request` 对象就会传入到路由函数。

下面这些变量都是 `Request` 对象中可以访问的属性：

- `json` (any) - JSON 请求体

  ```python
  from sanic.response import json

  @app.route("/json")
  def post_json(request):
      return json({ "received": True, "message": request.json })
  ```

- `args` (dict) - 查询字段变量。查询字段是 URL 中一段类似于 `?key1=value1&key2=value2` 的部分。中如果解析刚才的 URL，`args` 字典看上去会是这样的 `{'key1': ['value1'], 'key2': ['value2']}`。请求的 `query_string` 变量保存着未被解析的字符串值。

  ```python
  from sanic.response import json

  @app.route("/query_string")
  def query_string(request):
      return json({ "parsed": True, "args": request.args, "url": request.url, "query_string": request.query_string })
  ```

- 许多情况下你需要从已经封装好的字典中获取 URL 参数。对于和之前相同的 URL `?key1=value1&key2=value2` 来说，`raw_args` 字典是这样的 {'key1': 'value1', 'key2': 'value2'}`。

- `files` (`File` 对象的字典) - 拥有 `name`、`body` 和 `type` 属性的文件对象列表

  ```python
  from sanic.response import json

  @app.route("/files")
  def post_json(request):
      test_file = request.files.get('test')

      file_parameters = {
          'body': test_file.body,
          'name': test_file.name,
          'type': test_file.type,
      }

      return json({ "received": True, "file_names": request.files.keys(), "test_file_parameters": file_parameters })
  ```

- `form` (dict) - POST 表格变量

  ```python
  from sanic.response import json

  @app.route("/form")
  def post_json(request):
      return json({ "received": True, "form_data": request.form, "test": request.form.get('test') })
  ```

- `body` (bytes) - 原始 POST 请求体。这个属性可以无视内容类型直接获取请求的原始数据

  ```python
  from sanic.response import text

  @app.route("/users", methods=["POST",])
  def create_user(request):
      return text("You are trying to create a user with the following POST: %s" % request.body)
  ```

- `headers` (dict) - 一个包含请求头部并且大小写不敏感的字典

- `method` (str) - 请求的 HTTP 方法（即 `GET`，`POST`）

- `ip` (str) - 请求者的 IP 地址

- `port` (str) - 请求者的端口地址

- `socket` (tuple) - 请求者的 `(IP, port)`

- `app` - 一个用于处理当前请求的 Sanic 应用程序对象的引用。当内部蓝图或是模块中的处理函数无法访问全局 `app` 对象时，这个属性会很有帮助

  ```python
  from sanic.response import json
  from sanic import Blueprint

  bp = Blueprint('my_blueprint')

  @bp.route('/')
  async def bp_root(request):
      if request.app.config['DEBUG']:
          return json({'status': 'debug'})
      else:
          return json({'status': 'production'})
  ```

- `url`: 完整的请求 URL， 即 `http://localhost:8000/posts/1/?foo=bar`
- `scheme`: 请求的 HTTP 模式： `http` 或 `https`
- `host`: 请求的主机：`localhost:8080`
- `path`: 请求的路径：`/posts/1/`
- `query_string`: 请求的查询字段：`foo=bar` 或是一个空字符串 `''`
- `uri_template`: 用于匹配路由处理函数的模板：`/posts/<id>/`
- `token`: 授权请求头部的值： `Basic YWRtaW46YWRtaW4=`

## 使用 `get` 和 `getlist` 获取值

作为字典返回的请求属性实际上是名为 `RequestParameters` 的 `dict` 子类。这个对象使用 `get` 和 `getlist` 两个方法所得到的结果是有所区别的。

- `get(key, default=None)` 正常情况下， 当 `key` 对应的值是一个列表时*只有第一项会被返回*
- `getlist(key, default=None)` 正常情况下*返回整个列表*

```python
from sanic.request import RequestParameters

args = RequestParameters()
args['titles'] = ['Post 1', 'Post 2']

args.get('titles') # => 'Post 1'

args.getlist('titles') # => ['Post 1', 'Post 2']
```