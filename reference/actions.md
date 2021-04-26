# 动作

- 本文来自[Odoo 13官方文档之开发者文档](../README.md)系列文章

动作定义对应用户动作的系统行为：登录、动作按钮、发票选取 … 动作可存储在数据库中或直接返回如按钮方法的字典。所有的动作有两个必选属性：

- `type`

  当前动作的分类，定义可使用哪些字段以及如何解析动作

- `name`

  针对动作的用户可读的简短描述，可在客户端界面中显示

- `binding_model_id`

  若进行设置，在给定模型的动作栏中的可用动作对于服务端动作，使用 `model_id`。

客户端可以获取4种形式的动作：

- - `False`

    若当前有动作对话框处于打开状态，进行关闭

- - 字符串

    若匹配 [客户端动作](#reference-actions-client)，解析为客户端动作标签，否则视为数字

- - 数字

    读取数据库中的相应动作记录，可为数据库标识符或 [外部id](https://www.odoo.com/documentation/13.0/glossary.html#term-external-id)

- - 字典

    视为客户端描述符并执行

## 绑定

除两个必选属性外，所有的动作还可以有用于在任意模型的上下文菜单中展示动作的*可选*属性：

- `binding_model_id`

  指定动作所绑定的模型

- `binding_type`

  指定所绑定的类型，大多为动作出现在哪个上下文菜单之下`action` (默认)指定动作会出现在绑定模型Action上下文菜单中。`report`指定动作会出现在所绑定模型的Print上下文菜单中

- `binding_view_types`

  视图的逗号分隔列表，其中动作出现在上下文菜单中，大部分“列表”及/或“表单”。默认为`list,form` (列表及表单)



## 窗口动作(`ir.actions.act_window`)

大部分常用动作类型，用于展示通过[视图](https://alanhou.org/odoo-13-views/#reference-views)的模型可视化：定义一组视图类型的模型（可能为模型的指定记录）窗口动作 (及可能的指定视图) f

其字段有：

- `res_model`

  用于展示视图的模型

- `views`

  `(view_id, view_type)` 对的列表。每组的第二个元素是视图的分类 (tree, form, graph, …) ，第一个是可选的数据库 id (或 `False`)。若未提供 id，客户端应获取对所请求模型指定类型的默认视图(默认通过[`fields_view_get()`](https://alanhou.org/odoo-13-orm-api/#odoo.models.Model.fields_view_get)来实现). 列表的第一个类型是默认视图类型列表，在动作执行时会默认打开。每个视图类型在列表中最多出现一次。

- `res_id` (可选)

  若默认视图为 `form`，指定要加载的记录(否则应创建一条新记录)

- `search_view_id` (可选)

  `(id, name)` 对， `id` 为针对动作所加载的指定搜索视图的数据库标识符。默认获取模型的默认搜索视图

- `target` (可选)

  是否应在主内容区 (`current`)中以全屏模式（`fullscreen`）或对话框/弹窗（`new`）中打开视图。使用 `main` 代替 `current` 来清除面包屑导航。默认为`current`。

- `context` (可选)

  传递给视图的额外上下文数据

- `domain` (可选)

  隐式添加给所有视图搜索查询的过滤作用域

- `limit` (可选)

  默认在列表中显示的记录数。在网页客户端中默认为80

例如，要通过列表及表单视图打开客户 (配合 `customer` 标记集) ：

```
{
    "type": "ir.actions.act_window",
    "res_model": "res.partner",
    "views": [[False, "tree"], [False, "form"]],
    "domain": [["customer", "=", true]],
}
```

或在新对话框中打开指定产品（单独获取）的表单视图：

```
{
    "type": "ir.actions.act_window",
    "res_model": "product.product",
    "views": [[False, "form"]],
    "res_id": a_product_id,
    "target": "new",
}
```

数据库中的窗口动作有一些应由客户端忽略的不同字段，大多数用于组成`views` 列表：

- `view_mode` (默认= `tree,form` )

  视图类型的逗号分隔列表作为字符串(/!\ 无空格 /!\)。所有这些类型会在生成的`views` 列表中出现 (至少有一个`False` view_id)

- `view_ids`

  M2M[1](#notquitem2m) 的视图对象，定义 `views`的初始内容Act_window视图也可通过`ir.actions.act_window.view`清晰地定义。如果你打算允许模型的多个视图，推荐使用ir.actions.act_window.view 而不是动作 `view_ids``<record model="ir.actions.act_window.view" id="test_action_tree">   <field name="sequence" eval="1"/>   <field name="view_mode">tree</field>   <field name="view_id" ref="view_test_tree"/>   <field name="act_window_id" ref="test_action"/> </record>`

- `view_id`

  在类型是`view_mode`列表的一部分且没有由`view_ids`中的视图进行填充时，添加到 `views` 中的指定视图

These are mostly used when defining actions from这些大多在 [数据文件](https://www.odoo.com/documentation/13.0/reference/data.html#reference-data)中定义动作时使用：

```
<record model="ir.actions.act_window" id="test_action">
    <field name="name">A Test Action</field>
    <field name="res_model">some.model</field>
    <field name="view_mode">graph</field>
    <field name="view_id" ref="my_specific_view"/>
</record>
```

即使不是模型的默认视图，也将使用“my_specific_view”视图。

服务端的 `views` 序列的组合如下：

- 通过 `view_ids` 获取每个`(id, type)`(通过`sequence`排序)
- 若定义了 `view_id` 且未填充其类型，追加其 `(id, type)`
- 对每个 `view_mode`中未填充的类型，追加 `(False, type)`

[[1\]](#id1) 技术上不是 M2M: 添加序列字段及可能只由视图类型组成，不带有视图id。



## URL动作 (`ir.actions.act_url`)

(website/web page) via an 允许Odoo动作打开URL（网站/网页）。可由两个字段进行自定义：

- `url`

  在激活动作时打开的地址

- `target`

  若为 `new`在新窗口/页面打开该地址，若为 `self`使用该页面替换当前内容。默认值为 `new`

```
{
    "type": "ir.actions.act_url",
    "url": "https://odoo.com",
    "target": "self",
}
```

将使用Odoo首页替换当前内容版块。



## 服务端动作 (`ir.actions.server`)

###### `*class* odoo.addons.base.models.ir_actions.IrActionsServer(*pool*, *cr*)`

服务端动作模型。服务端动作在基模型上运作并且提供种类可自动执行的动作类型，例如，手动通过在 More 上下文菜单中添加动作来使用基动作规则。

从Odoo 8.0开始，在动作表单视图中出现了‘Create Menu Action’按钮。它在基模型的More菜单中创建一个入口。这允许在界面中创建服务端动作并以批量模式进行运行。

可以使用的动作有：

- ‘Execute Python Code’: 将会执行的 Python 代码块
- ‘Create a new Record’: 使用新值新建记录
- ‘Write on a Record’: 更新记录值
- ‘Execute several actions’: 定义触发一些其它服务端动作的动作

允许通过任意有效动作位置触发复杂服务端代码。仅有两个字段与客户端相关：

- `id`

  待运行服务端动作的数据库中的标识符

- `context` (可选)

  在运行服务端动作时使用的上下文数据

In-database records 数据库中的记录很明显的更丰富并可执行一些基于它们的`state`的特定或通用动作。一些字段 (及相应行为) 在状态之间进行分享：

- `model_id`

  链接到动作的Odoo模型。

```
state
```

- `code`: 执行通过`code` 参数给定的python代码。
- `object_create`: 根据`fields_lines` 规格新建`crud_model_id`模型的记录。
- `object_write`: 按照`fields_lines` 规格更新当前记录。
- `multi`: 执行通过`child_ids` 参数给定的一些动作。

### 状态字段

根据其状态通过不同的字段来定义行为。相关的状态在每个字段后给出。

- `code` (代码)

  指定调用动作时所要执行的Python代码块`<record model="ir.actions.server" id="print_instance">    <field name="name">Res Partner Server Action</field>    <field name="model_id" ref="model_res_partner"/>    <field name="code">        raise Warning(record.name)    </field> </record>`代码块中可定义`action`的变量，它将以要执行的下一个动作返回客户端：`    <field name="name">Res Partner Server Action</field>    <field name="model_id" ref="model_res_partner"/>    <field name="code">        if record.some_condition():            action = {                "type": "ir.actions.act_window",                "view_mode": "form",                "res_model": record._name,                "res_id": record.id,            }    </field> </record>`会要求客户端在满足某些条件时打开记录的表单

- `crud_model_id` (创建)(必传)

  新建记录的所在模型

- `link_field_id` (创建)

  `ir.model.fields`的many2one关联，指定当前记录的m2o字段，在其中应设置新创建字段（模型应匹配）

- `fields_lines` (创建/写入)

  在创建或拷贝记录时重载的字段。与字段的关联为 [`One2many`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.One2many)：`col1`在相关模型中设置`ir.model.fields` (`crud_model_id` 用于创建， `model_id` 用于更新)`value`字段的值，通过 `type`解析`type` (value|reference|equation)若为 `value`, `value` 字段解析为字面量值(可能会进行格式转换)，若为`equation`，`value` 字段解析为Python 表达式并进行运行

- `child_ids` (multi)

  指定多个子动作 (`ir.actions.server`) 来启用多状态。如子动作本身返回动作，上一个会作为multi自己的下一个动作返回客户端



### 运行上下文

在服务端动作的运行上下文或周边存在很多键可以使用：

- `model` 通过 `model_id`关联动作的模型对象
- `record`/`records` 动作所触发的记录/记录集，可以为空。
- `env` Odoo环境
- `datetime`, `dateutil`, `time`, `timezone` 相应Python模块
- `log: log(message, level='info')` 在ir.logging表中用于记录信息的日志函数
- `Warning` 针对`Warning` 异常的构造器



## 报表动作(`ir.actions.report`)

触发报表打印

- `name` (必传)

  仅在查询一些排序列表中的某一个时用于报表的助记符/描述

- `model` (必传)

  报表相关的模型

- `report_type` (default=qweb-pdf)

  要么是用于PDF报表的 `qweb-pdf` ，要么是用于HTML报表的 `qweb-html`

- `report_name` (必传)

  报表的名称 (将会与PDF的输出名称相同)

- `groups_id`

  允许查看/使用当前报表用户组的[`Many2many`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Many2many) 字段

- `multi`

  若设置为`True`, 该动作将不会在表单视图中显示。

- `paperformat_id`

  希望用于此报表的对应于纸张格式的[`Many2one`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Many2one)字段(若未指定，会使用公司格式)

- `attachment_use`

  若设置为`True`, 该报表仅在请求的初次进行生成，随后会从存储的报表中重新打印，而不会每次都重新生成。可用于必须仅生成一次的报表 (例如出于法律原因)

- `attachment`

  定义报表名称的python表达式；记录可以 `object`变量进行访问



## 客户端动作(`ir.actions.client`)

触发完全在客户端实现的动作。

- `tag`

  动作的客户端标识符， 客户端应知道如何响应的任意字符串

- `params` (可选)

  与客户端动作标记一起发送到客户端的附加数据的Python字典

- `target` (可选)

  客户端动作是否应在主内容区(`current`)、以全屏模式(`fullscreen`)或对话框/弹窗(`new`)中打开。使用 `main` 代替`current` 来清除面包屑。默认值为 `current`。

```
{
    "type": "ir.actions.client",
    "tag": "pos.ui"
}
```

告诉客户端启动POS 界面，服务端并不了解 POS 界面的运行方式。



## 自动化动作(`ir.cron`)

以预定义频率自动触发的动作。

- `name`

  自动化动作的名称 (主要在日志显示中使用)

- `interval_number`

  在两个动作执行之间的*nterval_type* 计量单位数

- `interval_type`

  频率间隔的计量单位(`minutes`, `hours`, `days`, `weeks`, `months`,

- `numbercall`

  这一动作需要运行的次数。如果动作要进行无限动作，设置为 `-1`。

- `doall`

  在服务重启时指定是否执行未执行动作的布尔值。

- `model_id`

  这一动作将会被调用的模型

- `code`

  动作的代码内容。可以是对模型方法的简单调用：`model.<method_name>()`

- `nextcall`

  该动作的下一次计划执行日期(date/time 格式)

 