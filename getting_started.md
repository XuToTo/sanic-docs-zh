# 快速入门

在开始之前，请确保你已经安装了 [pip](https://pip.pypa.io/en/stable/installing/) 和至少是 3.5 版本的 Python。因为 Sanic 使用了 `async`/`await` 语法，而这些在早期版本的 Python 中是不能正常工作的。

1. 安装 Sanic: `python3 -m pip install sanic`
2. 使用以下代码创建一个 `main.py` 文件:

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

3. 运行服务: `python3 main.py`
4. 在浏览器中打开地址 `http://0.0.0.0:8000`，这时候你应该就能看到 *Hello world!* 了

现在你已经成功运行了一个 Sanic 服务器了！