# Odoo email 网关

本文来自[Odoo 13官方文档之开发者文档](../README.md)系列文章

Odoo邮件网关允许我们在Odoo中直接注入所有已接收邮件。其原理直截了当：你的SMTP服务为每个新新收邮件执行“mailgate”脚本。该脚本通过XML-RPC处理与Odoo数据库的连接，并通过MailThread.message_process()功能发送邮件。

## 前提条件

- 对Odoo数据库的管理员权限。
- 自己的邮件服务器，如 Postfix 或 Exim。
- 有关如何配置邮件服务器的技术知识。

## Postfix篇

在alias配置 (/etc/aliases)中配置：

```
email@address: "|/odoo-directory/addons/mail/static/scripts/odoo-mailgate.py -d <database-name> -u <userid> -p <password>"
```

郑源 - `Postfix` - `Postfix aliases` - `Postfix virtual`

## Exim篇

```
*: |/odoo-directory/addons/mail/static/scripts/odoo-mailgate.py -d <database-name> -u <userid> -p <password>
```

资源 - `Exim`

如果你可以访问/管理自己的邮件服务器，使用`inbound messages`.