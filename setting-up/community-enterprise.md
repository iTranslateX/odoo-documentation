# 社区版转企业版

本文来自[Odoo 13官方文档之开发者文档](../README.md)系列文章

根据当前的安装方式，有多种升级社区版的方法。各种情况下的基本指导思想是：

- 备份社区版数据库![img](https://www.odoo.com/documentation/13.0/_images/db_manager.png)
- 关闭服务
- 安装web_enterprise模块
- 重启服务
- 输入Odoo企业版订阅码

![img](https://www.odoo.com/documentation/13.0/_images/enterprise_code.png)

## 在Linux中使用安装包

- 备份社区版数据库

- 停止odoo服务

  ```
  $ sudo service odoo stop
  ```

- 安装企业版 .deb (应该在社区安装包之上安装)

  ```
  $ sudo dpkg -i <path_to_enterprise_deb>
  ```

- 使用如下命令更新数据库为企业版安装包

  ```
  $ python3 /usr/bin/odoo-bin -d <database_name> -i web_enterprise --stop-after-init
  ```

- 应当能够使用常规的认证方式连接到 Odoo企业版实例。然后你可以通过在表单输入框中输入邮件中所收到的订阅码将数据库与你的Odoo企业版订阅进行关联。

## 在Linux中使用源代码

有很多种使用源码启动服务器的方式，你可能已有自己的偏好。可以将下面部分调整为你自己的常规工作流。

- 关闭你的服务

- 备份社区版数据库

- 更新启动命令中的 `--addons-path` 参数 (参见 [源码安装](https://alanhou.org/odoo-13-installing-odoo/#setup-install-source))

- 通过使用如下命令安装 web_enterprise 模块

  ```
  $ -d <database_name> -i web_enterprise --stop-after-init
  ```

  根据数据库的大小不同，可能会花费一些时间。

- 使用第3点中更新的addons路径重新启动服务。 此时应该可以连接到实例。可以通过在表单输入框中输入邮件中所收到的订阅码将数据库与你的Odoo企业版订阅进行关联。

## 在Windows上

- 备份社区版数据库

- 卸载Odoo 社区版 (使用安装文件夹中的 Uninstall可执行文件) - PostgreSQL将保持已安装状态![img](https://www.odoo.com/documentation/13.0/_images/windows_uninstall.png)

- 启动 Odoo企业版安装器并按照常规步骤执行。在选择安装路径时，可以设置社区版软件的文件夹(这个文件夹中还包含PostgreSQL软件 )。 在安装的最后取消勾选 `Start Odoo` ![img](https://www.odoo.com/documentation/13.0/_images/windows_setup.png)

- 使用命令行窗口，使用下面的命令（通过服务的子文件夹中的Odoo 安装路径）更新Odoo数据库

  ```
  $ odoo.exe -d <database_name> -i web_enterprise --stop-after-init
  ```

- 无需手动启动服务，服务已运行。这时应该可以使用常规的认证方式连接到Odoo企业版实例了。然后可以通过在表单输入框中输入邮件中所收到的订阅码将数据库与你的Odoo企业版订阅进行关联。