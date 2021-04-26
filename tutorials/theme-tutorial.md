# 主题教程

本文来自[Odoo 13官方文档之开发者文档](../README.md)系列文章

Odoo热爱自由。自由让设计师可以更进一步，自由让用户可以根据自己的需求来进行自定义。

准备好创建自己的主题了吗？很好。在开始前应该要知道一些内容。本教程旨在引导读者创建一个Odoo主题。

![Odoo 13开发者文档：主题教程](https://www.odoo.com/documentation/13.0/_images/Intro.jpg)

## 针对网页设计师的导言

如果你是初次使用Odoo的网页设计师，就来对地方了。本导言将勾勒出Odoo主题创建的基本框架。

Odoo团队创建了一个强大且易于使用的框架。使用这套工具无需预知任何特殊的语法。

### 从常用CMS转向Odoo

如果你经常使用同一种方式思考和工作，那么很可能得到的是相同的结果。如果想要一些全新的结果，那么请尝试一些新的事物。

> header.php文件来哪里呢?

这通常是习惯于使用Wordpress或Joomla的网页设计师在初次使用 Odoo 时所会问到的问题。

![img](https://www.odoo.com/documentation/13.0/_images/cms.jpg)

确实，在使用常见的CMS时，你需要对多个文件进行代码编写(如header.php, page.php, post.php等等)来为网站创建基本结构。使用这些系统，这一基本结构作为一个设计基础，可以随着时间推移不断更新来确保CMS的兼容性。因此可能在编写几个小时的代码之后还没有真正的开始设计的工作。

对于 Odoo 主题则**并非**如此。

我们认为主题设计应当简洁（而强大）。在创建我们的网站构造器时，我们决定从头开始而不是依赖于已有的其它框架。这一方法让我们有足够的自由去集中到对设计师真正重要的部分：样式、内容和它们背后的逻辑。而无需再挣扎于技术的细枝末节。

### Odoo默认主题结构

Odoo自带默认的主题结构。这是一个基本的“主题”，它提供最小的结构和布局。在新建主题时，你实际上是在对进行扩展。它在设置中保持启用并扮演像前面提到的CMS基础结构相似的角色，只是你无需创建并维护。它会在你的Odoo安装包中自动进行升级，因为其包含于网站构造器模块之中，默认所有部分都平滑的集成于其中

结果是，你可以完全的自由集中于设计上，由这个结构来完成集成和功能的任务。

![img](https://www.odoo.com/documentation/13.0/_images/def_structure.jpg)

**主要功能：**

- 页面、博客和电商的基本布局
- 集成网站构造器
- 基础代码片断
- 自动化的Less/Sass编译
- 自动化的Js和CSS最小化和合并

**主要技术:**

- Twitter Bootstrap
- jQuery
- jQuery UI
- underscore.js

## “模块化”思维

Odoo主题不是一个包含HTML或PHP文件的文件夹，它是一个使用XML编写的模块化的框架。以前从来没使用过XML文件？不必担心，在学习了本课程后，你就可以通过很基础的 HTML 知识来创建自己的每一个主题了。

使用经典的网页设计工作流，你通常要编写整个页面的布局。这样的结果是一个“静态”网页。当然可以更新内容，但你的客户会需要你来完成哪怕是最基础的修改。

为Odoo创建主题是一个全新的体验。你无需定义页面的整个布局，只要创建代码块（代码片断）来让用户选择在何处“拖放”它们，自己创建页面布局。我们称其为模块化设计。

想像Odoo主题是一个由创建并定义样式的元素和选项“列表”。作为设计师，你的目标是设计这些元素的样式来实现美妙的效果，而无需关注终端用户把它们放在哪里。

让我们开启“列表”元素之旅吧：

![img](https://www.odoo.com/documentation/13.0/_images/snippet.jpg)

#### 代码片断 (或构造版块)

一段HTML代码。 用户可以使用我们内置的网站构造器界面拖放、修改并修改它们。你可以定义为每个片断定义一组选项和样式。用户可以根据他们的需求进行选择。

![img](https://www.odoo.com/documentation/13.0/_images/page.jpg)

#### 页面

这些是普通的网页，只是对于终端用户可编辑并且可以你可以定义一个空的区域让用户可以通过把代码片断拖放到其中进行“填充”。

 

![img](https://www.odoo.com/documentation/13.0/_images/styles.jpg)

#### 样式

样式通过使用标准CSS文件(或Less/Sass)。你可以将样式定义为 **默认**或 **可选**。默认样式在主题中总是处于活跃状态，可选样式可以由用户启用或禁用。

![img](https://www.odoo.com/documentation/13.0/_images/functionalities.jpg)

#### 功能

借助于Odoo的模块化，所有内容可以做进下的自定义化。这表示有无限发挥你的创意的可能。添加功能很容易，为终端用户提供自定义选项也很简单。

### Odoo的XML文件概览

任意Odoo XML文件都以编码的定义开启。之后，可以在 `<odoo>` 标签中编写代码

```
[XML] 
<?xml version="1.0" encoding="utf-8" ?> 
<odoo>
   ## 此处编写代码 
</odoo>
```

几乎你所创建的所有元素和选项都要放在 `<template>` 标签中，如下例所示。

```
[XML]
 <template id="my_title" name="My title">
	<h1>This is an HTML block</h1>
  <h2 class="lead">And this is a subtitle</h2
</template>
```

### 划重点

不要误解 `template` 的含义。template标签仅用于定义一段html代码或选项 - 但不定与元素的视觉排列相关。

上面的代码定义了一个标题，但它不会在任何地方显示，因为该*template*没有与**Odoo默认结构** 存在任何关联。要进行关联，可以使用**xpath**, **qWeb** 或两者混用。

继续阅读本教程来学习如何使用你自己的代码进行适当的扩展。

### 更新你的主题

因为XML文件仅在安装主题时进行加载，你需要在每次对xml文件进行修改时强制进行重载。

要进行重载，点击模块页面中的升级（Upgrade）按钮。

![img](https://www.odoo.com/documentation/13.0/_images/restart.png)

![img](https://www.odoo.com/documentation/13.0/_images/upgrade_module.png)

## 创建一个主题模块

Odoo的主题像模块那样进行了打包。即使是为公司或客户设计一个很简单的网站，也需要将主题打包成Odoo模块。

- `main folder`

  创建一个文件夹并将其命名为： `theme_` 后接主题名。

- `__manifest__.py`

  创建一个空文档并将其保存到文件夹中，名为 `__manifest__.py`。这将包含你主题的配置信息。

- `__init__.py`

  创建一个空文件并命名为 `__init__.py`。这是必需的系统文件。创建后内容留空。

- `views` 和 `static` 文件夹

  在主文件夹中创建这两个文件夹。 在 `views` 中放置定义代码片断、页面和选项的 xml文件。`static` 文件夹是放置样式、图片和自定义 js 代码的相应位置。

### 划重点

.在odoo 和 init 文件开头和结束处分别使用两个下划线字符，

最终结果类似下面这样：

![img](https://www.odoo.com/documentation/13.0/_images/folder.jpg)

### 编辑 `__manifest__.py`

打开所创建的 `__manifest__.py` 文件并复制/粘贴如下内容：

```
{
  'name':'Tutorial theme',
  'description': 'A description for your theme.',
  'version':'1.0',
  'author':'Your name',

  'data': [
  ],
  'category': 'Theme/Creative',
  'depends': ['website'],
}
```

将前4个属性值替换为你所想要的内容。这些值将用于标识你在Odoo后台中的新主题。

`data` 属性将包含xml文件列表。现在是空的，但我们将添加新新建的文件。

`category` 定义你的模块分类（一直是“Theme”）并且在斜杠后放置子类别。可以使用Odoo应用分类列表的子分类。 (https://www.odoo.com/apps/themes)

`depends` 指定我们的主题正常运行所需的模块。对本课程的主题，只需要用到website。如果你还需要用到博客或电商功能，还需要添加下面这些模块。

```
...
'depends': ['website', 'website_blog', 'sale'],
...
```

### 安装你的主题

要安装主题，需要将主题文件夹放到Odoo安装所在的addons目录中。

然后，导航到Odoo的 **Website** 模块，进入Configuration ‣ Settings。

在 **Website** 版块中，点击 **Choose a theme** 按钮，然后悬停在主题上并点击 **Use this theme**。

## Odoo页面的结构

Odoo页面是一个两种元素组合的视觉结果： **cross-pages** 和 **unique**。默认，Odoo提供一个**Header** 和一个 **Footer** (跨页面) 以及包含让页面独特的内容的一个的主元素。

跨页面元素在每个页面都相同。独立元素仅与特定页面相关。

![img](https://www.odoo.com/documentation/13.0/_images/page_structure.jpg)To inspect the 默认布局，仅使用网站构造器新建一个页面。点击Content ‣ New Page并添加一个页面名称。使用浏览器查看页面元素。

```
<div id=“wrapwrap”>
  <header />
  <main />
  <footer />
</div>
```

### 扩展默认 Header

默认，Odoo的header 包含一个响应式的导航菜单和公司的logo。你可以轻易地对已有内容添加新元素或样式。.

要进行添加，在**views**文件夹中创建一个**layout.xml** 文件并添加默认的Odoo xml 标记。

```
<?xml version="1.0" encoding="utf-8" ?>
<odoo>



</odoo>
```

在`<odoo>` 标签中新建一个模板，复制-粘贴以下代码：

```
<!-- Customize header  -->
<template id="custom_header" inherit_id="website.layout" name="Custom Header">

  <!-- Assign an id  -->
  <xpath expr="//div[@id='wrapwrap']/header" position="attributes">
    <attribute name="id">my_header</attribute>
  </xpath>

  <!-- Add an element after the top menu  -->
  <xpath expr="//div[@id='wrapwrap']/header/div" position="after">
    <div class="container">
      <div class="alert alert-info mt16" role="alert">
        <strong>Welcome</strong> in our website!
      </div>
    </div>
  </xpath>
</template>
```

第一个xpath将在header添加 id `my_header` 。最好的方法是将css规则定位到该元素并且避免影响到页面上的其它内容。

### 警告

在替换默认元素属性时要非常小心。因为你的主题会继承默认主题，你的修改优先级会高于未来Odoo的更新。

第二个xpath将紧随导航菜单之后添加一条欢迎消息。

最后一步是将layout.xml添加到主题所使用的xml文件列表中。此时应像下面这样编辑 `__manifest__.py` 文件：

```
'data': [ 'views/layout.xml' ],
```

更新你的主题

![img](https://www.odoo.com/documentation/13.0/_images/restart.png)非常棒！我们成功的在header中添加了一个id并在导航菜单中添加了一个元素。这些修改将应用于网站的每个页面。

![img](https://www.odoo.com/documentation/13.0/_images/after-menu.png)

## 创建一个具体的页面布局

想像我们要创建一个服务页面的具体布局。对于这一页面，我们需要在顶部添加一个服务列表并赋予客户使用代码片段设置页面剩余内容的可能性。

在*views* 文件夹内，创建一个 **pages.xml** 文件并添加默认的Odoo 标记。在 `<odoo>`中，代替 `<template>`的定义，我们将创建一个*page* 对象。

```
<?xml version="1.0" encoding="utf-8" ?>
<odoo>

     <!-- === Services Page === -->
     <record id="services_page" model="website.page">
         <field name="name">Services page</field>
         <field name="website_published">True</field>
         <field name="url">/services</field>
         <field name="type">qweb</field>
         <field name="key">theme_tutorial.services_page</field>
         <field name="arch" type="xml">
             <t t-name="theme_tutorial.services_page_template">
                 <h1>Our Services</h1>
                 <ul class="services">
                     <li>Cloud Hosting</li>
                     <li>Support</li>
                     <li>Unlimited space</li>
                 </ul>
             </t>
         </field>
     </record>

 </odoo>
```

可以看到，页面中有很多额外的属性，如 *name* 或 可以进行访问的 *url* 。

我们成功地创建了一个新页面布局，但还没有告诉系统**如何使用它**。这时，我们可以使用 **QWeb**。将html代码包裹在 `<t>` 标签中，如下例所示。

```
<!-- === Services Page === -->
<record id="services_page" model="website.page">
    <field name="name">Services page</field>
    <field name="website_published">True</field>
    <field name="url">/services</field>
    <field name="type">qweb</field>
    <field name="key">theme_tutorial.services_page</field>
    <field name="arch" type="xml">
        <t t-name="theme_tutorial.services_page_template">
            <t t-call="website.layout">
                <div id="wrap">
                    <div class="container">
                        <h1>Our Services</h1>
                        <ul class="services">
                            <li>Cloud Hosting</li>
                            <li>Support</li>
                            <li>Unlimited space</li>
                        </ul>
                    </div>
                </div>
            </t>
        </t>
    </field>
</record>
```

使用 `<t t-call="website.layout">`我们将通过自己的代码扩展Odoo的默认页面布局。

可以看坏以，我们将自己的代码放到了两个 `<div>`中，一个带有 ID `wrap` ，另一个带有class `container`。这是提供最小化的布局。

下一步是添加一个空区域，用户可以在其中添加代码片断。要实现这点，只需在封闭`div#wrap`元素前创建一个`oe_structure`类的 `div` 。

```
<?xml version="1.0" encoding="utf-8" ?>
<odoo>

    <!-- === Services Page === -->
    <record id="services_page" model="website.page">
        <field name="name">Services page</field>
        <field name="website_published">True</field>
        <field name="url">/services</field>
        <field name="type">qweb</field>
        <field name="key">theme_tutorial.services_page</field>
        <field name="arch" type="xml">
            <t t-name="theme_tutorial.services_page_template">
                <t t-call="website.layout">
                    <div id="wrap">
                        <div class="container">
                            <h1>Our Services</h1>
                            <ul class="services">
                                <li>Cloud Hosting</li>
                                <li>Support</li>
                                <li>Unlimited space</li>
                            </ul>

                            <!-- === Snippets' area === -->
                            <div class="oe_structure" />
                        </div>
                    </div>
                </t>
            </t>
        </field>
    </record>

</odoo>
```

你可以创建所要数量的代码片断区域，并将它们放在页面的任意位置。

值得一提的是我们前面所见的`<template>`指令有替代方法可创建页面。

```
<?xml version="1.0" encoding="utf-8" ?>
<odoo>

    <!-- === Services Page === -->
    <template id="services_page_template">
        <t t-call="website.layout">
            <div id="wrap">
                <div class="container">
                    <h1>Our Services</h1>
                    <ul class="services">
                        <li>Cloud Hosting</li>
                        <li>Support</li>
                        <li>Unlimited space</li>
                    </ul>

                    <!-- === Snippets' area === -->
                    <div class="oe_structure" />
                </div>
            </div>
        </t>
    </template>
    <record id="services_page" model="website.page">
        <field name="name">Services page</field>
        <field name="website_published">True</field>
        <field name="url">/services</field>
        <field name="view_id" ref="services_page_template"/>
    </record>

</odoo>
```

这将允许页面内容使用 `<xpath>`做更进一步的自定义

我们的页面准备基本完毕。现在需要在**__manifest__.py** 文件中添加**pages.xml**

```
'data': [
  'views/layout.xml',
  'views/pages.xml'
],
```

更新你的主题

![img](https://www.odoo.com/documentation/13.0/_images/restart.png)太好了，我们的服务页面已准备就绪，可以通过`<yourwebsite>/services`（以上我们所选择的URL）来进行访问了。

你会注意到可以在 *Our Services*列表下方拖放代码片断。

![img](https://www.odoo.com/documentation/13.0/_images/services_page_nostyle.png)现在让回到我们的*pages.xml* ，在页面模板后复制粘贴如下代码。

```
<record id="services_page_link" model="website.menu">
  <field name="name">Services</field>
  <field name="page_id" ref="services_page"/>
  <field name="parent_id" ref="website.main_menu" />
  <field name="sequence" type="int">99</field>
</record>
```

这段代码会为主菜单添加一个链接，指向我们所创建的页面。

![img](https://www.odoo.com/documentation/13.0/_images/services_page_menu.png)**sequence** 属性定义了链接在顶部菜单的位置。在本例中，我们设置其值为 `99` 来将其放到最后。如果你希望将其放在具体的位置上，需要替换该值为所需的值。

如你在`website`模块的*data.xml* 文件中所见，默认 **Home** 链接设置为`10` 而 **Contact us** 页面则设置为 `60` 。例如假如你想要将自己的链接放在**中间**，可以设置链接的排序为 `40`。

## 添加样式

Odoo默认包含了 Bootstrap。这表示你可以马上使用所有的 Bootstrap 样式以及布局功能。

当然如果你想要提供一个独特的页面 Bootstrap 是不够的。以下步骤将引导你了解如何为自己的主题添加自定义样式。最终的结果不会很漂亮，但可以提供足够的信息让你自己继续进行构建。

我们先创建一个名为 **style.scss** 的空文件将将其放到static目录下名为 **scss** 的文件夹下。以下的规则会为我们的*服务*页面添加样式。复制粘贴并保存文件。

```
.services {
    background: #EAEAEA;
    padding: 1em;
    margin: 2em 0 3em;
    li {
        display: block;
        position: relative;
        background-color: #16a085;
        color: #FFF;
        padding: 2em;
        text-align: center;
        margin-bottom: 1em;
        font-size: 1.5em;
    }
}
```

我们的文件准备就绪了，但还没有包含到主题当中。

让我们导航到 view 文件夹并创建一个名为*a**ssets.xml*的XML文件。添加默认的 Odoo xml 标记并复制粘贴如下代码。记得用你主题的主文件夹名称替换掉 `theme folder` 。

```
<template id="mystyle" name="My style" inherit_id="website.assets_frontend">
    <xpath expr="link[last()]" position="after">
        <link rel="stylesheet" type="text/scss" href="/theme folder/static/scss/style.scss"/>
    </xpath>
</template>
```

我们刚刚创建了一个指定scss文件的template。如你所见，我们的模板有一个特殊属性，名为 `inherit_id`。 该属性告诉Odoo我们的模板引用了k 另一个template来实现正常操作。

本例中，我们引用了 `assets_frontend` 模板，它位于 `website` 模块中。 `assets_frontend` 指定了由网站构造器所加载的资源列表，我们的目标是在这个列表中添加自己的scss 文件。

这可以通过使用带有`expr="link[last()]"` 和 `position="after"`属性的xpath来实现，它表示“*将我的样式文件并将其放到资源列表的最后一个链接后面*”。

将其放到最后面，这样我们可以确保它最后加载并具有优先级。

最后在**__manifest__.py**文件中添加 **assets.xml。**

更新主题

![img](https://www.odoo.com/documentation/13.0/_images/restart.png)我们的 scss 文件现在包含在主题当中了，它将会自动编译、最小化并与所有 Odoo 资源进行合并。

![img](https://www.odoo.com/documentation/13.0/_images/services_page_styled.png)

## 创建代码片断

因代码片断是用户设计和布局页面的方式，它们是你设计时最重要的元素。我们来为服务页面创建一个代码片断。代码片断将显示3段文字并可由终端用户通过网站构造器界面进行编辑。导航至view文件夹并创建名为**snippets.xml** 的XML文件。添加默认的Odoo xml标记并复制/粘贴如下代码。template包含有可由代码片断显示的HTML标记。

```
<template id="snippet_testimonial" name="Testimonial snippet">
  <section class="snippet_testimonial">
    <div class="container text-center">
      <div class="row">
        <div class="col-lg-4">
          <img alt="client" class="rounded-circle" src="/theme_tutorial/static/src/img/client_1.jpg"/>
          <h3>Client Name</h3>
          <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit.</p>
        </div>
        <div class="col-lg-4">
          <img alt="client" class="rounded-circle" src="/theme_tutorial/static/src/img/client_2.jpg"/>
          <h3>Client Name</h3>
          <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit.</p>
        </div>
        <div class="col-lg-4">
          <img alt="client" class="rounded-circle" src="/theme_tutorial/static/src/img/client_3.jpg"/>
          <h3>Client Name</h3>
          <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit.</p>
        </div>
      </div>
    </div>
  </section>
</template>
```

可以看到，我们对这三栏使用了Bootstrap默认类。不仅是关乎布局，这些类将**由网站构造器触发让用户可以调整它们的尺寸**。

前面的代码将创建代码片断内容，但我们需要将其放到编辑器栏中，这样用户可以将其拖放到页面中。复制/粘贴这个模板到你的 **snippets.xml** 文件中。

```
<template id="place_into_bar" inherit_id="website.snippets" name="Place into bar">
  <xpath expr="//div[@id='snippet_structure']/div[@class='o_panel_body']" position="inside">
    <t t-snippet="theme_tutorial.snippet_testimonial"
       t-thumbnail="/theme_tutorial/static/src/img/ui/snippet_thumb.jpg"/>
  </xpath>
</template>
```

使用xpath，我们通过 id `snippet_structure`定位到具体的元素。这表示这个代码片段会出现在Structure标签栏中。如果希望修改目标标签栏，需要在xpath中替换掉`id`值。

![img](https://www.odoo.com/documentation/13.0/_images/snippet_bar.png)

| 标签栏名称 | Xpath表达式                      |
| ---------- | -------------------------------- |
| Structure  | `//div[@id='snippet_structure']` |
| Content    | `//div[@id='snippet_content']`   |
| Feature    | `//div[@id='snippet_feature']`   |
| Effect     | `//div[@id='snippet_effect']`    |

`<t>`标签将调用我们的代码片断模板并赋值一个img文件夹中放置的缩略图。现在可以从代码片断栏中将其播放到页面中并查看效果。

![img](https://www.odoo.com/documentation/13.0/_images/snippet_default.png)

## 代码片断选项

选项允许发面都使用网站构造器界面编辑代码片断的外观。借助网站构造器的功能，你可以轻松地创建一个代码片断选项并自动将它们添加到界面中。

### 选项组属性

选项封装成组。组中有属性来定义所包含选项如何与用户界面进行交互。

- `data-selector="[css selector(s)]"`

  将所有选项绑定到针对具体元素的组。

- `data-js=" custom method name "`

  用于绑定自定义Javascript方法。

- `data-drop-in="[css selector(s)]"`

  定义代码片断所放置入的元素列表。

- `data-drop-near="[css selector(s)]"`

  定义可将代码片断放置在其附近的代码片断。

### 默认选项方法

选项向脚本应用标准CSS类。根据你所选择的方法，用户界面的行为会有所区别。

- `data-select-class="[class name]"`

  在相同组内的多个data-select-class定义一系列用户可选择应用的类。一次仅能启用一个选项。

- `data-toggle-class="[class name]"`

  data-toggle-class用于从代码片断列表中应用一个或多个 CSS类。可同时应用多个。

我们来演示默认选项与用于一个基本示例。

我们一开始在views文件夹中新增一个文件，将其命名为**options.xml** 并在其中滞默认的Odoo XML标记。新建模板复制、粘贴如下内容：

```
<template id="snippet_testimonial_opt" name="Snippet Testimonial Options" inherit_id="website.snippet_options">
  <xpath expr="//div[@data-js='background']" position="after">
    <div data-selector=".snippet_testimonial"> <!-- Options group -->
      <li class="dropdown-submenu">
        <a href="#">Your Option</a>
        <div class="dropdown-menu"> <!-- Options list -->
          <a href="#" class="dropdown-item" data-select-class="opt_shadow">Shadow Images</a>
          <a href="#" class="dropdown-item" data-select-class="opt_grey_bg">Grey Bg</a>
          <a href="#" class="dropdown-item" data-select-class="">None</a>
        </div>
      </li>
    </div>
  </xpath>
 </template>
```

以上的模板将扩展默认的**snippet_options模板**，将我们的选项添加到 **background** 选项之后options (xpath expr 属性)。要以具体的顺序放置选项，查看**website模块**中的 **snippet_options模板** 并将你的选项添加到所需位置的之前或之后。

可以看到，我们将所有的选项放到一个 DIV 标签内，那会将它们定位到正确的选择器上 (`data-selector=".snippet_testimonial"`)。

要定义我们的选项，我们对`li`元素中应用了 `data-select-class` 属性。在用户选择一个选项时，属性中所包含的类将自动应用于该元素。

因为 `selectClass` 方法避免了多选，最后一个“empty”选项将重置代码片断为默认值。

在 `__manifest__.py` 中添加**options.xml**并更新主题。

![img](https://www.odoo.com/documentation/13.0/_images/restart.png)把我们的代码片断放到页面中，你会注意到新选项自动添加到了自定义菜单中。查看日志，你会注意到在选择一个选项时该类会应用到元素上。

![img](https://www.odoo.com/documentation/13.0/_images/snippet_options.png)我们来创建一些css规则，来为我们的选择添加视觉效果。打开 **style.scss** 文件并添加如下内容：

```
.snippet_testimonial {
  border: 1px solid #EAEAEA;
  padding: 20px;
}

// These lines will add a default style for our snippet. Now let's create our custom rules for the options.

.snippet_testimonial {
  border: 1px solid #EAEAEA;
  padding: 20px;

  &.opt_shadow img {
    box-shadow: 0 2px 5px rgba(51, 51, 51, 0.4);
  }

  &.opt_grey_bg {
    border: none;
    background-color: #EAEAEA;
  }
}
```

![img](https://www.odoo.com/documentation/13.0/_images/snippet_options2.png)太棒了！我们成功地为代码片断创建了选项。

每当发布者点击一个选项时，系统会添加在 data-select-class中指定的类。

通过将 `data-select-class` 替换为 `data-toggle-class` ，你将能够同时选择多个类。

### Javascript选项

如果你需要执行简单的类修改操作的话`data-select-class` 和 `data-toggle-class` 会非常好。但如果代码片断的自定义要求有更多操作呢？

如前面所述， `data-js` 属性可分配给一个选项组，来定义一个自定义方法。我们通过向早前创建的选项组div添加`data-js`属性来为*testimonials*代码片断创建一个选项。

```
<div data-js="snippet_testimonial_options" data-selector=".snippet_testimonial">
  [...]
</div>
```

搞定。 从现在开始，网站构造器在发布者每次进入编辑模式时都会查找 `snippet_testimonial_options` 方法。

我们再进一步，创建一个名为**tutorial_editor.js** 的javascript文件并将其放到 **static** 文件夹下。复制、粘贴如下代码：

```
(function() {
    'use strict';
    var website = odoo.website;
    website.odoo_website = {};
})();
```

很好，我们成功地创建了自己的javascript编辑器文件。 该文件将包含编辑模式下我们的代码片断所使用的javascript函数。我们来使用前面创建的`snippet_testimonial_options`方式】法为证言代码片断新建一个函数。

```
(function() {
    'use strict';
    var website = odoo.website;
    website.odoo_website = {};

    website.snippet.options.snippet_testimonial_options = website.snippet.Option.extend({
        onFocus: function() {
            alert("On focus!");
        }
    })
})();
```

可以注意到，我们使用了一个名为 `onFocus`的方法来触发我们的函数。网站构造器提供了一些你可以用于触发自定义函数的事件。

| 事件           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| `start`        | 在编辑会话中发布者第一次选中代码片断时或播放到页面中时触发。 |
| `onFocus`      | 每次用户选择脚本时或将脚本播放到页面中时触发。               |
| `onBlur`       | 在代码片断失去焦点是发生该事件。                             |
| `onClone`      | 在复制了代码片断后触发。                                     |
| `onRemove`     | 在删除代码片断之彰触发。                                     |
| `onBuilt`      | 在将代码片断拖放到拖放区域时触发。事件触发时，内容已插入到页面中。 |
| `cleanForSave` | 在发布者保存页面时触发。                                     |

我们来往编辑器资源列表中新增一些javascript文件。回到 **assets.xml** 并新建像前一个那样的 template。这次我们需要继承 `assets_editor`来代替 `assets_frontend`。

```
<template id="my_js" inherit_id="website.assets_editor" name="My Js">
  <xpath expr="script[last()]" position="after">
    <script type="text/javascript" src="/theme_tutorial/static/src/js/tutorial_editor.js" />
  </xpath>
</template>
```

更新你的主题

![img](https://www.odoo.com/documentation/13.0/_images/restart.png)我们来测试新的javascript函数。进入编辑模式并播放入页面。你应该可以看到我们绑定到`onFocus`事件上的javascript警告。如果关闭然后点击代码片断外部，接着再次在其中进行点击，该事件会再次被触发。

![img](https://www.odoo.com/documentation/13.0/_images/snippet_custom_method.png)

## 编辑手册指南

Basically all the elements in a page can be edited by the publisher. Besides that, some element types and css classes will trigger special Website Builder functionalities when edited.

### 布局

- `<section />`

  Any section element can be edited like a block of content. The publisher can move or duplicate it. It’s also possible to set a background image or color. Section is the standard main container of any snippet.

- `.row > .col-lg-*`

  Any large bootstrap columns directly descending from a .row element, will be resizable by the publisher.

- `contenteditable="False"`

  This attribute will prevent editing to the element and all its children.

- `contenteditable="True"`

  Apply it to an element inside a contenteditable=”False” element in order to create an exception and make the element and its children editable.

- `<a href=”#” />`

  In Edit Mode, any link can be edited and styled. Using the “Link Modal” it’s also possible to replace it with a button.

### Media

- `<span class=”fa” />`

  Pictogram elements. Editing this element will open the Pictogram library to replace the icon. It’s also possible to transform the elements using CSS.

- `<img />`

  Once clicked, the Image Library will open and you can replace images. Transformation is also possible for this kind of element.

```
<div class="media_iframe_video" data-src="[your url]" >
  <div class="css_editable_mode_display"/>
  <div class="media_iframe_video_size"/>
  <iframe src="[your url]"/>
</div>
```

This html structure will create an `<iframe>` element editable by the publisher.

## SEO最佳实践

### 利用内容插入

现代搜索引擎算法越来越关注内容，这表示对于**关键词密度**的关注减少而对于内容是否与**关键词真实相关**的关注增多。

因为内容对于SEO如此重要，你应当集中于给发布者可以轻松插入内容的工具。代码片断为“响应式内容”也会重要，即应适配发布者的内容，不论其长短。

我们来看这个经典两列代码一片断的示例，通过两种不同的方式来实现。

![img](https://www.odoo.com/documentation/13.0/_images/seo_snippet_wrong.png)

**不好的做法**

使用固定大小图片，发布者将不得不限制文本来遵循这一布局。

![img](https://www.odoo.com/documentation/13.0/_images/seo_snippet_good.png)

**好的做法**

使用适配列高的背景图片，发布者可自由添加内容，而无需顾及图片的高度。

### 页面分隔

基本上，页面分区表示页面被分成几个独立的部分，而这些部分被搜索引擎看作独立的词条。在你设计页面或代码片断时，应当确保使用正确的标签来有助于搜索引擎的索引。

- `<article>`

  指定一个独立的内容区。其中应为一块自包含的内容并且本身具有意义。你可以对 `<article>` 元素进行彼此间的嵌套。在这种情况下，它表示嵌套的元素与外部的 `<article>` 元素相关联。

- `<header>`

  表示内容自包含区块（`<article>`）的header区域。

- `<section>`

  是代码片断的默认标签，并且它指定内容区块的子区域。它可用于将`<article>` 内容分隔为几个部分。推荐使用标题元素(`<h1>` – `<h6>`)来定义section主题。

- `<hgroup>`

  用于幸运糖果屋标题 (`<h1>` - `<h6>`)区域。一个较好的例子是在顶部包含标题和子标题的article：`<hgroup>  <h1>Main Title</h1>  <h2>Subheading</h2> </hgroup>`

### 描述你的页面

#### 定义关键词

你应当使用恰当、相关的关键词及这些关键词的近义词。可以使用页面顶栏中可以找到的内置“Promote”函数来对每个页面定义这些关键词。

#### 定义标题和描述

使用“Promote”函数来定义它们。 保持页面标题简短并为页面添加主关键词词组。好标题会带来情感响应、设问或预示某些内容。

描述（Description），虽然对于搜索引擎排名不太重要，但对于获取用户点击率极其重要。这些是为内容打广告的机会，让那些搜索的人们可以精准地知道页面中是否包含他们所需要的内容。每个页面的标题和描述保持唯一性非常重要。