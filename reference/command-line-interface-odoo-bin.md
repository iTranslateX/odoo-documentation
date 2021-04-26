# 命令行接口：odoo-bin

- 本文来自[Odoo 13官方文档之开发者文档](../README.md)系列文章

## 运行服务

###### `-d <database>, --database <database>`

在安装或更新文件时使用的数据库。提供一个逗号分隔列表，列表中提供数据库限制权限。

高级数据库选项请参见 [下方内容](#reference-cmdline-server-database)。

###### `-i <modules>, --init <modules>`

在运行服务前要安装的模块的逗号分隔列表(要求有 [`-d`](#cmdoption-odoo-bin-d)选项)。

###### `-u <modules>, --update <modules>`

在运行服务前要更新的模块的逗号分隔列表(要求有 [`-d`](#cmdoption-odoo-bin-d)选项)。

###### `--addons-path <directories>`

模块所存储的目录的逗号分隔列表。这些目录供模块扫描。

###### `-c <config>, --config <config>`

指供一个替代 [配置文件](#reference-cmdline-config)

###### `-s, --save`

保存服务端配置到当前配置文件中 (默认为`*$HOME*/.odoorc` ，可通过使用 [`-c`](#cmdoption-odoo-bin-c)进行重载)。

###### `--without-demo`

在模块安装时禁止加载演示数据的逗号分隔列表，使用 `all` 来针对所有模块。

###### `--test-enable`

在安装模块之后运行测试

###### `--test-tags 'tag_1,tag_2,...,-tag_n'`

选择通过使用标签运行的测试。

###### `--screenshots`

指定在HttpCase.browser_js测试失败时写入截屏快照的路径。 默认值为`/tmp/odoo_tests/*db_name*/screenshots`

###### `--screencasts`

启用录屏并指定写入录屏文件的路径。需要安装 `ffmpeg`工具来将各帧编码入视频文件中。否则将保存帧而不是视频文件。

`1`, `t` 或 `true` 可用于使用同一目录作为上述的 `--screenshots` 选项。



### 数据库

###### `-r <user>, --db_user <user>`

数据库用户名，用于连接PostgreSQL.

###### `-w <password>, --db_password <password>`

若使用了 [密码验证](https://www.postgresql.org/docs/9.3/static/auth-methods.html#AUTH-PASSWORD)时，为数据库密码。

###### `--db_host <hostname>`

数据库服务端的主机

- Windows上的`localhost`
- 否则为UNIX套接字

###### `--db_port <port>`

数据库监听的端口，默认为5432

###### `--db-filter <filter>`

隐藏不匹配`<filter>`的数据库。过滤器是一个[正则表达式](https://docs.python.org/3/library/re.html)，额外之处有：

- `%h` 由所进行请求的完整主机名替换。
- `%d`由所进行请求的子域名进行替换，`www` 除外(因此域名`odoo.com` 和 `www.odoo.com` 同时匹配数据库`odoo`)。这些运算是区分大小写的。添加选项 `(?i)` 来匹配所有数据库(因此域名 `odoo.com` 使用 `(?i)%d` 匹配数据库 `Odoo`).

自版本11开始，还可以通过使用–database参数并指定一个逗号分隔的数据库列表来限定对指定数据库的访问

在合并这两个参数时，db-filter替代限定数据库列表的逗号分隔列表，而逗号分隔列表用于执行像升级模块这类请求运算。

```
$ odoo-bin --db-filter ^11.*$
```

限制对名称以11开头的数据库的访问

```
$ odoo-bin --database 11firstdatabase,11seconddatabase
```

限定仅访问两个数据库， 11firstdatabase 和 11seconddatabase

```
$ odoo-bin --database 11firstdatabase,11seconddatabase -u base
```

限定仅访问两个数据库，11firstdatabase 和 11seconddatabase，并升级一个数据库中的 base模块：11firstdatabase。如果数据库11seconddatabase 不存在，会创建数据库并安装base模块

```
$ odoo-bin --db-filter ^11.*$ --database 11firstdatabase,11seconddatabase -u base
```

限定对名称以11开头的数据库的访问，并对一个数据库的 base模块进行升级：11firstdatabase。如数据库11seconddatabase 不存在，会创建该数据库并安装 base 模块

###### `--db-template <template>`

在通过数据库管理界面新建数据库时，使用指定的[模板数据库](https://www.postgresql.org/docs/9.3/static/manage-ag-templatedbs.html)。默认为`template0`。

###### `--pg_path </path/to/postgresql/binaries>`

由数据库管理器用于导出及还原数据库的PostgreSQL二进制的路径。应仅在这些二进制文件位于非标准目录内时才进行指定。

###### `--no-database-list`

限制在系统中列出可用数据库列表的功能

###### `--db_sslmode`

控制在Odoo 和 PostgreSQL之间连接的SSL安全。值应为‘disable’, ‘allow’, ‘prefer’, ‘require’, ‘verify-ca’ 或 ‘verify-full’之一。默认值为‘prefer’



### Email

###### `--email-from <address>`

在Odoo需要发送邮件时用作<FROM>的Email地址

###### `--smtp <server>`

连接来发送邮件的SMTP服务器地址

###### `--smtp-port <port>`

###### `--smtp-ssl`

若进行了设置，odoo 应使用 SSL/STARTSSL SMTP 连接

###### `--smtp-user <name>`

连接到SMTP服务器的用户名

###### `--smtp-password <password>`

连接到SMTP服务器的密码



### 国际化

使用这些选项来翻译Odoo为另一咱语言。参见用户手册的 i18n 版块。选项‘-d’是必传的。在导入时选项‘-l’是必传的

###### `--load-language <languages>`

指定希望加载翻译的语言 (由逗号分隔)

###### `-l, --language <language>`

指定翻译文件的语言。通过–i18n-export 或 –i18n-import来进行使用

###### `--i18n-export <filename>`

导出所有要翻译的句子到一个CSV文件、一个PO文件或一个TGZ 存档中并退出。

###### `--i18n-import <filename>`

导入一个带有翻译的CSV 或 PO文件并退出。 要求使用 ‘-l’选项。

###### `--i18n-overwritec`

对更新的模块重写已有翻译词汇或导入一个CSV 或 PO 文件

###### `--modules`

指定导出的模块。与 –i18n-export一并使用



### 高级选项



#### 开发者功能

###### `--dev <feature,feature,...,feature>`

- `all`: 激活下面所有的功能
- `xml`: 从xml文件而非数据库直接读取模板。一旦在数据库中修改了模板，在下一次更新/初始化之前它将不会从 xml 中进行读取。
- `reload`: 在python文件更新时重启服务端 (根据所使用的文本编辑器可能不会被监测到)
- `qweb`: 在节点包含 `t-debug='debugger'`时qweb模板执行的断点
- `(i)p(u)db`: 在日志和返回错误前抛出预期外错误时在代码中开启所选中的python调试器。



#### HTTP

###### `--no-http`

不启动HTTP 或 长轮询 worker (可能仍会启动[cron](https://alanhou.org/odoo-13-actions/#reference-actions-cron) worker)

### ⚠️警告

若设置了[`--test-enable`](#cmdoption-odoo-bin-test-enable) 时没有效果，因为测试要求可访问HTTP 服务

###### `--http-interface <interface>`

HTTP服务监听的TCP/IP地址，默认为 `0.0.0.0` (所有地址)

###### `--http-port <port>`

HTTP服务监听的端口，默认为 8069。

###### `--longpolling-port <port>`

在多进程或gevent模式下长轮询连接的TCP端口，默认为8072。在默认（线程）模式下不进行使用。

###### `--proxy-mode`

启用通过 [Werkzeug代理支持](http://werkzeug.pocoo.org/docs/contrib/fixers/#werkzeug.contrib.fixers.ProxyFix)的`X-Forwarded-*`头部的使用。

### ⚠️警告

在反向代理场景之外**不可**启动代理模式



#### 日志

默认Odoo显示除工作流日志(仅为`warning` )以外的所有 `info` [级别](https://docs.python.org/3/library/logging.html#logging.Logger.setLevel) 的日志，并且输出被发送到`stdout`。也有重定向日志到其它目的地及自定义日志输出的量的各种选项可以使用。

###### `--logfile <file>`

发送日志输出到指定的文件而非stdout。在Unix中，文件 [可由外部日志轮转程序管理](https://docs.python.org/3/library/logging.handlers.html#watchedfilehandler)并在替换时自动被重新打开=

###### `--syslog`

记录到系统的事件记录器中： [Unix 系统中的syslog](https://docs.python.org/3/library/logging.handlers.html#sysloghandler)及 [Windowsk 的事件日志](https://docs.python.org/3/library/logging.handlers.html#nteventloghandler)。

都不可进行配置。

###### `--log-db <dbname>`

记录到指定数据库的 `ir.logging`模型 (`ir_logging` 数据表) 中。数据库可以是在“当前”PostgreSQL中的数据库名称或一个针对日志聚合等的 [PostgreSQL URI](https://www.postgresql.org/docs/9.2/static/libpq-connect.html#AEN38208)。

###### `--log-handler <handler-spec>`

`*LOGGER*:*LEVEL*`，在所提供的 `LEVEL` 中启用`LOGGER`，如`odoo.models:DEBUG` 会启用所有模型中`DEBUG`及以上级别的所有日志消息。

- 冒号 `:` 为必选
- 可省略日志器来配置根 (默认)处理器
- 如果省略了日志级别，日志设置为 `INFO`级别

可重复选项来配置多个日志器，如

```
$ odoo-bin --log-handler :DEBUG --log-handler werkzeug:CRITICAL --log-handler odoo.fields:WARNING
```

###### `--log-request`

对RPC请求启用DEBUG日志，等价于 `--log-handler=odoo.http.rpc.request:DEBUG`

###### `--log-response`

对RPC响应启用DEBUG日志，等价于 `--log-handler=odoo.http.rpc.response:DEBUG`

###### `--log-web`

启用HTTP请求和响应的 DEBUG日志，等价于 `--log-handler=odoo.http:DEBUG`

###### `--log-sql`

启用SQL查询的DEBUG日志，等价于 `--log-handler=odoo.sql_db:DEBUG`

###### `--log-level <level>`

对指定日志器更易于设置预定义级别的快捷方式。“真实” 级别 (`critical`, `error`, `warn`, `debug`) 在`odoo` 和 `werkzeug`日志器中进行设置loggers (除了仅对`odoo`设置的`debug` )。

Odoo还提供应用于不同日志设置的调试伪日志级别：

- `debug_sql`

  设置SQL日志器为 `debug`等价于 `--log-sql`

- `debug_rpc`

  设置 `odoo` 及HTTP请求日志器为 `debug`等价于`--log-level debug --log-request`

- `debug_rpc_answer`

  设置 `odoo` 和 HTTP 请求及响应日志器为 `debug`等价于 `--log-level debug --log-request --log-response`

在[`--log-level`](#cmdoption-odoo-bin-log-level) 和 [`--log-handler`](#cmdoption-odoo-bin-log-handler)产生冲突时， 使用后者



#### 多进程

###### `--workers <count>`

若`count` 不为0 (默认值)，启用多进程并设置指定的HTTP worker数量(子进程处理 HTTP 及 RPC 请求)。

多进程模式仅在基于Unix的系统中可以使用

一些选项允许限制及回收worker：

###### `--limit-request <limit>`

在回收和重启前将处理的worker请求数。

默认值为*8196*。

###### `--limit-memory-soft <limit>`

每个worker所允许使用的最大虚拟内存。如超出限制，会在杀死worker并在当前请求结束时回收它。

默认值为*2048MiB*.

###### `--limit-memory-hard <limit>`

对虚拟内存的硬性限制，任何超过这一限制的worker都会立即被杀死，不会等待当前请求进程的结束。

默认值为 *2560MiB*。

###### `--limit-time-cpu <limit>`

防止 worker每次请求使用超出<limit> CPU 秒数。若超出了限制，会杀死该worker。

默认值为*60*。

###### `--limit-time-real <limit>`

防止worker处理请求时超出 <limit>秒。如超出限制，会杀死该worker。

不同于[`--limit-time-cpu`](#cmdoption-odoo-bin-limit-time-cpu) ，这是包含SQL查询在内的“wall time”（绝对时间） 限制。

默认值为 *120*。

###### `--max-cron-threads <count>`

用于 [cron](https://alanhou.org/odoo-13-actions/#reference-actions-cron)任务的worker数。默认值为*2*。 worker是多线程模式中的线程，在多进程模式中进行处理。

对于多进程模式，这是对HTTP worker进程的补充。



## 配置文件

大部分命令行选项也可通过配置文件进行指定。大部分时候，它们使用去除了`-`前缀的相似名称，而其它`-` 由 `_` 进行替换，如 [`--db-template`](#cmdoption-odoo-bin-db-template) 变为 `db_template`.

一些转换与该模式不相匹配：

- [`--db-filter`](#cmdoption-odoo-bin-db-filter) 变为 `dbfilter`
- [`--no-http`](#cmdoption-odoo-bin-no-http) 对应 `http_enable` 的布尔值
- 日志预设(所有带有`--log-` 但除[`--log-handler`](#cmdoption-odoo-bin-log-handler) 和 [`--log-db`](#cmdoption-odoo-bin-log-db)以外的选项) 只会将内容添加到 `log_handler`中，在配置文件中直接使用它
- [`--smtp`](#cmdoption-odoo-bin-smtp) 存储为 `smtp_server`
- [`--database`](#cmdoption-odoo-bin-d)存储为 `db_name`
- [`--i18n-import`](#cmdoption-odoo-bin-i18n-import) 及 [`--i18n-export`](#cmdoption-odoo-bin-i18n-export) 在配置文件中完全不可用

默认配置文件是 `*$HOME*/.odoorc` ，它可使用[`--config`](#cmdoption-odoo-bin-c)进行重置。指定 [`--save`](#cmdoption-odoo-bin-s) 会保存回当前配置状态到该文件中。

## Shell

Odoo命令行还允许以python终端环境启动odoo。这会启用与[orm](https://alanhou.org/odoo-13-orm-api/#reference-orm)及其功能的直接交互。

```
$ odoo_bin shell
```

###### `--shell-interface (ipython|ptpython|bpython|python)`

指定在shell模式下使用推荐的 REPL。



## 脚手架

脚手架是简化初始搭建的（在Odoo中为模块）自动化框架结构的创建。 虽然不是必须，但它避免了一些配置基础结构及查看启动要求的无聊操作。

脚本架可通过 **odoo-bin scaffold** 子命令进行使用。

###### `name (required)`

要创建的模块名，可以不同方式生成程序名称 (如模块目录名、模型名称 …)

###### `destination (default=current directory)`

新建模块的目录，默认为当前目录

###### `-t <template>`

一个模板目录，文件通过[jinja2](http://jinja.pocoo.org/) 传递然后拷贝到 `destination`目录中

```
$ odoo_bin scaffold my_module /addons/
```

这将在 */addons/*中创建*my_module*模块。