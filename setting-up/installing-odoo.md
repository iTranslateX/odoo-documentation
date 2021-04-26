# 安装Odoo

安装 Odoo 有很多种方式，或者是根本无需进行安装，这取决于你的用例。

本文旨在描述大部分的安装选项。

- [在线使用](#setup-install-online)

  在生产环境或测试使用Odoo最简便的方式。

- [安装包](#setup-install-packaged)

  适于测试Odoo、开发模块及用于通过额外的部署和维护工作来在长期生产环境使用。

- [源码安装](#setup-install-source)

  提供更强的灵活性：例如允许在同一个系统中运行多个Odoo版本。有益于开发模块，可用作生产部署的基础。

- [Docker](https://www.odoo.com/documentation/13.0/setup/install.html#setup-install-docker)

  如果你经常使用[docker](https://www.docker.com/)来进行开发或部署，可使用获取官方[docker](https://www.docker.com/)基础镜像。



## 版本

Odoo有两种不同的[版本](https://www.odoo.com/pricing#pricing_table_features)：社区版和企业版。使用企业版可以通过[SaaS](https://www.odoo.com/page/start)并获取仅限于企业版客户和合作伙伴的代码。社区版免费向所有人开放。

如果你在使用社区版并希望升级到企业版，请参考[社区版升级到企业版](https://www.odoo.com/documentation/13.0/setup/enterprise.html#setup-enterprise) ([源码安装](#setup-install-source)除外)。



## 在线使用

### Demo

要快速了解Odoo, 可使用[demo](https://demo.odoo.com/)实例。这是仅供使用几小时的共享实例，可在不进行安装购买的情况下浏览及测试功能。

[Demo](https://demo.odoo.com/)实例无需本地安装，仅需通过浏览器访问即可。

### SaaS

轻松开启, 由Odoo S.A.全权管理及进行迁移，Odoo的 [SaaS](https://www.odoo.com/page/start)(Software as a Service)提供私有实例并可免费开始使用。可用于发现和测试Odoo并进行无本地安装非代码自定义（即与Odoo应用商店中的自定义模块不相兼容）。

既可用于测试Odoo，也可供长期生产使用。

类似[demo](https://demo.odoo.com/)实例，[SaaS](https://www.odoo.com/page/start)实例无需本地安装，仅通过浏览器即可访问。



## 安装包

Odoo提供针对Windows、基于deb的发行版(Debian, Ubuntu, …)及基于RPM的发行版 (Fedora, CentOS, RHEL, …) 的安装包，包括社区版和企业版。

这些安装包自动安装（社区版的）所有依赖，但可能会很难进行最新版的更新。

带有所要求相关依赖的官方社区版安装包可通过[nightly](https://nightly.odoo.com/)服务器获取。 社区版和企业版的安装包均可通过[下载](https://www.odoo.com/page/download)页面进行下载（必须要通过付费用户或合作伙伴账户登录才能下载企业版安装包）。

### Windows

-  通过[nightly](https://nightly.odoo.com/)服务器的下载安装包（仅社区版）或通过[下载页面](https://www.odoo.com/page/download)下载Windows 安装包（任意版本）。

- 运行所下载文件

  ### 警告

  在Windows 8中，你可能会看到标题为“Windows protected your PC”的警告。点击更多信息并继续运行。

- 接受 [UAC](http://en.wikipedia.org/wiki/User_Account_Control) 弹窗

- 执行各个安装步骤

Odoo会在安装的最后自动启动。

### Linux

#### Debian/Ubuntu

Odoo 13.0的deb安装包当前支持[Debian Buster](https://www.debian.org/releases/buster/), [Ubuntu 18.04](http://releases.ubuntu.com/18.04/)及以上版本。

##### 准备工作

Odoo需要有[PostgreSQL](http://www.postgresql.org/)服务才能正常运行。Odoo的deb包默认配置是使用在Odoo实例的相同主机上的PostgreSQL。以 root 用户执行如下命令来安装PostgreSQL服务：

```
 apt-get install postgresql -y
```

要进行PDF报表的打印，必须要自己安装[wkhtmltopdf](http://wkhtmltopdf.org/)： Debian仓库中的[wkhtmltopdf](http://wkhtmltopdf.org/)版本无法支持页眉和页脚，因此不能直接用作依赖。推荐的版本是 0.12.5，可通过[wkhtmltopdf的下载页面](https://github.com/wkhtmltopdf/wkhtmltopdf/releases/tag/0.12.5)的存档版块进行获取。之前推荐的0.12.1版本是一个不错的替代。针对各种版本的更多详情以及各自的特殊情况可通过[wiki](https://github.com/odoo/odoo/wiki/Wkhtmltopdf)页面进行了解。

##### 仓库

Odoo S.A.提供一个可用于Debian和Ubuntu不同发行版的仓库。可通过以 root 执行如下命令来安装Odoo社区版：

```
wget -O - https://nightly.odoo.com/odoo.key | apt-key add -
 echo "deb http://nightly.odoo.com/13.0/nightly/deb/ ./" >> /etc/apt/sources.list.d/odoo.list
apt-get update && apt-get install odoo
```

然后可以通过常规的 `apt-get upgrade` 命令来保持安装为最新版本。

截至目前，还没有针对企业版的仓库。

##### Deb安装包

作为上述仓库的替代方法，可通过如下方式下载deb安装包：

- 社区版: [nightly](https://nightly.odoo.com/)
- 企业版[下载页面](https://www.odoo.com/page/download)

然后可以使用 `gdebi`：

```
gdebi 
```

或 `dpkg`:

```
dpkg -i  # 这步可能会由于缺少依赖而失败
apt-get install -f # 应该会安装所缺少的依赖
dpkg -i <path_to_installation_package>
```

这会将Odoo安装为一个服务、创建必要的[PostgreSQL](http://www.postgresql.org/)用户并自动启动服务。

### 警告

python3-xlwt Debian包在Debian Buster和Ubuntu 18.04中不存在。这个python模块在导出为 xls 格式时需要用到。

如果需要这一功能，可以手动进行安装。一种方式是简单地通过下面的pip3命令来安装：

```
$ sudo pip3 install xlwt
```

### 警告

Debian 9和Ubuntu中不提供针对python模块num2words的包。文本数量值在Odoo中不会进行渲染，使用l10n_mx_edi模块时会产生问题。

如果你需要这一功能，可以像下面这样安装该 python模块：

```
$ sudo pip3 install num2words
```

#### Fedora

Odoo 13.0的rpm包支持Fedora 26。截至2017年，CentOS还没有对Odoo 13.0的所要求最小Python版本（3.5）的支持。

**译者注：**CentOS 8中已添加了对 Python 3的默认支持

##### 准备工作

Odoo需要有[PostgreSQL](http://www.postgresql.org/)服务来正常运行。这里假设可以使用sudo命令并进行了适当的配置，运行如下命令：

```
$ sudo dnf install -y postgresql-server
$ sudo postgresql-setup --initdb --unit postgresql
$ sudo systemctl enable postgresql
$ sudo systemctl start postgresql
```

要进行PDF报表的打印，必须要自己安装[wkhtmltopdf](http://wkhtmltopdf.org/)： Debian仓库中的[wkhtmltopdf](http://wkhtmltopdf.org/)版本无法支持页眉和页脚，因此不能直接用作依赖。推荐的版本是 0.12.5，可通过[wkhtmltopdf的下载页面](https://github.com/wkhtmltopdf/wkhtmltopdf/releases/tag/0.12.5)的存档版块进行获取。之前推荐的0.12.1版本是一个不错的替代。针对各种版本的更多详情以及各自的特殊情况可通过[wiki](https://github.com/odoo/odoo/wiki/Wkhtmltopdf)页面进行了解。

##### 仓库

Odoo S.A.提供一个可用于Fedora发行版的仓库。可通过执行如下命令来安装Odoo社区版：

```
$ sudo dnf config-manager --add-repo = https://nightly.odoo.com/13.0/nightly/rpm/odoo.repo
$ sudo dnf install -y odoo
$ sudo systemctl enable odoo
$ sudo systemctl start odoo
```

##### RPM包

代替使用以上所描述的仓库，可通过下面下载‘rpm’ 包：

- 社区版： [nightly](https://nightly.odoo.com/)
- 企业版 [下载](https://www.odoo.com/page/download)

下载后，包通过使用‘dnf’包管理器来进行安装：

```
$ sudo dnf localinstall odoo_13.0.latest.noarch.rpm
$ sudo systemctl enable odoo
$ sudo systemctl start odoo
```



## 源码安装

源码安装并不真的是关乎安装，而是直接通过源码进行运行。

对于模块开发者这会更为方便，因为这样会比使用安装包安装更容易获取Odoo源代码（相关信息或构建这一文件并离线获取）。

这也会让启动和停止Odoo比通过安装包配置的服务更为灵活和显式，同时它允许使用[命令行参数](https://www.odoo.com/documentation/13.0/reference/cmdline.html#reference-cmdline)无需编辑配置文件即重载设置。

最后，它提供对于系统设置更强的控制，并允许在系统中同时轻易地保留（及运行）多个版本的Odoo。

### Windows

#### 获取源码

有两种获取Odoo源代码的方式：zip**压缩包**或通过**git。**

##### 压缩包

社区版：

- [官方下载页面](https://www.odoo.com/page/download)
- [GitHub仓库](https://github.com/odoo/odoo)
- [Nightly服务器](https://nightly.odoo.com/)

企业版：

- [官方下载页面](https://www.odoo.com/page/download)
- [GitHub仓库](https://github.com/odoo/enterprise)

##### Git

以下部分要求在电脑主安装了[git](https://git-scm.com/)，并且你对于 git 命令有基础知识的掌握。

社区版：

```
C:\> git clone https://github.com/odoo/odoo.git
```

企业版：(参见 [版本](#setup-install-editions)部分进行获取)

```
C:\> git clone https://github.com/odoo/enterprise.git
```

**企业版git仓库并不包含完整的Odoo源代码。**它仅是一个附加插件的集合。主服务端代码在社区版中。运行企业版实际是将addons-path选项设置为企业版的文件夹并运行社区版服务。你需要同时克隆社区版和企业版来得到一个运行中的Odoo企业版安装。

#### 准备工作

##### Python

Odoo运行要求Python 3.5或更新的版本。使用官方的[Python 3安装包](https://www.python.org/downloads/windows/)来在你的电脑上下载并安装Python 3。

在安装期间，勾选**Add Python 3 to PATH**，然后点击**Customize Installation**并确保勾选了**pip**。

如果已安装了Python 3，确保版本为3.5或以上，因为之前的版本与Odoo不兼容。

```
C:\> python3 --version
```

同时验证针对该版本是否安装了[pip](https://pip.pypa.io/)。

```
C:\> pip3 --version
```

##### PostgreSQL

Odoo使用PostgreSQL来作为数据库管理系统。下载并安装 [最新版本的 PostgreSQL](https://www.postgresql.org/download/windows/)。

默认唯一的用户是`postgres`，但Odoo禁止通过`postgres`来进行连接，因此你需要新建一个PostgreSQL用户：

1. 在`PATH`中添加PostgreSQL的 `bin` 目录（默认为`C:\Program Files\PostgreSQL\<version>\bin`）。
2. 使用pg admin 图形化工具创建一个postgres用户并设置密码：
   - 打开**pgAdminIII**.
   - 双击服务端来创建一个连接。
   - 选择Edit ‣ New Object ‣ New Login Role。
   - 在**Role Name**字段中输入用户名（例如`odoo`）。
   - 打开**Definition**标签栏并输入密码（如 `odoo`），然后点击**OK**。

**译者注：**目前使用较多的为 pgAdmin 4，在 Login/Group Roles下进行操作

##### 依赖

Odoo的依赖在Odoo社区版根目录下的`requirements.txt` 文件中列出了所需的依赖。其中大部分都可以通过**pip**进行安装。

推荐不要将不同Odoo实例间或者与系统的python模块包混放在到一起。可以使用[virtualenv](https://pypi.python.org/pypi/virtualenv)来创建隔离的Python环境。

导航到Odoo社区安装位置的路径 (`YourOdooCommunityPath`) 并对requirements文件运行**pip**命令：

```
C:\> cd \YourOdooCommunityPath
C:\YourOdooCommunityPath> C:\Python35\Scripts\pip.exe install -r requirements.txt
```

### 警告

一些依赖无法通过pip进行安装，要求进行手动安装。尤其是：

- `psycopg` 必须通过[这个安装包](http://www.stickpeople.com/projects/python/win-psycopg/)进行安装。
- 要支持页眉和页脚，`wkhtmltopdf`的安装版本必须为 [0.12.5](https://github.com/wkhtmltopdf/wkhtmltopdf/releases/tag/0.12.5)。查看[wiki](https://github.com/odoo/odoo/wiki/Wkhtmltopdf)页面获取更多有关更多版本的详情。

对于从右向左的语言 (如阿拉伯语或希伯来语)，需要使用 `rtlcss` 包：

1. 下载并安装[nodejs](https://nodejs.org/en/download/)。

2. 安装 

   ```
   rtlcss
   ```

   :

   ```
   C:\> npm install -g rtlcss
   ```

3. 编辑系统环境变量`PATH` 来添加 `rtlcss.cmd` 所处的文件夹(通常为: `C:\Users\<user>\AppData\Roaming\npm\`).

#### 运行Odoo

一旦设置了所有的依赖，可通过运行服务端的命令行`odoo-bin`来启动Odoo。它位于Odoo社区版的根目录下。

要进行服务端的配置，你可以指定 [命令行参数](https://www.odoo.com/documentation/13.0/reference/cmdline.html#reference-cmdline-server) 或 [配置文件](https://www.odoo.com/documentation/13.0/reference/cmdline.html#reference-cmdline-config)。

对于企业版，必须将`enterprise`插件的路径添加`addons-path`参数中。注意要将它放在`addons-path`中的其它路径之前，以正确地加载插件。

常用的必要配置有：

- PostgreSQL用户和密码。
- 默认以外的自定义插件路径，来加载你自己的模块。

一个典型的方式是通过如下命令运行服务端：

```
C:\YourOdooCommunityPath> python3 odoo-bin -r dbuser -w dbpassword --addons-path=addons,../mymodules --db-filter=mydb$
```

其中`YourOdooCommunityPath` 是Odoo社区版所安装的路径， `dbuser`是PostgreSQL的登录用户，`dbpassword`是PostgreSQL的密码，`../mymodules`是一个带有附加插件的路径，而 `mydb` 是通过`localhost:8069`提供服务的默认数据库。

### Linux

#### 获取源码

有两种获取Odoo源代码的方式：zip**压缩包**或通过**git**。

##### 压缩包

社区版：

- [官方下载页面](https://www.odoo.com/page/download)
- [GitHub仓库](https://github.com/odoo/odoo)
- [Nightly服务器](https://nightly.odoo.com/)

企业版：

- [官方下载页面](https://www.odoo.com/page/download)
- [GitHub仓库](https://github.com/odoo/enterprise)

##### Git

以下部分要求在电脑主安装了[git](https://git-scm.com/)，并且你已掌握了git 命令的基础知识。

社区版：

```
$ git clone https://github.com/odoo/odoo.git
```

**译者注：**可使用sudo git clone -b 13.0 --depth 1 https://github.com/odoo/odoo.git 来减少下载的代码量

企业版：(参见 [版本](#setup-install-editions)部分进行获取)

```
$ git clone https://github.com/odoo/enterprise.git
```

**企业版git仓库并不包含完整的Odoo源代码。**它仅是一个附加插件的集合。主服务端代码在社区版中。运行企业版实际是将addons-path选项设置为企业版的文件夹并运行社区版服务。你需要同时克隆社区版和企业版来得到一个运行中的Odoo企业版安装。

#### 准备工作

##### Python

Odoo运行要求Python 3.5或更新的版本。如尚未安装Python 3请使用官方的包管理器来在你的电脑上下载并安装。

如果已经安装了Python 3，请确保版本为3.5或以上，因为更早的版本与Odoo并不兼容。

```
$ python3 --version
```

同时验证是否安装了相应版本的 [pip](https://pip.pypa.io/)。

```
$ pip3 --version
```

##### PostgreSQL

Odoo使用PostgreSQL来作为数据库管理系统。使用包管理器来下载并安装最新版本的PostgreSQL。

默认仅有用户`postgres`，但Odoo禁止通过`postgres`来进行连接，因此你需要新建一个PostgreSQL用户：

```
$ sudo -u postgres createuser -s $USER
$ createdb $USER
```

因为你的PostgreSQL用户名称与Unix登录名相同，可以无需密码来连接数据库。

##### 依赖

Odoo的依赖在Odoo社区版根目录下的`requirements.txt` 文件中列出了所需的依赖。其中大部分都可以通过**pip**进行安装。

推荐不要将不同Odoo实例间或者与系统的python模块包混放在到一起。可以使用[virtualenv](https://pypi.python.org/pypi/virtualenv)来创建隔离的Python环境。

导航到Odoo社区安装位置的路径 (`YourOdooCommunityPath`) 并对requirements文件运行**pip**命令：

```
$cd /YourOdooCommunityPath
/YourOdooCommunityPath$ pip3 install -r requirements.txt
```

### 警告

对于使用原生代码的库 (Pillow, lxml, greenlet, gevent, psycopg2, ldap)，可能在pip在能够安装依赖之前需要安装开发工具和原生依赖。在针对Python, PostgreSQL, libxml2, libxslt, libevent, libsasl2和libldap2的`-dev` 或 `-devel`包中均可获取。

### 警告

一些依赖无法通过pip进行安装，要求手动安装。尤其是：

- 要支持页眉和页脚，`wkhtmltopdf`的安装版本必须为 [0.12.5](https://github.com/wkhtmltopdf/wkhtmltopdf/releases/tag/0.12.5)。查看[wiki](https://github.com/odoo/odoo/wiki/Wkhtmltopdf)页面获取更多有关更多版本的详情。

对于从右向左的语言 (如阿拉伯语或希伯来语)，需要使用 `rtlcss` 包：

1. 通过包管理器下载并安装 **nodejs**和**npm**。

2. 安装

   ```
   rtlcss
   ```

   :

   ```
   $ sudo npm install -g rtlcss
   ```

#### 运行Odoo

一旦配置了所有依赖， 可通过运行服务端命令行工具`odoo-bin`来启动Odoo。它位于Odoo社区版所在的根目录下。

要进行服务端的配置，可以通过[命令行参数](https://www.odoo.com/documentation/13.0/reference/cmdline.html#reference-cmdline-server)或 [配置文件](https://www.odoo.com/documentation/13.0/reference/cmdline.html#reference-cmdline-config)进行指定。

企业版中必须将`enterprise`插件的路径添加`addons-path`参数中。注意要将它放在`addons-path`中的其它路径之前，这样才能正确地加载插件。

通常所需配置有：

- PostgreSQ用户和密码。除[psycopg2默认值](http://initd.org/psycopg/docs/module.html)以外没有其它的默认项： 通过当前用户无需密码在`5432`端口上连接 UNIX套接字。
- 默认值以外的自定义插件路径，来加载你自己的模块。

通常运行服务端的方式为：

```
/YourOdooCommunityPath$ python3 odoo-bin --addons-path=addons,../mymodules --db-filter=mydb$
```

`YourOdooCommunityPath`是 Odoo社区版安装的路径， `../mymodules` 是带有附加插件的目录，而 `mydb`是通过`localhost:8069`提供服务的默认数据库。

### Mac OS

#### 获取源码

有两种获取Odoo源代码的方式：zip**压缩包**或通过**git**。

##### 压缩包

社区版：

- [官方下载页面](https://www.odoo.com/page/download)
- [GitHub仓库](https://github.com/odoo/odoo)
- [Nightly服务器](https://nightly.odoo.com/)

企业版：

- [官方下载页面](https://www.odoo.com/page/download)
- [GitHub仓库](https://github.com/odoo/enterprise)

##### Git

以下部分要求在电脑主安装了[git](https://git-scm.com/)，并且你已掌握了git 命令的基础知识。

社区版：

```
$ git clone https://github.com/odoo/odoo.git
```

企业版：(参见 [版本](#setup-install-editions)部分进行获取)

```
$ git clone https://github.com/odoo/enterprise.git
```

**企业版git仓库并不包含完整的Odoo源代码。**它仅是一个附加插件的集合。主服务端代码在社区版中。运行企业版实际是将addons-path选项设置为企业版的文件夹并运行社区版服务。你需要同时克隆社区版和企业版来得到一个运行中的Odoo企业版安装。

#### 准备工作

##### Python

Odoo要求使用Python 3.5或之后的版本进行运行。如未安装，请使用你喜欢的包管理器 ([homebrew](http://brew.sh/), [macports](https://www.macports.org/)) 来在电脑上下载并安装Python 3。

如果已安装了Python 3，请确保版本为3.5或以上，因为更老的版本与Odoo不兼容。

```
$ python3 --version
```

同时验证已安装了对应版本的 [pip](https://pip.pypa.io/)。

```
$ pip3 --version
```

##### PostgreSQL

Odoo使用PostgreSQL来作为数据库管理系统。使用[postgres.app](https://postgresapp.com/)来下载并安装最新版本的PostgreSQL。

默认仅有用户`postgres`，但Odoo禁止通过`postgres`来进行连接，因此你需要新建一个PostgreSQL用户：

```
$ sudo -u postgres createuser -s $USER
$ createdb $USER
```

因为PostgreSQL用户与Unix登录名相同，所以可以无需密码直接连接该数据库。

##### 依赖

Odoo的依赖在Odoo社区版根目录下的`requirements.txt` 文件中列出了所需的依赖。其中大部分都可以通过**pip**进行安装。

推荐不要将不同Odoo实例间或者与系统的python模块包混放在到一起。可以使用[virtualenv](https://pypi.python.org/pypi/virtualenv)来创建隔离的Python环境。

导航到Odoo社区安装位置的路径 (`YourOdooCommunityPath`) 并对requirements文件运行**pip**命令：

```
$ cd  /YourOdooCommunityPath
/YourOdooCommunityPath$ pip3 install -r requirements.txt
```

### 警告

非Python依赖需要通过包管理器来进行安装：

1. 下载交安装

   命令行工具：

   ```
   $ xcode-select --install
   ```

2. 下载并安装你所选择的包管理器 ([homebrew](http://brew.sh/), [macports](https://www.macports.org/))。

3. 安装非python依赖。

### 警告

有些依赖无法通过pip进行安装，要求进行手动安装。尤其是：

- 要支持页眉和页脚，`wkhtmltopdf`安装的版本必须为[0.12.5](https://github.com/wkhtmltopdf/wkhtmltopdf/releases/tag/0.12.5)。参见[wiki](https://github.com/odoo/odoo/wiki/Wkhtmltopdf)页面获取更多版本的详情。

对于从右向左阅读的语言(如阿拉伯语或希伯来语)，需要使用 `rtlcss` 包：

1. 通过自己喜欢的包管理器([homebrew](http://brew.sh/), [macports](https://www.macports.org/))下载并安装**nodejs** 。

2. 安装 

   ```
   rtlcss
   ```

   :

   ```
   $ sudo npm install -g rtlcss
   ```



## Docker

如何通过Docker使用Odoo的完整文档中参见官方的Odoo [docker镜像](https://registry.hub.docker.com/_/odoo/)页面。

## 译者补充安装方法（Ubuntu）

[![Odoo 13后台页面](https://alanhou.org/homepage/wp-content/uploads/2019/10/2019101009424619.jpg)](https://alanhou.org/homepage/wp-content/uploads/2019/10/2019101009424619.jpg)

```
# 更新为国内apt源（以阿里云镜像为例）
sudo sed -i "s/archive.ubuntu.com/mirrors.aliyun.com/g" /etc/apt/sources.list
sudo sed -i "s/security.ubuntu.com/mirrors.aliyun.com/g" /etc/apt/sources.list
sudo apt-get update 
sudo adduser -system -home=/opt/odoo -group odoo -shell /bin/bash # 添加odoo用户和组
sudo apt-get install -y postgresql # 安装PostgreSQL数据库
sudo service postgresql start # 启动 PostgreSQL
# 创建数据库用户
sudo su - postgres
createuser --createdb --username postgres --no-createrole --no-superuser --pwprompt odoo
exit
sudo service postgresql restart
# 安装依赖
sudo apt-get install -y python3-pip
sudo apt-get install virtualenv
sudo apt-get install libsasl2-dev python-dev libldap2-dev libssl-dev python3-pypdf2

sudo su - odoo
git clone -b 13.0 --depth 1 https://github.com/odoo/odoo.git 
virtualenv -p python3 odoo-env
source odoo-env/bin/activate
pip3 install -r odoo/requirements.txt

# 配置文件
sudo vim /etc/odoo-server.conf
# 示例配置内容
[options]
; This is the password that allows database operations:
; admin_passwd = admin
db_host = False
db_port = False
db_name = odoo
db_user = odoo
db_password = False
logfile = /var/log/odoo/odoo-server.log
addons_path = /opt/odoo/odoo/addons,/opt/odoo/odoo/odoo/addons

# 设置权限
sudo chown odoo: /etc/odoo-server.conf 
sudo chmod 640 /etc/odoo-server.conf
sudo mkdir /var/log/odoo 
sudo chown odoo:root /var/log/odoo

# 安装 wkhmtltopdf（选择对应的 Ubuntu 版本）
wget "https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.xenial_amd64.deb" -O /tmp/wkhtml.deb 
sudo dpkg -i /tmp/wkhtml.deb  # 忽略此处报错，直接执行下一步
sudo apt-get -fy install

# 设置服务启动

vi /etc/systemd/system/odoo13.service

# 内容
[Unit]
Description=Odoo13
Requires=postgresql.service
After=network.target postgresql.service

[Service]
Type=simple
SyslogIdentifier=odoo13
PermissionsStartOnly=true
User=odoo
Group=odoo
WorkingDirectory=/opt/odoo
Environment=/opt/odoo/odoo-env/bin/activate
ExecStart=/opt/odoo/odoo-env/bin/python3 /opt/odoo/odoo/odoo-bin -c /etc/odoo-server.conf
StandardOutput=journal+console

[Install]
WantedBy=multi-user.target

# 启动服务
sudo systemctl start odoo13
```

  