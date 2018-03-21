# Sanic 中文文档 ![License](https://img.shields.io/badge/license-CC%20BY--NC--SA%204.0-brightgreen.svg) [![Build Status](https://travis-ci.org/XuToTo/sanic-docs-zh.svg?branch=master)](https://travis-ci.org/XuToTo/sanic-docs-zh)

Sanic 是一个类似 Flask 的 Python 3.5 Web 服务框架，在[这篇文章](https://magic.io/blog/uvloop-blazing-fast-python-networking/)的启发下，由一群来自 magicstack 的开发人员完成的，编写出来的目的是为了提供更高的执行效率。

除了有着类似 Flask 风格之外，Sanic 还支持异步请求处理。这就意味着你可以使用 Python 3.5 中新的 `async`/`await` 语法来编写出非阻塞并且执行速度更快的代码。

Sanic [在 GitHub](https://github.com/channelcat/sanic/) 上进行开发。欢迎贡献代码！

Sanic 力求简洁

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

如果发现了文档中的错误或是翻译不正确的地方，欢迎批评指正并[提交 PR](https://github.com/XuToTo/sanic-docs-zh/pulls) (ﾉ◕ヮ◕)ﾉ*:･ﾟ✧

如果文档没有及时更新请[提交 issue](https://github.com/XuToTo/sanic-docs-zh/issues) (๑•̀ω•́)ノ

---

## 许可

> Creative Commons Attribution 4.0 International License (CC BY-NC-SA 4.0)
>
> https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh
