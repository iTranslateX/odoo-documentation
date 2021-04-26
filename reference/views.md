# 视图

- 本文来自[Odoo 13官方文档之开发者文档](../README.md)系列文章

## 通用结构

视图对象暴露一些字段，除非指定它们是可选的。

- `name` (必传)

  仅在查找某个列表中内容时用作视图的助记符/描述

- `model`

  若适用为关联到视图的模型

- `priority`

  客户端程序可通过 `id` 或 by `(model, type)`请求视图。对于后者，会搜索所有相应类型及模型的视图，并且会返回 `priority` 值最小的那个 (它是“默认视图).`priority` 还在[视图继承](#reference-views-inheritance)时定义应用的排序

- `arch`

  视图布局的描述

- `groups_id`

  允许查看/使用当前视图的组对应的[`Many2many`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Many2many) 字段

- `inherit_id`

  当前视图的父级视图，参见 [继承](#reference-views-inheritance), 默认未设置

- `mode`

  继承模式，参见 [继承](#reference-views-inheritance)。如未设置 `inherit_id`则 `mode` 仅可为`primary`。如设置了`inherit_id`，默认 `extension`可显式地设置为`primary`

- `application`

  定义可切换视图的网站功能。默认，视图都会应用

- `banner_route`

  获取及前置到视图的路由地址。如果设置了这一属性，会获取[控制器路由url](https://alanhou.org/odoo-13-web-controllers/#reference-controllers)将展示上面的视图。控制器中响应的json应包含“html”键。在html中包含一个样式<link>标记，会进行删除并附加到 <head>中。要与后端交互，可以使用 <a type=”action”> 标签。请查看AbstractController (*addons/web/static/src/js/views/abstract_controller.js*)的_onActionClicked方法的文档了解更多详情。仅继承AbstractView 和 AbstractController的视图可使用这一属性， [表单](#reference-views-form), [看板](#reference-views-kanban), [列表](#reference-views-list), …例:`<tree banner_route="/module_name/hello" />``class MyController(odoo.http.Controller):    @http.route('/module_name/hello', auth='user', type='json')    def hello(self):        return {            'html': """                <div>                    <link href="/module_name/static/src/css/banner.css"                        rel="stylesheet">                    <h1>hello, world</h1>                </div> """        }`



## 继承

### 视图匹配

- 如通过 `(model, type)`请求视图，具有相应模型和类型的视图、`mode=primary` 及最优先级值最小的会进行匹配
- 在通过 `id`请求视图时，如其模式不是 `primary` ，会匹配具有`primary` 模式的*最近*的父级

### 视图解析

解析为所请求/匹配的`primary`视图生成最终的`arch`：

1. 若视图存在父级，会完全解析父级，然后应用当前视图的详细参数
2. 若视图没有父级，会按原内容使用 `arch`
3. 会查询当前视图带有 `extension`模式的子视图并且它们的继承参数会按照深度优先的原则进行应用(先应用子视图，接着是它的子视图及兄弟视图)

应用子视图的结果会产生最终的`arch`

### 继承详情

继承详情由元素定位符所组成，来匹配父级视图中所继承的元素及用于修改所继承元素的子级元素。

有三类用于匹配目录元素的元素定位符：

- 带有 `expr` 属性的`xpath`元素。 `expr` 是一个应用于当前`arch`的 [XPath](http://en.wikipedia.org/wiki/XPath) 表达式[2](#hasclass) ，它所找到的第一个节点就是匹配内容
- 带有`name`属性的 `field`元素，匹配带有相同`name`的第一个 `field` 。所有其它元素在匹配过程中会进行忽略
- 匹配第一个元素具有相同名称和属性的任意其它元素 (忽略 `position` 和 `version` 属性)

继承详情可有一个可选的 `position` 属性，指定如何修改所匹配的节点：

- `inside` (默认)

  继承详情的内容添加到所匹配的节点中

- `replace`

  继承详情的内容替换掉所匹配的节点。详情内容中仅包含 `$0` 的文本节点会由一个完整的所匹配节点进行替换，有效的封装所匹配节点。

- `after`

  继承详情的内容添加到所匹配节点的父级中，在所匹配节点之后

- `before`

  继承详情的内容添加到所匹配节点的父级中匹配节点之前

- `attributes`

  继承详情的内容应为带有`name`属性可可选内容体的 `attribute` 元素：如果 `attribute` 元素拥有内容体，会使用 `attribute` 元素的文本作为值在匹配节点上按照`name`名称新建一个属性若`attribute` 元素没有内容体，以 `name`命名的属性名会从匹配节点中删除。如果不存在这种属性，会抛出错误

此外，`position` `move` 可用于带有`inside`, `replace`, `after` 的直接子视图，或是 `before` `position`属性用于移动节点。

```
<xpath expr="//@target" position="after">
    <xpath expr="//@node" position="move"/>
</xpath>

<field name="target_field" position="after">
    <field name="my_field" position="move"/>
</field>
```

视图详情按照顺序进行应用。



## 列表

列表视图的根元素是 `<tree>`[3](#treehistory)。列表视图的根可拥有如下属性：

- `editable`

  默认，选择列表视图的行打开对应的 [表单视图](#reference-views-form)。`editable` 属性让视图本身在原处可编辑。有效的值有 `top` 和 `bottom`，让新记录分别出现在列表的顶部或底部。行内[表单视图](#reference-views-form)的结构通过列表视图获取。大部分在 [表单视图](#reference-views-form)的字段和按钮中有准备的属性由列表视图所支持，但在列表视图不可编辑时可能不具备什么意义。如果`edit` 属性设置为 `false`，会忽略掉 `editable`选项。

- `multi_edit`

  可编辑或不可编辑列表可通过定义multi_edit=1来激活多编辑功能

- `default_order`

  重载视图的顺序，替换模型的默认顺序。值为一个字段的逗号分隔列表，后接 `desc` 来进行反向排序：`<tree default_order="sequence,name desc">`

- `decoration-{$name}`

  允许根据相应的记录属性在悠行级文本的样式。值为Python表达式。对于每条记录，通过记录的属性作为上下文值运行表达式，若为 `true`，相应的样式应用于行之上。其它的上下文值有 `uid` (当前用户的 id) 和 `current_date` ( `yyyy-MM-dd`格式的当前日期字符串)。`{$name}` 可为 `bf` (`font-weight: bold`), `it` (`font-style: italic`)或任意[bootstrap上下文颜色 ](https://getbootstrap.com/docs/3.3/components/#available-variations) (`danger`, `info`, `muted`, `primary`, `success` or `warning`).

- `create`, `edit`, `delete`, `duplicate`, `import`, `export_xlsx`

  允许通过设置相应属性为 `false`来在视图中*禁用*相应的动作

- `limit`

  页面的默认大小。必须为正整数

- `groups_limit`

  在分组列表视图时，一页中组的默认数量。必须是正整数

- `expand`

  在分组列表视图时， 在设置为true (默认: false)时自动打开组的第一级

列表视图的可能的子元素有：

- `button`

  在列表单元格中显示按钮`icon`用于显示按钮的图标`string`如时没有 `icon`, 为按钮的文本如果存在 `icon`，为图标的 `alt` 文本`type`按钮类型， 表明点击时对 Odoo产生的效果：`object`在列表模型中调用方法。按钮的`name` 是方法，通过当前行记录id和当前上下文进行调用。`action`加载并执行 `ir.actions`，按钮的`name`为动作的数据库id。上下文使用列表模型（以`active_model`）进行扩展 ，当前行的记录 (`active_id`) 和列表中当前加载的所有记录 (`active_ids`, 可以仅是匹配当前搜索的数据库记录的子集)`name`参见 `type``args`参见 `type``attrs`基于记录值的动态属性。属性对作用域的映射，作用域在当前行记录的上下文中运行，如为`True` ，在单元格中设置相应属性。可用的属性有 `invisible` (隐藏按钮).`states``invisible` `attrs`的简写：状态列表、逗号分隔，要求模型有一个`state` 字段并在视图中使用。如记录是在列出的状态中让按钮`不可见`🚫危险在使用`attrs`的同时使用 `states`可能会导致预期外的结果，因为作用域使用逻辑与AND进行拼接。`context`在执行按钮的Odoo调用时合并入视图的上下文`confirm`在进行按钮的 Odoo 调用之前显示的（供用户接受的）确认消息

- `field`

  定义相应字段应针对每条记录显示的列。可使用如下属性：`name`在当前模型中显示的字段名。给定的名称一个视图中仅可使用一次`string`列字段标题(默认，使用模型字段的 `string` )`invisible`获取并存储字段，但不显示表中的列。针对不应显示查由如`@colors`所使用的必要字段`groups`不能看到该字段的用户组列表`widget`针对字段显示的替代展现。（一部分）可以使用的视图值有：`progressbar`以`float` 字段显示为进度条。`handle`针对 `sequence` (或 `integer`) 字段，记录通过它们排序，而不是中通过拖拽图标对记录重新排序来显示字段值。`sum`, `avg`在列的询问显示相应的聚合。仅对当前显示记录计算聚合。聚合运算必须匹配相应字段的`group_operator``attrs`基于记录值的动态属性。仅影响当前字段，因此如`invisible` 会隐藏该字段，但保留其它记录的同一字段为可见，它不会隐藏列本身`width` (针对 `editable`)在列表中没有数据时， 列宽可由设置这一属性来进行强制。该值可为绝对宽度(如‘100px’)，或相对权重 (如 ‘3’, 表示该列比其它列大3倍)。注意在列表中有记录时，我们让浏览器自动根据其内容来适配列宽，因此会忽略这一属性。如果列表视图`可编辑`， 任意来自 [表单视图](#reference-views-form)的属性仍然有效并将会在设置行内表单视图时使用。在子视图列表中 (表单视图中显示的One2many/Many2many )，`column_invisible` 属性可用于根据父级对象隐藏列。`<field name="product_is_late" attrs="{'column_invisible': [('parent.has_late_products', '=', False)]}"/>`在对列表视图分组时，数值字段会进行聚合并按组显示。同时，如果在组中有过多记录，会在组的行中显示分页。因此，在列表视图可以分组时在最后一列中带有数值字段不是一种良好实践 (对于表单视图中的x2manys字段不存在问题： 它们无法进行分组)。

- `groupby`

  字段分组记录时定义当前视图的自定义头部（带按钮）。对于可用作修饰符的 `groupby` 内部还可添加`field`。因此这些字段属于many2one comodel。这些额外字段会进行批量获取。`name`（当前模型）many2one字段的名称。自定义状况会在对这一字段名视图分组时显示。`<groupby name="partner_id">  <field name="name"/> <!-- name of partner_id -->    <button type="edit" name"edit" string="Edit/>    <button type="object" name="my_method" string="Button1"      attrs="{'invisible': [('name', '=', 'Georges')]}"/> </groupby>`可定义一个特殊按钮(`type="edit"`) 来打开many2one 表单视图。

- `control`

  对当前视图定义自义控件。这在父级 `tree` 视图位于One2many 内部时具有意义。不支持任何属性，但可以有子标签：`create` 添加按钮来在当前列表中新建元素。如果定义了任意`create` ，它将重写默认的“add a line” 按钮。支持以下属性：`string` (必传)按钮中显示的文本。`context`在获取新记录的默认值时该上下文会合并到已有上下文中。例如它可用于重载默认值。以下示例会通过替换3个新按钮来重载默认的 “add a line”按钮：“Add a product”, “Add a section” 和 “Add a note”.“Add a product” 会设置字段 ‘display_type’ 为其默认值。两个其它按钮会分别设置字段的‘display_type’ 为 ‘line_section’ 和 ‘line_note’.`<control>  <create    string="Add a product"  />  <create    string="Add a section"    context="{'default_display_type': 'line_section'}"  />  <create    string="Add a note"    context="{'default_display_type': 'line_note'}"  /> </control>`



## 表单

表单视图用于显示单条记录的数据。它们的根元素是`<form>`。它们由带有额外结构和语法组件的常规 [HTML](http://en.wikipedia.org/wiki/HTML)所组成。

### 结构组件

结构组件提供结构或具有少量逻辑的“视觉”功能。它们在表单视图中用途元素或元素集。

- `notebook`

  定义标签区块。每个标签通过一个 `page` 子元素进行定义。page可包含如下属性：`string` (必传)标签的标题`accesskey` HTML [accesskey](http://www.w3.org/TR/html5/editing.html#the-accesskey-attribute)`attrs`基于记录值的标准动态属性注意`notebook` 不应放在`group`中

- `group`

  用于定义表单中的列布局。默认，组定义2列而最直接的组的子元素接收单列。组的直接`field` 子元素默认显示一个标签（label），而标签 和字段本身的colspan分别为1。`group` 中的列数可使用`col` 属性进行自定义，元素所接收的列数可使用 `colspan`进行自定义。子元素横向布局 (在修改行之前尝试填充下一列)。组可拥有 `string` 属性，显示为组的标题

- `newline`

  仅在 `group`元素中可用， 提前结束当前行并立即切换到新行 (不预先填入任何剩余列)

- `separator`

  小的横向空格，带 `string` 属性时作为区块的标题

- `sheet`

  可用于 `form`的直接子元素，针对更窄或更具响应式的表单布局

- `header`

  与`sheet`拼接，在表单上方提供全宽的位置，通常用于显示工作流按钮及状态小组件

### 语法组件

语法组件嵌入Odoo系统中并与系统交互。可用的语法组件有：

- `button`

  对Odoo系统进行调用，类似于[表单视图按钮](#reference-views-list-button)。此外，可指定如下属性：`special`对于在对话框中打开的表单视图：用 `save` 来保存记录及关闭对话框，用 `cancel` 关闭会话框但不进行保存。

- `field`

  渲染(并在可能时允许编辑)当前记录的单个字段。支持在表单视图中多次使用字段并且这些字段可接受针对‘invisible’ 及 ‘readonly’修饰符的不同值。 但是，在存在多个不同‘required’修饰符值的字段时不能保证行为。 field节点的可用属性有：`name` (必须)待渲染的字段名称`widget`根据其类型(如 [`Char`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Char), [`Many2one`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Many2one))默认渲染的字段。`widget`属性允许使用不同的渲染方法及上下文。`options`指定字段组件（包含默认组件）配置项的JSON对象`class`对所生成的元素设置的HTML类，常用的字段类有：`oe_inline`防止常见的字段后换行`oe_left`, `oe_right`[浮动](https://developer.mozilla.org/en-US/docs/Web/CSS/float)字段到相应的方向`oe_read_only`, `oe_edit_only`仅在相应的表单模式中显示字段`oe_no_button`避免在[`Many2one`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Many2one)中显示导航按钮`oe_avatar`针对图片字段，显示图片为“头像” (方形，最大90x90，有一些图片修饰)`groups`仅对特定用户显示该字段`on_change`在编辑字段时调用指定方法，可为用户生成其它字段或显示警告从8.0版本开始淘汰: 对模型使用 [`odoo.api.onchange()`](https://alanhou.org/odoo-13-orm-api/#odoo.api.onchange)`attrs`基于记录值的动态元信息参数`domain`仅针对关联字段，在显示已有选区记录时过滤应用对象`context`仅针对关联字段，在获取可用值时传递的上下文`readonly`在只读和编辑模式下均显示该字段，但不可编辑`required`生成错误并在字段没有值时阻止记录的保存`nolabel`不自动显示字段标签，只有在字段是`group`的直接子元素时才有意义`placeholder`在*空字段*中显示的帮助消息。 可在复杂表单中替换字段标签。不应数据的示例，因为用户会混淆占位符与所填写的字段`mode`针对 [`One2many`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.One2many), 字段关联字段所使用的显示模式（视图类型）。`tree`, `form`, `kanban` 或 `graph`之一。默认为`tree` (一种列表显示)`help`在悬浮到字段或其标签上时对用户显示的提示信息`filename`针对二进制字段，提供文件名的关联字段名称`password`表明为一个存储密码的[`Char`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Char)字段，不应显示其数据`kanban_view_ref`针对在移动环境中从when selecting records from m2o/m2m选取字段时打开的指定看板视图



## 图形

图形视图用于对一些记录或记录组的聚合进行可视化。其根元素是 `<graph>` ，接收如下属性：

- `type`

  `bar` (默认), `pie` 及 `line`为图形可使用的类型

- `stacked`

  仅用于 `柱状`图。若存在且设置为 `True`，在组内堆叠柱形图

在图形视图中允许使用的唯一元素是 `field` ，可拥有如下属性：

- `name` (必填)

  在视图中使用的字段名。用于分组(而非聚合)

- `title` (可选)

  在图形之上显示的字=符串。

- `type`

  表明该字段是否应用于分组条件或作为组内的聚合值。可用的值有：`row` (默认)通过指定字段分组。所有图形类型至少支持一组级别的分组，有些可支持更多种。`col`在图形视图中授权，但仅供透视表使用`measure`在组内供聚合的字段

- `interval`

  针对日期和日期时间字段，由指定间隔(`day`, `week`, `month`, `quarter` 或 `year`) 进行分组，而非具体日期时间 (固定的秒级解析) 或日期 (固定的日级解析)分组。

聚合项自动由模型字段生成；仅使用可聚合的字段。这些聚合项还会对字段的字符串进行按字母排序。

> **⚠️警告**
>
> 图形聚合对数据库内容执行，非存储的函数字段无法在图形视图中使用。



## 数据透视表

透视表视图用于可视化聚合为[透视表](http://en.wikipedia.org/wiki/Pivot_table)。其根元素为 `<pivot>` ，可接收如下属性：

- `disable_linking`

  设置为`True` 来删除表中单元格对列表视图的链接

- `display_quantity`

  设置为`true`来显示默认的数量列。

- `default_order`

  聚合项的名称及用于视图中默认排序的顺序 (asc 或 desc) 。`<pivot default_order="foo asc">   <field name="foo" type="measure"/> </pivot>`

在透视表视图中所允许的唯一元素是 `field`，可使用如下属性：

- `name` (必填)

  在视图中使用字段名。 用于分组 (而非聚合)

- `string`

  在透视表视图中用于显示字段的名称，重载该字段默认的python 字符串属性。

- `type`

  表明字段是否应用作分组条件或在组内作为一个聚合值。可用的值有：`row` (默认)按指定字段分组，每个组有各自的行。`col`创建列级分组`measure`f在组内聚合的字段`interval`用于日期和日期日间字段， 由指定间隔 (`日`, `周`, `月`, `季` 或 `年`)分组，而非对指定日期时间 (固定的秒级解析) 或日期 (固定的日级解析)分组。

- `invisible`

  若为true，该字段不会在活跃的聚合项或可选聚合项 (对于聚合无意义的字段很有用，如不同的单位字段 e.g. € 和 $)中显示。

聚合项通过模型字段自动生成； 仅使用可聚合字段。这些聚合项还对字段的字符串按字母排序。

**⚠️警告**
类似图形视图，透视表仅对数据库内容进行聚合，也就是说非存储的函数字段不能在透视表视图中使用。

在透视表视图中 `字段`可以有一个`widget` 属性来描述其格式。组件应为一个字段格式化器，其中最有意义的有 `date`, `datetime`, `float_time`和 `monetary`。

例如时间表透视表视图可定义为：

```
<pivot string="Timesheet">
    <field name="employee_id" type="row"/>
    <field name="date" interval="month" type="col"/>
    <field name="unit_amount" type="measure" widget="float_time"/>
</pivot>
```



## 看板

看板视图是一种 is a [看板](http://en.wikipedia.org/wiki/Kanban_board)可视化：它将记录显示为“卡片”，介于 [列表视图](#reference-views-list)和不可编辑的[表单视图](#reference-views-form)之间。记录可在列中分组，用于工作流可视化或操控(如任务或工作进程管理)，或不进行分组 (仅用于可视化记录)。

看板视图会加载并显示最多10列。之后的任何列都会被关闭 (但可仍可由用户打开)。

看板视图的根元素是 `<kanban>`，它使用如下属性：

- `default_group_by`

  是否应对看板分组，如未通过动作或当前搜索指定分组。应为无其它指定分组时的分组字段名。

- `default_order`

  若用户没有分组字段的话为用于（通过列表视图）对卡片进行的排序

- `class`

  对看板视图的根HTML元素添加的HTML类

- `examples`

  若在[KanbanExamplesRegistry](https://github.com/odoo/odoo/blob/99821fdcf89aa66ac9561a972c6823135ebf65c0/addons/web/static/src/js/views/kanban/kanban_examples_registry.js)中设置键，列设置的示例会在分组看板视图中可用。 [这里](https://github.com/odoo/odoo/blob/99821fdcf89aa66ac9561a972c6823135ebf65c0/addons/project/static/src/js/project_task_kanban_examples.js#L27)是一个如何定义这些设置的示例。

- `group_create`

  “新增列” 栏是否可见。默认为: true。

- `group_delete`

  通过上下文菜单是否可删除组。默认为: true。

- `group_edit`

  通过上下文菜单是否可编辑组。默认为：true。

- `archivable`

  如在模型中定义了 `active` 字段，记录是否属于可存档/可还原的列。默认为: true。

- `quick_create`

  是否可不切换表单视图就创建记录。默认，在由 many2one, selection, char 或 boolean字段分组时启用`quick_create` ，否则禁用。

- `records_draggable`

  是否可在看板进行分组时拖拽记录。默认为: true。设置为 `true` 来保持启用它，设为`false` 来保持禁用。

该视图元素可用的子元素有：

- `field`

  声明在看板*逻辑*中使用的字段。如字段仅是在看板视图中显示，无需进行预声明。可用属性有：`name` (必填)待获取的字段名

- `progressbar`

  声明放在看板列顶部的进度条元素。可用的属性有：`field` (必填)在进度条中用于子分组列记录值的字段名称`colors` (必填)将以上字段值映射到“danger”, “warning”, “success” 或 “muted”颜色的 JSON`sum_field` (可选)对列记录进行汇总并显示在进度条旁的字段名称（若省略，则显示记录的总数）

- `templates`

  定义[QWeb](https://alanhou.org/odoo-13-qweb/#reference-qweb)模板的列表。卡片定义为保持清晰可分成多个模板，但看板视图必须定义至少一个根模板 `kanban-box`，它会对每条记录渲染一半个多。看板视图使用最标准的[javascript qweb](https://alanhou.org/odoo-13-qweb/#reference-qweb-javascript) 并提供如下上下文变量：`widget`当前`KanbanRecord()`，可用于获取一些元信息。这些方法也在模板上下文中可用，并且无需通过`widget`访问`record`带有所请求字段的对象作为其属性。每个字段有两具属性 `value` 和 `raw_value`，前者根据当前用户参数格式化，后者是来自 [`read()`](https://alanhou.org/odoo-13-orm-api/#odoo.models.Model.read) 的直接值(除[按用户所在地格式化的](https://github.com/odoo/odoo/blob/a678bd4e/addons/web_kanban/static/src/js/kanban_record.js#L102)日期和日期时间外)`context`,当前上下文，来自动作，以及表单视图中内嵌看板视图时的one2many 或 many2many字段`user_context`字面意思`read_only_mode`字面意思`selection_mode`当看板视图在移动环境中打开选择记录的m2o/m2m字段时设置为true在移动环境中点击m2o/m2m字段会打开看板视图按钮和字段虽然大部分看板模板都是标准的[QWeb](https://alanhou.org/odoo-13-qweb/#reference-qweb), 看板视图对`field`, `button` 和 `a` 元素进行了特殊处理：默认使用其格式化值替换字段，除非指定了`widget` 属性，这时它们的渲染和行为将取决于相应的组件。可用的值有(还有未列出的):`handle`对于对记录进行排序的 `序列` (或`整型`)字段，允许拖拽记录进行重新排序。带有`type`属性的按钮及链接执行Odoo相关运算而非标准的 HTML函数。类型有：`action`, `object`[Odoo按钮](#reference-views-list-button)的标准行为，可使用磊部与标准Odoo按钮相关的属性。`open`以只读模式在表单视图中打开卡片记录`edit`以可编辑模式在表单视图中打开卡片记录`delete`删除卡片记录并删除该卡片

如果你需要扩展看板视图，参见：js:class::`JS API <KanbanRecord>`.



## 日历视图

日历视图按天、周、或月日历显示记录为事件。它们的根元素是 `<calendar>`。日历视图中可用的属性有：

- `date_start` (必填)

  存储事件开始日期的记录字段名称

- `date_stop`

  存储事件结束日期的记录字段名称，若提供了`date_stop` 则记录在日历中可（通过拖拽）进行移动

- `date_delay`

  `date_stop`的另一种实现，提供了事件时长而非结束日期(单位：天)

- `color`

  name of a record field to use for 针对*颜色分段*所使用的记录字段名。相同颜色区的记录在日历中被分配相同的高亮颜色，颜色半随机分配。在侧边栏显示可见记录的display_name/avatar

- `form_view_id`

  在用户创建或编辑事件时打开的视图。注意如果未设置这个属性，日历视图会回到当前所存在的动作中的表单视图 id。

- `event_open_popup`

  如果‘event_open_popup’选项设置为true，那么日历视图会在表单视图对话框中打开事件（或记录）。否则，它会在新的表单视图中打开事件(带有do_action)

- `quick_add`

  在点击时启用快速事件创建：仅向用户询问 `name` 并尝试使用该名称和点击事件时间新建事件。若快速创建失败则回到完整表单对话框

- `all_day`

  记录中的布尔字段名称，表明相应事件是否标记为整天 (时长就不再相关)

- `mode`

  在加载日历时的默认显示模式。可用的属性有： `day`, `week`, `month`

- `<field>`

  声明在看板逻辑中聚合或使用的字段。字段是否仅在日历卡片中显示。字段可以有额外的属性：`invisible`使用 “True”来隐藏卡片中的值`avatar_field`仅针对x2many字段，用于在卡片中显示avatar而非display_name`write_model` 及 `write_field`可以添加一个过滤器并在所定义模型中保存结果，在侧边栏中添加过滤器

- `templates`

  定义[QWeb](https://alanhou.org/odoo-13-qweb/#reference-qweb) 模板`calendar-box`。为保持清晰卡片定义可分割成多个模板，每条记录渲染一次。看板视图使用最标准的 [javascript qweb](https://alanhou.org/odoo-13-qweb/#reference-qweb-javascript) 并提供如下上下评论变量：`widget`当前 `KanbanRecord()`可用于获取一些元信息。这些也在模板上下文中可用并且无需通过`widget` 访问，`getColor` 来将颜色整型`getAvatars` 在不可见字段头像图片`displayFields`列表中进行转化`record`带有所有请求字段作为属性的对象。每个字段有两个属性`value` 和 `raw_value``event`日历事件对象`format`通过用户参数转化值为可读字段串的格式方法`fields`所有模型字段参数的定义`user_context`字面含义`read_only_mode`字面含义



## 甘特图

甘特视图可显示甘特图(用于计划任务)。

甘特视图的根元素是 `<gantt/>`，它没有子元素但可以接收如下属性：

- `date_start` (必填)

  为每条记录提供事件起始datetime的字段名称。

- `date_stop` (必填)

  为每条记录提供终止时长的字段名称。

- `color`

  用于根据值显示颜色块的字段名称。

- `decoration-{$name}`

  允许根据相应记录的属性修改行级文本样式。值为Python表达式。针对每条记录，表达式使用记录的属性作为上下文值运行，若为 `true`，相应的样式应用于行。其它的上下文值有`uid` (当前用户有的id )及 `current_date` (`yyyy-MM-dd`格式的当前日期字符串)。`{$name}` 可为任意[bootstrap上下文颜色 ](https://getbootstrap.com/docs/3.3/components/#available-variations)(`danger`, `info`, `muted`, `primary`, `success` 或 `warning`).

- `default_group_by`

  对任务分组的字段名

- `consolidation`

  在记录单元格中显示合并值的字段名

- `consolidation_max`

  以“group by”字段作为键的宜聚网， 以及在单元格中显示为红色之前的最大合并值 (如`{"user_id": 100}`)

- `consolidation_exclude`

  描述任务是否应从合并中排除的字段名，若设置为 true，它在合并行中显示条状区域

- `create`, `edit`, `plan`

  允许通过设置相应的属性为 `false`来*禁用*视图中相应的动作。 若启用了 `create` ，会在每次悬停在新建记录的位置处显示“+”按钮，而如果启用了 `edit` ，会在时间区的计划记录处显示“放大镜”按钮。

- `offset`

  根据规模，添加到今天计算默认时期的单位数量。例如： default_scale week中的+1偏移会打开下周的甘特视图，而 default_scale month中-2的偏移会打开2个月之前的甘特视图。

- `progress`

  为记录的事件提供完成百分比的字段名，在0 和 100之间

- `string`

  甘特图的标题

- `precision`

  指定每个区间中版块精度的JSON对象。区间 `day` 可选的值有are (默认: `hour`):`hour`: 记录变为整点 (如e： 7:12 变为 8:00)`hour:half`: 记录时间按半小时变化 (如: 7:12 变为 7:30)`hour:quarter`: 记录时间按一刻钟变化(如: 7:12 变为 7:15)对于 `week` 区间可用的值有 (默认: `day:half`):`day`: 记录时间按整天(ex: 7:28 AM 变为 11:59:59 PM)`day:half`: 记录时间r 按每半天 (ex: 7:28 AM 变为 12:00 PM)对于 `month`区间可用的值有 (默认: `day:half`):`day`: 记录时间按整天 (ex: 7:28 AM 变为 11:59:59 PM)`day:half`: 记录时间按每半天 (ex: 7:28 AM 变为 12:00 PM)`year`区间总是按全天来的。精度属性的示例： `{"day": "hour:quarter", "week": "day:half", "month": "day"}`

- `total_row`

  控制行包含记录总数的行是否显示的布尔值。(默认: `false`)

- `collapse_first_level`

  在通过一个字段分组时控制各行是否折叠的布尔值。 (默认: `false`, 在通过两个字段分组时开始折叠)

- `display_unavailability`

  标记由甘特视图内可用模型的函数 `gantt_unavailability` 返回的日期的布尔值。记录仍可在它们之中计划任务，但它们的不可用性会进行视觉展示。(默认: `false`)

- `default_scale`

  渲染视图时的默认区间。可用的传下有 (默认: `month`):`day``week``month``year`

- `scales`

  针对这个视图允许的区间的逗号分隔列表。默认允许所有区间。这个列表中可用的区间值，参见 `default_scale`。

- `templates`

  定义 [QWeb](https://alanhou.org/odoo-13-qweb/#reference-qweb) 模板 `gantt-popover`，在用户悬停在甘特视图的一个记录上时使用。甘特视图使用最标准的 [javascript qweb](https://alanhou.org/odoo-13-qweb/#reference-qweb-javascript) 并提供如下上下文变量：`widget`当前 `GanttRow()`，可用于获取一些元信息。转化颜色整型的 `getColor`方法也可以在不使用`widget`的模板上下文中直接使用。`on_create` 如在点击视图中添加按钮时指定，它不打开通用对话框，而是启动一个客户端动作，其中包含动作的xmlid (eg: `on_create="%(my_module.my_wizard)d"`

- `form_view_id`

  在用户创建或编辑记录时打开的视图。注意如果未设置这个属性，甘特视图会在当前动作中存在表单视图时用回它的 id。

- `thumbnails`

  如果组是一个关联字段的话这允许显示组名旁的缩略图。应是一个python字典，键是活跃模型中的字段名。值是在关联模型中存储缩略图的字段名称。例： 引用res.users的有user_id字段的任务。res.users模型有一个存储头像的字段图像，那么：

```
<gantt
  date_start="date_start"
  date_stop="date_stop"
  thumbnails="{'user_id': 'image_128'}"
>
</gantt>
```

会在通过user_id分组时在名称旁显示用户的头像



## 图表

图表（diagram）视图可用于记录的直接图表。根元素为`<diagram>` 且不接收任何属性。

图表视图中可用的子元素有：

- `node` (必填, 1)

  定义图表的节点。其属性有：`object`节点的Odoo模型`shape`类似[列表视图](#reference-views-list)中的颜色和字体的条件形状映射。 唯一有效的形状是 `矩形` (默认开头是椭圆)`bgcolor`类似`shape`，但按条件对节点映射背景色。默认背景色是白色，唯一有效的替代值是`grey`。

- `arrow` (必填, 1)

  定义图表的边。属性有：`object` (必填)边的Odoo模型`source` (required)指向边的源节点记录的边模型的[`Many2one`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Many2one) 字段`destination` (required)指向边的目标节点记录的边械的[`Many2one`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Many2one) 字段`label`（带引号字符串）属性Python列表 。相应属性的值会进行合并并显示为边的模型。

- `label`

  图表的解释提示，`string`属性定义提示的内容。每个 `label` 输入出为图表状况中的段落，非常易见但不添加任何强调。



## 仪表盘

类似透视表和图形视图，仪表盘视图用于显示聚合数据。但仪表盘可以内嵌子视图，这让其可以对指定数据集有更完整、美观的展示。

### ⚠️警告

仪表盘视图仅在Odoo企业版中可用。

仪表盘视图可以显示子视图、一些（某个域）字段的聚合或甚至一些公式 (包含一个或更多聚合的表达式)。例如，下面是一个简单的仪表盘：

```
<dashboard>
    <view type="graph" ref="sale_report.view_order_product_graph"/>
    <group string="Sale">
        <aggregate name="price_total" field="price_total" widget="monetary"/>
        <aggregate name="order_id" field="order_id" string="Orders"/>
        <formula name="price_average" string="Price Average"
            value="record.price_total / record.order_id" widget="percentage"/>
    </group>
    <view type="pivot" ref="sale_report.view_order_product_pivot"/>
</dashboard>
```

该仪表盘的根元素是<dashboard>, 它不接收任何属性。

在仪表盘视图中有5种可用的标签类型：

- `view`

  声明一个子视图。可接受的属性有：`type` (必传)子视图的类型。例如 *graph* 或 *pivot。*`ref` (可选)视图的一个xml id。如未给定，会使用模型的默认视图。`name` (可选)定义这个元素的字符串。它多用作一个xpath的目标。

- `group`

  定义列布局。这实际非常类型表单视图中的组元素。可用的属性有：`string` (可选)将会以组标题显示 的描述。`colspan` (可选)这一group标签中的子列数。 默认为6。`col` (可选)这一group标签横跨的列数 (仅在另一个组中有意义)。默认值为 6。

- `aggregate`

  声明一个聚合。这是对当前域给定字段的聚合值。注意聚合应用在group标签内 (否则不会正确应用样式)。可接受的属性有：`field` (必填)用于计算聚合的字段名。可用的字段属性有：`integer` (默认的组运算符是求和)`float` (默认组运算符是求和)`many2one` (默认组运算符是唯一计数)`name` (必填)标识这一聚合的字符串 (用于方程)`string` (可选)在值上显示的短描述。如未指定，会用回字段字符串。`domain` (可选)我们想要聚合的一组记录的额外限制。这个域会与当前域合并。`domain_label` (可选)在用户点击带有域的聚合时，它会添加到搜索视图中作为一个 facet。针对这个显示的字符串可通过这个属性自定义。`group_operator` (可选)在聚合值 (see https://www.postgresql.org/docs/9.5/static/functions-aggregate.html)时使用的有效postgreSQL聚合函数标识符。如未提供，默认，使用字段定义中的 group_operator。如group_operator 值为“”时不会实现任何字段值聚合。特殊聚合函数`count_distinct` (odoo中定义)也可在这里使用`<aggregate name="price_total_max" field="price_total" group_operator="max"/>``col` (可选)这个标签横跨的列数 (仅在组内有意义)。默认为1。`widget` (可选)格式化该值的组件 (类似字段中的widget属性)。例如，monetary。`help` (可选)在提示信息中显示的帮助消息 (等价于python中字段的帮助信息)`measure` (可选)这个属性是描述度量的字段，在点击聚合时在图形和透视表视图中使用。特殊值 __count__ 可用于计数度量。`<aggregate name="total_ojects" string="Total Objects" field="id" group_operator="count" measure="__count__"/>``clickable` (可选)表明聚合是否可点击的布尔值(默认为true)。点击可点击的聚合会使用子视图改变度量并在搜索视图中添加域属性（若存在）的值。`value_label` (可选)放在聚合值压路机的字符串。例如，它可用于表示聚合值的度量单位。

- `formula`

  声明所获取值。方程为通过聚合所计算的值。注意类似聚合，方程可在标签中使用 (否则样式会正确地应用)。可用的属性有：`value` (必填)将进行运行的字符串表达式，其中有内置的python运算器(在网页客户端中)。每个聚合可在`record`变量的上下文中使用。例如， `record.price_total / record.order_id`.`name` (可选)标识该方程的字符串`string` (可选)A short description that will be displayed above the formula.`col` (optional)The number of columns spanned by this tag (only makes sense inside a group). By default, 1.`widget` (optional)A widget to format the value (like the widget attribute for fields). For example, monetary. By default, it is ‘float’.`help` (optional)A help message to dipslay in a tooltip (equivalent of help for a field in python)`value_label` (optional)A string put on the right of the formula value. For example, it can be useful to indicate the unit of measure of the formula value.

- `widget`

  声明用于显示该信息的特殊组件。这种机制类似于表单视图中的组件。可接受的属性有：`name` (必填)识别应实例化组件的字符串。视图会查看`widget_registry` 来获取相应的类。`col` (可选)通过这个标签所跨的列数 (仅在组内有意义)。默认值为1。



## 同期群

同期群（cohort）视图用于显示和理解一个时期中一些数据的变化。如，设想有一个给定的业务，客户可订阅一些服务。同期群视图可以显示每月订阅的总数，并研究哪个用记会离开服务（流失）。在点击单元格时，同期群视图会将你重定向到一个新的动作，其中你仅能看到该单元格时间间隔中的记录，这个动作包含包含一个列表视图和一个表单视图。

### ⚠️警告

同期群视图仅在Odoo企业版中可用。

默认同期群视图会使用与动作中定义的相同列表和表单视图。可以向动作的上下文传递一个列表视图和表单视图来设置/重载所使用的视图 (上下文键为`form_view_id` 和 `list_view_id`)

例如，以下是一个很简单的同期群视图：

```
<cohort string="Subscription" date_start="date_start" date_stop="date" interval="month"/>
```

同期图视图的根元素是<cohort>, 它接受如下属性：

- - `string` (必填)

    标题，应描述视图

- - `date_start` (必填)

    有效日期或日期时间字段。这个字段由视图解析为记录的开始日期

- - `date_stop` (必填)

    有效的日期或日期时间字段。该字段由视图解析为记录的结束日期。这是决定流失的字段。

- - `mode` (可选)

    描述模式的字段。它应为‘churn’（流失） 或 ‘retention’（留存） (默认). 流失模式从 0%开始必随时间增长，而留存则从100%开始并随时间下降。

- - `timeline` (可选)

    描述时间轴的字符串。应为‘backward’ 或 ‘forward’ (默认)。前向时间轴会从date_start 到 date_stop显示数据，而后向时间轴 会从date_stop 到 date_start显示数据 (在date_start未来的一个时间或是大于date_stop时)。

- - `interval` (可选)

    描述时间间隔的字符串。应为‘day’, ‘week’, ‘month’’ (默认) 或 ‘year’.

- - `measure` (可选)

    可进行聚合的字段。该字段可用于计算每个单元格的值。如未设置，同期群视图会计算所发生的次数。



## 活动

活动视图用于显示关联到记录的活动。数据在图表中显示，记录为行、活动类型为列。每行的第一个单元格显示一个表示相应记录的卡片 (可自定义， 参见 `templates`，相当类似于[看板](#reference-views-kanban))。在点击其它单朝夕相处时，会显示该记录同一类型所有活动的详细描述。

### ⚠️警告

活动视图仅在安装了`mail`模块时可用，针对那些继承了`mail.activity.mixin`的模型。

活动视图的根元素是 `<activity>`，它接受如下属性：

- - `string` (必填)

    标题，应描述视图

该视图可用的子元素有：

- `field`

  声明活动逻辑中使用的字段。如该字段仅在活动视图中显示，无需预先声明。可用的属性有：`name` (必填)要获取的字段名称

- `templates`

  定义 [QWeb](https://alanhou.org/odoo-13-qweb/#reference-qweb)模板。 卡片定义为保持清晰可分割成多个模板，但活动视图必须至少定义一个根模板`activity-box`，它将为每条记录进行一次渲染。活动视图使用最标准的[javascript qweb](https://alanhou.org/odoo-13-qweb/#reference-qweb-javascript) 并提供如下上下文变量 (参见 [看板](#reference-views-kanban)了解更多详情)：`widget`当前 `ActivityRecord()`，可用于获取一些元信息。这些方法也可以直接在模板上下文中获取且无需通过`widget`访问`record`将所有请求字段作为其属性的对象。每个字段有两个属性 `value` 和 `raw_value`



## 搜索

搜索视图是此前那些视图类型的分解，在其中不显示内容，虽然它们应用于具体的模型，却用于过滤其它视图的内容(通常为 [列表视图](#reference-views-list) 或 [图形视图](#reference-views-graph)等的聚合视图)。除用例的这种不同外，它们的定义方式相同。

搜索视图的根元素是 `<search>`。它不接收属性。

搜索视图中可用的子元素有：

- `field`

  字段定义含用户提供值的域或上下文。在生成搜索视图时，字段域由彼此和使用**AND**的过滤器组成。字段可有如下属性：`name`待过滤的字段名`string`字段的标签`operator`默认，字段生成表单作用域`[(*name*, *operator*, *provided_value*)]`，其中`name` 是字段名，而 `provided_value` 是由用户所提供的值，可进行过滤或转化(如用户应提供一个选择字段值的标签，而非值本身)。`operator` 属性允许根据字段类型重载默认运算符 (如针对浮点字段的`=` 、针对字符字段的`ilike` )`filter_domain`用作字段搜索作用域的完整域，可使用 `self` 变量来在自定义域中注入所提供的值。可用于生成比`运算符` 本身灵活得多的作用域 (如同时对多个字段的搜索)如果同时提供了`operator` 和 `filter_domain` ，`filter_domain` 作为主导。`context`允许添加上下文键，包含用户所提供的值(对`domain`可用的对 `self`变量一样可用，针对[`Many2one`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Many2one)为一个数组，如 `[id_1, id_2]` )。默认，字段不生成作用域。作用域和上下文是包含在内的，如指定了`context` 会同时生成两者。若要仅生成上下文值，设置 `filter_domain` 为一个空列表：`filter_domain="[]"``groups`让字段仅有特定用户可用`widget`对字段使用具体的搜索组件(标准Odoo 8.0中的唯一用例是针对[`Many2one`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Many2one)的 `selection` 组件)`domain`若字段可提供自动完成 (e.g. [`Many2one`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Many2one))，过滤可能的完成结果。

- `filter`

  过滤器是搜索视图中预定义的切换，仅能启用或禁用。其主要目的是向搜索上下文添加数据(对搜索/过滤的数据视图传递的上下文)，或向搜索视图添加版块。过滤器可拥有如下属性：`string` (必填)过滤器的标签`domain` (可选)一个Odoo [作用域](https://alanhou.org/odoo-13-orm-api/#reference-orm-domains)，会添加到动作作用域中作为搜索作用域的一部分。`date` (可选)`date` 或 `datetime`类型的字段名称。使用这一属性具有在过滤器菜单的子菜单中创建一组过滤器的效果。所提供的过滤器与时间相关，但在控制面板初始化时运行作用域的层面不是动态的。例：`<filter name="filter_create_date" date="create_date" string="Creation Date"/>`以上的示例允许在以下时间区间中通过创建日期字段值轻松搜索 (假定当前月分为2019年8月)。`Create Date >  August  July  June  Q4  Q3  Q2  Q1 --------------  2019  2018  2017`允许多个选择的选择。`default_period` (可选)仅对拥有非空`date` 属性的过滤器有意义，在视图初始化时若激活默认过滤器组中过滤器激活哪个时间区间。若未提供，默认使用‘this_month’。在以下选项中进行选择： today, this_week, this_month, last_month, antepenultimate_month, fourth_quarter, third_quarter, second_quarter, first_quarter, this_year, last_year, antepenultimate_year。例：`<filter name="filter_create_date" date="create_date" string="Creation Date" default_period="this_week"/>``context`一个Python字典，合并入动作的作用域来生成搜索作用域`group_by` 键可用于定义在‘Group By’中可使用的分组。‘group_by’的值可为一个有效的字段名。`<filter name="groupby_category" string="Category" context = {'group_by': 'category_id'}/>`以上定义的groupby允许按分类分组数据。在字段类型是`date` 或 `datetime`的时候，该过滤器生成Group By菜单的子菜单，其中可使用如下的间隔选项： day, week, month, quarter, year。在过滤器位于视图初始化时激活的默认过滤器组中是，记录默认按月进行分组。这可使用像下例中的‘date_field:interval’句法进行修改。例：`<filter name="groupby_create_date" string="Creation Date" context = {'group_by': 'create_date:week'}/>``name`过滤器的逻辑名称，可用于[默认启用它](#reference-views-search-defaults)，也可用作[继承钩子](#reference-views-inheritance)`help`过滤器的更长解析文本，可作为提示信息进行显示`groups`让过滤器仅对指定用户可用在7.0版本中新增。（不带有分隔它们的非过滤器的）过滤器的序列可被看作包含组合：它们由 `OR`或常见的`AND`进行组合，如`<filter domain="[('state', '=', 'draft')]"/> <filter domain="[('state', '=', 'done')]"/>`如果同时选中这两个过滤器，会选中那些`state` 是 `draft` 或 `done`的记录，但`<filter domain="[('state', '=', 'draft')]"/> <separator/> <filter domain="[('delay', '<', 15)]"/>`如果同时选中这两个过滤器，会选中 `state` 是 `draft` **且** `delay` 小于15的记录

- `separator`

  可用于在简单搜索视图中分隔过滤器组

- `group`

  可用于分隔过滤器组，在搜索视图中比 `separator` 可读性更强

- `searchpanel`

  允许在任意多个记录视图的左侧显示搜索面板。默认，列表和看板视图启用了搜索面板。搜索面板可以对其它带有如下属性的视图进行激活：`view_types` 启用搜索页面的视图类型的逗号分隔列表 默认: ‘tree,kanban’这个工具允许根据给定字段快速过滤数据。这些字段指定为带有`field`标签名以及如下属性的 `searchpanel` 的直接子元素：`name` (mandatory) the name of the field to filter on`select` 决定行为和显示。可用的值有 `one` (默认) 最多可先用一个值。支持的字段类型有many2one 和 selection`multi` 可选择（复选框）的一些值。支持的字段类型为many2one, many2many 和 selection。`groups`: 限定指定用户`string`: 决定显示的标签`icon`: 指定使用的图标`color`: 决定图标的颜色其它的可选属性在`multi` 的情况下可用：`domain`: 决定comodel记录要满足的条件。作用域可用于表达对另一个搜索面板的另一个（带select=”one”）字段的依赖。思考`<searchpanel>  <field name="department_id"/>  <field name="manager_id" select="multi" domain="[('department_id', '=', department_id)]"/> <searchpanel/>`在以上的示例中， 屏幕上可用的manager_id (manager 名称) 的值的范围取决于 `department_id`字段当前选择的值。`groupby`: comodel的字段名 (仅对many2one 和 many2many字段可用)。值将通过该字段分组。`disable_counters`: 默认为false。若设置为true，计数器不会进行计算。实现了这一功能来防止性能太并的问题。另一种解决性能问题的方法是相应地重载 `search_panel_select_multi_range` 方法。



### 搜索默认值

搜索字段和过滤器可使用`search_default_*name*` 键使用动作的`context`进行配置。对于字段，值应为在字段中设置的值，对于过滤器，为布尔值或是一个数字。例如，假定 `foo` 是一个字段，而 `bar` 是动作上下文的过滤器：

```
{
  'search_default_foo': 'acro',
  'search_default_bar': 1
}
```

将自动启用 `bar` 过滤器并搜索对*acro*的`foo` 字段。

数值 (1 和 99之间)可用于描述默认groupby的排序。例如，若 `foo` 和 `bar` 对应两个two groupby

```
{
  'search_default_foo': 2,
  'search_default_bar': 1
}
```

可起到先后激活 `bar` 和 `foo`的效果。



## 地图

此视图可在地图中显示记录并且它们之间路由。记录以小图标表示。它还允许与记录图标绑定的弹窗中的模型字段的可视化。

视图所应用的模型应包含一个res.partner many2one，因为视图依靠res.partner的地址和坐标字段来定位记录。

### ⚠️警告

地图视图仅在Odoo企业版中可用



### Api

此视图使用地点数据平台的 api来获取矢量瓦片(地图的背景)，执行地理转发 (转化地址为一组坐标)及获取路线。该视图实现两个api，一个默认的，openstreet 地图可获取[矢量瓦片](https://wiki.openstreetmap.org/wiki/Tile_data_server)及进行[地理转发](https://nominatim.org/release-docs/develop/)。这个api不要求使用令牌。一旦在通用设置中提供有效[MapBox](https://docs.mapbox.com/api/) 令牌，视图就切换为Mapbox api。这个 api更为快速并允许对路线的计算。令牌可通过[注册](https://account.mapbox.com/auth/signup/)MapBox来获取。



### 结构组件

该视图的根元素是 `<map>` ，允许使用多个属性

- `res_partner`

  包含res.partner many2one。如不提供，视图会致力于创建一个空映射。

- `default_order`

  如果提供字段，视图会重载模型的默认排序。字段必须是模型的一部分，其中视图的应用不来自res.partner

- `routing`

  若为 `true`，则显示记录之间的路线。视图仍需一个有效的MapBox令牌并至少有两条地点记录。(即记录有res.partner many2one 并且 partner 有地址和有效坐标)

在`<map>`元素中允许的唯一元素是 `<marker-popup>`。该元素可包含多个`<field>` 元素。每个这种元素又会解析为地标弹出的一行。字段的属性如下：

- `name`

  显示的字段。

- `string`

  该字符串会在字段的内容前显示。可用作描述。

属性或元素都不是必填，但前面提过如未提供res.partner many2one，视图不会定位记录。

- 例如以下是一个地图：

  `<map res_partner="partner_id" default_order="date_begin" routing="true">    <marker-popup>        <field name="name" string="Task: "/>    </marker-popup> </map>`



## QWeb

QWeb视图是视图的`arch`之内的标准 [QWeb](https://alanhou.org/odoo-13-qweb/#reference-qweb)模板。它们没有指定的根元素。因为 QWeb视图没有特定的根元素，其类型必须显式地指定 (它不能通过 `arch` 字段的根元素进行推导)。

QWeb视图有两个用例：

- 可用作前台模板，其中 [template](https://alanhou.org/odoo-13-data-files/#reference-data-template) 应用作快捷方式。
- 它可用作实际qweb视图 (在动作中打开)，这时应定义为带有显式 `type` (无法推导) 和模型的常规视图。

对基本qweb-as-template的主要添加有：

- qweb-as-view对 `<nav>` 元素有一个特殊情况，包含CSS 类 `o_qweb_cp_buttons`：其内容应为按钮且会进行提取并移动到控制面板的按钮区， `<nav>`本身会被删除，这是对不存在控制面板视图的替代方法

- qweb-as-view渲染对标准qweb上下文添加一些子项：

  - `model`

    qweb视图所绑定的模型

  - `domain`

    由搜索视图所提供的作用域

  - `context`

    由搜索视图所提供的上下文

  - `records`

    对`model.search(domain)`的一个惰性代理 ，如果只希望迭代记录并不执行更复杂的操作(如 分组)则可以使用

- qweb-as-view 还提供额外的渲染钩子：

  - `_qweb_prepare_context(view_id, domain)`准备具体到 qweb-as-view的渲染上下文
  - `qweb_render_view(view_id, domain)` 是由客户端调用的方法并将调用上下文就绪的方法及最终的 `env['ir.qweb'].render()`.

[1] 出于向后兼容考虑

[[2\]](#id1) 在QWeb视图中添加一个护函数进行更简单的匹配： 如果上下文节拥有所有指定类则`hasclass(*classes)` 匹配

[[3\]](#id2) 出于历史原因，它源自树状视图，之后重构为更为表格/列表类型的显示

[4] 或没有模板它是一个继承视图，那么 [它应仅包含xpath元素](#reference-views-inheritance)