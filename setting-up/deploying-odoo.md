# 部署Odoo

本文来自[Odoo 13官方文档之开发者文档](../README.md)系列文章

本文档描述在生产或面向因特网的环境下配置Odoo的步骤。它紧承[安装](https://alanhou.org/odoo-13-installing-odoo/)一节，对于不暴露到互联网的开发系统并不是很有必要。

### ⚠️警告

如果在建立对外服务，确保查看 [安全权限](#security)中的建议！



## dbfilter

Odoo是一套多租户系统：单个 Odoo系统可在多个服务库实际上运行并提供服务。它也是高度可定制的，自定义内容（通过所加载的模块开始）可在“当前数据库”中实现。

这对于以公司用户在后台（客户端）登录时并不会成为问题：数据库可在登录时进行选择，并在之后加载自定义内容。

但对于未登录的用户（门户、网站）则会形成问题，他们未与数据库进行绑定：Odoo需要知道应当用于加载网页或执行操作的数据库。未使用多租户不会构成问题，这时仅有一个数据库，但如果存在多个数据库时， Odoo需要知道该使用哪个数据库的规则。

这也是使用 [`--db-filter`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-db-filter)的目的之一：它指定应如何根据所请求的主机名（域名）来选择数据。其值是一个 [正则表达式](https://docs.python.org/3/library/re.html)，可能包含动态注入的用于访问系统的主机名 (`%h`) 或首个子域名 (`%d`) 。

对于在生产环境托管多个数据库的服务器，尤其是那些使用了 `website` 模块的，**必须**要设置dbfilter，否则一些功能将无法正确运行。

### 配置样例

- 显示仅以‘mycompany’名称开头的数据库

在 `/etc/odoo.conf` 中设置：

```
[options] 
dbfilter = ^mycompany.*$
```

- 仅显示匹配 `www`之后首个子域名的数据库：例如如果请求是发送到 `www.mycompany.com` 或 `mycompany.co.uk`而非`www2.mycompany.com` 或 `helpdesk.mycompany.com`的话则会显示“mycompany”数据库

在 `/etc/odoo.conf` 中设置：

```
[options] 
dbfilter = ^%d$
```

设置适当的 [`--db-filter`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-db-filter) 是确保部署正常的一个重要部分。一旦正确运行且每个主机名仅匹配一个数据库的话，强烈推荐限制对数据库管理器界面的访问，可以使用 `--no-database-list` 启动参数在防止列出数据库，并阻止对数据库管理界面的访问，参见 [权限安全](#security)。

## PostgreSQL

默认，PostgreSQL仅允许通过UNIX套接字及回环地址（PostgreSQL服务所安装的同一台机器，使用localhost）连接来进行连接。

如果Odoo和PostgreSQL在同一台机器上执行的话UNIX套接字是没有问题的，并且也是在未提供主机参数时的默认值，但如果希望Odoo 和 PostgreSQL在不同机器上进行执行 [1](#different-machines)，它就需要通过之一[监听网卡](https://www.postgresql.org/docs/9.6/static/runtime-config-connection.html) [2](#remote-socket)：

- 仅接受回环连接并在Odoo和PostgreSQL所运行的机器之间[使用一条SSH通道](https://www.postgresql.org/docs/9.6/static/ssh-tunnels.html)，然后配置Odoo来连接通道的另一端
- 接受来自Odoo所安装的机器的连接，可能是通过ssl (参见 [PostgreSQL连接设置](https://www.postgresql.org/docs/9.6/static/runtime-config-connection.html)了解详情)，然后配置Odoo来在网络上进行连接

### 配置样例

- 允许在localhost上的tcp连接
- 允许来自网络上192.168.1.x 的tcp连接

在`/etc/postgresql/9.5/main/pg_hba.conf` 中设置：

```
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
host    all             all             192.168.1.0/24          md5
```

在 `/etc/postgresql/9.5/main/postgresql.conf` 中设置：

```
listen_addresses = 'localhost,192.168.1.2'
port = 5432
max_connections = 80
```



### 配置Odoo

默认，Odoo在5432端口上通过UNIX套接字连接本地postgres。这对Postgres不在本地部署Postgres或软件不使用默认配置时可以使用[数据库选项](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#reference-cmdline-server-database)进行重置。[安装包安装](https://alanhou.org/odoo-13-installing-odoo/#setup-install-packaged)时会自动新建用户 (`odoo`) 并将其设置为数据库用户。

- 数据库管理界面由 `admin_passwd` 设置进行保护。这一设置仅能通过配置文件进行设置，并且仅在执行数据库修改前进行检查。可以设置为一个随机生成的值来确保第三方不会使用到这个界面。

- 所有的数据库操作使用 [数据库选项](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#reference-cmdline-server-database)，包括数据库管理界面。要让数据库管理界面运行，要求 PostgreSQL 用户拥有 `createdb` 权限。

- 用户总是可以删除他们所拥有的数据库。要让数据库管理界面完成不可使用，PostgreSQL用户需要通过

  ```
  no-createdb
  ```

  创建数据库，并且该数据库必须要为另一个PostgreSQL 用户所持有。

  ### ⚠️警告

  该PostgreSQL用户*必须*不能是超级用户

#### 配置样例

- 连接192.168.1.2上的PostgreSQL服务
- 端口为 5432
- 使用 ‘odoo’ 用户
- 通过‘pwd’ 来作为密码
- 过滤出以‘mycompany’名称开头的数据库

在 `/etc/odoo.conf` 中设置：

```
[options]
admin_passwd = mysupersecretpassword
db_host = 192.168.1.2
db_port = 5432
db_user = odoo
db_password = pwd
dbfilter = ^mycompany.*$
```



### Odoo和PostgreSQL之间的SSL

自从Odoo 11.0开始，可以强制在Odoo 和 PostgreSQL之间使用ssl连接，在 Odoo中， db_sslmode控制着ssl安全连接，值有‘disable’, ‘allow’, ‘prefer’, ‘require’, ‘verify-ca’ 或 ‘verify-full’

[PostgreSQL 文档](https://www.postgresql.org/docs/current/static/libpq-ssl.html)



## 内置服务端

Odoo包含一个内置的HTTP服务端，可以使用多线程或多进程。

在生产使用中，推荐使用多进程服务端，因其增强了稳定性，让其更好的利用了计算资源并可以更好地进行监控和资源管控。

- 多进程通过配置 [`worker进程数据为非0的数字`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-workers)来启用， worker的数量应根据服务器的核数来计算(可能要根据预期有多少个cron任务来为cron的worker预留空间)
- Worker的限定可根据硬件配置来设置，这样才能避免资源耗尽

### ⚠️警告

多进程模式当前在Windows中尚无法使用

### Worker数量计算

- 基本准备则 : (#CPU * 2) + 1
- Cron worker需要使用CPU
- 1 worker ~= 6个并发用户

### 内存大小的计算

- 我们认为20%的请求是复杂的重度请求，而80%是简单请求
- 重度worker，在对所有的计算字段及SQL请求...都进行了良好设计时， 可能会消耗约1G的内存
- 轻量的worker，在相同的场景下，预计会消耗约150MB 的 RAM

所需RAM = #worker * ( (light_worker_ratio * light_worker_ram_estimation) + (heavy_worker_ratio * heavy_worker_ram_estimation) )

### LiveChat

在多进程中，会自动启动一个独立的LiveChat工作进程并监听[`长轮询端口`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-longpolling-port) 但客户端不会对其进行连接。

而你必须要有一个将以 `/longpolling/` 开头的URL请求重定向到长轮询端口的代理。其它的请求应当代理至[`普通HTTP端口`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-http-port)。

要实现这点，你需要在Odoo之前部署一个反向代理 ，比如Nginx或Apache。这么做时，需要转发更多的http头给Odoo，并在Odoo中激活proxy_mode来让 Odoo读取这些头部信息。

### 配置样例

- 4 CPU, 8 线的服务器
- 60个并发用户
- 60 个用户 / 6 = 10 <- 理论上所需的worker数量
- (4 * 2) + 1 = 9 <-理论最大worker数
- 我们将使用8个worker + 1个用于cron。我们还将使用监控系统来测量cpu负载，并检查其是否在 7 和 7.5之间。
- RAM = 9 * ((0.8*150) + (0.2*1024)) ~= 3个针对Odoo的Go RAM

`/etc/odoo.conf`中的配置：

```
[options]
limit_memory_hard = 1677721600
limit_memory_soft = 629145600
limit_request = 8192
limit_time_cpu = 600
limit_time_real = 1200
max_cron_threads = 1
workers = 8
```



## HTTPS

不论是通过网站/网页客户端还是网页服务来进行访问，Odoo都以明文传输认证信息。这表示Odoo的安全部署必须使用 HTTPS[3 ](#switching)。SSL终端可通过任意SSL终端代理来实现，但要求进行如下设置：

- 启用Odoo的[`代理模式`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-proxy-mode)。这应当仅在Odoo位于反向代理之后时启用
- 设置SSL终端代理([Nginx终端示例](https://nginx.com/resources/admin-guide/nginx-ssl-termination/))
- 设置代理本身 ([Nginx代理示例](https://nginx.com/resources/admin-guide/reverse-proxy/))
- 你的SSL终端代理也就自动重定向非安全连接到安全端口上

### ⚠️警告

万一你使用 [POSBox](https://www.odoo.com/page/point-of-sale-hardware#part_2)来配合销售点（POS）模块，则必须对`/pos/web` 路由禁用HTTPS配置，以避免混合内容的错误。

### 配置样例

- 重定向http请求到 https
- 重定向请求至odoo

在 `/etc/odoo.conf`中设置：

```
proxy_mode = True
```

在 `/etc/nginx/sites-enabled/odoo.conf` 中设置：

```
#odoo server
upstream odoo {
 server 127.0.0.1:8069;
}
upstream odoochat {
 server 127.0.0.1:8072;
}

# http -> https
server {
   listen 80;
   server_name odoo.mycompany.com;
   rewrite ^(.*) https://$host$1 permanent;
}

server {
 listen 443;
 server_name odoo.mycompany.com;
 proxy_read_timeout 720s;
 proxy_connect_timeout 720s;
 proxy_send_timeout 720s;

 # Add Headers for odoo proxy mode
 proxy_set_header X-Forwarded-Host $host;
 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
 proxy_set_header X-Forwarded-Proto $scheme;
 proxy_set_header X-Real-IP $remote_addr;

 # SSL parameters
 ssl on;
 ssl_certificate /etc/ssl/nginx/server.crt;
 ssl_certificate_key /etc/ssl/nginx/server.key;
 ssl_session_timeout 30m;
 ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
 ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
 ssl_prefer_server_ciphers on;

 # log
 access_log /var/log/nginx/odoo.access.log;
 error_log /var/log/nginx/odoo.error.log;

 # Redirect longpoll requests to odoo longpolling port
 location /longpolling {
 proxy_pass http://odoochat;
 }

 # Redirect requests to odoo backend server
 location / {
   proxy_redirect off;
   proxy_pass http://odoo;
 }

 # common gzip
 gzip_types text/css text/scss text/plain text/xml application/xml application/json application/javascript;
 gzip on;
}
```

## 运行 Odoo为WSGI应用

也可以将Odoo挂载为标准的 [WSGI](https://wsgi.readthedocs.org/) 应用。Odoo 通过`odoo-wsgi.example.py`提供了WSGI的基础启动脚本。该脚本应当进行自定义 (可能是将其从配置目录拷贝出来)以在 `odoo.tools.config`中进行正确的配置，而不是通过命令行或配置文件配置。

WSGI服务仅向网页客户端、网站及网页服务 API暴露主要的HTTP 端点。因Odoo不再控制worker的创建，它法再设置cron或 livechat工作进程。

### Cron工作进程

要将Odoo部署的cron任务运行为WSGI应用，要求：

- 经典的Odoo ( 通过`odoo-bin`运行)
- 连接到cron所要运行的数据库 (通过 [`odoo-bin -d`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-d))
- 不应暴露到网络中。要确保cron运行器不能通过网络访问，可以通过 [`odoo-bin --no-http`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-no-http) 或在配置文件中设置 `http_enable = False` 完全禁用内置HTTP服务

### LiveChat

第二个 WSGI部署会存在问题的子系统是LiveChat：这里大部分 HTTP连接是相对较短的，并且为下一个请求快速清理worker进程， LiveChat要求对每个客户端的长连接以实现接近补时的通知。

这与基于进程的worker模型是相悖的，因为它会绑住 worker进程并阻止新用户访问系统。但时，这些长连接所做的事甚少，大多数时间停在那里等待通知。

在WSGI中支持livechat/通知的方案是：

- 部署一个Odoo的线程版本 (代替基于进程的预创建版本)并仅将以 `/longpolling/` 开头的 URL 重定向到Odoo，这是最简单的并且长轮询URL会叠加成cron实例。
- 通过 `odoo-gevent` 部署带事件Odoo并代理以 `/longpolling/`开头的请求至 [`长轮询端口`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-longpolling-port)。

## 静态文件服务

为开发方便，Odoo直接在模块中提供静态文件服务。这在性能方面可能并不算理想，因为静态文件通常会通过静态HTTP服务器来提供服务。

Odoo静态文件位于每个模块的 `static/` 文件夹中，因此静态文件可通过将所有请求拦截至 `/*MODULE*/static/*FILE*`来进行服务的提供，并在各个addon路径中查找正确的模块（及文件）。



## 安全权限

对于入门人员，请记住保证信息系统的安全是一个持之以恒的过程，没有一劳永逸的法门。此时，你的安全状况和环境中最脆弱的链接如出一辙。

因此不要把本部分看作能够防止所有安全问题的终极宝典。它仅是对在安全动作计划中首要需做的事情的总结。其余的自于你的操作系统及发行版的最佳安全实践，这些最佳实践有用户、密码及访问控制管理等。

在部署面向互联网的服务时，应确保考虑如下安全相关的主题：

- 保持设置一个复杂的超级管理员admin的密码，并在建立系统伊始就限制对数据库管理页面的访问。参见 [数据库管理器权限](#db-manager-security)。
- 为所有的数据库管理员账户选择独立的登录名及复杂密码。不要使用‘admin’作为登录名。 不要使用这些登录名进行日常运维，仅在控制/管理软件时进行使用。*永远不要*使用默认密码，如admin/admin，即使在测试/上线前数据库中也不要使用。
- **不要**在面向互联网的服务中安装 demo数据。带有演示数据的数据库包含默认登录名和密码，即使在上线前/dev 系统中也可能会用于进入你的系统并产生巨大的麻烦。
- 使用适当的数据库过滤器( [`--db-filter`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-db-filter)) 来根据主机名限制你的数据识图的可见性。参见[dbfilter](#db-filter)。你也可以使用[`-d`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-d) 来提供自己的 (逗号分隔) 列表来用于过滤可用的数据库，而不是让系统获取数据库后台中的所有数据库。
- 一旦配置了 `db_name` 和 `db_filter` 并仅对每个主机名匹配单个数据库，应当设置 `list_db` 配置项为 `False`来防止整个列出所有气数库，并阻止对数据库管理界面的访问(也可使用 [`--no-database-list`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-no-database-list) 命名行选项进行实现)
- 确保PostgreSQL用户 ([`--db_user`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-r)) 不是超级用户，并且数据库为另一个用户所持有。例如，如果你在使用独立非特权的`db_user`时可能会由 `postgres` 超级用户所拥有。参见 [配置Odoo](#setup-deploy-odoo)。
- 通过 GitHub或从 https://www.odoo.com/page/download 及[http://nightly.odoo.com](http://nightly.odoo.com/)下载最新版本定期安装最新构建在保持软件的更新。
- 使用多进程模式配置服务端，做与你的日常使用相匹配的适当限制 (内存y/CPU/超时)。参见 [内置服务器](#builtin-server)。
- 使用有效的 SSL 证书在提供 HTTPS 终端的网页服务端背后台运行Odoo，以防止对纯文本通讯的窃听。SSL证书很便宜，并且还存在很多免费的证书。配置web 代理来限制请求的大小、设置适当的超时，然后启用[`proxy mode`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-proxy-mode) （代理模式）选项。参见 [HTTPS](#https-proxy)。
- 如果需要允许对服务端的远程 SSH访问，确保对**所有**账户而非`root`仅仅设置复杂密码。强烈推荐完全禁止基于密码的验证，而只使用公钥验证。同时考虑限制通过VPN的访问，仅允许防火墙中受信 IP 的访问，可同时运行诸如 `fail2ban` 或相似的暴力破解监测系统。
- 考虑在代理或防火墙上实施相应的频率限制，以防止暴力破解攻击及DoS（拒绝服务）攻击。参见 [防止暴力攻击](#login-brute-force) 获取更多具体的方法。很多网络提供商提供对DDoS（分布式拒绝服务攻击）的自动防护，但这通常是可选的服务，请进行咨询了解更多详情。
- 只有条件允许，请将面向公众的 demo/test/staging 实例放在与生产实际不同的机器上。并应用与生产相同的安全防护。
- 如果托管着多个客户，使用容器或相应的“jail”技术来相互隔离用户数据及文件。
- 设置对数据库和文件存储数据的日常备份，并将它们拷贝到服务端自身无法访问到的远程存储服务器。



### 防止暴力攻击

对于面向因特网的部署，针对用户密码的暴力破解攻击很常见，这一威胁在 Odoo 服务端也不应忽视。在尝试执行登录时Odoo发送一条日志记录，并报告结果：成功或失败，以及目标登录名和源 IP。

日志记录有如下的形式：

登录失败：

```
2018-07-05 14:56:31,506 24849 INFO db_name odoo.addons.base.res.res_users: Login failed for db:db_name login:admin from 127.0.0.1
```

登录成功：

```
2018-07-05 14:56:31,506 24849 INFO db_name odoo.addons.base.res.res_users: Login successful for db:db_name login:admin from 127.0.0.1
```

这些日志可轻易地由 `fail2ban`等入侵防护系统进行分析。

例如，如下的 fail2ban过滤器定义应当会匹配一次登录失败：

```
[Definition]
failregex = ^ \d+ INFO \S+ \S+ Login failed for db:\S+ login:\S+ from <HOST>
ignoreregex =
```

这可用于 jail定义来禁用对 HTTP(S)进行攻击的IP。

以下是在监测到1分钟内有10次登录尝试失败时禁用该 IP 15分钟的示例：

```
[odoo-login]
enabled = true
port = http,https
bantime = 900  ; 15 min ban
maxretry = 10  ; if 10 attempts
findtime = 60  ; within 1 min  /!\ Should be adjusted with the TZ offset
logpath = /var/log/odoo.log  ;  set the actual odoo log path here
```



### 数据库管理器权限

[ 配置Odoo](#setup-deploy-odoo) 中提到了传递 `admin_passwd` 。

这一设置用于所有的数据库管理界面 (创建、删除、导出或还原数据库)。

如果必须禁止所有的管理界面， 应当设置 `list_db` 配置选项为`False`，以阻止对所有数据库选择及管理界面的访问。

### ⚠️警告

强烈建议禁用面向因特网系统中的数据库管理器。它是用作开发/演示的工作，以方便快速创建及管理数据库。并不是设计用于生产环境中的，可能会向攻击者暴露一些危险的功能。它也不是设计用来处理大量数据库的，并可能会触发内存极限。

在生产系统中，数据库管理操作应保持由系统管理员执行，包括配置新数据库及自动化备份。

确保配置一个适当的`db_name` 参数 (可选项还有`db_filter`)，这样系统可以决定每个请求的目标数据库，否则会因被禁止自己选择数据库而无法访问。

如果管理界面必须仅能由一部分机器访问，使用代理服务器的功能来阻止所有以`/web/database`开头的路由的访问，但（可能）不包含用于显示数据库选择界面的`/web/database/selector` 。

如果数据库管理界面应保持可供访问，`admin_passwd` 设置必须由其 `admin` 默认值进行修改：这个密码在允许进行数据库修改操作前会进行检查。

它应进行安全的存储，并应是随机生成的，例如：

```
$ python3 -c 'import base64, os; print(base64.b64encode(os.urandom(24)))'
```

这会在成一个32个字符的伪随机可打印字符串。

## 所支持的浏览器

Odoo支持多个版本的多种浏览器。在浏览器版本间保持最新并没有什么不同。Odoo在当前浏览器版本中都支持。所支持的浏览器列表如下：

- IE11,
- Mozilla Firefox,
- Google Chrome,
- Safari,
- Microsoft Edge

[[1\]](#id1) 让多个Odoo软件使用同一个PostgreSQL数据库，或对这两个服务提供更多的计算资源。

[[2\]](https://alanhou.org/odoo-13-deploying-odoo/#id2) 技术上存在一个 [socat](http://www.dest-unreach.org/socat/)这样的工具，可用于在网络中代理UNIX套接字，但那多针对仅通过UNIX套接字才能使用的软件

[[3\]](#id6) 或通过任何内部包交换的网络进行访问，但那会要求安全交换、对 [ARP攻击](https://en.wikipedia.org/wiki/ARP_spoofing)的保护以及防止使用WiFi。哪怕是通过安全的包交换网络，也推荐使用HTTPS进行部署，可能产生的费用可通过“自签署”证书来降低，并且自控的网络中要比在互联网上的部署更为简单。