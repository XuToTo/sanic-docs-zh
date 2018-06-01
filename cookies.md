# Cookies

Cookies 是一些列持久存储在用户浏览器内的数据。Sanic 可以读写以键值对形式存储的 cookies。


> 注意：
>
> Cookies 是可以被客户端随意更改的。因此你不能只是存储这些数据，特别是一些 cookies 中的登录信息，因为它们可以被客户端随意更改。所以，为了确保你存储在 cookies 中的数据不会被客户端伪造或篡改，应该利用一些像是 [itsdangerous](https://pythonhosted.org/itsdangerous/) 的工具来对数据进行加密签名。

## 读取 cookies

用户的 cookies 可以通过 `Request` 对象中的 `cookies` 字典获取。

```python
from sanic.response import text

@app.route("/cookie")
async def test(request):
    test_cookie = request.cookies.get('test')
    return text("Test cookie set to: {}".format(test_cookie))
```

---

## 写入 cookies

在返回一个响应时，可以在 `Response` 对象中设置 cookies。

```python
from sanic.response import text

@app.route("/cookie")
async def test(request):
    response = text("There's a cookie up in this response")
    response.cookies['test'] = 'It worked!'
    response.cookies['test']['domain'] = '.gotta-go-fast.com'
    response.cookies['test']['httponly'] = True
    return response
```

---

## 删除 cookies

Cookies 可以通过语义的形式（译注：设置 `max-age`）或显式地移除。

```python
from sanic.response import text

@app.route("/cookie")
async def test(request):
    response = text("Time to eat some cookies muahaha")

    # 这个 cookie 将会在 0 秒内到期
    del response.cookies['kill_me']

    # 这个 cookie 将会在 5 秒后自己销毁
    response.cookies['short_life'] = 'Glad to be here'
    response.cookies['short_life']['max-age'] = 5
    del response.cookies['favorite_color']

    # 这个 cookie 将会保持不变
    response.cookies['favorite_color'] = 'blue'
    response.cookies['favorite_color'] = 'pink'
    del response.cookies['favorite_color']

    return response
```

响应中的 cookies 可以像设置字典的值一样设置下面这些参数：

- `expires` (datetime)：cookie 在客户端浏览器中过期的时间。
- `path` (string): 可以应用 cookie 的 URL 子集。默认是 `/`。
- `comment` (string)：备注（元数据）。
- `domain` (string)：指定 cookie 生效的域名。一个显式指定的域名必须总是要以一个点开头。
- `max-age` (number)：cookie 的存活秒数。
- `secure` (boolean)：指定 cookie 是否只能通过 HTTPS 发送。
- `httponly` (boolean): 指定 cookie 是否可以被 JavaScript 读取。