# 网页控制器

本文来自[Odoo 13官方文档之开发者文档](../README.md)系列文章

## 控制器

控制器需要提供扩展性，很像 [`Model`](https://alanhou.org/odoo-13-orm-api/#odoo.models.Model)，但无法使用相同的机制作为预设条件 (带有已加载模块的数据库) 尚不可使用(如未创建或未选择数据库)。

因此控制器具有其自己的扩展机制，与模型的机制相分离：

控制器通过[继承](https://docs.python.org/3/tutorial/classes.html#tut-inheritance) `Controller`来进行创建。 路由通过由 [`route()`](https://alanhou.org/odoo-13-web-controllers/#odoo.http.route)装饰的方法定义：

```
class MyController(odoo.http.Controller):
    @route('/some_url', auth='public')
    def handler(self):
        return stuff()
```

要*重载*控制器，[继承](https://docs.python.org/3/tutorial/classes.html#tut-inheritance)其类并重载相关方法，如果需要重新暴露它们：

```
class Extension(MyController):
    @route()
    def handler(self):
        do_before()
        return super(Extension, self).handler()
```

- 有必要使用 [`route()`](https://alanhou.org/odoo-13-web-controllers/#odoo.http.route) 进行装饰来保持方法（及路由）可见：如方法未进行装饰就重新定义，会变成“未发布”

- 合并所有方法的装饰器，如重载方法的装饰器没有参数则会保留此前的所有参数，所提供的任意参数会覆盖此前所定义参数，例：

  ```
  class Restrict(MyController):
      @route(auth='user')
      def handler(self):
          return super(Restrict, self).handler()
  ```

  会修改对用户公共认证的`/some_url` (要求登录)

## API



### 路由

###### `odoo.http.route(*route=None*, ***kw*)`

装饰器让所装饰的方法成为请求的处理器。该方法必须要是`Controller`子类的一部分。

参数

- **route** – 字符串或数组。决定哪种http请求匹配所装饰方法的路由部分。可以是单个字符串或字符串数组。参见 werkzeug的路由文档了解路由表达式的格式( http://werkzeug.pocoo.org/docs/routing/ )。

- **type** – 请求的类型，可为`'http'` 或 `'json'`.

- auth

   – 认证方法的类型，可为以下类型：

  - `user`: 用户必须认证且当前请求将使用用户的权限进行执行。
  - `public`: 用户可认证也可不认证。如未认证，当前请求会使用共享Public用户进行执行。
  - `none`: 即使用没有数据库，方法也一直是活跃的。主要由框架和认证模块使用。它们的请求代码对访问数据库没有任何作用，也没有表明当前数据库或当前用户的配置。

- **methods** – 一个这个路由所应用的http方法的序列。如未指定，允许使用所有方法。

- **cors** – Access-Control-Allow-Origin cors 指令值。

- **csrf** ([`bool`](https://docs.python.org/3/library/functions.html#bool)) – 是否应为路由启用CSRF保护。默认为 `True`。参见[CSRF保护](https://alanhou.org/odoo-13-web-controllers/#csrf)了解更多内容。

### CSRF保护

版本9.0中新增。

Odoo实现基于令牌的[CSRF保护](https://en.wikipedia.org/wiki/CSRF)。

CSRF保护默认启用并应用于[**RFC 7231**](https://tools.ietf.org/html/rfc7231.html)中所定义的*不安全* HTTP 方法 ( `GET`, `HEAD`, `TRACE` 和 `OPTIONS`之外的所有方法)。

CSRF保护通过名为`csrf_token`值作为请求表单数据的一部分以检查请求来实现。 该值作为验证的一部分从表单中删除并在你自己的表单处理过程中不进行考虑。

在为不安全方法（大多为表单等的POST）新增控制器时：

- 如果表单在Python中生成，csrf令牌借由`request.csrf_token() <odoo.http.WebRequest.csrf_token()`获取，`request`对象默认在QWeb (python)模块中可用，如果你不使用 QWeb它可能会被显式地添加。

- 如果表单在Javascript中生成， CSRF信息默认添加到 QWeb (js) 渲染上下文中作为

  ```
  csrf_token
  ```

   ，否则会在 

  ```
  web.core
  ```

   模块中作为

  ```
  csrf_token
  ```

  :

  ```
  require('web.core').csrf_token
  ```

- 如果端点（endpoint）可由外部(非Odoo内部)调用， 如其为REST API 或一个 [webhook](https://en.wikipedia.org/wiki/Webhook)，必须对端点禁用CSRF保护。如果可能，你可能会希望实现其它验证的方法(来确保它不由不相关的第三方调用)。



### 请求

请求对象在请求的一开始自动对`odoo.http.request` 进行设置

###### `*class* odoo.http.WebRequest(*httprequest*)`

所有Odoo Web请求类型的父类，大多处理初始化及请求对象的设置 (调度本身须由子类处理)

参数

**httprequest** ([`werkzeug.wrappers.BaseRequest`](https://werkzeug.palletsprojects.com/en/0.16.x/wrappers/#werkzeug.wrappers.BaseRequest)) – 封装了的werkzeug Request对象

###### `httprequest`

提供给请求的原始[`werkzeug.wrappers.Request`](https://werkzeug.palletsprojects.com/en/0.16.x/wrappers/#werkzeug.wrappers.Request)对象

###### `params`

请求参数的`Mapping`，通常没有用，因为它们直接提供给handler方法作为关键字参数

###### `cr`

对当前方法调用初始化的`游标` 。

在当前请求使用 `none` 认证时访问游标会报出异常。

###### `context`

针对当前请求上下文值的`映射`

###### `env`

绑定到当前请求的 `环境` 。

###### `session`

`OpenERPSession` 存储针对当前 http 会话的 HTTP session数据

###### `registry`

关联当前请求的数据库的仓库、如当前请求使用`none`认证的话可以为 `None` 。

自版本8.0开始淘汰：使用 [`env`](https://alanhou.org/odoo-13-web-controllers/#odoo.http.WebRequest.env)

###### `db`

关联这一请求的数据库。如当前请求使用`none`认证的话可以为 `None` 。

###### `csrf_token(*time_limit=3600*)`

生成并返回当前会话的CSRF令牌

Parameters

**time_limit** (`int | None`) – CSRF令牌 token 应仅对指定时长（单位为秒）有效 ，默认为1小时，设为 `None` 表示令牌在当前用户的会话结束前有效。

返回

ASCII 令牌字符串

###### `*class* odoo.http.HttpRequest(**args*)`

`http` 请求类型的处理器。

匹配作为关键字参数传递给handler方法的路由参数、查询字符串参数、[表单](http://www.w3.org/TR/html401/interact/forms.html#h-17.13.4.2)参数和文件。

在名称冲突时，路由参数的优先级最高。

handler方法的结果可为：

- 一个假值，些时HTTP响应将为一个 [HTTP 204](http://tools.ietf.org/html/rfc7231#section-6.3.5) (无内容)
- werkzeug响应对象，返回原有内容
- `str` 或 `unicode`，将在响应对象中封装并解析为 HTML

###### `make_response(*data*, *headers=None*, *cookies=None*)`

非HTML对象或带有自定义响应头呀 cookie 的HTML响应的帮助方法。

如未返回非HTML数据，在handler仅能返回他们想要作为字符串发送页面的 HTML标记时，他们需要创建完整的响应对象，否则返回数据将不能由客户端正确地解析。

参数

- **data** (`basestring`) – 响应体
- **headers** (`[(name, value)]`) – 对响应设置的HTTP头
- **cookies** (`collections.Mapping`) – 对客户端设置的cookie

###### `not_found(*description=None*)`

[HTTP 404](http://tools.ietf.org/html/rfc7231#section-6.5.4) (Not Found) 响应的捷方式

###### `render(*template*, *qcontext=None*, *lazy=True*, ***kw*)`

QWeb模板的懒渲染。

给定模块的实际渲染发生在调度的最后。同时，模板及/或qcontext 可由静态响应修改甚至是替换掉。

参数

- **template** (`basestring`) – 等渲染的模板
- **qcontext** ([`dict`](https://docs.python.org/3/library/stdtypes.html#dict)) – 要使用的渲染上下文
- **lazy** ([`bool`](https://docs.python.org/3/library/functions.html#bool)) – 是否应将模板渲染延时到最后一刻
- **kw** – 转发到werkzeug的Response对象

###### `*class* odoo.http.JsonRequest(**args*)`

透过HTTP的[JSON-RPC 2](http://www.jsonrpc.org/specification) 请求handler

- 忽略`method`
- `params`须为一个JSON对象 (并非数组)且传递为handler方法的关键字参数
- handler方法的结果返回为 JSON-RPC `结果`并在 [JSON-RPC Response](http://www.jsonrpc.org/specification#response_object)中进行封装

成功的请求：

```
--> {"jsonrpc": "2.0",
     "method": "call",
     "params": {"context": {},
                "arg1": "val1" },
     "id": null}

<-- {"jsonrpc": "2.0",
     "result": { "res1": "val1" },
     "id": null}
```

产生错误的请求：

```
--> {"jsonrpc": "2.0",
     "method": "call",
     "params": {"context": {},
                "arg1": "val1" },
     "id": null}

<-- {"jsonrpc": "2.0",
     "error": {"code": 1,
               "message": "End user error message.",
               "data": {"code": "codestring",
                        "debug": "traceback" } },
     "id": null}
```

### 响应

###### `*class* odoo.http.Response(**args*, ***kw*)`

通过控制器路由链传递的Response对象。

[`werkzeug.wrappers.Response`](https://werkzeug.palletsprojects.com/en/0.16.x/wrappers/#werkzeug.wrappers.Response) 参数之外，这个类的构造方法可针对QWeb懒渲染接收如下的额外参数。

参数

- **template** (`basestring`) – 等渲染的模板
- **qcontext** ([`dict`](https://docs.python.org/3/library/stdtypes.html#dict)) – 要使用的渲染上下文
- **uid** ([`int`](https://docs.python.org/3/library/functions.html#int)) – 针对ir.ui.view渲染调用所使用的用户 id，`None` 时使用请求的用户 (默认)

这些属性在Response对象中作为参数使用且可在渲染之前的任意时刻进行更改

还暴露[`werkzeug.wrappers.Response`](https://werkzeug.palletsprojects.com/en/0.16.x/wrappers/#werkzeug.wrappers.Response)的所有属性和方法。

###### `render()`

渲染Response的模板，返回结果

###### `flatten()`

强制渲染响应模板，设置结果为响应体并清除`template`