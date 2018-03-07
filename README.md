# Sanic 中文文档 [![Build Status](https://travis-ci.org/XuToTo/sanic-docs-zh.svg?branch=master)](https://travis-ci.org/XuToTo/sanic-docs-zh)

Sanic is a Flask-like Python 3.5+ web server that's written to go fast.  It's based on the work done by the amazing folks at magicstack, and was inspired by [this article](https://magic.io/blog/uvloop-blazing-fast-python-networking/).

On top of being Flask-like, Sanic supports async request handlers.  This means you can use the new shiny async/await syntax from Python 3.5, making your code non-blocking and speedy.

Sanic is developed [on GitHub](https://github.com/channelcat/sanic/). Contributions are welcome!

Sanic aspires to be simple

```python
from sanic import Sanic
from sanic.response import json

app = Sanic()

@app.route("/")
async def test(request):
    return json({"hello": "world"})

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000)
```

---

[官方文档](http://sanic.readthedocs.io/en/latest) · [GitHub](https://github.com/XuToTo/sanic-docs-zh) · [GitBook](http://book.xutoto.im/sanic-docs-zh)

业余翻译，总结自用

如果发现了文档中的错误或是翻译不正确的地方，欢迎 [PR](https://github.com/XuToTo/sanic-docs-zh/pulls) (ﾉ◕ヮ◕)ﾉ*:･ﾟ✧

如果文档没有及时更新请[提交 issue](https://github.com/XuToTo/sanic-docs-zh/issues) (๑•̀ω•́)ノ