# 构建模块

本文来自[Odoo 13官方文档之开发者文档](../README.md)系列文章

### ⚠️警告

本教程要求[已安装了 Odoo](../setting-up/installing-odoo/)

## 启动/停止Odoo服务

Odoo使用客户端/服务端架构，其中的客户户端是通过 RPC 访问 Odoo 服务的浏览器。

业务逻辑和扩展通常由服务端来实现，但可以对客户端添加支持功能（例如新的交互式地图的数据展现）。

要启动服务，只需在shell中调用命令 [odoo-bin](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#reference-cmdline)，必要时为文件添加完整路径：

```
odoo-bin
```

通过在终端按下两次 `Ctrl-C` 键或者通过杀死相应操作系统进程来停止服务。

## 构建Odoo模块

服务端和客户端的扩展都封装在*模块*中，可以选择在数据库中进行加载。

Odoo模块可以向系统添加全新的业务逻辑，或者修改并继承已有的业务逻辑：可以创建模块来向 Odoo 的通用会计支持来添加你所在国家的会计规则，而用另一个模块添加对车队实时可视化的支持。

Odoo的一切始于模块也终于模块。

### 模块的组成

Odoo中的模块可以包含很多元素：

- 业务对象

   在Python类中声明， 这些资源根据配置由 Odoo 自动地进行持久化存储

- 数据文件

  声明元数据（视图或报表）、配置文件（模块参数）、演示数据等的XML或CSV文件 网页控制器

  Handle requests from web browsers

- 静态网页数据

  用于网页界面或网站的图片CSS或javascript文件

### 模块结构

每个模块都是放在*模块目录*下的一个目录。模块目录使用[`--addons-path`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-addons-path)选项来进行指定。

大部分命令行选择也可以通过 [配置文件](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#reference-cmdline-config)来进行设置。

Odoo模块在[模块声明文件](https://alanhou.org/odoo-13-module-manifests/#reference-module-manifest)中进行声明。参见[声明文件文档](https://alanhou.org/odoo-13-module-manifests/#reference-module-manifest)了解更多知识。

模块也是一个带有 `__init__.py` 文件的[Python 包](http://docs.python.org/3/tutorial/modules.html#packages)文件，其中包含对模块中不同 Python文件的重要指示。

例如，如果模块中在单个 `mymodule.py` 文件， `__init__.py` 中可能包含：

```
from . import mymodule
```

Odoo提供一种帮助设置新模块的机制， [odoo-bin](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#reference-cmdline-server) 有一个子命令 [scaffold](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#reference-cmdline-scaffold) 可创建空的模块

```
$ odoo-bin scaffold <模块名> <所放位置>
```

该命令为你的模块创建一个子目录，并自动为模块创建一系列标准文件。其中大部分只包含注释代码或XML。大部分这些文件的使用将在本教程中进行讲解。

### 📝练习

模块创建

使用以上命令行来创建一个空模块Open Academy，并在 Odoo中进行安装。

1. 调用命令 `odoo-bin scaffold openacademy addons`.
2. 调用模块中的声明文件。
3. 暂时不要动其它文件。

*openacademy/__manifest__.py*

```
# -*- coding: utf-8 -*-
{
    'name': "Open Academy",

    'summary': """Manage trainings""",

    'description': """
        Open Academy module for managing trainings:
            - training courses
            - training sessions
            - attendees registration
    """,

    'author': "My Company",
    'website': "http://www.yourcompany.com",

    # Categories can be used to filter modules in modules listing
    # Check https://github.com/odoo/odoo/blob/12.0/odoo/addons/base/data/ir_module_category_data.xml
    # for the full list
    'category': 'Test',
    'version': '0.1',

    # any module necessary for this one to work correctly
    'depends': ['base'],

    # always loaded
    'data': [
        # 'security/ir.model.access.csv',
        'templates.xml',
    ],
    # only loaded in demonstration mode
    'demo': [
        'demo.xml',
    ],
}
```

*openacademy/__init__.py*

```
# -*- coding: utf-8 -*-
from . import controllers
from . import models
```

*openacademy/controllers.py*

```
# -*- coding: utf-8 -*-
from odoo import http

# class Openacademy(http.Controller):
#     @http.route('/openacademy/openacademy/', auth='public')
#     def index(self, **kw):
#         return "Hello, world"

#     @http.route('/openacademy/openacademy/objects/', auth='public')
#     def list(self, **kw):
#         return http.request.render('openacademy.listing', {
#             'root': '/openacademy/openacademy',
#             'objects': http.request.env['openacademy.openacademy'].search([]),
#         })

#     @http.route('/openacademy/openacademy/objects/<model("openacademy.openacademy"):obj>/', auth='public')
#     def object(self, obj, **kw):
#         return http.request.render('openacademy.object', {
#             'object': obj
#         })
```

*openacademy/demo.xml*

```
<odoo>

        <!--  -->
        <!--   <record id="object0" model="openacademy.openacademy"> -->
        <!--     <field name="name">Object 0</field> -->
        <!--   </record> -->
        <!--  -->
        <!--   <record id="object1" model="openacademy.openacademy"> -->
        <!--     <field name="name">Object 1</field> -->
        <!--   </record> -->
        <!--  -->
        <!--   <record id="object2" model="openacademy.openacademy"> -->
        <!--     <field name="name">Object 2</field> -->
        <!--   </record> -->
        <!--  -->
        <!--   <record id="object3" model="openacademy.openacademy"> -->
        <!--     <field name="name">Object 3</field> -->
        <!--   </record> -->
        <!--  -->
        <!--   <record id="object4" model="openacademy.openacademy"> -->
        <!--     <field name="name">Object 4</field> -->
        <!--   </record> -->
        <!--  -->

</odoo>
```

*openacademy/models.py*

```
# -*- coding: utf-8 -*-

from odoo import models, fields, api

# class openacademy(models.Model):
#     _name = 'openacademy.openacademy'

#     name = fields.Char()
```

*openacademy/security/ir.model.access.csv*

```
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
access_openacademy_openacademy,openacademy.openacademy,model_openacademy_openacademy,,1,0,0,0
```

*openacademy/templates.xml*

```
<odoo>

        <!-- <template id="listing"> -->
        <!--   <ul> -->
        <!--     <li t-foreach="objects" t-as="object"> -->
        <!--       <a t-attf-href="{{ root }}/objects/{{ object.id }}"> -->
        <!--         <t t-esc="object.display_name"/> -->
        <!--       </a> -->
        <!--     </li> -->
        <!--   </ul> -->
        <!-- </template> -->
        <!-- <template id="object"> -->
        <!--   <h1><t t-esc="object.display_name"/></h1> -->
        <!--   <dl> -->
        <!--     <t t-foreach="object._fields" t-as="field"> -->
        <!--       <dt><t t-esc="field"/></dt> -->
        <!--       <dd><t t-esc="object[field]"/></dd> -->
        <!--     </t> -->
        <!--   </dl> -->
        <!-- </template> -->

</odoo>
```

### 对象关系映射

Odoo的关键组件是 ORM 层，该层避免手动编写大部分的SQL 并提供扩展性和安全服务[2](https://alanhou.org/odoo-13-building-module/#rawsql).

业务对象以Python类进行声明，它继承集成了自动化持久系统的 `Model` 。

模型可通过在定义中设置一系列属性来进行配置。最重要的属性 `_name` 必须要有，它定义 Odoo 系统中模型的名称。以下是一个模型的最小化完整定义：

```
from odoo import models
class MinimalModel(models.Model):
    _name = 'test.model'
```

### 模型字段

字段用于定义模型存储内容及位置。字段在模型类中以属性进行定义：

```
from odoo import models, fields

class LessMinimalModel(models.Model):
    _name = 'test.model2'

    name = fields.Char()
```

#### 通用属性

类似于模型本身，字段也可通过传递配置属性来作为参数进行配置：

```
name = field.Char(required=True)
```

一些属性对所有字段都可用，以下是一些通用的属性

- `string` (`unicode`, 默认值：字段名)

  用户界面中（对用户可见）的字段标签

- `required` (`bool`, 默认值: `False`)

  若为 `True`，该字段不能为空，必须要么带有默认值，要么保持在创建记录时给定值。

- `help` (`unicode`, 默认值： `''`)

  长形，在用户界面中向用户提供提示信息。

- `index` (`bool`, default: `False`)

  请求 Odoo 对字段创建[数据库索引](http://use-the-index-luke.com/sql/preface) 。

#### 简单字段

有两大类字段：“简单”字段是在模型表中直接存储的原子值，而“关联”字段关联（同一模型或不同模型中的）记录。

简单字段的示例有 `Boolean`, `Date`, `Char`。

#### 保留字段

Odoo对所有模型创建一些字段[1](https://alanhou.org/odoo-13-building-module/#autofields)。这些字段由系统管理，不应进行写入。在有用和必要时可进行读取：

- `id` (`Id`)

  模型中对一条记录的唯一标识符。

- `create_date` (`Datetime`)

  记录的创建日期。

- `create_uid` (`Many2one`)

  创建记录的用户。

- `write_date` (`Datetime`)

  记录的最后修改日期。

- `write_uid` (`Many2one`)

  最近修改记录的用户。

#### 特殊字段

默认， Odoo还对不同的展示和搜索行为要求在所有模型中有一个 `name` 字段。用于这些目的的字段可以通过设置`_rec_name`进行重载。

### 📝练习

定义模型

D在*openacademy*模块中定义一个新数据模型*Course* 。课程有标题和描述。课程必须要有标题。

E编辑文件 `openacademy/models/models.py` 来包含*Course* 类。

*openacademy/models.py*

```
from odoo import models, fields, api

class Course(models.Model):
    _name = 'openacademy.course'
    _description = "OpenAcademy Courses"

    name = fields.Char(string="Title", required=True)
    description = fields.Text()
```

### 数据文件

Odoo是一套高度数据驱动的系统。但行为在数据加载时由模块中的[Python](http://python.org/)代码进行自定义。

 一些模块的存在只是为了向Odoo中添加数据

模块文件通过带有`<record>`的XML[数据文件](https://alanhou.org/odoo-13-data-files/#reference-data)进行声明。每个 `<record>` 元素创建或更新数据库记录。

```
<odoo>

        <record model="{model name}" id="{record identifier}">
            <field name="{a field name}">{a value}</field>
        </record>

</odoo>
```

- `model` 是针对记录的 Odoo 模型的名称。
- `id` 是[外部标识符](https://www.odoo.com/documentation/13.0/glossary.html#term-external-identifier)，它允许引用记录（而无需知道其数据库中的内部标识符）。
- `<field>` 元素有一个 `name` ，它是模型中的字段名 (例如 `description`).。它们的内部是字段值。

数据文件是在要加载的声明文件中声明的，它们可以在 `'data'` 列表（一直都加载）或`'demo'`列表（仅在演示模式下加载）中进行声明。

### 📝练习

定义演示数据

使用一些演示课程来创建演示数据填充*Courses* 模型。

编辑`openacademy/demo/demo.xml`文件来包含一些数据。

*openacademy/demo.xml*

```
<odoo>

        <record model="openacademy.course" id="course0">
            <field name="name">Course 0</field>
            <field name="description">Course 0's description

Can have multiple lines
            </field>
        </record>
        <record model="openacademy.course" id="course1">
            <field name="name">Course 1</field>
            <!-- no description for this one -->
        </record>
        <record model="openacademy.course" id="course2">
            <field name="name">Course 2</field>
            <field name="description">Course 2's description</field>
        </record>

</odoo>
```

数据文件的内容仅在模块安装或更新时进行加载。

在进行修改之后，不要忘记使用[odoo-bin -u openacademy](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#reference-cmdline) 来在数据库中保存修改。

### 动作和菜单

动作和菜单是数据库的常规记录，通常通过数据文件进行声明。动作可通过三种方式触发：

1. 通过点击菜单项(链接具体动作)
2. 通过点击视图中的按钮(如若关联动作的话
3. 作为对象的上下文动作

因为菜单的声明有些复杂，有一个 `<menuitem>` 快捷方式来声明 `ir.ui.menu` 并更轻松地将其连接到对应的动作。

```
<record model="ir.actions.act_window" id="action_list_ideas">
    <field name="name">Ideas</field>
    <field name="res_model">idea.idea</field>
    <field name="view_mode">tree,form</field>
</record>
<menuitem id="menu_ideas" parent="menu_root" name="Ideas" sequence="10"
          action="action_list_ideas"/>
```

### ⛔️危险

这个动作必须在XML文件中相应的菜单前进行声明。

数据文件按顺序执行，动作的 `id`必须在菜单可被创建前出现在数据库中。

### 练习

定义新的菜单项

菜单下定义新的菜单项访问课程。用户应当能够：

- 显示所有课程的列表
- 创建/修改课程

1. 创建带有动作和触发动作的菜单的`openacademy/views/openacademy.xml`
2. `openacademy/__manifest__.py`的`data`列表中

*openacademy/__manifest__.py*

```
    'data': [
        # 'security/ir.model.access.csv',
        'templates.xml',
        'views/openacademy.xml',
    ],
    # only loaded in demonstration mode
    'demo': [
```

*openacademy/views/openacademy.xml*

```
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

        <!-- window action -->
        <!--
            The following tag is an action definition for a "window action",
            that is an action opening a view or a set of views
        -->
        <record model="ir.actions.act_window" id="course_list_action">
            <field name="name">Courses</field>
            <field name="res_model">openacademy.course</field>
            <field name="view_type">form</field>
            <field name="view_mode">tree,form</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">Create the first course
                </p>
            </field>
        </record>

        <!-- top level menu: no parent -->
        <menuitem id="main_openacademy_menu" name="Open Academy"/>
        <!-- A first level in the left side menu is needed
             before using action= attribute -->
        <menuitem id="openacademy_menu" name="Open Academy"
                  parent="main_openacademy_menu"/>
        <!-- the following menuitem should appear *after*
             its parent openacademy_menu and *after* its
             action course_list_action -->
        <menuitem id="courses_menu" name="Courses" parent="openacademy_menu"
                  action="course_list_action"/>
        <!-- Full id location:
             action="openacademy.course_list_action"
             It is not required when it is the same module -->

</odoo>
```

## 基础视图

视图定义模型记录展示的方式。每种视图类型展示一种可视化模式(一个记录列表，它们汇总的图表…)。视图通常可由它们的类型（如伙伴列表）或特别通过它们的 id 进行请求。对于通用请求，将使用具有正确类型的视图和最低的优先级（这样每种类型的最低优先级是该类型的默认视图）。

[视图继承](https://alanhou.org/odoo-13-views/#reference-views-inheritance) 允许修改其它地方声明的视图 （添加或删除内容）。

### 通用视图声明

视图声明为 `ir.ui.view`模型的一条记录。视图类型在`arch`字段的根元素中进行指定：

```
<record model="ir.ui.view" id="view_id">
    <field name="name">view.name</field>
    <field name="model">object_name</field>
    <field name="priority" eval="16"/>
    <field name="arch" type="xml">
        <!-- view content: <form>, <tree>, <graph>, ... -->
    </field>
</record>
```

### 🚫危险

视图的内容是XML。

因此`arch` 字段必须要声明为`type="xml"` 来获得正确地解析。

### 树状视图

.树状视图，也称为列表视图，以列表形式显示记录。

它们的根元素是 `<tree>`。树状视图的最简单形式是仅将所有字段在表格中进行展示（每个字段一列）：

```
<tree string="Idea list">
    <field name="name"/>
    <field name="inventor_id"/>
</tree>
```

### 表单视图

表单可用于创建及编辑单个记录。

它们的根元素是`<form>`。它们由高级别的结构元素（group，notebook）及互动元素（button和field）组成：

```
<form string="Idea form">
    <group colspan="4">
        <group colspan="2" col="2">
            <separator string="General stuff" colspan="2"/>
            <field name="name"/>
            <field name="inventor_id"/>
        </group>

        <group colspan="2" col="2">
            <separator string="Dates" colspan="2"/>
            <field name="active"/>
            <field name="invent_date" readonly="1"/>
        </group>

        <notebook colspan="4">
            <page string="Description">
                <field name="description" nolabel="1"/>
            </page>
        </notebook>

        <field name="state"/>
    </group>
</form>
```

### 📝练习

使用XML自定义表单视图

为Course对象创建你自己的表单视图。 展示的数据应当为：课程的名称或描述。

*openacademy/views/openacademy.xml*

```
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

        <record model="ir.ui.view" id="course_form_view">
            <field name="name">course.form</field>
            <field name="model">openacademy.course</field>
            <field name="arch" type="xml">
                <form string="Course Form">
                    <sheet>
                        <group>
                            <field name="name"/>
                            <field name="description"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <!-- window action -->
        <!--
            The following tag is an action definition for a "window action",
```

### 📝练习

Notebook

在Course表单视图中， 将描述字段放在一个标签下，这样会更易于稍后添加包含附加信息的其它标签。

修改Course表单视图如下：

*openacademy/views/openacademy.xml*

```
                    <sheet>
                        <group>
                            <field name="name"/>
                        </group>
                        <notebook>
                            <page string="Description">
                                <field name="description"/>
                            </page>
                            <page string="About">
                                This is an example of notebooks
                            </page>
                        </notebook>
                    </sheet>
                </form>
            </field>
```

表单视图也可以使用普通HTML来获得更灵活的布局：

```
<form string="Idea Form">
    <header>
        <button string="Confirm" type="object" name="action_confirm"
                states="draft" class="oe_highlight" />
        <button string="Mark as done" type="object" name="action_done"
                states="confirmed" class="oe_highlight"/>
        <button string="Reset to draft" type="object" name="action_draft"
                states="confirmed,done" />
        <field name="state" widget="statusbar"/>
    </header>
    <sheet>
        <div class="oe_title">
            <label for="name" class="oe_edit_only" string="Idea Name" />
            <h1><field name="name" /></h1>
        </div>
        <separator string="General" colspan="2" />
        <group colspan="2" col="2">
            <field name="description" placeholder="Idea description..." />
        </group>
    </sheet>
</form>
```

### 搜索视图

搜索视图自定义与列表视图（以及其它聚合视图）相关联的搜索字段 (and other aggregated views). 它们的根元素是 `<search>` 并且由定义可供搜索的字段组成：

```
<search>
    <field name="name"/>
    <field name="inventor_id"/>
</search>
```

如果针对模型不存在搜索视图，Odoo生成仅允许搜索 `name` 字段的搜索视图。

### 📝练习

搜索课程

允许根据标题或描述来搜索课程。

*openacademy/views/openacademy.xml*

```
            </field>
        </record>

        <record model="ir.ui.view" id="course_search_view">
            <field name="name">course.search</field>
            <field name="model">openacademy.course</field>
            <field name="arch" type="xml">
                <search>
                    <field name="name"/>
                    <field name="description"/>
                </search>
            </field>
        </record>

        <!-- window action -->
        <!--
            The following tag is an action definition for a "window action",
```

## 模型间的关联

一个模型中的记录可能与另一个模型中的记录相关联。例如，与包含客户数据相关联的客户记录相关联的销售订单记录；它还与销售销售订单明细记录相关联。

### 📝练习

创建一个课时（session）模型

对于Open Academy模块，我们考虑为*课时*添加模型：课时是在给定时间教授给指定学员的课程形式。

为*课时*创建一个模型。课时有名称、开始时间、时长和坐席数。添加一个动作和展示它们的菜单项。让新模型可通过菜单项进行访问。

1. 在 `openacademy/models/models.py`中创建一个*Session*类
2. 在`openacademy/view/openacademy.xml`中添加对session对象的访问

*openacademy/models.py*

```
    name = fields.Char(string="Title", required=True)
    description = fields.Text()


class Session(models.Model):
    _name = 'openacademy.session'
    _description = "OpenAcademy Sessions"

    name = fields.Char(required=True)
    start_date = fields.Date()
    duration = fields.Float(digits=(6, 2), help="Duration in days")
    seats = fields.Integer(string="Number of seats")
```

*openacademy/views/openacademy.xml*

```
             action="openacademy.course_list_action"
             It is not required when it is the same module -->

        <!-- session form view -->
        <record model="ir.ui.view" id="session_form_view">
            <field name="name">session.form</field>
            <field name="model">openacademy.session</field>
            <field name="arch" type="xml">
                <form string="Session Form">
                    <sheet>
                        <group>
                            <field name="name"/>
                            <field name="start_date"/>
                            <field name="duration"/>
                            <field name="seats"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record model="ir.actions.act_window" id="session_list_action">
            <field name="name">Sessions</field>
            <field name="res_model">openacademy.session</field>
            <field name="view_type">form</field>
            <field name="view_mode">tree,form</field>
        </record>

        <menuitem id="session_menu" name="Sessions"
                  parent="openacademy_menu"
                  action="session_list_action"/>

</odoo>
```

`digits=(6, 2)` 指定了浮点数的精度：6是总位数，而是在点号后的位数。注意这会导致点号前的最大位数是4.

### 关联字段

关联字段链接相同模型（等级）或不同模型间的记录。

关联字段类型有：

- `Many2one(other_model, ondelete='set null')`

  对另一个对象的简单链接：`print foo.other_id.name`参见其它[外键](http://www.postgresql.org/docs/9.3/static/tutorial-fk.html)

- `One2many(other_model, related_field)`

  一个虚拟关联，`Many2one`的反向。`One2many` 作为记录的容器，访问它会产生一个记录集（有可能为空）：`for other in foo.other_ids:    print other.name`🚫危险因`One2many`是一个虚拟关联，必须要在`*other_model*`中有一个 `Many2one` 字段，并且其名称*必须* 是`*related_field*`

- `Many2many(other_model)`

   双向的多对多关联，在一侧的任意记录可以与另一侧任意数量的记录进行关联。作为记录的容易，访问它也可能会产生空记录集：`for other in foo.other_ids:    print other.name`

### 📝练习

Many2one关联

使用many2one修改*Course* 和 *Session* 模型来反映它们与其它模型之间的关联：

- 一个课程有一个*负责人*用户，该字段的值记录在内置的模型 `res.users`中。
- 一个课时对应一个*导师*，该字段的值是内置模型 `res.partner`中的一条记录。
- 课时与*课程*之间存在关联，该字段的值是`openacademy.course` 模型中的一条记录并且是必须的。
- 调整视图。

1. 在模型中添加相关联的 `Many2one` 字段，并
2. 在视图中添加它们。

*openacademy/models.py*

```
    name = fields.Char(string="Title", required=True)
    description = fields.Text()

    responsible_id = fields.Many2one('res.users',
        ondelete='set null', string="Responsible", index=True)


class Session(models.Model):
    _name = 'openacademy.session'
    start_date = fields.Date()
    duration = fields.Float(digits=(6, 2), help="Duration in days")
    seats = fields.Integer(string="Number of seats")

    instructor_id = fields.Many2one('res.partner', string="Instructor")
    course_id = fields.Many2one('openacademy.course',
        ondelete='cascade', string="Course", required=True)
```

*openacademy/views/openacademy.xml*

```
                    <sheet>
                        <group>
                            <field name="name"/>
                            <field name="responsible_id"/>
                        </group>
                        <notebook>
                            <page string="Description">
            </field>
        </record>

        <!-- override the automatically generated list view for courses -->
        <record model="ir.ui.view" id="course_tree_view">
            <field name="name">course.tree</field>
            <field name="model">openacademy.course</field>
            <field name="arch" type="xml">
                <tree string="Course Tree">
                    <field name="name"/>
                    <field name="responsible_id"/>
                </tree>
            </field>
        </record>

        <!-- window action -->
        <!--
            The following tag is an action definition for a "window action",
                <form string="Session Form">
                    <sheet>
                        <group>
                            <group string="General">
                                <field name="course_id"/>
                                <field name="name"/>
                                <field name="instructor_id"/>
                            </group>
                            <group string="Schedule">
                                <field name="start_date"/>
                                <field name="duration"/>
                                <field name="seats"/>
                            </group>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <!-- session tree/list view -->
        <record model="ir.ui.view" id="session_tree_view">
            <field name="name">session.tree</field>
            <field name="model">openacademy.session</field>
            <field name="arch" type="xml">
                <tree string="Session Tree">
                    <field name="name"/>
                    <field name="course_id"/>
                </tree>
            </field>
        </record>

        <record model="ir.actions.act_window" id="session_list_action">
            <field name="name">Sessions</field>
            <field name="res_model">openacademy.session</field>
```

### 📝练习

反向的one2many关联

使用反向的关联字段one2many，修改模型在反映课程与课时之间的关联。

1. 修改 `Course` 类，并
2. 在课程表单视图中添加字段。

*openacademy/models.py*

```
    responsible_id = fields.Many2one('res.users',
        ondelete='set null', string="Responsible", index=True)
    session_ids = fields.One2many(
        'openacademy.session', 'course_id', string="Sessions")


class Session(models.Model):
```

*openacademy/views/openacademy.xml*

```
                            <page string="Description">
                                <field name="description"/>
                            </page>
                            <page string="Sessions">
                                <field name="session_ids">
                                    <tree string="Registered sessions">
                                        <field name="name"/>
                                        <field name="instructor_id"/>
                                    </tree>
                                </field>
                            </page>
                        </notebook>
                    </sheet>
```

### 📝练习

many2many多对多关联

使用关联字段many2many，修改*Session* 模型来关联每个课时到一组*参加人员*。参见人员由伙伴记录业体现，因此我们将关联到内置的模型 `res.partner`中。相应地调整视图。

1. 修改 `Session` 类，并
2. 在表单视图中添加字段。

*openacademy/models.py*

```
    instructor_id = fields.Many2one('res.partner', string="Instructor")
    course_id = fields.Many2one('openacademy.course',
        ondelete='cascade', string="Course", required=True)
    attendee_ids = fields.Many2many('res.partner', string="Attendees")
```

*openacademy/views/openacademy.xml*

```
                                <field name="seats"/>
                            </group>
                        </group>
                        <label for="attendee_ids"/>
                        <field name="attendee_ids"/>
                    </sheet>
                </form>
            </field>
```

## 继承

### 模型继承

Odoo提供两种以模块化的方式继承已有模型的*继承*机制。

第一种继承允许一个模块修改在另一个模块中所定义的模型的行为：

- 在模型中添加字段，
- 重载模型中字段的定义，
- 为模型添加约束，
- 向模型添加方法，
- 在模型中重载已有方法。

第二种继承机制（代理继承）允许将模型中的每条记录链接到父级模型中的记当中，并提供对父级记录的透明访问。

![img](https://www.odoo.com/documentation/13.0/_images/inheritance_methods.png)

### 参见其它

- `_inherit`
- `_inherits`

### 视图继承

除在原处（通过重写）修改已有视图之外，Odoo还提供了视图继承，此时“继承”视图应用于根视图之上，并且可以从它们的父级添加或删除内容。

继承视图使用 `inherit_id` 字段引用其父级，而不同于单个视图，`arch`字段由任意数量的选择并修改父级视图内容的`xpath`元素组成：

```
<!-- improved idea categories list -->
<record id="idea_category_list2" model="ir.ui.view">
    <field name="name">id.category.list2</field>
    <field name="model">idea.category</field>
    <field name="inherit_id" ref="id_category_list"/>
    <field name="arch" type="xml">
        <!-- find field description and add the field
             idea_ids after it -->
        <xpath expr="//field[@name='description']" position="after">
          <field name="idea_ids" string="Number of ideas"/>
        </xpath>
    </field>
</record>
```

- `expr`

  [XPath](http://w3.org/TR/xpath) 表达式选择父级视图中的单个元素。在没有匹配任何元素或匹配到一个以上元素时抛出错误

- `position`

  用于匹配元素的运算：`inside`在匹配元素后添加`xpath`的内容体`replace`使用`xpath`的内容体替换已匹配元素，使用始元素替换所有 `$0`节点内容`before`在匹配元素之前以兄弟节点插入`xpath`内容体`after`在匹配元素之后以兄弟节点插入 `xpaths`内容体`attributes`在`xpath`内容体中使用特殊的`attribute`元素修改所匹配元素的属性

在匹配单个元素时， `position` 属性可对所查找到的元素直接进行设置。以下两种继承可得到相同的结果。

```
<xpath expr="//field[@name='description']" position="after">
    <field name="idea_ids" />
</xpath>

<field name="description" position="after">
    <field name="idea_ids" />
</field>
```

### 📝练习

修改已有内容

- 使用模型继承，修改已有的*Partner* 模型来添加一个 `instructor` 布尔字段，以及一个与课时-伙伴关联相对应的many2many字段
- 使用视图继承，在伙伴表单视图中展示这些字段

是时候介绍检查视图的开发者模式了，查找其外部ID及放置新字段的位置。

1. 创建 `openacademy/models/partner.py` 文件并在 `__init__.py`中导入它
2. 创建 `openacademy/views/partner.xml` 文件并效期添加到 `__manifest__.py`中

*openacademy/__init__.py*

```
# -*- coding: utf-8 -*-
from . import controllers
from . import models
from . import partner
```

*openacademy/__manifest__.py*

```
        # 'security/ir.model.access.csv',
        'templates.xml',
        'views/openacademy.xml',
        'views/partner.xml',
    ],
    # only loaded in demonstration mode
    'demo': [
```

*openacademy/partner.py*

```
# -*- coding: utf-8 -*-
from odoo import fields, models

class Partner(models.Model):
    _inherit = 'res.partner'

    # Add a new column to the res.partner model, by default partners are not
    # instructors
    instructor = fields.Boolean("Instructor", default=False)

    session_ids = fields.Many2many('openacademy.session',
        string="Attended Sessions", readonly=True)
```

*openacademy/views/partner.xml*

```
<?xml version="1.0" encoding="UTF-8"?>
 <odoo>

        <!-- Add instructor field to existing view -->
        <record model="ir.ui.view" id="partner_instructor_form_view">
            <field name="name">partner.instructor</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form"/>
            <field name="arch" type="xml">
                <notebook position="inside">
                    <page string="Sessions">
                        <group>
                            <field name="instructor"/>
                            <field name="session_ids"/>
                        </group>
                    </page>
                </notebook>
            </field>
        </record>

        <record model="ir.actions.act_window" id="contact_list_action">
            <field name="name">Contacts</field>
            <field name="res_model">res.partner</field>
            <field name="view_mode">tree,form</field>
        </record>
        <menuitem id="configuration_menu" name="Configuration"
                  parent="main_openacademy_menu"/>
        <menuitem id="contact_menu" name="Contacts"
                  parent="configuration_menu"
                  action="contact_list_action"/>

</odoo>
```

#### 作用域

在Odoo中， [作用域](https://alanhou.org/odoo-13-orm-api/#reference-orm-domains) 是对记录的条件进行编码的值。作用域是一个用于选择模型记录子集的条件列表。每个条件有三项，包含字段名、运算符及值。

例如，在用于*Product* 模型时，以下作用域选取所有单价大于*1000*的*服务*：

```
[('product_type', '=', 'service'), ('unit_price', '>', 1000)]
```

默认条件通过隐式的AND进行组合。逻辑运算符 `&` (AND)、 `|` (OR) 以及 `!` (NOT) 可用于显式地组合条件。它们在前置位置中使用（运算符在参数之前而非之间插入）。例如要选择“是服务***或\*** 单价***不在\*** 1000到2000之间”的产品：

```
['|',
    ('product_type', '=', 'service'),
    '!', '&',
        ('unit_price', '>=', 1000),
        ('unit_price', '<', 2000)]
```

在尝试选择用户界面中的记录时，`domain` 参数可添加到关联字段中来限制针对关联的有效记录。

### 📝练习

关联字段的作用域

在为*Session选择导师时，*仅*导师（`instructor` 设置为`True`的伙伴）*可见。

*openacademy/models.py*

```
    duration = fields.Float(digits=(6, 2), help="Duration in days")
    seats = fields.Integer(string="Number of seats")

    instructor_id = fields.Many2one('res.partner', string="Instructor",
        domain=[('instructor', '=', True)])
    course_id = fields.Many2one('openacademy.course',
        ondelete='cascade', string="Course", required=True)
    attendee_ids = fields.Many2many('res.partner', string="Attendees")
```

声明为字面量的作用域在服务端运行并且无法引用右侧的动态值，以字符串声明的作用域在客户端运行并允许使用右侧的字段名。

### 📝练习

更复杂的作用域

创建新的伙伴分类*Teacher / Level 1* and *Teacher / Level 2*。针对课时的导师可以是导师或者（任意级别的）教师。

1. 修改*Session* 模型的作用域
2. 修改 `openacademy/view/partner.xml`来访问 *Partner 分类*

*openacademy/models.py*

```
    seats = fields.Integer(string="Number of seats")

    instructor_id = fields.Many2one('res.partner', string="Instructor",
        domain=['|', ('instructor', '=', True),
                     ('category_id.name', 'ilike', "Teacher")])
    course_id = fields.Many2one('openacademy.course',
        ondelete='cascade', string="Course", required=True)
    attendee_ids = fields.Many2many('res.partner', string="Attendees")
```

*openacademy/views/partner.xml*

```
                  parent="configuration_menu"
                  action="contact_list_action"/>

        <record model="ir.actions.act_window" id="contact_cat_list_action">
            <field name="name">Contact Tags</field>
            <field name="res_model">res.partner.category</field>
            <field name="view_mode">tree,form</field>
        </record>
        <menuitem id="contact_cat_menu" name="Contact Tags"
                  parent="configuration_menu"
                  action="contact_cat_list_action"/>

        <record model="res.partner.category" id="teacher1">
            <field name="name">Teacher / Level 1</field>
        </record>
        <record model="res.partner.category" id="teacher2">
            <field name="name">Teacher / Level 2</field>
        </record>

</odoo>
```

## 计算字段和默认值

截至目前字段都在数据库中直接存储及获取。字段也可*被计算*。在这种情况下，字段值不是从数据库中获取而是通过调用模型方法实时计算。

要创建计算字段，创建一个字段并设置其属性 `compute` 为方法名。计算字段应只需对`self`中的每条记录设置计算的字段值。

### 🚫危险

`self` 是一个集合

对象 `self`是一个*记录集*，如一个记录的有序集合。它支持对集合的标准Python运算，如 `len(self)` 和 `iter(self)`，其它的集合运算如 `recs1 + recs2`。

遍历 `self` 会逐条给出记录，每个记录本身又是一个大小为1的集合。你可以使用点号运算符来对访问/赋值单条记录中的字段，如 `record.name`。

```
import random
from odoo import models, fields, api

class ComputedModel(models.Model):
    _name = 'test.computed'

    name = fields.Char(compute='_compute_name')

    @api.multi
    def _compute_name(self):
        for record in self:
            record.name = str(random.randint(1, 1e6))
```

### 依赖

计算字段的值通常依赖于所计算记录中的其它字段的值。 ORM要求开发者通过`depends()`装饰器对计算字段指定这些依赖。给定的依赖由ORM用于在依赖中发生更改时触发字段的重新计算：

```
from odoo import models, fields, api

class ComputedModel(models.Model):
    _name = 'test.computed'

    name = fields.Char(compute='_compute_name')
    value = fields.Integer()

    @api.depends('value')
    def _compute_name(self):
        for record in self:
            record.name = "Record with value %s" % record.value
```

### 📝练习

计算字段

- 添加*Session* 模型座位的占用比
- 在树状和表单视图中显示该字段
- 以进行条展示该字段

1. 向 *Session*添加一个计算字段
2. 在*Session* 视图中显示该字段：

*openacademy/models.py*

```
   course_id = fields.Many2one('openacademy.course',
        ondelete='cascade', string="Course", required=True)
    attendee_ids = fields.Many2many('res.partner', string="Attendees")

    taken_seats = fields.Float(string="Taken seats", compute='_taken_seats')

    @api.depends('seats', 'attendee_ids')
    def _taken_seats(self):
        for r in self:
            if not r.seats:
                r.taken_seats = 0.0
            else:
                r.taken_seats = 100.0 * len(r.attendee_ids) / r.seats
```

*openacademy/views/openacademy.xml*

```
                                <field name="start_date"/>
                                <field name="duration"/>
                                <field name="seats"/>
                                <field name="taken_seats" widget="progressbar"/>
                            </group>
                        </group>
                        <label for="attendee_ids"/>
                <tree string="Session Tree">
                    <field name="name"/>
                    <field name="course_id"/>
                    <field name="taken_seats" widget="progressbar"/>
                </tree>
            </field>
        </record>
```

### 默认值

任何字段都可赋默认值。在字段定义中，添加参数`default=X` ，其中 `X` 可以是Python字面量值（布尔型、整型、字符串），或接收记录集并返回值的函数：

```
name = fields.Char(default="Unknown")
user_id = fields.Many2one('res.users', default=lambda self: self.env.user)
```

`self.env` 对象提供对请求参数及其它有用内容的访问：

- `self.env.cr` 或 `self._cr` 是数据库的 *游标* 对象；它用于查询该数据库
- `self.env.uid` 或 `self._uid` 是当前用户的数据库id
- `self.env.user` 是当前用户的记录
- `self.env.context` 或 `self._context` 是上下文字典
- `self.env.ref(xml_id)` 返回与XML id相对应的记录
- `self.env[model_name]` 返回给定模型的实例

### 📝练习

活跃对象 - 默认值

- 将start_date默认值定义为当天 (参见 `Date`)。
- 在Session类中添加 `active` 字段，并默认设置课时为活跃状态。

*openacademy/models.py*

```
    _description = "OpenAcademy Sessions"

    name = fields.Char(required=True)
    start_date = fields.Date(default=fields.Date.today)
    duration = fields.Float(digits=(6, 2), help="Duration in days")
    seats = fields.Integer(string="Number of seats")
    active = fields.Boolean(default=True)

    instructor_id = fields.Many2one('res.partner', string="Instructor",
        domain=['|', ('instructor', '=', True),
```

*openacademy/views/openacademy.xml*

```
                                <field name="course_id"/>
                                <field name="name"/>
                                <field name="instructor_id"/>
                                <field name="active"/>
                            </group>
                            <group string="Schedule">
                                <field name="start_date"/>
```

Odoo拥有内置的规则 ，它会让`active`字段值为`False` 的记录不可见。

## Onchange

“onchange”机制提供一种当用户填写字段值时在用户界面更新表单的方式，它尚未在数据库中进行保存。

例如，假设模型有三个字段 `amount`, `unit_price` 和 `price`，并且你希望在其它字段修改时对表单更新价格。要实现这点，定义一个方法，其中`self` 表示表单视图中的记录，并通过`onchange()`对其进行装饰来指定要触发的字段。对 `self` 所做的任何修改都会在表单中进行反映。

```
<!-- content of form view -->
<field name="amount"/>
<field name="unit_price"/>
<field name="price" readonly="1"/>
# onchange handler
@api.onchange('amount', 'unit_price')
def _onchange_price(self):
    # set auto-changing field
    self.price = self.amount * self.unit_price
    # Can optionally return a warning and domains
    return {
        'warning': {
            'title': "Something bad happened",
            'message': "It was very bad indeed",
        }
    }
```

对于计算字段，带值的 `onchange` 行为是内置的，因为可以通过操作*Session*表单来进行查看：修改座席或参与人员的数量，`taken_seats`进度t 条会自动更新。

### 📝练习

⚠️警告

添加一个显式的onchange来对无效值报出警告，如座席为负值或参与人员大于座席数。

*openacademy/models.py*

```
                r.taken_seats = 0.0
            else:
                r.taken_seats = 100.0 * len(r.attendee_ids) / r.seats

    @api.onchange('seats', 'attendee_ids')
    def _verify_valid_seats(self):
        if self.seats < 0:
            return {
                'warning': {
                    'title': "Incorrect 'seats' value",
                    'message': "The number of available seats may not be negative",
                },
            }
        if self.seats < len(self.attendee_ids):
            return {
                'warning': {
                    'title': "Too many attendees",
                    'message': "Increase seats or remove excess attendees",
                },
            }
```

## 模型约束

Odoo提供设置自动地验证不定式： `Python约束` 及 `SQL约束`。

Python约束定义为一个由`constrains()`装饰的方法，对记录集进行调用。装饰器指定在约束中包含哪个字段，因此约束在其中之一修改时会自动运行。该方法会在未满足不定式时抛出异常：

```
from odoo.exceptions import ValidationError

@api.constrains('age')
def _check_something(self):
    for record in self:
        if record.age > 20:
            raise ValidationError("Your record is too old: %s" % record.age)
    # all records passed the test, don't return anything
```

### 📝练习

添加Python约束

添加一个导师未在他/她自己的课时中列席的约束检查。

*openacademy/models.py*

```
# -*- coding: utf-8 -*-

from odoo import models, fields, api, exceptions

class Course(models.Model):
    _name = 'openacademy.course'
                    'message': "Increase seats or remove excess attendees",
                },
            }

    @api.constrains('instructor_id', 'attendee_ids')
    def _check_instructor_not_in_attendees(self):
        for r in self:
            if r.instructor_id and r.instructor_id in r.attendee_ids:
                raise exceptions.ValidationError("A session's instructor can't be an at
```

SQL约束通过模型属性`_sql_constraints`来进行定义。后者赋值给一个三字符串列表 `(name, sql_definition, message)`，其中 `name`是有效的SQL约束名，`sql_definition` 是一个 [table_constraint](http://www.postgresql.org/docs/9.3/static/ddl-constraints.html)表达式，而 `message` 是报错消息。

### 📝练习

添加SQL约束

在[PostgreSQL文档](http://www.postgresql.org/docs/9.3/static/ddl-constraints.html)的帮助下，添加如下约束：

1. **检查**课程描述和课程标题是不同的
2. 让课程的名称**唯一**

*openacademy/models.py*

```
    session_ids = fields.One2many(
        'openacademy.session', 'course_id', string="Sessions")

    _sql_constraints = [
        ('name_description_check',
         'CHECK(name != description)',
         "The title of the course should not be the description"),

        ('name_unique',
         'UNIQUE(name)',
         "The course title must be unique"),
    ]


class Session(models.Model):
    _name = 'openacademy.session'
```

### 📝练习

练习6 - 添加复制选项

因为我们为课程名添加了唯一性约束，就无法再使用“duplicate”功能了 (Form ‣ Duplicate).

重新实现你自己的“copy”方法，来允许复制课程对象，修改原名称为“Copy of [original name]”。

*openacademy/models.py*

```
    session_ids = fields.One2many(
        'openacademy.session', 'course_id', string="Sessions")

    @api.multi
    def copy(self, default=None):
        default = dict(default or {})

        copied_count = self.search_count(
            [('name', '=like', u"Copy of {}%".format(self.name))])
        if not copied_count:
            new_name = u"Copy of {}".format(self.name)
        else:
            new_name = u"Copy of {} ({})".format(self.name, copied_count)

        default['name'] = new_name
        return super(Course, self).copy(default)

    _sql_constraints = [
        ('name_description_check',
         'CHECK(name != description)',
```

## 高级视图

### 树状视图

树状视图可以接收额外的属性来进一步自定义它们的行为：

- `decoration-{$name}`

  允许根据对应记录属性的行文本样式。值为Python表达式。对于每条记录，表达式通过记录的属性来作为上下文值并在为`true`时运行，将样式应用到行上。其它上下文值有 `uid` (当前用户的 id) 和 `current_date` ( `yyyy-MM-dd`格式的当前日期字符串)。`{$name}` 可以为 `bf` (`font-weight: bold`)， `it` (`font-style: italic`)或任意 [bootstrap上下文颜色](https://getbootstrap.com/docs/3.3/components/#available-variations) (`danger`, `info`, `muted`, `primary`, `success` 或 `warning`)。`<tree string="Idea Categories" decoration-info="state=='draft'"    decoration-danger="state=='trashed'">    <field name="name"/>    <field name="state"/> </tree>`

- `editable`

  为 `"top"` 或 `"bottom"`。让树状视图在当前位置可编辑(而无需进入表单视图), 值为新行出现的位置。

### 📝练习

列表上色

修改Session视图，让课时周期小于5天的标记为蓝色，超过15天的标记为红色。

修改课时树状列表视图如下：

*openacademy/views/openacademy.xml*

```
            <field name="name">session.tree</field>
            <field name="model">openacademy.session</field>
            <field name="arch" type="xml">
                <tree string="Session Tree" decoration-info="duration&lt;5" decoration-danger="duration&gt;15">
                    <field name="name"/>
                    <field name="course_id"/>
                    <field name="duration" invisible="1"/>
                    <field name="taken_seats" widget="progressbar"/>
                </tree>
            </field>
```

### 日历

按照日历事件来显示记录。它们的根元素为 `<calendar>`，最常用的属性有：

- `color`

  字段名用于*颜色分段*。颜色自动分配给事件，但处于相同颜色分段的事件（`@color`字段值相同的记录）会被给予相同的颜色。

- `date_start`

  记录的字段中存储事件的开始日期/时间

- `date_stop` (可选)

  字段中保存事件的结束日期/时间

- `string`

  记录的字段中定义针对每个日历事件的标签

```
<calendar string="Ideas" date_start="invent_date" color="inventor_id">
    <field name="name"/>
</calendar>
```

### 📝练习

日历视图

为*Session* 模型添加日历视图来启用让用户浏览与Open Academy关联的事件。

1. 添加通过

   ```
   start_date
   ```

    和 

   ```
   duration
   ```

   计算的

   ```
   end_date
   ```

   字段

   让字段可写的反向函数，并允许在日历视图中（通过拖放）移动课时

2. 向 *Session* 模型添加日历视图

3. 并向*Session* 模型的动作添加菜单视图

*openacademy/models.py*

```
# -*- coding: utf-8 -*-

from datetime import timedelta
from odoo import models, fields, api, exceptions

class Course(models.Model):
    attendee_ids = fields.Many2many('res.partner', string="Attendees")

    taken_seats = fields.Float(string="Taken seats", compute='_taken_seats')
    end_date = fields.Date(string="End Date", store=True,
        compute='_get_end_date', inverse='_set_end_date')

    @api.depends('seats', 'attendee_ids')
    def _taken_seats(self):
                },
            }

    @api.depends('start_date', 'duration')
    def _get_end_date(self):
        for r in self:
            if not (r.start_date and r.duration):
                r.end_date = r.start_date
                continue

            # Add duration to start_date, but: Monday + 5 days = Saturday, so
            # subtract one second to get on Friday instead
            duration = timedelta(days=r.duration, seconds=-1)
            r.end_date = r.start_date + duration

    def _set_end_date(self):
        for r in self:
            if not (r.start_date and r.end_date):
                continue

            # Compute the difference between dates, but: Friday - Monday = 4 days,
            # so add one day to get 5 days instead
            r.duration = (r.end_date - r.start_date).days + 1

    @api.constrains('instructor_id', 'attendee_ids')
    def _check_instructor_not_in_attendees(self):
        for r in self:
```

*openacademy/views/openacademy.xml*

```
            </field>
        </record>

        <!-- calendar view -->
        <record model="ir.ui.view" id="session_calendar_view">
            <field name="name">session.calendar</field>
            <field name="model">openacademy.session</field>
            <field name="arch" type="xml">
                <calendar string="Session Calendar" date_start="start_date" date_stop="end_date" color="instructor_id">
                    <field name="name"/>
                </calendar>
            </field>
        </record>

        <record model="ir.actions.act_window" id="session_list_action">
            <field name="name">Sessions</field>
            <field name="res_model">openacademy.session</field>
            <field name="view_type">form</field>
            <field name="view_mode">tree,form,calendar</field>
        </record>

        <menuitem id="session_menu" name="Sessions"
```

### 搜索视图

搜索视图 `<field>` 元素可有一个 `@filter_domain` ，重载生成用于搜索给定字段的作用域。在给定的作用域中，`self` 表示由用户输入的值。在下面的示例中，它用于对 `name` 和 `description`字段进行搜索。

搜索视图也可以包含 `<filter>` 元素，胜任预定义搜索的切换器。过滤器必须有一个下面的属性：

- `domain`

  对当前搜索添加给定作用域

- `context`

  对当前搜索添加一些上下文件；使用 `group_by` 键来对给定字段名的结果进行分组

```
<search string="Ideas">
    <field name="name"/>
    <field name="description" string="Name and description"
           filter_domain="['|', ('name', 'ilike', self), ('description', 'ilike', self)]"/>
    <field name="inventor_id"/>
    <field name="country_id" widget="selection"/>

    <filter name="my_ideas" string="My Ideas"
            domain="[('inventor_id', '=', uid)]"/>
    <group string="Group By">
        <filter name="group_by_inventor" string="Inventor"
                context="{'group_by': 'inventor_id'}"/>
    </group>
</search>
```

要在动作中使用非默认的搜索视图，应当关联使用动作记录的 `search_view_id` 字段。

动作也可以通过其 `context` 字段来为搜索字段设置默认值：`search_default_*field_name*` 表单的上下文键将会通过所提供值初始化 *field_name* 。搜索过滤器必须有一个可选的 `@name` 来作为默认值或作为布尔值（仅在默认情况下启用）。

### 📝练习

搜索视图

1. 添加按钮在课程搜索视图中过滤出当前用户所负责的课程。默认为选中状态。
2. 添加按钮通过所负责用户进行课程分组。

*openacademy/views/openacademy.xml*

```
                <search>
                    <field name="name"/>
                    <field name="description"/>
                    <filter name="my_courses" string="My Courses"
                            domain="[('responsible_id', '=', uid)]"/>
                    <group string="Group By">
                        <filter name="by_responsible" string="Responsible"
                                context="{'group_by': 'responsible_id'}"/>
                    </group>
                </search>
            </field>
        </record>
            <field name="res_model">openacademy.course</field>
            <field name="view_type">form</field>
            <field name="view_mode">tree,form</field>
            <field name="context" eval="{'search_default_my_courses': 1}"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">Create the first course
                </p>
```

### 甘特图

### ⚠️警告

甘特图视图要求有web_gantt模型，仅在 [企业版](https://alanhou.org/odoo-13-installing-odoo/#setup-install-editions)中存在。

横向柱状图常用于展示项目规划和进展，其根元素为 `<gantt>`。

```
<gantt string="Ideas"
       date_start="invent_date"
       date_stop="date_finished"
       progress="progress"
       default_group_by="inventor_id" />
```

### 📝练习

甘特图表

添加一个甘特图表来让用户可浏览与Open Academy模块关联的计划课时。课时应由导师进行分组。

1. 创建一个计算字段来表达按小时的课时时长
2. 添加甘特视图的定义，并为*Session*模型的动作添加甘特视图

*openacademy/models.py*

```
    end_date = fields.Date(string="End Date", store=True,
        compute='_get_end_date', inverse='_set_end_date')

    hours = fields.Float(string="Duration in hours",
                         compute='_get_hours', inverse='_set_hours')

    @api.depends('seats', 'attendee_ids')
    def _taken_seats(self):
        for r in self:
            # so add one day to get 5 days instead
            r.duration = (r.end_date - r.start_date).days + 1

    @api.depends('duration')
    def _get_hours(self):
        for r in self:
            r.hours = r.duration * 24

    def _set_hours(self):
        for r in self:
            r.duration = r.hours / 24

    @api.constrains('instructor_id', 'attendee_ids')
    def _check_instructor_not_in_attendees(self):
        for r in self:
```

*openacademy/views/openacademy.xml*

```
            </field>
        </record>

        <record model="ir.ui.view" id="session_gantt_view">
            <field name="name">session.gantt</field>
            <field name="model">openacademy.session</field>
            <field name="arch" type="xml">
                <gantt string="Session Gantt"
                       date_start="start_date" date_delay="hours"
                       default_group_by='instructor_id'>
                    <!-- <field name="name"/> this is not required after Odoo 10.0 -->
                </gantt>
            </field>
        </record>

        <record model="ir.actions.act_window" id="session_list_action">
            <field name="name">Sessions</field>
            <field name="res_model">openacademy.session</field>
            <field name="view_type">form</field>
            <field name="view_mode">tree,form,calendar,gantt</field>
        </record>

        <menuitem id="session_menu" name="Sessions"
```

### 图表视图

图表视图允许对模型的聚合预览和分析，它们的根元素为 `<graph>`。

透视表视图(`<pivot>`元素) 是一个多维表格，允许选择过滤器和维度来获取移动到更为图表化预览的正确聚合数据集。透视表视图分享定义为图表视图的相同内容。

图表视图有4种展示模式，默认使用`@type`属性选中默认模式。

- 柱状图 (默认)

  柱状图， 第一维度用于对横轴定义分组，另一个维度定义每个分组中的聚合条柱。默认条柱是并排的，可以对`<graph>`使用`@stacked="True"`来进行叠放。

- 折线图

  2维折线图

- 饼状图

  2维饼状图

图表视图包含必填的`@type`属性的 `<field>` ，接收如下值：

- `row` (默认)

  字段应默认聚合

- `measure`

  字段应进行聚合而非分组

```
<graph string="Total idea score by Inventor">
    <field name="inventor_id"/>
    <field name="score" type="measure"/>
</graph>
```

### ⚠️警告

表单视图对数据库值执行聚合，它们不通过非存储的计算字段运行。

### 📝练习

图表视图

在Session对象中添加图表视图，对每个课程显示柱状图表单下的参与人员的数量。

1. 添加参与人数作为已存储的计算字段
2. 然后添加相关的视图

*openacademy/models.py*

```
    hours = fields.Float(string="Duration in hours",
                         compute='_get_hours', inverse='_set_hours')

    attendees_count = fields.Integer(
        string="Attendees count", compute='_get_attendees_count', store=True)

    @api.depends('seats', 'attendee_ids')
    def _taken_seats(self):
        for r in self:
        for r in self:
            r.duration = r.hours / 24

    @api.depends('attendee_ids')
    def _get_attendees_count(self):
        for r in self:
            r.attendees_count = len(r.attendee_ids)

    @api.constrains('instructor_id', 'attendee_ids')
    def _check_instructor_not_in_attendees(self):
        for r in self:
```

*openacademy/views/openacademy.xml*

```
            </field>
        </record>

        <record model="ir.ui.view" id="openacademy_session_graph_view">
            <field name="name">openacademy.session.graph</field>
            <field name="model">openacademy.session</field>
            <field name="arch" type="xml">
                <graph string="Participations by Courses">
                    <field name="course_id"/>
                    <field name="attendees_count" type="measure"/>
                </graph>
            </field>
        </record>

        <record model="ir.actions.act_window" id="session_list_action">
            <field name="name">Sessions</field>
            <field name="res_model">openacademy.session</field>
            <field name="view_type">form</field>
            <field name="view_mode">tree,form,calendar,gantt,graph</field>
        </record>

        <menuitem id="session_menu" name="Sessions"
```

### 看板

用于组织任务、生产进程等等...它们的根元素是 `<kanban>`。

看板视图显示一组可按列进行分组的卡片。每个卡片表示一条记录，并且每列为聚合字段的值。

例如，项目任务可通过阶段(每列为一个阶段) 或负责人（每列一个用户）等进行组织。

看板视图以表单元素的组合（包含基本HTML）及[QWeb](https://alanhou.org/odoo-13-qweb/#reference-qweb)定义每个卡片的结构。

### 📝练习

看板视图

添加一个由课程分组（列即为课程）的展示课时的看板裤视图。

1. 为*Session* 模型添加整型`color`字段
2. 添加看板视图并更新动作

*openacademy/models.py*

```
    duration = fields.Float(digits=(6, 2), help="Duration in days")
    seats = fields.Integer(string="Number of seats")
    active = fields.Boolean(default=True)
    color = fields.Integer()

    instructor_id = fields.Many2one('res.partner', string="Instructor",
        domain=['|', ('instructor', '=', True),
```

*openacademy/views/openacademy.xml*

```
            </field>
        </record>

        <record model="ir.ui.view" id="view_openacad_session_kanban">
            <field name="name">openacademy.session.kanban</field>
            <field name="model">openacademy.session</field>
            <field name="arch" type="xml">
                <kanban default_group_by="course_id">
                    <field name="color"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div
                                    t-attf-class="oe_kanban_color_{{kanban_getcolor(record.color.raw_value)}}
                                                  oe_kanban_global_click_edit oe_semantic_html_override
                                                  oe_kanban_card {{record.group_fancy==1 ? 'oe_kanban_card_fancy' : ''}}">
                                <div class="oe_dropdown_kanban">
                                    <!-- dropdown menu -->
                                    <div class="oe_dropdown_toggle">
                                        <i class="fa fa-bars fa-lg" title="Manage" aria-label="Manage"/>
                                        <ul class="oe_dropdown_menu">
                                            <li>
                                                <a type="delete">Delete</a>
                                            </li>
                                            <li>
                                                <ul class="oe_kanban_colorpicker"
                                                    data-field="color"/>
                                            </li>
                                        </ul>
                                    </div>
                                    <div class="oe_clear"></div>
                                </div>
                                <div t-attf-class="oe_kanban_content">
                                    <!-- title -->
                                    Session name:
                                    <field name="name"/>
                                    <br/>
                                    Start date:
                                    <field name="start_date"/>
                                    <br/>
                                    duration:
                                    <field name="duration"/>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record model="ir.actions.act_window" id="session_list_action">
            <field name="name">Sessions</field>
            <field name="res_model">openacademy.session</field>
            <field name="view_type">form</field>
            <field name="view_mode">tree,form,calendar,gantt,graph,kanban</field>
        </record>

        <menuitem id="session_menu" name="Sessions"
```

## 权限

必须配置访问控制机制来实现连续的权限策略。

### 基于组的权限控制机制

组在 `res.groups`模型中以普通记录进行创建，并通过菜单定义来进行菜单访问的授权。但便没有菜单，对象也可以进行间接的访问，因此必须为组定义实际的对象级权限（读、写、创建、删除）。它们通常通过CSV文件内部模块进行插入。也可使用字段的groups属性来限制视图或对象中具体字段的访问。

### 访问权限

访问权限在 `ir.model.access`模型的记录中进行定义。每个访问权限关联模型、组（或没有全局访问的组）或一组权限：读取、写入、创建、删除。这种访问权限通常由按模型命名的CSV文件进行创建： `ir.model.access.csv`。

```
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
access_idea_idea,idea.idea,model_idea_idea,base.group_user,1,1,1,0
access_idea_vote,idea.vote,model_idea_vote,base.group_user,1,1,1,0
```

### 📝练习

通过Odoo界面添加访问控制

新建用户“John Smith”。然后对*Session*模型创建具有读取权限的“OpenAcademy / Session Read”组。

1. 通过Settings ‣ Users ‣ Users新建用户*John Smith*
2. 通过Settings ‣ Users ‣ Groups,新建组`session_read`，它应当对*Session* 模型拥有读取的权限
3. 编辑 *John Smith* 让其成为`session_read`的成员
4. 以*John Smith* 进行登录来检查访问权限是否正确

### 练习

通过模块中的数据文添加访问控制

使用数据文件，

- 创建对OpenAcademy模型拥有完全访问权限的 *OpenAcademy / Manager* 组
- 让 *Session* 和 *Course* 对所有用户可读

1. 新建文件 `openacademy/security/security.xml` 来放置OpenAcademy Manager组
2. 编辑具有对模型访问权限的 `openacademy/security/ir.model.access.csv` 文件
3. 最后更新 `openacademy/__manifest__.py` 来将新数据文件添加到其中

*openacademy/__manifest__.py*

```
    # always loaded
    'data': [
        'security/security.xml',
        'security/ir.model.access.csv',
        'templates.xml',
        'views/openacademy.xml',
        'views/partner.xml',
```

*openacademy/security/ir.model.access.csv*

```
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
course_manager,course manager,model_openacademy_course,group_manager,1,1,1,1
session_manager,session manager,model_openacademy_session,group_manager,1,1,1,1
course_read_all,course all,model_openacademy_course,,1,0,0,0
session_read_all,session all,model_openacademy_session,,1,0,0,0
```

*openacademy/security/security.xml*

```
<odoo>

        <record id="group_manager" model="res.groups">
            <field name="name">OpenAcademy / Manager</field>
        </record>

</odoo>
```

### 记录规则

记录规则限制对给定模型为记录子集的访问权限。规则是 `ir.rule`模型的一条记录，并且关联模型、一些组 (many2many 字段)、限制所应用的访问权限及作用域。作用域指定访问权限所限定的记录。

这里的示例是防止对状态不为 `cancel`的线索的删除。 注意 `groups` 字段的值必须按照与ORM中`write()`方法相同的规则。

```
<record id="delete_cancelled_only" model="ir.rule">
    <field name="name">Only cancelled leads may be deleted</field>
    <field name="model_id" ref="crm.model_crm_lead"/>
    <field name="groups" eval="[(4, ref('sales_team.group_sale_manager'))]"/>
    <field name="perm_read" eval="0"/>
    <field name="perm_write" eval="0"/>
    <field name="perm_create" eval="0"/>
    <field name="perm_unlink" eval="1" />
    <field name="domain_force">[('state','=','cancel')]</field>
</record>
```

### 📝练习

记录规则

对Course模型和“OpenAcademy / Manager”组添加一条记录规则 ，用于限定负责课程的`write` 和 `unlink` 权限。如果课程没有负责人，则的帮助组中用户均可修改它。

在 `openacademy/security/security.xml`中新建一条规则：

*openacademy/security/security.xml*

```
        <record id="group_manager" model="res.groups">
            <field name="name">OpenAcademy / Manager</field>
        </record>
    
        <record id="only_responsible_can_modify" model="ir.rule">
            <field name="name">Only Responsible can modify Course</field>
            <field name="model_id" ref="model_openacademy_course"/>
            <field name="groups" eval="[(4, ref('openacademy.group_manager'))]"/>
            <field name="perm_read" eval="0"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_create" eval="0"/>
            <field name="perm_unlink" eval="1"/>
            <field name="domain_force">
                ['|', ('responsible_id','=',False),
                      ('responsible_id','=',user.id)]
            </field>
        </record>

</odoo>
```

## 向导

向导通过动态表单描述与用户（或对话框）的交互会话。向导只是继承`TransientModel`类而非`Model`的一个模型。`TransientModel`类继承了`Model` 并且通过如下内容利用所有已有机制：

- 向导记录不是持久化的；它们在一定时间后会自动从数据库中删除。这也是称之为 *transient(临时)*的原因。
- 向导模型不要求有具体的访问权限；用户对向导记录有所有的权限。
- 向导记录可能会通过many2one字段引用普通记录或向导记录，但普通记录无法通过many2one字段引用向导记录。

我们将创建一个允许用户针对具体课时或同时对一组课时创建参与人员的向导。

### 📝练习

定义向导

使用many2one关联对 *Session* 模型及many2many关联对*Partner*模型创建一个向导模型。

新增文件 `openacademy/wizard.py`：

*openacademy/__init__.py*

```
from . import controllers
from . import models
from . import partner
from . import wizard
```

*openacademy/wizard.py*

```
# -*- coding: utf-8 -*-

from odoo import models, fields, api

class Wizard(models.TransientModel):
    _name = 'openacademy.wizard'
    _description = "Wizard: Quick Registration of Attendees to Sessions"

    session_id = fields.Many2one('openacademy.session',
        string="Session", required=True)
    attendee_ids = fields.Many2many('res.partner', string="Attendees")
```

### 启动向导

向导由 `ir.actions.act_window` 记录启动，字段 `target` 的值设置为 `new`。后者将向导视图打开为弹窗。可通过菜单项触发该动作。

还有启动向导的其它方式：使用像上面那样的 `ir.actions.act_window` 记录，但包含在上下文中指定动作可用的模型的额外字段 `binding_model_id`。向导会出现在主视图上方的模型的上下文动作中。因为在ORM中一些内部钩子，这种动作通过`act_window`在XML中进行声明。

```
<act_window id="launch_the_wizard"
            name="Launch the Wizard"
            src_model="context.model.name"
            res_model="wizard.model.name"
            view_mode="form"
            target="new"
            key2="client_action_multi"/>
```

向导使用普通视图，并且它们的按钮可使用属性 `special="cancel"` 来不进行保存关闭向导窗口。

### 📝练习

启动向导

1. 为向导定义表单视图。
2. 在*Session* 模型的上下文中添加动作来启动向导。
3. 在向导中为session字段定义默认值；使用上下文参数`self._context` 来获取当前会话。

*openacademy/wizard.py*

```
    _name = 'openacademy.wizard'
    _description = "Wizard: Quick Registration of Attendees to Sessions"

    def _default_session(self):
        return self.env['openacademy.session'].browse(self._context.get('active_id'))

    session_id = fields.Many2one('openacademy.session',
        string="Session", required=True, default=_default_session)
    attendee_ids = fields.Many2many('res.partner', string="Attendees")
```

*openacademy/views/openacademy.xml*

```
                  parent="openacademy_menu"
                  action="session_list_action"/>

        <record model="ir.ui.view" id="wizard_form_view">
            <field name="name">wizard.form</field>
            <field name="model">openacademy.wizard</field>
            <field name="arch" type="xml">
                <form string="Add Attendees">
                    <group>
                        <field name="session_id"/>
                        <field name="attendee_ids"/>
                    </group>
                </form>
            </field>
        </record>

        <act_window id="launch_session_wizard"
                    name="Add Attendees"
                    src_model="openacademy.session"
                    res_model="openacademy.wizard"
                    view_mode="form"
                    target="new"
                    key2="client_action_multi"/>

</odoo>
```

### 📝练习

注册参与人员

为向导添加按钮，并为给定课时实现添加对参与人员的对应方法。

*openacademy/views/openacademy.xml*

```
                        <field name="session_id"/>
                        <field name="attendee_ids"/>
                    </group>
                    <footer>
                        <button name="subscribe" type="object"
                                string="Subscribe" class="oe_highlight"/>
                        or
                        <button special="cancel" string="Cancel"/>
                    </footer>
                </form>
            </field>
        </record>
```

*openacademy/wizard.py*

```
    session_id = fields.Many2one('openacademy.session',
        string="Session", required=True, default=_default_session)
    attendee_ids = fields.Many2many('res.partner', string="Attendees")

    @api.multi
    def subscribe(self):
        self.session_id.attendee_ids |= self.attendee_ids
        return {}
```

### 📝练习

向多个课时注册参与人员

修改向导模型来让参与人员可注册到多个课时中。

*openacademy/views/openacademy.xml*

```
            <field name="arch" type="xml">
                <form string="Add Attendees">
                    <group>
                        <field name="session_ids"/>
                        <field name="attendee_ids"/>
                    </group>
                    <footer>
```

*openacademy/wizard.py*

```
    _name = 'openacademy.wizard'
    _description = "Wizard: Quick Registration of Attendees to Sessions"

    def _default_sessions(self):
        return self.env['openacademy.session'].browse(self._context.get('active_ids'))

    session_ids = fields.Many2many('openacademy.session',
        string="Sessions", required=True, default=_default_sessions)
    attendee_ids = fields.Many2many('res.partner', string="Attendees")

    @api.multi
    def subscribe(self):
        for session in self.session_ids:
            session.attendee_ids |= self.attendee_ids
        return {}
```

## 国际化

每个模块可在 i18n目录中通过命名为LANG.po的文件提供其自己的翻译，其中LANG为语句的locale编号或在有不同时使用语言和国家的组合（pt.po 或 pt_BR.po）。对所有启用语言将由Odoo自动加载翻译。开发人员在创建模块时保持使用英语，然后使用Odoo的gettext POT导出功能（Settings ‣ Translations ‣ Import/Export ‣ Export Translation without specifying a language）导出模块用词。这会创建一个模型模板POT文件，然后获取所翻译的PO文件。很IDE都有编辑及合并PO/POT文件的插件或模式。

由Odoo所生成的便携对象（Portable Object ） 在 [Transifex](https://www.transifex.com/odoo/public/) 上进行发布，让翻译该软件变得容易。

```
|- idea/ # The module directory
   |- i18n/ # Translation files
      | - idea.pot # Translation Template (exported from Odoo)
      | - fr.po # French translation
      | - pt_BR.po # Brazilian Portuguese translation
      | (...)
```

默认 Odoo的POT导出及提供XML文件内部的标签或Python代码内部的字段定义，但任何Python字符串都可通过在周围包裹`odoo._()`（如`_("Label")`）函数的方式来进行翻译。

### 📝练习

翻译模块

为Odoo安装软件选择第二种语言。使用Odoo所提供的工具来翻译模块。

1. 创建目录 `openacademy/i18n/`
2. 安装你所需要的任意语言( Administration ‣ Translations ‣ Load an Official Translation)
3. 同时可翻译词汇(Administration ‣ Translations ‣ Application Terms ‣ Synchronize Translations)
4. 无需指定语言通过导出( Administration ‣ Translations -> Import/Export ‣ Export Translation) 创建一个模板翻译文件，保存到 `openacademy/i18n/`中
5. 通过导出 ( Administration ‣ Translations ‣ Import/Export ‣ Export Translation) 并指定语言创建一个翻译文件。保存到`openacademy/i18n/`中
6. 使用文本翻辑器或独立的PO文件编辑器（如[POEdit](http://poedit.net/)）打开导出的翻译文件并翻译缺少的用词
7. 在 `models.py`中，为函数`odoo._` 添加导入语句，产将缺失的字符串标识为可翻译
8. 重复3-6步

*openacademy/models.py*

```
# -*- coding: utf-8 -*-

from datetime import timedelta
from odoo import models, fields, api, exceptions, _

class Course(models.Model):
    _name = 'openacademy.course'
        default = dict(default or {})

        copied_count = self.search_count(
            [('name', '=like', _(u"Copy of {}%").format(self.name))])
        if not copied_count:
            new_name = _(u"Copy of {}").format(self.name)
        else:
            new_name = _(u"Copy of {} ({})").format(self.name, copied_count)

        default['name'] = new_name
        return super(Course, self).copy(default)
        if self.seats < 0:
            return {
                'warning': {
                    'title': _("Incorrect 'seats' value"),
                    'message': _("The number of available seats may not be negative"),
                },
            }
        if self.seats < len(self.attendee_ids):
            return {
                'warning': {
                    'title': _("Too many attendees"),
                    'message': _("Increase seats or remove excess attendees"),
                },
            }
    def _check_instructor_not_in_attendees(self):
        for r in self:
            if r.instructor_id and r.instructor_id in r.attendee_ids:
                raise exceptions.ValidationError(_("A session's instructor can't be an attendee"))
```

## 报表

### 打印报表

Odoo使用一个基于[QWeb](https://alanhou.org/odoo-13-qweb/#reference-qweb), [Twitter Bootstrap](http://getbootstrap.com/) 和 [Wkhtmltopdf](http://wkhtmltopdf.org/)的报表引擎。

报表是两大元素的组合：

- ```
  ir.actions.report
  ```

  ，为其提供了一个快捷元素 

  ```
  <report>
  ```

   ，它为报表设置了各种基础参数(默认类型、报表是否应在生成后保存到数据库中...)

  ```
  <report
      id="account_invoices"
      model="account.invoice"
      string="Invoices"
      report_type="qweb-pdf"
      name="account.report_invoice"
      file="account.report_invoice"
      attachment_use="True"
      attachment="(object.state in ('open','paid')) and
          ('INV'+(object.number or '').replace('/','')+'.pdf')"
  />
  ```

- 针对实际报表的标准

  QWeb视图

   ：

  ```
  <t t-call="web.html_container">
      <t t-foreach="docs" t-as="o">
          <t t-call="web.external_layout">
              <div class="page">
                  <h2>Report title</h2>
              </div>
          </t>
      </t>
  </t>
  
  the standard rendering context provides a number of elements, the most
  important being:
  
  ``docs``
      the records for which the report is printed
  ``user``
      the user printing the report
  ```

因报表是标准的网页，它们可通过URL访问，并且输出参数可通过这个URL进行控制，例如*Invoice*报表的HTML版本可通过  http://localhost:8069/report/html/account.report_invoice/1 (若已安装 `account` ) 进行访问并可通过http://localhost:8069/report/pdf/account.report_invoice/1访问PDF版本。

### 🚫危险

如果你的PDF报表中缺失了样式 (如出现了文字但样式/布局与 html 版本不同)，可能是你的 [wkhtmltopdf](http://wkhtmltopdf.org/) 进程无法访问网页服务器来进行下载。

如果查看服务端日志，发现在生成PDF报表时无法下载CSS样式，那么就是这个问题无疑了。

[wkhtmltopdf](http://wkhtmltopdf.org/) 进行将使用 `web.base.url` 系统参数作为对所有链接文件的 *根* 路径，但这个参数会在管理员每次登录时自动更新。如果我们的服务处于某些代理背后，可能就无法访问。可以通过添加如下系统参数来进行修复：

- `report.url`，指向一个访问到服务端的 URL (可能是 `http://localhost:8069` 或类似的地址)。它将仅用于具体目的。
- `web.base.url.freeze`，在设置为`True`时，将会停止对 `web.base.url`的自动更新。

### 📝练习

为Session模型创建报表

对于每个课时，应当显示课时名称、开始和结束时间，以及其参与人员。

*openacademy/__manifest__.py*

```
        'templates.xml',
        'views/openacademy.xml',
        'views/partner.xml',
        'reports.xml',
    ],
    # only loaded in demonstration mode
    'demo': [
```

*openacademy/reports.xml*

```
<odoo>

    <report
        id="report_session"
        model="openacademy.session"
        string="Session Report"
        name="openacademy.report_session_view"
        file="openacademy.report_session"
        report_type="qweb-pdf" />

    <template id="report_session_view">
        <t t-call="web.html_container">
            <t t-foreach="docs" t-as="doc">
                <t t-call="web.external_layout">
                    <div class="page">
                        <h2 t-field="doc.name"/>
                        <p>From <span t-field="doc.start_date"/> to <span t-field="doc.end_date"/></p>
                        <h3>Attendees:</h3>
                        <ul>
                            <t t-foreach="doc.attendee_ids" t-as="attendee">
                                <li><span t-field="attendee.name"/></li>
                            </t>
                        </ul>
                    </div>
                </t>
            </t>
        </t>
    </template>

</odoo>
```

### 仪表盘

### 📝练习

定义仪表盘

定义的仪表盘可包含你所创建的图表视图、课时的日历视图以及课程的列表视图（可切换为表单视图）。 仪表盘应可通过菜单中的菜单项进行访问，并在选中OpenAcademy主菜单时自动在网页客户端中显示。

1. 创建 

   ```
   openacademy/views/session_board.xml
   ```

   文件。 它应当包含仪表盘视图、该视图所引用的动作、打开仪表盘的动作及添加仪表盘动作的菜单项的再定义。

   可用的仪表盘样式有 `1`, `1-1`, `1-2`, `2-1` 和 `1-1-1`

2. 更新 `openacademy/__manifest__.py` 来引用新的数据文件

*openacademy/__manifest__.py*

```
    'version': '0.1',

    # any module necessary for this one to work correctly
    'depends': ['base', 'board'],

    # always loaded
    'data': [
        'templates.xml',
        'views/openacademy.xml',
        'views/partner.xml',
        'views/session_board.xml',
        'reports.xml',
    ],
    # only loaded in demonstration mode
```

*openacademy/views/session_board.xml*

```
<?xml version="1.0"?>
<odoo>

        <record model="ir.actions.act_window" id="act_session_graph">
            <field name="name">Attendees by course</field>
            <field name="res_model">openacademy.session</field>
            <field name="view_type">form</field>
            <field name="view_mode">graph</field>
            <field name="view_id"
                   ref="openacademy.openacademy_session_graph_view"/>
        </record>
        <record model="ir.actions.act_window" id="act_session_calendar">
            <field name="name">Sessions</field>
            <field name="res_model">openacademy.session</field>
            <field name="view_type">form</field>
            <field name="view_mode">calendar</field>
            <field name="view_id" ref="openacademy.session_calendar_view"/>
        </record>
        <record model="ir.actions.act_window" id="act_course_list">
            <field name="name">Courses</field>
            <field name="res_model">openacademy.course</field>
            <field name="view_type">form</field>
            <field name="view_mode">tree,form</field>
        </record>
        <record model="ir.ui.view" id="board_session_form">
            <field name="name">Session Dashboard Form</field>
            <field name="model">board.board</field>
            <field name="type">form</field>
            <field name="arch" type="xml">
                <form string="Session Dashboard">
                    <board style="2-1">
                        <column>
                            <action
                                string="Attendees by course"
                                name="%(act_session_graph)d"
                                height="150"
                                width="510"/>
                            <action
                                string="Sessions"
                                name="%(act_session_calendar)d"/>
                        </column>
                        <column>
                            <action
                                string="Courses"
                                name="%(act_course_list)d"/>
                        </column>
                    </board>
                </form>
            </field>
        </record>
        <record model="ir.actions.act_window" id="open_board_session">
          <field name="name">Session Dashboard</field>
          <field name="res_model">board.board</field>
          <field name="view_type">form</field>
          <field name="view_mode">form</field>
          <field name="usage">menu</field>
          <field name="view_id" ref="board_session_form"/>
        </record>

        <menuitem
            name="Session Dashboard" parent="base.menu_reporting_dashboard"
            action="open_board_session"
            sequence="1"
            id="menu_board_session"/>

</odoo>
```

## 网页服务

web-service模块提供对所有网页服务的通用接口：

- XML-RPC
- JSON-RPC

业务对象也可通过分布式对象机制来进行访问。它们都可以通过带有上下文视图的客户端接口进行修改。

Odoo 可通过 XML-RPC/JSON-RPC 接口进行访问，针对这类接口很多编程语言中都存在语言库。

### XML-RPC库

以下示例是通过`xmlrpc.client`库与Odoo服务端交互的Python 3程序：

```
import xmlrpc.client

root = 'http://%s:%d/xmlrpc/' % (HOST, PORT)

uid = xmlrpc.client.ServerProxy(root + 'common').login(DB, USER, PASS)
print("Logged in as %s (uid: %d)" % (USER, uid))

# Create a new note
sock = xmlrpc.client.ServerProxy(root + 'object')
args = {
    'color' : 8,
    'memo' : 'This is a note',
    'create_uid': uid,
}
note_id = sock.execute(DB, uid, PASS, 'note.note', 'create', args)
```

### 📝练习

向客户端新增服务

编写一个可向PC中运行的Odoo（你自己的或你导师的）发送XML-RPC请求的 Python程序。这个程序应当显示所有课时，以及它们相对应的座位号。它还应为其中的课程新建课时。

```
import functools
import xmlrpc.client
HOST = 'localhost'
PORT = 8069
DB = 'openacademy'
USER = 'admin'
PASS = 'admin'
ROOT = 'http://%s:%d/xmlrpc/' % (HOST,PORT)

# 1. Login
uid = xmlrpc.client.ServerProxy(ROOT + 'common').login(DB,USER,PASS)
print("Logged in as %s (uid:%d)" % (USER,uid))

call = functools.partial(
    xmlrpc.client.ServerProxy(ROOT + 'object').execute,
    DB, uid, PASS)

# 2. Read the sessions
sessions = call('openacademy.session','search_read', [], ['name','seats'])
for session in sessions:
    print("Session %s (%s seats)" % (session['name'], session['seats']))
# 3.create a new session
session_id = call('openacademy.session', 'create', {
    'name' : 'My session',
    'course_id' : 2,
})
```

除使用硬编码的课程 id外，代码还可通过名称查找课程：

```
# 3.create a new session for the "Functional" course
course_id = call('openacademy.course', 'search', [('name','ilike','Functional')])[0]
session_id = call('openacademy.session', 'create', {
    'name' : 'My session',
    'course_id' : course_id,
})
```

### JSON-RPC库

以下示例是通过Python标准库`urllib.request` 和 `json`来与Odoo服务进行交互的Python 3程序。下面的示例假定已安装了 **生产力** 应用 (`note`) ：

```
import json
import random
import urllib.request

HOST = 'localhost'
PORT = 8069
DB = 'openacademy'
USER = 'admin'
PASS = 'admin'

def json_rpc(url, method, params):
    data = {
        "jsonrpc": "2.0",
        "method": method,
        "params": params,
        "id": random.randint(0, 1000000000),
    }
    req = urllib.request.Request(url=url, data=json.dumps(data).encode(), headers={
        "Content-Type":"application/json",
    })
    reply = json.loads(urllib.request.urlopen(req).read().decode('UTF-8'))
    if reply.get("error"):
        raise Exception(reply["error"])
    return reply["result"]

def call(url, service, method, *args):
    return json_rpc(url, "call", {"service": service, "method": method, "args": args})

# log in the given database
url = "http://%s:%s/jsonrpc" % (HOST, PORT)
uid = call(url, "common", "login", DB, USER, PASS)

# create a new note
args = {
    'color': 8,
    'memo': 'This is another note',
    'create_uid': uid,
}
note_id = call(url, "object", "execute", DB, uid, PASS, 'note.note', 'create', args)
```

示例可轻易地从XML-RPC 调整为 JSON-RPC。

在不同语言中有很多高级别的API来访问Odoo系统，而无需*显式地*通过 XML-RPC 或 JSON-RPC，如：

- https://github.com/akretion/ooor
- https://github.com/OCA/odoorpc
- https://github.com/nicolas-van/openerp-client-lib
- http://pythonhosted.org/OdooRPC
- https://github.com/abhishek-jaiswal/php-openerp-lib

[[1\]](https://alanhou.org/odoo-13-building-module/#id2) 可以`禁用一些字段的自动创建`

[[2\]](https://alanhou.org/odoo-13-building-module/#id1) 也可以编写原生的SQL查询，但需要注意它会超过所有的Odoo验证以及安全机制。