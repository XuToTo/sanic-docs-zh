# 静态文件

当通过 `app.static` 方法注册了静态文件和目录（比如一个图像文件）后，Sanic 就会为此提供静态文件服务。该方法需要指定端点 URL 和一个文件名。指定的文件可以通过所给的端点路径获取。

```python
from sanic import Sanic
from sanic.blueprints import Blueprint

app = Sanic(__name__)

# 为静态文件夹中的文件提供 URL 为 /static 的静态文件服务 
app.static('/static', './static')
# 通过 url_for 来构建 URL，name 参数默认是 ‘static’ 并且是可以忽略的
app.url_for('static', filename='file.txt') == '/static/file.txt'
app.url_for('static', name='static', filename='file.txt') == '/static/file.txt'

# 当请求 URL /the_best.png 时获取 /home/ubuntu/test.png 文件
app.static('/the_best.png', '/home/ubuntu/test.png', name='best_png')

# 你可以使用 url_for 来构建静态文件的 URL
# 你可以忽略 name 和 filename 参数，如果你没有定义这些的话
app.url_for('static', name='best_png') == '/the_best.png'
app.url_for('static', name='best_png', filename='any') == '/the_best.png'

# 你需要为其它的静态文件定义名称
app.static('/another.png', '/home/ubuntu/another.png', name='another')
app.url_for('static', name='another') == '/another.png'
app.url_for('static', name='another', filename='any') == '/another.png'

# 你还可以为蓝图提供静态文件服务
bp = Blueprint('bp', url_prefix='/bp')
bp.static('/static', './static')

# 直接为文件提供服务
bp.static('/the_best.png', '/home/ubuntu/test.png', name='best_png')
app.blueprint(bp)

app.url_for('static', name='bp.static', filename='file.txt') == '/bp/static/file.txt'
app.url_for('static', name='bp.best_png') == '/bp/test_best.png'

app.run(host="0.0.0.0", port=8000)
```