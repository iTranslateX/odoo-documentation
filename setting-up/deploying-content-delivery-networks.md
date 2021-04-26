# 通过 CDN 进行部署

本文来自[Odoo 13官方文档之开发者文档](https://alanhou.org/odoo-13-developer-documentation/)系列文章

## 通过[KeyCDN](https://www.keycdn.com/)进行部署

本文档将指导你为Odoo网站配置一个 [KeyCDN](https://www.keycdn.com/) 账户。

### 第1步：在KeyCDN仪表盘中创建一个pull 区

![img](https://www.odoo.com/documentation/13.0/_images/keycdn_create_a_pull_zone.png)创建该区时，在高级功能子菜单中启用CORS选择。 (更多参见后述内容)

![img](https://www.odoo.com/documentation/13.0/_images/keycdn_enable_CORS.png)一旦完成后，需要等待片刻，此时[KeyCDN](https://www.keycdn.com/)会爬取你的网站。

![img](https://www.odoo.com/documentation/13.0/_images/keycdn_progressbar.png)

会为你的Zone生成一个新的 URL，本例中链接为 `http://pulltest-b49.kxcdn.com`

### 第2步：使用你的zone 配置odoo实例

在Odoo后台中，进入到网站设置（Website Settings）菜单，然后启用CDN并复制/拷贝你的zone URL到 CDN Base URL字段中。该字段仅在启用了开发者模式之后才可见并可配置。

![img](https://www.odoo.com/documentation/13.0/_images/odoo_cdn_base_url.png)现在你的网站会对匹配CDN过滤器正则表达式的资源使用 CDN。

可以查看网站的HTML源码来确定是否正确地进行了CDN集成。

![img](https://www.odoo.com/documentation/13.0/_images/odoo_check_your_html.png)

### 为什么要激活CORS？

在一些浏览器(编写本文档时有Firefox 和 Chrome )的安全限制中会防止远程链接的CSS文件获取该外部服务器上的相对资源。

如未在CDN区中启用 CORS选项，默认Odoo网站上最明显的问题是缺少 font-awesome图标，因为在 font-awesome的CSS中声明字体文件无法通过远程服务器进行加载。

以下是在这种情况下首页的样子：

![img](https://www.odoo.com/documentation/13.0/_images/odoo_font_file_not_loaded.png)一条安全错误信息会出现在浏览器的控制台中：

![img](https://www.odoo.com/documentation/13.0/_images/odoo_security_message.png)在 CDN 中启用CORS选择可解决这一问题。