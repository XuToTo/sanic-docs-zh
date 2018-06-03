# 类视图

类视图只是种实现了请求对应的响应行为的类。它提供了一种可以将同一个端点的不同 HTTP 请求类型划分开来的处理方式。端点可以被分配一个类视图，而不是通过在不同的处理函数上定义和装饰上每个端点所支持的请求类型。

## 定义视图

一个类视图应该是 `HTTPMethodView` 的子类。你可以为你每个想支持的 HTTP 请求类型实现对应的类方法。如果收到的请求类型没有定义对应的方法就会生成一个 `405: Method not allowed` 响应。

要在一个端点上注册类视图，需要使用 `app.add_route` 方法。第一个参数应该是一个调用了 `as_view` 方法的定义类，第二个参数是端点的 URL。

可用的方法有 `get`、`post`、`put`、`patch` 和 `delete`。下面是一个使用了所有这些方法的类：

```python
from sanic import Sanic
from sanic.views import HTTPMethodView
from sanic.response import text

app = Sanic('some_name')

class SimpleView(HTTPMethodView):

  def get(self, request):
      return text('I am get method')

  def post(self, request):
      return text('I am post method')

  def put(self, request):
      return text('I am put method')

  def patch(self, request):
      return text('I am patch method')

  def delete(self, request):
      return text('I am delete method')

app.add_route(SimpleView.as_view(), '/')
```

你还可以使用 `async` 关键字。

```python
from sanic import Sanic
from sanic.views import HTTPMethodView
from sanic.response import text

app = Sanic('some_name')

class SimpleAsyncView(HTTPMethodView):

  async def get(self, request):
      return text('I am async get method')

app.add_route(SimpleAsyncView.as_view(), '/')
```

---

## URL 参数

如果你需要获取 URL 参数，可以像之前路由章节中所说的那样，在定义方法的时候包含它们。

```python
class NameView(HTTPMethodView):

  def get(self, request, name):
    return text('Hello {}'.format(name))

app.add_route(NameView.as_view(), '/<name>')
```

---

## 装饰器

如果你想向视图类中添加装饰器的话，你可以设置 `decorators` 类变量。它们将会在调用 `as_view` 时应用到类上。

```python
class ViewWithDecorator(HTTPMethodView):
  decorators = [some_decorator_here]

  def get(self, request, name):
    return text('Hello I have a decorator')

  def post(self, request, name):
    return text("Hello I also have a decorator")

app.add_route(ViewWithDecorator.as_view(), '/url')
```

但是如果你只想装饰个别的函数而不是所有的话，你可以像下面这么做：

```python
class ViewWithSomeDecorator(HTTPMethodView):

    @staticmethod
    @some_decorator_here
    def get(request, name):
        return text("Hello I have a decorator")

    def post(self, request, name):
        return text("Hello I don't have any decorators")
```

---

## URL 的构建

如果你希望为一个 `HTTPMethodView` 构建 URL 的话，要记得将类名作为端点传入 `url_for` 中。例如：

```python
@app.route('/')
def index(request):
    url = app.url_for('SpecialClassView')
    return redirect(url)

class SpecialClassView(HTTPMethodView):
    def get(self, request):
        return text('Hello from the Special Class View!')

app.add_route(SpecialClassView.as_view(), '/special_class_view')
```

---

## 使用组合视图

你可以使用 `CompositionView` 来将处理函数移到视图类的外部，以此来替换 `HTTPMethodView`。

每个所支持的 HTTP 方法对应的处理函数，只要是在代码别处定义的，都需要通过 `Composition.add` 方法添加到视图中。第一个参数是能够处理的 HTTP 方法的列表（比如 `['GET', 'POST']`），第二个参数则是处理函数。下面的示例展示了在 `CompositionView` 中使用外部定义的处理函数以及内联化 lambda 的用法： 

```python
from sanic import Sanic
from sanic.views import CompositionView
from sanic.response import text

app = Sanic(__name__)

def get_handler(request):
    return text('I am a get method')

view = CompositionView()
view.add(['GET'], get_handler)
view.add(['POST', 'PUT'], lambda request: text('I am a post/put method'))

# 使用新的视图来处理根 URL 对应的请求
app.add_route(view, '/')
```

注意：目前你还不能通过 `url_for` 为 `CompositionView` 构建 URL。