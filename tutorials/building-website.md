# 建造网站

本文来自[Odoo 13官方文档之开发者文档](../README.md)系列文章

### ⚠️警告

- 本指南要求 [掌握Python基础知识](http://docs.python.org/3/tutorial/)
- 本指南要求[已安装了 Odoo](https://alanhou.org/odoo-13-installing-odoo/)

## 创建一个基本模块

Odoo中的任务通过创建模块来实现。

模块自定义Odoo安装软件的行为，要么通过新增行为，要么通过改变已有的行为（包含由其它模块所添加的行为）。

[Odoo脚手架](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#reference-cmdline-scaffold) 可配置一个基本的模块。仅需调用如下命令来快速开启：

```
$ ./odoo-bin scaffold Academy my-modules
```

这会自动创建一个`my-modules` *模块目录*，其中放置一个 `academy` 模块。这个目录可以是已有的模块目录，但模块名在目录内必须是唯一的。

## 演示模块

现在我们已经有一个“完整的” 模块可供安装了。

虽然它还没有什么功能，但我们可以进行安装：

- 启动Odoo服务

  ```
  $ ./odoo-bin --addons-path addons,my-modules
  ```

- 访问 [http://localhost:8069](http://localhost:8069/)

- 新建一个包含演示数据的数据库

- 访问Settings ‣ Modules ‣ Modules

- 在右上角移除*Installed* 过滤器并搜索 *academy*

- 点击 Install按钮在安装 *Academy* 模块

## 浏览器访问

[控制器](../reference/web-controllers) 解释浏览器请求并返回数据。

添加一个简单的控制器并确保它通过`__init__.py` 进行了导入(这样Odoo可以发现它):

*academy/controllers.py*

```
# -*- coding: utf-8 -*-
from odoo import http

class Academy(http.Controller):
    @http.route('/academy/academy/', auth='public')
    def index(self, **kw):
        return "Hello, world"

#     @http.route('/academy/academy/objects/', auth='public')
#     def list(self, **kw):
```

关闭服务(`^C`) 然后再重启：

```
$ ./odoo-bin --addons-path addons,my-modules
```

打开页面 http://localhost:8069/academy/academy/，你应该可以看到自己的 “页面” 出现：

![img](https://www.odoo.com/documentation/13.0/_images/helloworld.png)

## 模板

在Python中生成HTML并不友好。

通常的方案是 [templates](http://en.wikipedia.org/wiki/Web_template)，带有占位符并展示逻辑的伪文档。Odoo允许使用任何Python 模板系统，但提供了它自己的 [QWeb](https://alanhou.org/odoo-13-qweb/#reference-qweb) 模板系统，在其中集成了一些其它功能。

创建模板并确保模板文件在 `__manifest__.py` 声明文件中进行了注册，修改控制器来使用我们的模板：

*academy/controllers.py*

```
class Academy(http.Controller):
    @http.route('/academy/academy/', auth='public')
    def index(self, **kw):
        return http.request.render('academy.index', {
            'teachers': ["Diana Padilla", "Jody Caroll", "Lester Vaughn"],
        })

#     @http.route('/academy/academy/objects/', auth='public')
#     def list(self, **kw):
```

*academy/templates.xml*

```
<odoo>

        <template id="index">
            <title>Academy</title>
            <t t-foreach="teachers" t-as="teacher">
              <p><t t-esc="teacher"/></p>
            </t>
        </template>
        <!-- <template id="object"> -->
        <!--   <h1><t t-esc="object.display_name"/></h1> -->
        <!--   <dl> -->
```

模板遍历 (`t-foreach`) 所有的教师 (通过*template 上下文进行传递*),，并在单独的段落中分别打印都是。

最后重启Odoo并通过进入Settings ‣ Modules ‣ Modules ‣ Academy并点击Upgrade来更新模块的数据（来安装模板）。

另外，可重启Odoo [`并同时更新模块`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-u)

```
$ odoo-bin --addons-path addons,my-modules -d academy -u academy
```

访问 http://localhost:8069/academy/academy/ 应当会显示如下结果：

![img](https://www.odoo.com/documentation/13.0/_images/basic-list.png)

## 在Odoo中存储数据

[Odoo模型](https://alanhou.org/odoo-13-orm-api/#reference-orm-model) 映射数据表。

在前面的部分中我们只是展示Python代码中静态输入的字符串列表。这并不允许进行修改或持久化存储，因此现在我们需要将数据迁移到数据库中。

### 定义数据模型

定义teacher模型，并确保其在 `__init__.py` 中进行了导入来正确加载：

*academy/models.py*

```
from odoo import models, fields, api

class Teachers(models.Model):
    _name = 'academy.teachers'

    name = fields.Char()
```

然后为模型设置 [基本权限控制](https://alanhou.org/odoo-13-security-odoo/#reference-security-acl) 并将其添加到声明文件中：

*academy/__manifest__.py*

```
    # always loaded
    'data': [
        'security/ir.model.access.csv',
        'templates.xml',
    ],
    # only loaded in demonstration mode
```

*academy/security/ir.model.access.csv*

```
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_academy_teachers,access_academy_teachers,model_academy_teachers,,1,0,0,0
```

以上只是向所有用户 (`group_id:id` 留空).赋予了读权限 (`perm_read`) 。

[数据文件](https://alanhou.org/odoo-13-data-files/#reference-data) (XML or CSV) 必须要添加到模块声明文件中，Python文件(模型或控制器) 必须要在 `__init__.py` 中进行（直接或间接的）导入。

### ⚠️警告

超级用户可跳过权限控制，即使未进行授权他们也可以访问所有模型

### 演示数据

第二步是要添加演示数据来方便测试。这通过添加 `demo` [数据文件](https://alanhou.org/odoo-13-data-files/#reference-data)来实现，它必须要在声明文件中进行添加：

*academy/demo.xml*

```
<odoo>

        <record id="padilla" model="academy.teachers">
            <field name="name">Diana Padilla</field>
        </record>
        <record id="carroll" model="academy.teachers">
            <field name="name">Jody Carroll</field>
        </record>
        <record id="vaughn" model="academy.teachers">
            <field name="name">Lester Vaughn</field>
        </record>

</odoo>
```

[数据文件](https://alanhou.org/odoo-13-data-files/#reference-data) 可用于演示或非演示数据。演示数据仅在”演示模式“下进行加载，用于流程测试或演示，非演示数据总是会被加载，用于初始化系统配置。

本例中我们使用演示数据，因为系统中的真实用户会希望输入或导入他们自己的教师列表，演示列表仅用作测试。

### 访问数据

最后一步是修改模型和模板来使用我们自己的演示数据：

1. 用数据库代替静态列表来获取记录
2. 因 `search()` 返回一个匹配过滤器 （此处为“所有记录”）的记录集，修改模型来打印每个教师的 `name`

*academy/controllers.py*

```
class Academy(http.Controller):
    @http.route('/academy/academy/', auth='public')
    def index(self, **kw):
        Teachers = http.request.env['academy.teachers']
        return http.request.render('academy.index', {
            'teachers': Teachers.search([])
        })

#     @http.route('/academy/academy/objects/', auth='public')
```

*academy/templates.xml*

```
         <template id="index">
            <title>Academy</title>
            <t t-foreach="teachers" t-as="teacher">
                <p><t t-esc="teacher.id"/> <t t-esc="teacher.name"/></p>
            </t>
        </template>
        <!-- <template id="object"> -->
```

重启服务并更新模块（来更新声明文件和加载演示文件），然后访问http://localhost:8069/academy/academy/。页面会有一些不同：名词前有一个数字（教师的数据库标识符）。

## 网站支持

Odoo打包了一个针对构造网站的模块。

到此我们都直接地使用了控制器，但Odoo 8通过`website`模块新增了更深入的集成以及一些其它服务（如默认样式、主题）。

1. 首先将 `website` 添加为 `academy`的依赖
2. 然后对控制器添加`website=True` 的标记，这对[请求对象](https://alanhou.org/odoo-13-web-controllers/#reference-http-request)设置了一些新变量并且允许在我们的模板中使用网站布局
3. 在模板中使用网站布局

*academy/__manifest__.py*

```
    'version': '0.1',

    # any module necessary for this one to work correctly
    'depends': ['website'],

    # always loaded
    'data': [
```

*academy/controllers.py*

```
from odoo import http

class Academy(http.Controller):
    @http.route('/academy/academy/', auth='public', website=True)
    def index(self, **kw):
        Teachers = http.request.env['academy.teachers']
        return http.request.render('academy.index', {
```

*academy/templates.xml*

```
<odoo>

        <template id="index">
            <t t-call="website.layout">
                <t t-set="title">Academy</t>
                <div class="oe_structure">
                    <div class="container">
                        <t t-foreach="teachers" t-as="teacher">
                            <p><t t-esc="teacher.id"/> <t t-esc="teacher.name"/></p>
                        </t>
                    </div>
                </div>
            </t>
        </template>
        <!-- <template id="object"> -->
```

在重启服务更新模块后台后台 (来更新声明文件和模板)，访问http://localhost:8069/academy/academy/ 应该会看到一个带有品牌编号内置页面元素的页面(顶级菜单、footer, …)

![img](https://www.odoo.com/documentation/13.0/_images/layout.png)网站布局还提供对编辑工具的支持：点击右上角的Sign In，填写账号信息 (默认为`admin` / `admin` ) 然后点击Log In。

现在就进入了Odoo的 “地盘”: 管理界面。此时点击Website菜单项（在右上角）。

我们以管理员身份回到了网站，可以访问*website*所提供支持的高级编辑功能。

- 模板代码编辑器 (自定义 ‣ HTML编辑器) 中你可以看到并编辑用于当前页面的所有模板
- 左上角切换为“编辑模式”的Edit按钮，这里可以使用代码块（代码片断）及富文本编辑
- 一些其它功能如移动端预览或 SEO

## URL和路由

控制器方法通过 `route()`装饰器与路由进行关联， 装饰器接收一个路由字符串和一些用于自定义其行为或权限的属性。

我们看到了一个“字面量”路由字符串，与URL版块进行精准的匹配，但路由字符串也可以使用匹配URL位的 [转换器模式](http://werkzeug.pocoo.org/docs/routing/#rule-format) ，并将这些作为局部变量。例如我们可以新建一个控制器方法，其中接收URL 并将其打印出来：

*academy/controllers.py*

```
            'teachers': Teachers.search([])
        })

    @http.route('/academy/<name>/', auth='public', website=True)
    def teacher(self, name):
        return '<h1>{}</h1>'.format(name)


#     @http.route('/academy/academy/objects/', auth='public')
#     def list(self, **kw):
#         return http.request.render('academy.listing', {
```

重启Odoo，访问 http://localhost:8069/academy/Alice/ 和 http://localhost:8069/academy/Bob/ ，查看区别。

如名称所示， [转换器模式](http://werkzeug.pocoo.org/docs/routing/#rule-format) 不只是进行提取，还可以进行 *验证* 和 *转换*,，这样我可以修改新控制器来仅接收整型：*academy/controllers.py*

```
             'teachers': Teachers.search([])
        })

    @http.route('/academy/<int:id>/', auth='public', website=True)
    def teacher(self, id):
        return '<h1>{} ({})</h1>'.format(id, type(id).__name__)


#     @http.route('/academy/academy/objects/', auth='public')
```

重启Odoo，访问 http://localhost:8069/academy/2，注意如何将老的字符串值转换为整型的。试着访问 http://localhost:8069/academy/Carol/ 并注意页面是找不到的：因为“Carol” 不是个整型，路由被忽略而找不到路由。

Odoo 提供一个额外的转换器，名为 `model` ，直接在给出它们的 id 时提供记录。我们来使用它创建一个针对教师简介的通用页面：

*academy/controllers.py*

```
            'teachers': Teachers.search([])
        })

    @http.route('/academy/<model("academy.teachers"):teacher>/', auth='public', website=True)
    def teacher(self, teacher):
        return http.request.render('academy.biography', {
            'person': teacher
        })


#     @http.route('/academy/academy/objects/', auth='public')
```

*academy/templates.xml*

```
                 </div>
            </t>
        </template>
        <template id="biography">
            <t t-call="website.layout">
                <t t-set="title">Academy</t>
                <div class="oe_structure"/>
                <div class="oe_structure">
                    <div class="container">
                        <p><t t-esc="person.id"/> <t t-esc="person.name"/></p>
                    </div>
                </div>
                <div class="oe_structure"/>
            </t>
        </template>
        <!-- <template id="object"> -->
        <!--   <h1><t t-esc="object.display_name"/></h1> -->
        <!--   <dl> -->
```

然后修改模型列表来关联到我们的新控制器：

*academy/templates.xml*

```
                <div class="oe_structure">
                    <div class="container">
                        <t t-foreach="teachers" t-as="teacher">
                            <p><a t-attf-href="/academy/{{ slug(teacher) }}">
                              <t t-esc="teacher.name"/></a>
                            </p>
                        </t>
                    </div>
                </div>
                <div class="oe_structure"/>
                <div class="oe_structure">
                    <div class="container">
                        <h3><t t-esc="person.name"/></h3>
                    </div>
                </div>
                <div class="oe_structure"/>
```

重启Odoo并升级模块，然后你可以访问每个教师的页面。作为练习，在教师的页面中添加代码码来写入简介，然后进入其他教师页面进行同样的操作。你会发现简介在所有教师间是共享的，因为代码块是添加到 *template*中的， 而 *biography* t模板在所有教师间是共享的，编辑一个页面时同时也编辑了其它页面。

## 字段编辑

针对具体记录的数据应当对记录进行保存，所以我们来为教师新增一个简介字段：

*academy/models.py*

```
    _name = 'academy.teachers'

    name = fields.Char()
    biography = fields.Html()
```

*academy/templates.xml*

```
                <div class="oe_structure">
                    <div class="container">
                        <h3><t t-esc="person.name"/></h3>
                        <div><t t-esc="person.biography"/></div>
                    </div>
                </div>
                <div class="oe_structure"/>
```

重启Odoo并更新视图，重新加载教师页面，这个字段会不可见，因为其中不包含内容。

对于记录字段，模板可以使用特殊的 `t-field` 指令，它允许使用针对字段的页面编辑网站中的字段内容。修改 *person* 模板来使用 `t-field`：

*academy/templates.xml*

```
                 <div class="oe_structure"/>
                <div class="oe_structure">
                    <div class="container">
                        <h3 t-field="person.name"/>
                        <div t-field="person.biography"/>
                    </div>
                </div>
                <div class="oe_structure"/>
```

重启Odoo并升级模块，现在在教师姓名下有一个占位符，并在编辑模式下有一个新代码块区域。拖到这里的内容会存储到对应教师的 `biography` 字段中，因此会具体到对应教师。

教师的姓名也可以进行编辑，并在保存修改后在首页中可见。

`t-field` 也可以接收依赖于具体字段的格式化选项。例如如果我们要展示教师记录的修改日期：

*academy/templates.xml*

```
                <div class="oe_structure">
                    <div class="container">
                        <h3 t-field="person.name"/>
                        <p>Last modified: <i t-field="person.write_date"/></p>
                        <div t-field="person.biography"/>
                    </div>
                </div>
```

它以一个非常“计算机”的方式展示，难以阅读，但我们可以要求一个易于人类阅读的版本：

*academy/templates.xml*

```
                <div class="oe_structure">
                    <div class="container">
                        <h3 t-field="person.name"/>
                        <p>Last modified: <i t-field="person.write_date" t-options='{"format": "long"}'/></p>
                        <div t-field="person.biography"/>
                    </div>
                </div>
```

或一个相对的展示：

*academy/templates.xml*

```
                <div class="oe_structure">
                    <div class="container">
                        <h3 t-field="person.name"/>
                        <p>Last modified: <i t-field="person.write_date" t-options='{"widget": "relative"}'/></p>
                        <div t-field="person.biography"/>
                    </div>
                </div>
```

## 管理和ERP集成

### Odoo管理简介

在[网站支持](#website-support) 一节中我们简单地接触了Odoo管理。 可以使用菜单中的 Administrator ‣ Administrator加到这个页面(如果登出的话请进行登录)。

Odoo后台的概念结构很简单：

1. 首先是菜单，一个树状的记录（菜单中可包含子菜单）。不包含子集的菜单映射到…
2. 动作。动作有多种类型：链接、报表、Odoo应当执行代码或数据展现。数据展现也称为*window*动作，告诉Odoo根据一组视图...展示一个给定的模型。
3. 视图拥有类型，一个其响应的广泛类型（列表、图表、日历），自定义在视图顺模型展示方式的结构。

### 管理后台中进行编辑

默认，Odoo模型对用户基本是隐藏的。让要其可见，必须通过动作，动作本身通常通过菜单来让人们可对其进行访问。

我们来为自己的模型创建一个菜单：

*academy/__manifest__.py*

```
    'data': [
        'security/ir.model.access.csv',
        'templates.xml',
        'views.xml',
    ],
    # only loaded in demonstration mode
    'demo': [
```

*academy/views.xml*

```
<odoo>

  <record id="action_academy_teachers" model="ir.actions.act_window">
    <field name="name">Academy teachers</field>
    <field name="res_model">academy.teachers</field>
  </record>

  <menuitem sequence="0" id="menu_academy" name="Academy"/>
  <menuitem id="menu_academy_content" parent="menu_academy"
            name="Academy Content"/>
  <menuitem id="menu_academy_content_teachers"
            parent="menu_academy_content"
            action="action_academy_teachers"/>
```

然后访问 http://localhost:8069/web/ ，左上角应该会有一个菜单Academy，默认为已选状态，因为它是第一个菜单，并会打开一个教师列表。通过列表可以新建教师记录，并通过记录视图切换到“表单”视图。

如果没有对如何展示记录的定义([视图](https://alanhou.org/odoo-13-views/#reference-views)) ，Odoo会自动补时创建一个基本的展示。本例中现在“列表”视图的展示没有问题（仅展示教师姓名），但在“表单”视图中HTML `biography`字段与`name`字段并排展示，没有给予足够的空间。我们来自定义一个表单视图来让浏览和编辑教师记录的体验更好：

*academy/views.xml*

```
     <field name="res_model">academy.teachers</field>
  </record>

  <record id="academy_teacher_form" model="ir.ui.view">
    <field name="name">Academy teachers: form</field>
    <field name="model">academy.teachers</field>
    <field name="arch" type="xml">
      <form>
        <sheet>
          <label for="name"/> <field name="name"/>
          <label for="biography"/>
          <field name="biography"/>
        </sheet>
      </form>
    </field>
  </record>

  <menuitem sequence="0" id="menu_academy" name="Academy"/>
  <menuitem id="menu_academy_content" parent="menu_academy"
            name="Academy Content"/>
```

### 模型之间的关联

我们了解到了在记录中直接存储的一组”基础“字段。还有[很多基础字段](https://alanhou.org/odoo-13-orm-api/#reference-orm-fields-basic)。第二个广泛的分类是[关联](https://alanhou.org/odoo-13-orm-api/#reference-orm-fields-relational)，用于记录之间彼此链接（在同一个模型中或跨模型）。

我们来创建一个*courses* 模型。每个课程应有一个 `teacher` 字段，链接到单个教师记录，但每个教师可以教授多门课程：*academy/models.py*

```
    name = fields.Char()
    biography = fields.Html()

class Courses(models.Model):
    _name = 'academy.courses'

    name = fields.Char()
    teacher_id = fields.Many2one('academy.teachers', string="Teacher")
```

*academy/security/ir.model.access.csv*

```
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_academy_teachers,access_academy_teachers,model_academy_teachers,,1,0,0,0
access_academy_courses,access_academy_courses,model_academy_courses,,1,0,0,0
```

我们也来添加视图，这样可以查看并编辑一个课程的教师：

*academy/views.xml*

```
    </field>
  </record>

  <record id="action_academy_courses" model="ir.actions.act_window">
    <field name="name">Academy courses</field>
    <field name="res_model">academy.courses</field>
  </record>
  <record id="academy_course_search" model="ir.ui.view">
    <field name="name">Academy courses: search</field>
    <field name="model">academy.courses</field>
    <field name="arch" type="xml">
      <search>
        <field name="name"/>
        <field name="teacher_id"/>
      </search>
    </field>
  </record>
  <record id="academy_course_list" model="ir.ui.view">
    <field name="name">Academy courses: list</field>
    <field name="model">academy.courses</field>
    <field name="arch" type="xml">
      <tree string="Courses">
        <field name="name"/>
        <field name="teacher_id"/>
      </tree>
    </field>
  </record>
  <record id="academy_course_form" model="ir.ui.view">
    <field name="name">Academy courses: form</field>
    <field name="model">academy.courses</field>
    <field name="arch" type="xml">
      <form>
        <sheet>
          <label for="name"/>
          <field name="name"/>
          <label for="teacher_id"/>
          <field name="teacher_id"/>
        </sheet>
      </form>
    </field>
  </record>

  <menuitem sequence="0" id="menu_academy" name="Academy"/>
  <menuitem id="menu_academy_content" parent="menu_academy"
            name="Academy Content"/>
  <menuitem id="menu_academy_content_courses"
            parent="menu_academy_content"
            action="action_academy_courses"/>
  <menuitem id="menu_academy_content_teachers"
            parent="menu_academy_content"
            action="action_academy_teachers"/>
```

还应当可以直接通过教师页面来新建一个课程，或者查看他们所教授的所有课程，因此为*teachers*模型添加一个`反向关联`：

*academy/models.py*

```
    name = fields.Char()
    biography = fields.Html()

    course_ids = fields.One2many('academy.courses', 'teacher_id', string="Courses")

class Courses(models.Model):
    _name = 'academy.courses'
```

*academy/views.xml*

```
      <form>
        <sheet>
          <label for="name"/> <field name="name"/>

          <label for="biography"/>
          <field name="biography"/>

          <field name="course_ids">
            <tree string="Courses" editable="bottom">
              <field name="name"/>
            </tree>
          </field>
        </sheet>
      </form>
    </field>
```

### 讨论和通知

Odoo提供了技术模型，它们不直接满足业务需求，但增加了无需手动构建而实现业务对象的可能性。

其中之一就是*Chatter* 系统，Odoo的email和消息系统的一部分，它可以为任意模型添加通知和讨论线程。模型只需要通过 `_inherit` `mail.thread`，并向其表单视图添加 `message_ids` 字段来展示讨论线程。讨论线程是记录级的。

对于我们的academy项目，允许有课程的讨论来处理如计划更改或进行教师和助教之间的讨论是有现实意义的：

*academy/models.py*

```
class Courses(models.Model):
    _name = 'academy.courses'
    _inherit = 'mail.thread'

    name = fields.Char()
    teacher_id = fields.Many2one('academy.teachers', string="Teacher")
```

*academy/views.xml*

```
          <label for="teacher_id"/>
          <field name="teacher_id"/>
        </sheet>
        <div class="oe_chatter">
          <field name="message_follower_ids" widget="mail_followers"/>
          <field name="message_ids" widget="mail_thread"/>
        </div>
      </form>
    </field>
  </record>
```

在每个课程表单的底部， 现在都会有一个讨论线程并让系统的用户可以进行留言或关注及取消关注与具体课程相关联的讨论。

### 销售课程

Odoo还提供业务模型，它允许更直接地使用或实现业务需求。例如 `website_sale` 模型根据Odoo系统中的产品来建立一个电商网站。我们可以轻松地把课程变成某种商品来让课程可销售。

不同于前面的经典继承，这表示通过*product*模型替换我们的*course*模型，并在当前位置扩展商品模型（为我们所需的内容）。

首先我们需要添加一个对`website_sale` 的依赖，因此可以获取商品 (通过 `sale`)及电商界面：

*academy/__manifest__.py*

```
    'version': '0.1',

    # any module necessary for this one to work correctly
    'depends': ['website_sale'],

    # always loaded
    'data': [
```

重启Odoo，更新模块，在网站中会出一个Shop版块，列出一系第（通过演示数据）预填充的商品。

第二步是通过`product.template`替换 *courses* 模型，并新增一个针对课程的商品分类：

*academy/__manifest__.py*

```
        'security/ir.model.access.csv',
        'templates.xml',
        'views.xml',
        'data.xml',
    ],
    # only loaded in demonstration mode
    'demo': [
```

*academy/data.xml*

```
<odoo>
  <record model="product.public.category" id="category_courses">
    <field name="name">Courses</field>
    <field name="parent_id" ref="website_sale.categ_others"/>
  </record>
</odoo>
```

*academy/demo.xml*

```
            <field name="name">Lester Vaughn</field>
        </record>

        <record id="course0" model="product.template">
            <field name="name">Course 0</field>
            <field name="teacher_id" ref="padilla"/>
            <field name="public_categ_ids" eval="[(4, ref('academy.category_courses'), False)]"/>
            <field name="website_published">True</field>
            <field name="list_price" type="float">0</field>
            <field name="type">service</field>
        </record>
        <record id="course1" model="product.template">
            <field name="name">Course 1</field>
            <field name="teacher_id" ref="padilla"/>
            <field name="public_categ_ids" eval="[(4, ref('academy.category_courses'), False)]"/>
            <field name="website_published">True</field>
            <field name="list_price" type="float">0</field>
            <field name="type">service</field>
        </record>
        <record id="course2" model="product.template">
            <field name="name">Course 2</field>
            <field name="teacher_id" ref="vaughn"/>
            <field name="public_categ_ids" eval="[(4, ref('academy.category_courses'), False)]"/>
            <field name="website_published">True</field>
            <field name="list_price" type="float">0</field>
            <field name="type">service</field>
        </record>

</odoo>
```

*academy/models.py*

```
     name = fields.Char()
    biography = fields.Html()

    course_ids = fields.One2many('product.template', 'teacher_id', string="Courses")

class Courses(models.Model):
    _inherit = 'product.template'

    teacher_id = fields.Many2one('academy.teachers', string="Teacher")
```

*academy/security/ir.model.access.csv*

```
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_academy_teachers,access_academy_teachers,model_academy_teachers,,1,0,0,0
```

*academy/views.xml*

```
     </field>
  </record>

  <menuitem sequence="0" id="menu_academy" name="Academy"/>
  <menuitem id="menu_academy_content" parent="menu_academy"
            name="Academy Content"/>
  <menuitem id="menu_academy_content_teachers"
            parent="menu_academy_content"
            action="action_academy_teachers"/>
```

安装之后，一些课程现在在Shop中就可以使用了，但需要进行查找。

- 在当前扩展模型，这无需新增`_name`即进行`继承`
- `product.template` 已经使用了讨论系统，因此我们可以从自己的扩展模型中删除它
- 我们所创建的课程默认为 *published* ，因此可以无需登录就进行查看

### 更改已有视图

截至目前，我们简单的学习了：

- 新模型创建
- 新视图创建
- 新记录创建
- 已有模型的修改

剩下对已有记录的修改及已有视图的修改。我们将对Shop页面进行这两者的修改。

视图的修改通过创建*继承*视图来实现，它应用于原视图之上并对其进行修改。这些修改视图可以无需对原始视图进行修改来新增或删除，这让我们进行试验和回滚都变得更为简单。

因为我们的课程是免费的，不需要在shop页面显示价格，因此我们将修改视图来在价格为0时进行婚期。第一个任务是找到展示价格的视图，这可通过Customize ‣ HTML Editor来实现，它让我们读取涉及渲染页面的不同模型。进行了一些查看后我们发现“Product item”可能是负责这个内容的。=

修改视图结构通过3个步骤来完成：

1. 新建视图
2. 通过设置新视图的 `inherit_id` 为所修改视图的外部 id 来扩展修改视图
3. 在这个结构中，使用`xpath`标记来选择并修改来自所修改视图中的元素

*academy/templates.xml*

```
                <div class="oe_structure"/>
            </t>
        </template>

        <template id="product_item_hide_no_price" inherit_id="website_sale.products_item">
            <xpath expr="//div[hasclass('product_price')]/b" position="attributes">
                <attribute name="t-if">product.price &gt; 0</attribute>
            </xpath>
        </template>

        <!-- <template id="object"> -->
        <!--   <h1><t t-esc="object.display_name"/></h1> -->
        <!--   <dl> -->
```

我们所做的第二个修改是让商品边栏通过默认的Customize ‣ Product Categories可见，这样可以开启、关闭一个树状商品分类（用于过滤主显示内容）。

这通过继承模型的 `customize_show` 和 `active` 字段来实现：一个继承模板（例如我们所创建的）可以为 *customize_show=True*。这一选项会在自定义菜单中以复选框进行显示，允许管理员启用或禁用它们（并易于自定义他们的网页）。

我们只需修改*Product Categories* 记录并设置其默认为 *active=”True”：*

*academy/templates.xml*

```
            </xpath>
        </template>

        <record id="website_sale.products_categories" model="ir.ui.view">
            <field name="active" eval="True"/>
        </record>

        <!-- <template id="object"> -->
        <!--   <h1><t t-es
```

至此， *Product Categories*边栏就会在安装了*Academy*模块之后自动启用了。