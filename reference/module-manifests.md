# 模块声明文件

本文来自[Odoo 13官方文档之开发者文档](https://alanhou.org/odoo-13-developer-documentation/)系列文章

## 声明

声明文件用于声明python包为Odoo模块并指定模块元数据信息。

它是一个名为`__manifest__.py`的文件，包含一个 Python字典，其中每个键指定模块元数据单元。

```
{
    'name': "A Module",
    'version': '1.0',
    'depends': ['base'],
    'author': "Author Name",
    'category': 'Category',
    'description': """
    Description text
    """,
    # data files always loaded at installation
    'data': [
        'views/mymodule_view.xml',
    ],
    # data files containing optionally loaded demonstration data
    'demo': [
        'demo/demo_data.xml',
    ],
}
```

可用的声明字段有：

`name` (`str`, 必填)

可阅读的模块名

`version` (`str`)

该模块的版本，应遵循[语法版本号](https://semver.org/)规则

`description` (`str`)

模块的扩展描述，格式为reStructuredText

`author` (`str`)

模块作者的姓名

`website` (`str`)

模块作者的网站URL

`license` (`str`, 默认: `LGPL-3`)

模块的发行证书。可用的值有：

* `GPL-2`
* ``GPL-2 or any later version``
* GPL-3``GPL-3 or any later version``
* AGPL-3``LGPL-3``
* Other OSI approved licence
* ``OEEL-1` (Odoo企业版证书 v1.0)
* `OPL-1` (Odoo自有证书 v1.0)
* `其它资产`

``category` (`str`, 默认: `Uncategorized`)

Odoo中的分类，模块的大概业务领域。

虽然推荐使用 [已有分类](https://github.com/odoo/odoo/blob/13.0/odoo/addons/base/data/ir_module_category_data.xml)，但字段时自由形式的并会补时创建未知的分类。作业帮行者无疆有可使用分隔符 Category hierarchies can be cr `/`进行创建，如 `Foo / Bar` 会创建一个分类`Foo`，并创建分类 `Bar` 为 `Foo`的子分类，且会将`Bar`设置为模块的分类。

`depends` (`list(str)`)

必须在其之前加载的Odoo模块，要么因为该模板使用了它们创建的功能或因为其修改了它们定义的资源。在安装模块时，其所有依赖会在它之前安装。类似地依赖会在模块加载完之间加载。

`data` (`list(str)`)

必须保持通过模块安装或更新的数据文件列表。来自模块根目录的路径列表

`demo` (`list(str)`)

仅在*演示模式*下安装或更新的数据文件列表

`auto_install` (`bool`, 默认: `False`)

`若为True`，这个模块会自动在其所有依赖已安装时进行安装。

通常用于实现两个其它独立模块协同集成的“链接模块”。

例如 `sale_crm` 同时依赖于 `sale` 和 `crm` 并设置为 `auto_install`。在安装了`sale` 及 `crm` 的时候，它自动添加追踪订单的CRM活动，而`sale` 或 `crm` 并不知道彼此的存在

`external_dependencies` (`dict(key=list(str))`)

包含python 和/或二进制依赖的字典。

对于python依赖，`python`键必须为这个字典定义并应分配一个导入的 python模块列表。

对于二进制依赖， `bin` 键必须为定义典定义且应分配一个二进制可执行名称的列表。

在未安装python模块时或主机或二进制没在主机的PATH环境变量中找到时不会安装该模块。

`application` (`bool`, 默认: `False`)

模块是否应被看成是安整的应用 (`True`) 或仅是一个向已有应用模块提供功能支持的技术模块 (`False`) 。

`css` (`list(str)`)

指定带有导入的自定义规则的css文件，这些文件应位于该模块的`static/src/css` 中。

`images` (`list(str)`)

指定模块所使用的图片文件。

`installable` (`bool` 默认: `True`)

用户是否能通过Web UI安装该模块。

`maintainer` (`str`)

负责维护该模块的人或组织，默认假定作者就是维护人员。

`{pre_init, post_init, uninstall}_hook` (`str`)模块安装/卸载的钩子，它们的值应为表示定义在模块`__init__.py`内的函数名的字符串。

`pre_init_hook` 接收游标作为其唯一的参数，这个函数在模块安装之前执行。

`post_init_hook` 接收一个游标及仓库作为其参数，该函数在模块安装之后执行。

`uninstall_hook` 接收一个游标及仓库作为其参数，该函数在模块卸载之后执行。

这些钩子应仅在配置/清理时使用，要求使用这个模块的场景是通过 api 要么非常复杂要么无法办到时。 

