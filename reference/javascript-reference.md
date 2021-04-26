# Javascript手册

本文来自[Odoo 13官方文档之开发者文档](https://alanhou.org/odoo-13-developer-documentation/)系列文章

本文档讲解Odoo Javascript框架。这个框架按照代码行数来说不是个大应用，但非常通用，因为它基本是一个将声明式接口转换来在线应用的设备，能够与模型及数据库中的每条记录进行交互。甚至还能使用网页客户端来修改网页客户端界面。

Odoo中的所有 js 文档字符串的 html版本位于： [JS API](https://www.odoo.com/documentation/13.0/reference/javascript_api.html#api-js)

## 概览

该Javascript框架设计用来处理3种用例：

*   网页客户端：这是一个私有网页应用，人们可以浏览和编辑业务数据。这是一个单页面应用 (页面不会重新加载，仅在需要时从服务端获取新的数据)
*   网站：Odoo对外的部分。它允许未认证的用户作为客户端浏览某些内容、购物或执行一些动作。这是一个典型的网站：有一些控制器路由以及一些javascript代码来让其生效。
*   销售点 POS：这是一个用于 POS 的界面。是一个专业性的单页面应用。

某些javascript代码对这3种用例是通用的，打包在一起（参见下方的资源版块）。本文档会集中于网页客户端的设计上。

## 网页客户端

### 单页面应用

简言之， *webClient*，*WebClient* 的实例是整个用户界面的根控件。其职责是编排所有子控件并提供服务，如rpc、本地存储等。

在运行时，网页客户端是一个单页面应用。它不需要在每次用户执行动作时向服务端请求整个页面。而是只请求所需要的部分，然后在视图中进行替换/更新。同时，它管理着url：与网页客户端状态保持着同步。

这表示在用户使用Odoo时， 网页客户端类(及动作管理器)实际创建和销毁众多子控件。状态是高度动态的，每个控件在任意时刻都可被销毁。

### 网页客户端JS代码概览

这里，我们给出一个网页客户端代码的快速概览，位于*web/static/src/js* 插件中。注意我们刻意没有穷举所有内容。仅包含了最重要的一些文件/文件夹。

*   *boot.js*: 这是定义模块系统的文件。需要首先进行加载。
*   *core/*: 这是底层构建块的集合。它包含类系统、控件系统、并发工具以及很多其它类/函数。
*   *chrome/*: 在这个文件夹中，有大部分组成主要用户界面的大控件。
*   *chrome/abstract_web_client.js* 及 *chrome/web_client.js*: 这些文件共同定义WebClient控件，即网页客户端的根控件。
*   *chrome/action_manager.js*: 这是将动作转化为控件的代码 (如看板或表单视图)
*   *chrome/search_X.js* 所有这些文件定义了搜索视图 (不是网页客户端角度的视图，而是服务端角度的视图)
*   *fields*: 所有主要的字段控件都在这里定义
*   *views*: 视图所处的位置

## 资源管理

在Odoo中管理资源并不像其它应用那样直接。其中一个原因是有很多种状况，只需要使用到部分资源。例如，网页客户端、POS、网站甚至是移动端应用的需求都是不同的。同时，有些资源会很大，但不大会用到。这时，我们就会希望进行懒加载。

主要的思想是我们在 xml 中定义一组包(bundle)。这里的包是文件集合(javascript, css, scss)。在Odoo中，最重要的包在 *addons/web/views/webclient_templates.xml*文件中定义。像下面这样：

```
<template id="web.assets_common" name="Common Assets (used in backend interface and website)"> 
  <link rel="stylesheet" type="text/css" href="/web/static/lib/jquery.ui/jquery-ui.css"/> 
  ... 
  <script type="text/javascript" src="/web/static/src/js/boot.js"></script> 
  ... 
</template>
```

然后可以通过*t-call-assets* 指令将包中的文件插入到模板中：

```
<t t-call-assets="web.assets_common" t-js="false"/>
<t t-call-assets="web.assets_common" t-css="false"/>
```

通过这些指令由服务端渲染模板时发生的状况如下：

*   包中描述的所有*scss*文件都编译为css文件。名为*file.scss*的文件将会编译为*file.scss.css*文件。
*   如果在*debug=assets* 模式下，

    *   *t-js*属性设置为 false 的*t-call-assets* 指令会被一个指向 css文件的样式表标签列表替换
*   *t-css*属性设置为 false 的 *t-call-assets* 指令会由一个指向js文件的脚本标签列表替换
*   如果不在*debug=assets* 模式下，

    *   css文件会合并且最小化，然后生成样式表标签
    *   js 文件会合并且最小化，然后生成脚本标签


注意资源文件进行了缓存，因此理论上，浏览器仅需加载它们一次。

### 主要的包

在启动Odoo服务时，会检查包中每个文件的时间戳，如有必要会创建/重建相应的包。

以下是开发者需要知道的一些重要的包：

*   *web.assets_common*: 这个包含有网页客户端以及 POS 中通用的大部分资源。一般它包含 odoo 框架的底层构建块。注意它包含了定义 odoo 模块系统的*boot.js* 文件。
*   *web.assets_backend*: 它个包含有针对网页客户端的代码 (主要有网页客户端/动作管理器/视图)
*   *web.assets_frontend*: 这个包为对公网站的所有相关内容：电商、论坛、博客、事件管理…

### 在资源包中添加文件

将*addons/web* 中的文件添加到包中的正确方式很简单：只需在*webclient_templates.xml*文件中将*script* 或*stylesheet* 标签添加到包中即可。但在处理不同的插件时，我们需要添加该插件中的文件。此时，应按照如下3个步骤完成：

1.  在*views/*文件夹中添加*assets.xml* 文件
2.  在声明文件的‘data’键中添加字符串‘views/assets.xml’
3.  创建所需包的一个继承视图，并通过xpath表达式添加文件。例如，

```
<template id="assets_backend" name="helpdesk assets" inherit_id="web.assets_backend">
    <xpath expr="//script[last()]" position="after">
        <link rel="stylesheet" type="text/scss" href="/helpdesk/static/src/scss/helpdesk.scss"/>
        <script type="text/javascript" src="/helpdesk/static/src/js/helpdesk_dashboard.js"></script>
    </xpath>
</template>
```

注意包中的文件都在用户别裁 Odoo网页客户端时进行加载。这表示这些文件每次都会通过网络传输 (浏览器中启用了缓存的情况除外)。在某些情况下，对一些资源进行懒加载会更好。例如，若控件需要一个更大的库，并且该控件不是体验的核心部分，在控件实际创建时才加载该库可能是一个好r 的方式。控件类有针对这种用例的内置支持。 (参见[QWeb模板引擎](#reference-javascript-reference-qweb)一节)

### 如果文件不加载/更新的话怎么办

文件没有正确加载有很多种原因。下面是一些可以尝试解决的方面：

*   启动了服务端时，它不知道资源文件是否进行了修改。因此，你只需要重启服务再次生成资源。
*   （在开工具中）查看终端 (通常使用F12打开)来确保没有明显的错误
*   试着在文件开始处（模块定义之前）添加一个console.log，因此你可以查看文件是否进行了加载
*   在用户界面中的debug模式下 ，有一些强制服务端更新资源的选项。
*   使用*debug=assets* 模式。这会跳过资源打包 (注意这没有实际解决问题。服务端将使用过时的包)
*   最后，对开发者来说最方便的方式是使用 *–dev=all* 选项启动服务。 这会启动文件监视器选项，自动在必要时置资源为无效。注意在操作系统是Windows时效果不是很好。
*   记得刷新页面！
*   或者是保存代码文件…

一旦资源进行了重建，需要刷新页面来重新加载相应的资源 (如果没起效果，文件可能是被缓存了)。

## Javascript模块系统

一旦我们可以向浏览器加载文件，需要确保加载的顺序正确。要做到这点，Odoo定义了一个小模块系统 (位于 *addons/web/static/src/js/boot.js*文件中，需要先进行加载)。

Odoo模块系统受AMD启发，通过对全局odoo对象定义函数 *define* 来实现。然后我们通过调用该函数来定义各个javascript模块。在Odoo框架中，模块只是一段很快会执行的代码。它有名称并可能会有依赖。在加载其依赖时，也会加载模块。然后模块值会成为定义该模块函数的返回值。

示例如下：

```
// in file a.js
odoo.define('module.A', function (require) {
    "use strict";

    var A = ...;

    return A;
});

// in file b.js
odoo.define('module.B', function (require) {
    "use strict";

    var A = require('module.A');

    var B = ...; // something that involves A

    return B;
});

定义模块的一种替代方式是显式地在第二个参数中给定依赖列表。

```
odoo.define('module.Something', ['module.A', 'module.B'], function (require) {
    "use strict";

    var A = require('module.A');
    var B = require('module.B');

    // some code
});
```

如果有些依赖缺失/未准备就绪，就不会加载该模块。几秒扣在控制台中会打印警告。

注意不支持循环依赖。这完全讲得通，但也意味着开发者需要格外小心。

### 定义模块

*odoo.define* 方法给出了三个参数：

*   *moduleName*: javascript模块的名称。它应是一个唯一字符串。惯例是在特定的描述后加上odoo插件的名称。例如，‘web.Widget’ 描述在*web* 插件中定义模块，它导出一个 *Widget* 类 (因为第一个字母大写了)。如果名称不唯一，会抛出异常并在控制台中显示。
*   *dependencies*: 第二个参数是可选的。若给定，它应是一个字符串列表，每个对应一个 javascript模块。它描述了应在模块执行前要求加载的依赖。如果这里没有显式地给定依赖，那么模块系统会通过对其调用toString来从函数中提取，然后使用regexp 来查找所有的 *require* 语句。

```
odoo.define('module.Something', ['web.ajax'], function (require) {
    "use strict";

    var ajax = require('web.ajax');

    // some code here
    return something;
});
```

*   最后一个参数是定义模块的函数。其返回值是模块的值，可传递给其它使用它的模块。注意对于异步模块有一个小例外，参见下一部分。

如果发生了错误，（在调试模式下）会在控制台中进行记录：

*   `Missing dependencies`: 这些模块没在页面中出现。可能是 JavaScript文件不在页面中或者模块名错误
*   `Failed modules`: 监测到了javascript错误
*   `Rejected modules`: 模块返回失败的Promise。它（和它的依赖模块）没被加载。
*   `Rejected linked modules`: 依赖失败模块的模块
*   `Non loaded modules`: 依赖缺失和错误模块的模块

### 异步模块

模块可能会需要在准备就绪前执行一些任务。例如，它可以进行rpc请求加载数据。此时，模块可只返回一个promise。那样模块系统会在注册模块前等待promise完成。

```
odoo.define('module.Something', function (require) {
    "use strict";

    var ajax = require('web.ajax');

    return ajax.rpc(...).then(function (result) {
        // some code here
        return something;
    });
});
```

### 最佳实践

*   记住模块名的惯例：*addon名*后缀*模块名*。
*   在模块顶部声明所有依赖。同时，应该通过模块名按字母排序。这让掌握模块更加容易。
*   在最后声明所有导出的值
*   尽量避免从一个模块导出过多内容。通常在一个（小）模块中导出一项内容更好。
*   异步模块可用于简化一些用例。例如，*web.dom_ready* 模块在 dom 准备就绪时返回一个已成功的 promise。因而另一个需要该DOM的s 模块只需在某处添加*equire(‘web.dom_ready’)* 语句即可，该代码仅在DOM就绪时才会执行。
*   尽量避免在一个文件中定义多个模块。短期内可能更方便，但实际上更难以维护。

## 类系统

Odoo的开发出现 在 ECMAScript 6 类之前。在Ecmascript 5中，定义类的标准方式是定义一个函数并在原型对象中添加方法。这没有问题，但在我们想要使用实例和 mixins时就会有些复杂。

出于这些原因，Odoo受 John Resig启发决定使用自己的类系统。其类位于*web.Class中，在**class.js*文件中。

### 创建子类 

我们来讨论下如何创建类。使用的主要机制是*extend*方法 (这多多少少有些类似于ES6类中的*extend*)。

```
var Class = require('web.Class');

var Animal = Class.extend({
    init: function () {
        this.x = 0;
        this.hunger = 0;
    },
    move: function () {
        this.x = this.x + 1;
        this.hunger = this.hunger + 1;
    },
    eat: function () {
        this.hunger = 0;
    },
});
```

本例中，*init* 函数是构造函数。它会在创建实例时调用。通过使用*new* 关键字来产生实例。

### 继承

要能够继承已有类非常方便。只需通过在超类中使用*extend* 方法。在调用方法时，框架会秘密地重新绑定一个特殊方法: *_super* 到当前调用的方法。这允许我们在任何需要调用父方法时使用 *this._super*。

```
var Animal = require('web.Animal');

var Dog = Animal.extend({
    move: function () {
        this.bark();
        this._super.apply(this, arguments);
    },
    bark: function () {
        console.log('woof');
    },
});

var dog = new Dog();
dog.move()
```

### Mixin

odoo类系统不支持多继承， 但对那些我们需要分享一些行为的用例，有一个mixin系统：*extend* 方法实际上可接收任意数量的参数并会在新类中合并所有参数。

```
var Animal = require('web.Animal');
var DanceMixin = {
    dance: function () {
        console.log('dancing...');
    },
};

var Hamster = Animal.extend(DanceMixin, {
    sleep: function () {
        console.log('sleeping');
    },
});
```

本例中，*Hamster* 类是Animal的子类 ，但同时混杂了 DanceMixin。

### 对已有类打补丁

这并不普遍，但有时需要在原处修改另一个类。目标是有一个修改类及所有当前/未来实例的机制。这通过使用*include* 方法来实现：

```
var Hamster = require('web.Hamster');

Hamster.include({
    sleep: function () {
        this._super.apply(this, arguments);
        console.log('zzzz');
    },
});
```

很明显这是一个危险的操作，应当细心操作。但按照Odoo架构的方式，有时需要在一个插件中修改另一个插件中定义的控件/类的行为。注意它不会修改该类的所有实例，即便实例已创建。

## 控件

*Widget* 类真的是用户界面中的一个重要组成部分。几乎用户界面中的所有内容都是控件控制。Widget类在*widget.js*文件的*web.Widget*模块中定义。

简言之，由Widget类提供的功能包括：

*   控件间的父/子关系(*PropertiesMixin*)
*   具有安全特性的继承生命周期管理 (e.g. 在销毁父级时自动销毁子级)

*   自动通过[qweb](https://alanhou.org/odoo-13-qweb/#reference-qweb)渲染
*   有助与外部环境交互的各种工具函数。

以下是一个基本计数器控件的示例：

```
var Widget = require('web.Widget');

var Counter = Widget.extend({
    template: 'some.template',
    events: {
        'click button': '_onClick',
    },
    init: function (parent, value) {
        this._super(parent);
        this.count = value;
    },
    _onClick: function () {
        this.count++;
        this.$('.val').text(this.count);
    },
});

对于这个示例，假定模板*some.template* (并且相应地加载了：模板位于文件中，在模块的声明中相应地定认在 *qweb* 键中) 通过以下内容给定：

```
<div t-name="some.template">
    <span class="val"><t t-esc="widget.count"/></span>
    <button>Increment</button>
</div>

这个示例控件可以如下方式使用：

```
// Create the instance
var counter = new Counter(this, 4);
// Render and insert into DOM
counter.appendTo(".some-div");
```

这些示例描述了*Widget* 类的一些功能，包含事件系统、模板系统带有初始*parent*参数的构造函数。

### 控件的生命周期

类似很多控件系统，widget类有良好定义的生命周期。通常生命周期如下：调用 *init，*然后调用*willStart*，接着进行渲染，之后是*start* ，最终调用*destroy*。

###### `Widget.init(*parent*)`

这是构造方法。init方法用于初始化控件的基本状态。它是同步的并可通过控件创建者或父级接收参数进行重载。

参数

*   **parent** (`Widget()`) – 新控件的父级，用于处理自动析构和事件传输。可以为 `null` 来让控件无父级。

###### `Widget.willStart()`

这个方法将在创建控件时或在附加到 DOM 的过程中由框架进行一次调用。*willStart* 方法是应当返回promise的钩子。JS框架将等待promise在进入渲染步骤之前完成。注意此时，控件还没有DOM根元素。*willStart* 钩子多用于执行一些异步任务，如从服务端获取数据。

###### `[Rendering]()`

这个步骤由框架自动完成。具体为框架检查控件中模板的键是否定义。如果定义了，那么它将会通过在渲染上下文中绑定到*widget*键的模板(参见上面的示例：我们在QWeb模板中使用 *widget.count* 来从控件中读取值)。如未定义模板，我们读取*tagName* 键并创建相应的DOM元素。在渲染完成时，我们设置结果为控件的 $el属性。然后，乱动绑定events和custom_events键中的所有事件。

###### `Widget.start()`

在渲染完成时，框架会自动调用*start* 方法。这可用于执行一些专门的渲染后任务。例如，设置一个库。

必须返回一个promise来表明任务完成。

返回

promise

###### `Widget.destroy()`

这交流中心是控件生命的最后一部。在销毁控件时，基本上执行所有必要的清理操作：从控件树中删除控件、解绑事件…

在销毁控件父级时自动调用，如控件无父级或删除它但仍保留父级时必须显式地调用。

注意不一定要调用willStart 和 start方法。控件可以无需附加到 DOM 后就进行创建(将调用 *init* 方法) 然后进行销毁 (*destroy* 方法) 。如果是那样的y 话，都不会调用willStart 和 start。

### 控件API

###### `Widget.tagName`

如控件未定义模板时使用。默认为 `div`，将用作标签名来创建 DOM 元素设置控件的 DOM根。可以通过如下属性进一步的自定义所生成的DOM根：

###### `Widget.id`

用于在所生成的 DOM 根中生成一个`id` 属性。注意很少需要这么做，在控件可使用多次时这可能也不是一个好的想法。

###### `Widget.className`

用于在所生成的DOM根中生成`class` 属性。 注意它实际上可以包含多个css类：*‘some-class other-class’*

###### `Widget.attributes`

属性名对属性值的映射(对象字面量) 。每个k:v对将在所生成的 DOM 根中设置为一个DOM属性。

###### `Widget.el`

设置为控件根的原生DOM元素(仅在start生命周期方法之后可用)

###### `Widget.$el`

[`el`](#Widget.el "Widget.el")的jQuery封装 (仅在start生命周期方法之后可用)

###### `Widget.template`

应当设置为[QWeb模板](https://alanhou.org/odoo-13-qweb/#reference-qweb)的名称。若进行了设置，模板会在控件初始化后但在启动前进行渲染。模板所生成的根元素会设置为控件的DOM根。

###### `Widget.xmlDependencies`

需要在控件渲染前加载的xml文件的路径列表。这不会导致任何已加载的内容的加载。在想要懒加载模板或希望在网站和网站客户端界面之前分享控件时会用到。

```
var EditorMenuBar = Widget.extend({
    xmlDependencies: ['/web_editor/static/src/xml/editor.xml'],
    ...
```

###### `Widget.events`

events 是事件选择器 (由空格分隔的事件名称和可选CSS选择器) 对回调函数的映射.  回调函数可以是控件方法名或函数对象。在任一情况下， `this` 会设置到控件中：

```
events: {
    'click p.oe_some_class a': 'some_method',
    'change input': function (e) {
        e.stopPropagation();
    }
},
```

选择器用于jQuery的事件代理 ， 回调函数仅对 DOM 根匹配选择器的后代触发。如未提供选择器 (仅指定了事件名称)，事件会直接对控件的 DOM 根设置。

注意： 不鼓励使用行内函数，未来可能会移除这一支持。

###### `Widget.custom_events`

这和 *events* 属性几乎一样，但键可为任意字符串。它们代表一些由子控件触发的业务事件。在触发事件时，它会‘冒泡’控件树 (参见组件(component)通讯的部分获取更多详情)。

###### `Widget.isDestroyed()`

返回

在控件正在或已销毁时为`true` ，否则为`false`

###### `Widget.$(*selector*)`

应用指定给 DOM根作为参数的CSS选择器：

```this.$(selector);```

以上功能和下面的相同：

```this.$el.find(selector);```

参数

*   **选择器** (`String`) – CSS选择器

返回

jQuery对象

这个帮助方法类似于 `Backbone.View.<dt

###### `Widget.setElement(*element*)`

对所提供的元素重置控件的DOM根，同时处理 DOM 根的各种别名的重置以及取消设置和重置委托事件。

参数

*   **元素** (`Element`) – 设置为控件 DOM 根的DOM元素或jQuery对象

### 在DOM中插入控件

###### `Widget.appendTo(*element*)`

渲染控件并将其插入为目标的最后一个子元素，使用 [.appendTo()](https://api.jquery.com/appendTo/)

###### `Widget.prependTo(*element*)`

渲染控件并将其插入为目标的第一个子元素，使用[.prependTo()](https://api.jquery.com/prependTo/)

###### `Widget.insertAfter(*element*)`

渲染控件并将其插入为目标的前置兄弟元素，使用 [.insertAfter()](https://api.jquery.com/insertAfter/)

###### `Widget.insertBefore(*element*)`

渲染控件并将其插入为目标的后置兄弟元素，使用  [.insertBefore()](https://api.jquery.com/insertBefore/)

所有这些方法接收相应jQuery方法所接收的内容(CSS选择器、DOM节点或 jQuery对象)。它们者返回promise并带有3个任务：

*   通过以下渲染控件的根元素

    `renderElement()`

*   使用所匹配的任意jQuery方法在 DOM 中插入控件的根元素

*   启动控件并返回启动它的结果

### 控件指南

*   

    应避免标识符 (`id` 属性) 。在通常的应用和模块中，`id` 限制了控件的利用并会将代码打散。大部分时候，它们可通过空内容、类或保留对 DOM 节点的或jQuery的引用所替换。

    

    如`id`绝对必须 (因第三方的要求)，该 id应由 `_.uniqueId()` 进行部分的生成，如：

    ```this.id = _.uniqueId('my-widget-');```
    
    *   避免预测/通用CSS类名。“content” 或 “navigation”这样的类名可能与所需的含义/语法相匹配，但很可能其他开发者会有相同的需求，这会导致命名冲突及不想要的行为。通常类名应由它们所属的控件名称作为前缀 (创建 “非正式” 命名空间，类似C 或 Objective-C中那样)。
    *   应避免全局选择器。因为控件可能会在单个页面中多次使用 (Odoo中的仪表盘就是一个例子)，查询应限定为给定控件的作用域。未过滤r 的选择如 `$(selector)` 或 `document.querySelectorAll(selector)` 通常会带有预期外或错误的行为。Odoo Web的`Widget()` 有一个提供其DOM 根 ([`$el`](#Widget._S_el "Widget.$el"))的属性，以及对直接选定节点的快捷方式 ([`$()`](#Widget._S_ "Widget.$"))。
    *   更为常见的是，不要假定你的组件拥有或控制任何其自有 [`$el`](#Widget._S_el "Widget.$el")之外的内容 (因此避免使用对父级控件的引用)
    *   Html模板/渲染应使用 QWeb除非内容足够琐碎或微不足道。
    *   所有交互组件 (在屏幕显示信息或拦截DOM事件的组件) 必须继承自 `Widget()` 并正确地实现及使用其API和生命周期。
    *   在使用$el 时确保等待启动的完成，如：

 > ```var Widget = require('web.Widget');
 > var AlmostCorrectWidget = Widget.extend({
 >  start: function () {
 >      this.$el.hasClass(....) // 理论上，已设置了$el，但你不知道父级会使用它做什么，最好先调用 super
 >      return this._super.apply(arguments);
 >  },
 > });
 > 
 > var IncorrectWidget = Widget.extend({
 >  start: function () {
 >      this._super.apply(arguments); // 丢失了父级promise，没人会等待控件的开启
 >  },
 > });
 > 
 > var CorrectWidget = Widget.extend({
 >  start: function () {
 >      var self = this;
 >      return this._super.apply(arguments).then(function() {
 >          self.$el.hasClass(....) // 这会生效，promise都不会丢失并且按所控制的顺序执行代码：首先是super，然后是我们的代码。
 >      });
 >  },
 > });```

## QWeb模板引擎

网页客户端使用[QWeb](https://alanhou.org/odoo-13-qweb/)模板引擎来渲染控件 (除非它们重载*renderElement* 方法来执行其它操作)。Qweb JS模板引擎基于XML，并且大部分与 Python的实现相兼容。

下面我们来讲解模板是如何加载的。无论网页客户端在什么时候启动，会对*/web/webclient/qweb* 路由进行rpc调用。然后服务端会返回一个在针对 每个所安装模块的数据文件中定义的所有模板的列表。正确的让你说的在每个模块声明的 *qweb* 一条中列出。

网页客户端会在启动第一个控件时等待模板列表的加载。

这种机制对我们需求效果很好，但有时，我们需要对一个模板进行懒加载。例如，想象一下我们有一个很少使用的控件。在这种情况下，我们可能会倾向于不在主文件中加载模板，以让网页客户端更轻量。这时，可以在控件中使用*xmlDependencies* 键：

```
var Widget = require('web.Widget');

var Counter = Widget.extend({
    template: 'some.template',
    xmlDependencies: ['/myaddon/path/to/my/file.xml'],

    ...

});
```

通过它，*Counter* 控件会在其*willStart*方法中加载xmlDependencies文件，因此模板会在执行渲染时就绪。

## 事件系统

当前Odoo支持两种事件系统： 一个允许监听和触发事件的简单系统，一个让事件“冒泡”的更为完整的系统。

这两咱事件系统都在*mixins.js*.文件的*EventDispatcherMixin*中实现。这个 mixin在 *Widget* 类中包含。

### 基事件系统

这个事件系统在时间上最早。它实现了一个简单的总线模式。我们有4个主要的方法：

*   *on*: 这用于注册对事件的监听器。
*   *off*: 用于删除事件监听器。
*   *once*: 这用于注册仅调用一次的监听器。
*   *trigger*: 触发事件。这将导致调用每个监听器。

下例是使用这一事件系统的场景：

```
var Widget = require('web.Widget');
var Counter = require('myModule.Counter');

var MyWidget = Widget.extend({
    start: function () {
        this.counter = new Counter(this);
        this.counter.on('valuechange', this, this._onValueChange);
        var def = this.counter.appendTo(this.$el);
        return Promise.all([def, this._super.apply(this, arguments)]);
    },
    _onValueChange: function (val) {
        // do something with val
    },
});

// 在Counter控件中，我们需要调用这个trigger方法：

... this.trigger('valuechange', someValue);```
```

### ⚠️警告

不鼓励使用这一事件系统，我们计划通过继承的事件系统使用*trigger_up*方法来替换每个*trigger* 方法

### 继承事件系统

自定义事件控件是更高级的系统，它模仿了DOM事件API。在触发事件时，它会在组件树中“冒泡”，直至抵达了根控件或是停止了。

*   *trigger_up*: 这个方法创建小型的*OdooEvent* 并在组件树中调度它。注意它将伴随所触发事件的组件启动。
*   *custom_events*: 这等价于*event* 字段，但仅针对odoo事件。

OdooEvent类非常简单。它有三个公共属性：*target* (触发事件的控件), *name* (事件名) 和 *data* (负载)。它还有两个方法：*stopPropagation* 和 *is_stopped*。

可更新前例来使用自定义事件系统：

```
var Widget = require('web.Widget');
var Counter = require('myModule.Counter');

var MyWidget = Widget.extend({
    custom_events: {
        valuechange: '_onValueChange'
    },
    start: function () {
        this.counter = new Counter(this);
        var def = this.counter.appendTo(this.$el);
        return Promise.all([def, this._super.apply(this, arguments)]);
    },
    _onValueChange: function(event) {
        // 通过event.data.val执行一些操作
    },
});

// 在Counter控件中，我们需要调用这个trigger_up方法：

... this.trigger_up('valuechange', {value: someValue});
```

## 注册表

Odoo生态中一个常用的需求是借由外部（通过安装应用，如不同的模块）扩展/改变行为。例如，人们可能需要在一些视图中添加一个新的控件。这种情况以及很多其它情况下，通常的处理是创建所需要的控件，然后将其添加到注册表中（注册步骤），来让其它的网页客户端知道其存在。

在系统中存在一些注册表：

*   字段注册表 (通过‘web.field_registry’导出)。字段注册表包含

    所有对网页客户端可见的字段控件。在视图(典型的有表单或列表/看板视图) 需要字段控件时，这就是查看的地方。常见的用例如下：

```
    var fieldRegistry = require('web.field_registry');

    var FieldPad = ...;

    fieldRegistry.add('pad', FieldPad);
```

注意每个值应为*AbstractField*的子类

*   视图注册表：这个注册表包含所有对网页客户端可见的 JS视图(尤其是视图管理器)。每个注册表的值应为*AbstractView*的子类

*   动作注册表：我们需要记录注册表中的所有客户端动作。这是在动作管理器需要创建客户端动作时所要查找的地方。在版本 11中，每个值应仅为*Widget*的子类。但在版本12中，该值要求为*AbstractAction*。

## 组件间通讯

有很多种在控件间通讯的方式：

*   从父级到子级：

    这是一种简单用例。父级控件只需要调用其子级的方法：

    ```this.someWidget.update(someInfo);```

    *   从控件到其父级/某祖先级：
    
        在这种用例中，控件的任务只是通知环境发生了操作。因为我们不想要控件引用其父级 (这会让控件与其父级的实现相耦合)，通常最好的处理方式是触发事件，它会通过使用`trigger_up`方法在组件树中冒泡：
    
        ```this.trigger_up('open_record', { record: record, id: id});```
    
        这一事件会对控件触发，然后冒泡并最终由上游控件捕获：

```
    var SomeAncestor = Widget.extend({
        custom_events: {
            'open_record': '_onOpenRecord',
        },
        _onOpenRecord: function (event) {
            var record = event.data.record;
            var id = event.data.id;
            // do something with the event.
        },
    });

*   交叉组件：

    交叉组件通讯可使用总线实现。这不是一种推荐的通讯形式，因为它存在让代码更难于维护的劣势。但是它存在让组件解耦的优势。在这种情况下，只需通过在总线上触发和监听事件来实现。例如：

```
    // in WidgetA
    var core = require('web.core');

    var WidgetA = Widget.extend({
        ...
        start: function () {
            core.bus.on('barcode_scanned', this, this._onBarcodeScanned);
        },
    });

    // in WidgetB
    var WidgetB = Widget.extend({
        ...
        someFunction: function (barcode) {
            core.bus.trigger('barcode_scanned', barcode);
        },
    });
```

  In this example,在这个示例中，我们使用*web.core*所导出的总线，但并非必须。总线可针对 一个具体的目的进行创建。

## 服务

在版本11.0中，我们引入了*service*的概念。主要的想法是给子组件一种访问环境的控制方法，这种方法授予框架足够的控制权，并且可测试。

服务系统按照3种概念进行组织：服务、服务提供商和控件。它起作用的方式是控件（通过*trigger_up*）触发事件，这些事件冒泡至服务提供商，又让服务执行任务，然后可能会返回回复。

### 服务

服务是*AbstractService* 类的一个实例。基本上它只有一个名称和几个方法。其任务是执行一些操作，通常会依赖于环境。

例如，我们有一个*ajax* 服务(任务是执行rpc)、 *localStorage* (与浏览器本地存储交互) 及其它内容。

下面是简化了ajax服务如何实现的示例：

```
var AbstractService = require('web.AbstractService');

var AjaxService = AbstractService.extend({
    name: 'ajax',
    rpc: function (...) {
        return ...;
    },
});
```

该服务名称为‘ajax’并定义了一个方法 *rpc*。

### 服务提供者

要让服务生效， 有必要让服务提供者准备好分发自定义事件。在后台 (网页客户端)中， 这通过主客户端实例来实现。注意服务提供者的代码来自*ServiceProviderMixin*。

### 控件

控件是请求服务的部分。为实现这点，它只是触发了一个 *call_service* 事件(通常使用帮助函数*call*)。这个事件会冒泡并将意图与系统其它部分进行通讯。

实际上一些函数调用很频繁，因此我们有一些帮助函数来让其使用更为简单。例如， *_rpc* 方法是帮助进行rpc调用的帮助方法。

```
var SomeWidget = Widget.extend({
    _getActivityModelViewID: function (model) {
        return this._rpc({
            model: model,
            method: 'get_activity_view_id'
        });
    },
});
```

### ⚠️警告

如果销毁了这一控件，会从让控件树中脱离并不会有父级。这种情况下，事件不会冒泡，也即任务不会完成。这通常正是我们希望通过销毁控件所获取的结果。

### RPC

功能ajax 服务提供。但大部分人可能仅与 *_rpc* 帮助方法进行交互。

在操作Odoo时通常有两种用例：一种可能需要对（python）模型调用方法 (这通过控制器 *call_kw*)，或者是可以直接调用控制器 (在某些路由中可用)。

* 对python模型调用方法：

  ```
  return this._rpc({
      model: 'some.model',
      method: 'some_method',
      args: [some, args],
  });

*   直接调用控制器：

```
return this._rpc({
    route: '/some/route/',
    params: { some: kwargs},
});
```

## 通知

Odoo框架有一种与用户进行不同信息通讯的标准方式：通知，显示在用户界面的右上角。

有两种类型的通知：

*   *通知*: 用于显示一些反馈。例如，在用户退订频道时。
*   *警告：*用于显示重要/紧急信息。通常为系统中大部分（可还原）错误。

同时，通知还可用于在不影响工作流的情况下向用户询问问题。设想一下通过VOIP接收电话：会显示带有两个按钮的悬停通知： *接听* 和 *拒绝*。

### 通知系统

Odoo中的通知系统设计带有如下组件：

*   *Notification* 控件：这是一个用于创建和显示带有所需信息的简单控件
*   *NotificationService*： 该服务的职责是在完成请求时（通过custom_event）创建和销毁通知。注意网页客户端是一个服务提供者。
*   客户端动作*display_notification*：允许通过python触发通知的显示(如在用户点击一个对象类型的按钮时调用的方法)。
*   *ServiceMixin*中的两个帮助函数： *do_notify* 和 *do_warn*

### 显示通知

显示通知最常用的方式是使用通过来自*ServiceMixin*的两个方法：

*   *do_notify(title, message, sticky, className)*:

    显示*notification*类型的通知。

    *   *title*: 字符串。将在顶部作为标题显示
*   *message*: 字符串，通知的内容
    *   *sticky*: 布尔值，可选。若为true，通知会保持到用户取消它为止。否则，通知会在一个短暂的间隔后自动关闭。
*   *className*: 字符串，可选。这是一个会自动添加到通知中的css类名。对于操作样式会很有用，但不太推荐使用。

*   *do_warn(title, message, sticky, className)*:

    

    e显示*warning*类型的通知。

    *   *title*: 字符吕。将在顶部作为标题显示
    *   *message*: 字符串，通知的内容
    *   *sticky*: 布尔值，可选。若为true，通知会保持到用户取消它为止。否则，通知会在一个短暂的间隔后自动关闭。
    *   *className*: 字符串，可选。这是一个会自动添加到通知中的css类名。对于操作样式会很有用，但不太推荐使用。

以下是两个如何使用这些方法的示例：

```
// 注意我们对文本调用了_t 来确保进行相应的翻译。
this.do_notify(_t("Success"), _t("Your signature request has been sent."));

this.do_warn(_t("Error"), _t("Filter name is required."));
```

下面是python中的一个示例：

```
# 注意我们对文本调用了_(string)来确保进行相应的翻译。
def show_notification(self):
    return {
        'type': 'ir.actions.client',
        'tag': 'display_notification',
        'params': {
            'title': _('Success'),
            'message': _('Your signature request has been sent.'),
            'sticky': False,
        }
    }
```

## Systray

TSystray是界面中菜单栏的右边部分，这里网页客户端显示一些控件，如消息菜单。

在通过菜单创建 SystrayMenu 是，它会查找所有已注册控件并在相应的位置将它们添加为子控件。

当前针对systray控件没有特别的API。它们应为简单的控件，并可以像其它控件与*trigger_up*方法通讯那样与自身的环境通讯。

### 新增Systray项

不存在 systray注册表。 添加控件正确的方式是将其添加到类变量 SystrayMenu.items中。

```
var SystrayMenu = require('web.SystrayMenu');

var MySystrayWidget = Widget.extend({
    ...
});

SystrayMenu.Items.push(MySystrayWidget);
```

### 排序

在向自身添加控件之前，Systray菜单会按照属性对各项进行排序。如果在原型中没有该属性，会使用 50来替代。因此，如需将systray项放到右侧，可以设置一个很高的序号 (相反，设置一个小序号来居左显示)。

```MySystrayWidget.prototype.sequence = 100;```

## 翻译管理

一些翻译在服务端完成 (基本所有的文本字符串都由服务端处理和完成)，但在静态文件中存在需要翻译的字符串。当前运作的方式如下：

*   每个可翻译字符串通过特殊函数*_t*进行标记 (函数位于JS 模块*web.core中）*
*   这些字符串由服务端用于生成相应的PO文件
*   在加载网页客户端时，它会调用路由*/web/webclient/translations* ，返回所有翻译词汇的列表
*   在运行时， 无论何时调用*_t* 函数，会在这个列表中查找翻译并返回，如无翻译则返回原字符串。

在[翻译模块](https://alanhou.org/odoo-13-translating-modules/)文档中从服务端角度对翻译进行了更详细的讲解。

在javascript中针对翻译有两个重要函数： *_t* 和 *_lt。*区别是 *_lt* 进行了懒运行。

```
var core = require('web.core');

var _t = core._t;
var _lt = core._lt;

var SomeWidget = Widget.extend({
    exampleString: _lt('this should be translated'),
    ...
    someMethod: function () {
        var str = _t('some text');
        ...
    },
});
```

在上例中，the *_lt* 的必要性在于模块加载是翻译还未准备就绪。

注意翻译函数需要一些关注。参数中给定的字符串不应是动态的。

## 会话

网页客户端提供了一个包含有关用户当前会话的信息的特定模块。要点有：

*   uid: 当前用户ID (其作为 *res.users*的ID)
*   user_name: 用户名，为字符串
*   用户上下文 (用户 ID, 语言和时区)
*   partner_id: 与当前用户相关联的伙伴ID
*   db: 当前使用的数据库名

### 向会话添加信息

在加载/web路由时，服务端会在模板的script标签中注册一些会话信息。信息会通过*ir.http*模型的*session_info* 方法中读取。因此如果想要添加具体的信息，可通过重载session_info方法并将其添加到字典中来实现。

```
from odoo.http import request

class IrHttp(models.AbstractModel):
    _inherit = 'ir.http'

    def session_info(self):
        result = super(IrHttp, self).session_info()
        result['some_key'] = get_some_value_from_db()
        return result
```

现在，可通过javascript在会话中读取来获取值：

```
var session = require('web.session');
var myValue = session.some_key;
...
```

注意这一设计机制用于减少网页客户端在就绪前所需进行的通讯。更适合运算开销较少的数据 (慢速的session_info 调用会让每个人的网页客户端产生延时)，以及在初始化进程中需较早使用的数据。

## 视图

‘视图’一词的含义有好几种。这部分有关视图的javascript代码设计，而不是*arch*或其它内容的结构。

2017年， Odoo使用新的架构替换了之前的视图代码。主要的需求是将渲染逻辑从模型逻辑中分离出来。

（通常意义上的）视图通过4块进行描述：View、Controller、Renderer和Model。这4块的API在AbstractView、AbstractController、AbstractRenderer和AbstractModel类中进行描述。

ViewControllerRendererModel

*   View为工厂模式。其任务是获取一级字段、结构、上下文及一些其它参数，然后构造一个Controller/Renderer/Model三板斧。视图的角色是推荐正确的信息相应地设置MVC模式的每个部分。 通常，它需要处理 arch字符串并提取视图其它每个部分所需要的数据。注意视图j 是一个类，而非控件。一旦任务完成即可丢弃。
*   Renderer是一个job: 表示数据在DOM元素中被查看了。每个视图可以不同的方式渲染数据。同时，它应当监听相应的用户动作并在必要是通知其父级(Controller)。渲染器是MVC模式中的V。
*   Model: 其任务是获取并保留视图的状态。通常，它以某种方式代表数据库中的一组记录。Model是‘业务数据’的所有者。模型是MVC模式中的M。
*   Controller: 其任务是协调渲染器和模型。同时它是其它网页客户端的主入口。例如，当用户在搜索视图中修改内容时，控制器的*update*方法会使用相应的信息进行调用。控制器是MVC模式中的C。

视图的 JS代码设计用于在视图管理器/动作管理器的外部使用。它们可用在客户端动作中或，在对公网站中显示（对资源进行一些操作）。

## 字段控件

网页客户端体验很重要的一部分是关于编辑和创建数据的。大部分工作借助于字段控件完成，它知悉字段类型及应如何显示和编辑值的详情。

### AbstractField

*AbstractField* 类是针对所有支持它们的视图中所有控件的基类 (当前支持的有表单、列表、看板视图)。

在v11和之前版本的字段控件之间存在着很多差别。我们来讲一下最重要的那些：

*   控件在所有视图之间共享(即表单/列表/看板视图)。无需再复制其实现。注意可以通过在视图注册表中在视图名前加前缀对视图有一个特殊的控件版本：*list.many2one* 的优先级会高于*many2one*。
*   控件不再是字段值的所有者。它们只代表数据并与其它视图进行通讯。
*   控件不再需要能够在编辑和只读模式之间切换。目前在需要进行这一切换时，会销毁控件并再次进行渲染。这不是问题，因为反正它们也不再拥有值了
*   字段控件可在视图外部使用。它们的 API有些许尴尬，但它们就是设计用来独自使用的。

### 装饰

类似列表视图，字段控件对装饰有着简单的支持。装饰的目的在于根据记录当前状态拥有指定文本颜色的简单方式。例如，

```<field name="state" decoration-danger="amount &lt; 10000"/>```

有效的装饰名称有：

*   decoration-bf
*   decoration-it
*   decoration-danger
*   decoration-info
*   decoration-muted
*   decoration-primary
*   decoration-success
*   decoration-warning

每个装饰 *decoration-X* 会映射到一个css类 *text-X*, 这是标准的bootstrap css 类 (除*text-it* 和 *text-bf*外，由odoo进行处理并分别对应斜体和粗体)。注意装饰属性的值应为有效的python表达式，通过记录作为上下文来运行。

### 非关联字段

下面我们记录了默认在非关系记录中可用的文档，排名不分先后。

*   整型 (FieldInteger)

    

    这是针对 *integer*类型字段的默认字段类型

    *   支持字段类型：*integer*

    选项：

    *   type: 设置输入类型 (默认为*text* ，可对 *number*进行设置)

    在编辑模式下，字段渲染为设置于*number*之上带有HTML属性类型的输入(因此用户可受益于原生支持，尤其是在移动端)。在这种情况下，它的默认格式为避免不兼容性而做了禁用。

```<field name="int_value" options='{"type": "number"}'/>```

 *   

        step: 设置在用户点击按钮时增加及减小的步长

        (仅针对number类型的输入，默认为1)

        

    ```<field name="int_value" options='{"type": "number", "step": 100}'/>```

    

    

*   

    浮点型 (FieldFloat)

    

    这是针对*float*类型字段的默认字段类型。

    *   支持的字段类型：*float*

    属性：

    *   digits: 显示的精度

    ```<field name="factor" digits="[42,5]"/>```

    选项：

    *   type: 设置输入类型 (默认为*text*，可对 *number*进行设置)

    在编辑模式下，字段渲染为设置于*number*之上带有HTML属性类型的输入(因此用户可受益于原生支持，尤其是在移动端)。在这种情况下，它的默认格式为避免不兼容性而做了禁用：

    ```<field name="int_value" options='{"type": "number"}'/>```

    *   step: 设置在用户点击按钮时增加及减小的步长

        (仅针对number类型的输入，默认为1)

    
    ```<field name="int_value" options='{"type": "number", "step": 0.1}'/>```
    
*   float_time (FieldFloatTime)

    

    这个控件的目的是显示相应的表示时间间隔（按小时）的浮点值。例如0.5应当格式化为0:30，或是4.75对应 4:45。

    *   支持的字段类型：*float*

    * float_factor (FieldFloatFactor)
        这一控件旨在显示相应的按选项中给定的因子转化后的浮点值。例如，数据库中保存的值是0.5且因子为3，控件值就应格式化为1.5。
    
        *   支持的字段类型：*float*
    
    * float_toggle (FieldFloatToggle)
        这一控件的目的是通过包含（选项中给定的）可能值范围的控制来替换输入字段。每次单点允许用户在范围内进行循环。这里的目的是限制预定义选项的字段值。同时，该控件支持因子转化为 *float_factor* 控件 (范围值应为转化的结果)。
    
        *   所支持的字段类型：*float*
    
        ```<field name="days_to_close" widget="float_toggle" options='{"factor": 2, "range": [0, 4, 8]}'/>```
    
    * 布尔型 (FieldBoolean)
        这是针对*boolean*类型字段的默认字段类型。
        *   所支持的字段类型：*boolean*
        
    * 字符型 (FieldChar)
        这是针对*char*类型字段的默认字段类型。
        *  所支持的字段类型：*char*
        
    *  日期型 (FieldDate)
        这是针对*date*类型的默认字段类型。注意它对datetime 字段也同样有效。在格式化日期时使用会话的时区。
    
        *   所支持的字段类型：*date*, *datetime*
    
        选项：
    
        *   datepicker: 针对[datepicker](https://github.com/Eonasdan/bootstrap-datetimepicker) 控件的额外设置。
        
        ```
            <field name="datefield" options='{"datepicker": {"daysOfWeekDisabled": [0, 6]}}'/>
        ```
        
        *  日期时间型 (FieldDateTime)
        
            这是针对*datetime*类型的默认字段类型。
        
            *   所支持字段类型：*date*, *datetime*
        
            选项：
        
            *   datepicker: 针对 [datepicker](https://github.com/Eonasdan/bootstrap-datetimepicker) 控件的额外设置。

```
    <field name="datetimefield" options='{"datepicker": {"daysOfWeekDisabled": [0, 6]}}'/>

*  daterange (FieldDateRange)

    这个控件让用户可以在单个拾取器中选择开始和结束日期。

    *   所支持的字段类型： *date*, *datetime*

    选项：

    *   related_start_date: 应用于结束日期字段用来获取用于在拾取器中显示范围的开始日期值。
    *   related_end_date: 应用于开始日期字段用来获取用于在拾取器中显示范围的结束日期值。
    *   picker_options: 拾取器的额外设置。

    ```<field name="start_date" widget="daterange" options='{"related_end_date": "end_date"}'/>```

*   

    monetary (FieldMonetary)

    

    这是针对‘monetary’类型的默认字段类型。 它用于显示货币。如果在选项中给定了货币字段，就会使用它，否则会使用（会话中的）默认货币

    *   所支持的字段类型：*monetary*, *float*

    选项：

    *   currency_field: 另一个字段名应为针对倾向的 many2one。

    ```<field name="value" widget="monetary" options="{'currency_field': 'currency_id'}"/>```

* text (FieldText)
    这是针对*text*类型的默认字段类型。

    *   所支持的字段类型：*text*

* handle (HandleWidget)
    这一字段的任务是显示为一个句柄，并允许通过拖拽来重新排序各条记录。

    ### ⚠️警告

    必须对记录所排序的字段进行指定。

    ### ⚠️警告

    不支持多个对同一列表带有句柄控件的字段。

    *   所支持的字段类型： *integer*

*   email (FieldEmail)

    这一字段显示email地址。使用它的主要原因是它以只读模式渲染为一个带有相应href的锚标签。

    *   所支持的字段类型：*char*

*   phone (FieldPhone)

    这个字段显示电话号码。使用它的主要原因是它仅在某些情况下以只读模式渲染为一个带有相应href的锚标签：我们仅在设备能够调用具体的号码时才让其可点击。

    *   所支持字段类型：*char*

*   

    url (UrlWidget)

    （以只读模式）显示一段url的字段。使用它的主要原因是它通过相应的css类和href渲染为一个锚标签。

    *   所支持的字段类型： *char*

    同时，锚标签的文本可通过*text* 属性进行自定义 (它不会改变 href值)。

    ```<field name="foo" widget="url" text="Some URL"/>```

*  domain (FieldDomain)
    借助于树状接口 “Domain”字段让用记可以构那家一个技术前缀作用域并实时查看所选记录。在调试模式下，输入中也可直接输入前缀字符域（或构建树状接口所不允许的高级作用域）。

    注意这仅限于‘static’作用域 (无动态表达式或对上下文变量的访问)。

    *   所支持的字段类型： *char*

*  link_button (LinkButton)
    LinkButton控件实际上只是显示了一个带有图标和文本值作为内容的 span。链接可进行点击并以其值作为url打开一个新的浏览器窗口。

    *   所支持的字段类型：*char*

*  image (FieldBinaryImage)

    该控件用于表示一个作为图片的二进制值。在一些情况下，服务端返回一个‘bin_size’而非真实的图片(bin_size是表示文件大小的字符串，如6.5kb)。这时，控件会通过服务端对应图片的源属性生成图片。

    *   所支持的字段类型： *binary*

    选项：

    *   preview_image: 若图片仅加载为‘bin_size’，那么这个选项用于告知网页客户端默认字段名不是当前字段名，而是其它的字段名。

    ```<field name="image" widget='image' options='{"preview_image":"image_128"}'/>```

*   binary (FieldBinaryFile)
    允许保存/下载二进制文件的通用控件。

    *   所支持的字段类型： *binary*
    属性：
    *   filename: 保存二进制文件会丢失文件名，因为它仅保存二进制值。文件名可保存在另一个字段中。要进行实现，属性文件名应设置为视图中的字段。

    ```<field name="datas" filename="datas_fname"/>```

*  priority (PriorityWidget)
    该控件渲染为一组星标，允许用户进行点击来选取或不选取一个值。例如可用于标记一个任务为高优先级。

    注意这个控件也可在‘只读’模式下使用，但不常见。

    *   所支持字段类型：*selection*


*  attachment_image (AttachmentImage)
    many2one字段的图像控件。若设置了该字段，这个控件会通过相应的src url渲染为一张图片。该控件在编辑或只读模式下的行为并无不同，仅用于浏览图片。

    *   所支持的字段类型：*many2one*

    ```<field name="displayed_image_id" widget="attachment_image"/>```

*  image_selection (ImageSelection)
    允许用户通过点击图像来选取值。
    *   所支持的字段类型：*selection*
    选项: 所选值对带图片url（*image_link*）及预览图片（*preview_link*）的对象的映射。
    注意这不是个可选的选项！

```
    <field name="external_report_layout" widget="image_selection" options="{
        'background': {
            'image_link': '/base/static/img/preview_background.png',
            'preview_link': '/base/static/pdf/preview_background.pdf'
        },
        'standard': {
            'image_link': '/base/static/img/preview_standard.png',
            'preview_link': '/base/static/pdf/preview_standard.pdf'
        }
    }"/>
```

*  label_selection (LabelSelection)
    该控件渲染一个简单的不可编辑标签。仅用于显示一些信息，无法编辑。

    *   所支持的字段类型：*selection*

    选项：

    *   classes: 所选值对css类的映射

    ```
    <field name="state" widget="label_selection" options="{
        'classes': {'draft': 'default', 'cancel': 'default', 'none': 'danger'}
    }"/>
    ```

*  state_selection (StateSelectionWidget)
    这是一个特殊的选择控件。它假定在视图中有一些硬编码的字段：*stage_id*, *legend_normal*, *legend_blocked*, *legend_done*。多用于显示及修改项目中任务的状态，在下拉中显示额外的信息。
    *   所支持的字段类型： *selection*
    ```<field name="kanban_state" widget="state_selection"/>```
*  kanban_state_selection (StateSelectionWidget)
    与state_selection是完全一样的控件
    *   所支持字段类型：*selection*
*  boolean_favorite (FavoriteWidget)
    该控件根据布尔值显示为空（或非空）星标。注意它也可在只读模式下编辑。
    *   所支持的字段类型：*boolean*
*  boolean_button (FieldBooleanButton)
    布尔型按钮用于表单视图中的stat按钮。目标是显示带有布尔字段当前状态的一个美观的按钮 (例如‘Active’)，并允许用户在点击它时修改该字段。
    注意它可在只读模式下进行编辑。
    *   所支持的字段类型： *boolean*
    选项：
    *   terminology: 可为‘active’、‘archive’、‘close’或带有*string_true*, *string_false*, *hover_true*, *hover_false*键的自定义映射
    ```
    <field name="active" widget="boolean_button" options='{"terminology": "archive"}'/>
    ```

* boolean_toggle (BooleanToggle)

    显示一个表示布尔值的切换器。它是FieldBoolean的子字段，多用于不同的外观展示。
*   statinfo (StatInfo)
    该控件用于在*stat* 按钮中表示统计信息。它基本上只是一个带有数字的标签。
    *   所支持的字段类型：*integer, float*
    选项：
    *   label_field: 若给定，该控件会使用label_field的值作为文本。
    ```
    <button name="%(act_payslip_lines)d"
        icon="fa-money"
        type="action">
        <field name="payslip_count" widget="statinfo"
            string="Payslip"
            options="{'label_field': 'label_tasks'}"/>
    </button>
    ```

*  percentpie (FieldPercentPie)
    该控件用于在*stat*按钮中展示统计信息。它类似于一个 statinfo 控件，但信息在饼图（由空到完整）中进行表示。注意该值解释为百分值 (一个从0到100的数值)。
    *   所支持的字段类型： *integer, float*
    ```<field name="replied_ratio" string="Replied" widget="percentpie"/>```

* progressbar (FieldProgressBar)
    表示一个进度条的值(从0到某个值)
    *   所支持的字段类型： *integer, float*
    选项：
    *   editable: 值是否可编辑的布尔值
    *   current_value: 从字段中获取的必须在视图中展示的current_value
    *   max_value: 从字段中获取的必须在视图中展示的max_value
    *   edit_max_value: max_value是否可编辑的布尔值
    *   title: 进度条的标题，在顶部显示r –> 不进行翻译要使用参数 (非选项) “title” 来代替

```
    <field name="absence_of_today" widget="progressbar"
        options="{'current_value': 'absence_of_today', 'max_value': 'total_employee', 'editable': false}"/>
```

*   toggle_button (FieldToggleBoolean)

    这个控件用于布尔字段。它将按钮在绿色图标/灰色图标之间进行切换。它还根据值和一些选项设置提示消息。

    *   所支持自动类型：*boolean*

    选项：

    *   active:在布尔值为true时应设为提示信息的字符串
*   inactive: 在布尔值为false时应设为提示信息的字符串
    
```
    <field name="payslip_status" widget="toggle_button" options='{"active": "Reported in last payslips", "inactive": "To Report in Payslip"}' />
    ```

*   dashboard_graph (JournalDashboardGraph)

    

    这是更为专业的控件，用于显示表示数据集的图表。例如，它用在会计仪表盘看板视图中。

    它假定该字段是数据集的JSON序列化。

    *   所支持的字段类型：*char*

    属性

    *   graph_type: 字符串，可为‘line’ 或‘bar’

    ```
    <field name="dashboard_graph_data" widget="dashboard_graph" graph_type="line"/>
    ```
    
*   ace (AceEditor)

    该控件用于Text字段。它提供了一个Ace编辑器来编辑XML及Python。

    *   所支持的字段类型：*char, text*


### 关联字段

###### `*class *web.relational_fields.FieldSelection()`

所支持的字段类型：*selection*

###### `web.relational_fields.FieldSelection.placeholder`

在未选中值时用于显示信息的字符串

```<field name="tax_id" widget="selection" placeholder="Select a tax"/>```

*   radio (FieldRadio)

    这是FielSelection的子字段，但专门用于显示所有有效选项为单选按钮。

    注意若用于many2one记录时，那么会进行更多的rpc调用来获取相关记录的 name_get。

    *   所支持的字段类型：*selection, many2one*

    选项：

    *   horizontal: 若为true, 单选按钮会进行横向显示。

    ```
<field name="recommended_activity_type_id" widget="radio" options="{'horizontal':true}"/>
    ```

*   selection_badge (FieldSelectionBadge)

    这是FieldSelection的子字段，但专门用于显示所有有效选项为矩形徽章。

    *   所支持的字段类型：*selection, many2one*

    ```<field name="recommended_activity_type_id" widget="selection_badge"/>```

*   many2one (FieldMany2One)

    针对many2one 字段的默认控件。

    *   所支持的字段类型：*many2one*

    属性：

    *   can_create: 允许关联记录的创建(优先级高于no_create选项)
*   can_write: 允许对关联记录的编辑(默认值：true)
    
选项：
    
    *   no_create：防止关联记录的创建
*   quick_create：允许关联记录的快速创建(默认值：true)
    *   no_quick_create：防止关联记录的快速创建 (不要询问)
*   no_create_edit：可能与 no_create相同
    *   create_name_field：在创建关联记录时，若设置了该选项， *create_name_field* 的值会填充为输入值(默认值：*name*)
    *   always_reload: 布尔值，默认为false。若为true，该控件会一直进行额外的name_get 来获取其name值。它用在重载了name_get方法的场景 (请不要重载）
    *   no_open：布尔值，默认为false。若设置为true，（只读模式下）点击它时many2one 不会重定向到记录
    
    ```
    <field name="currency_id" options="{'no_create': True, 'no_open': True}"/>
    ```

*   list.many2one (ListFieldMany2One)

     （列表视图中）many2one字段的默认控件。

    专用于列表视图的many2one字段。主要原因是我们需要将（只读模式下的）many2one字段渲染为文本， 不允许打开关联记录。

    *   所支持的字段类型：*many2one*

*   many2one_barcode (FieldMany2OneBarcode)

    这一many2one字段的控件允许打开移动设备(Android/iOS)的摄像头扫描条形码。

    专门用于允许用户使用原生摄像头扫描二维码的many2one字段。然后它使用 name_search来搜索其值。

    如果设置了这一控件且用记没有使用移动应用，会回归到常规的many2one (FieldMany2One)

    *   所支持的字段类型： *many2one*

*   kanban.many2one (KanbanFieldMany2One)

    （看板视图中）many2one字段的默认控件。我们需要禁用掉看板视图中的所有版本。

    *   所支持的字段类型：*many2one*

*   many2many (FieldMany2Many)

    针对many2many字段的默认控件。

    *   所支持的字段类型：*many2many*

    属性：

    *   mode: 字符串，默认显示的视图
*   domain: 限定数据为指定作用域
    
选项：
    
    *   create_text: 允许在新增记录时显示文本的自定义

*   many2many_binary (FieldMany2ManyBinaryMultiFiles)

    这一控件有助于用户同时上传或删除一个或多个文件。

    注意该控件专门针对 ‘ir.attachment’模型。

    *   所支持的字段类型：*many2many*

*   many2many_tags (FieldMany2ManyTags)

    将many2many显示为一个标签列表。

    *   所支持的字段类型：*many2many*

    选项：

    *   color_field: 数值字段的名称，应当出现在视图中。会根据其值选择颜色。

    ```<field name="category_id" widget="many2many_tags" options="{'color_field': 'color'}"/>```

    *   no_edit_color: 设置为True来移除修改标签颜色的可能性(默认值： False).

    ```<field name="category_id" widget="many2many_tags" options="{'color_field': 'color', 'no_edit_color': True}"/>```

*   form.many2many_tags (FormFieldMany2ManyTags)

    专用于表单视图的many2many_tags控件。它有一些允许编辑标签颜色的额外代码。

    *   所支持的字段类型： *many2many*

*   kanban.many2many_tags (KanbanFieldMany2ManyTags)

    专门用于看板视图的many2many_tags控件。

    *   所支持的字段类型：*many2many*

*   many2many_checkboxes (FieldMany2ManyCheckBoxes)

    该字段显示一个复选框列表并允许用户选择一个选项的子集。

    *   所支持的字段类型： *many2many*

*   one2many (FieldOne2Many)

    one2many字段的默认控件。

    I它通常在子列表视图或子看板视图中显示数据。

    *   所支持的字段类型：*one2many*

    选项：

    *   create_text: 用于自定义‘Add’标签/文本的字符串。

    ```<field name="turtles" options="{\'create_text\': \'Add turtle\'}">```

*   statusbar (FieldStatus)

    它是专门用于表单视图的控件。是很多表示工作流的表单的顶栏，并允许选择具体的状态。

    *   所支持的字段类型：*selection, many2one*

*   reference (FieldReference)

    FieldReference是一个（模型）选择和FieldMany2One (值)的组合。允许选择一个任意模型的记录。

    *   所支持的字段类型：*char, reference*


## 客户端动作

客户端动作是一个集成于网页客户端界面中的自定义控件，就像 *act_window_action*。在需要与已有视图或具体模型密切连接的构建会很有用。例如，Discuss应用实际上是一个客户端动作。

根据上下文客户端动作一词可有多种含义：

*   从服务端的角度，它是一个带有char类型的*tag*字段的*ir_action*模型记录
*   从网页客户端的角度，它是一个继承自AbstractAction类的控件，并应在（从字段char）相应键下的动作注册表中注册

在菜单项与客户端动作相关联时，打开它会从服务端获取动作定义，然后查看其动作注册表来以相应的键获取Widget定义，最终，它会初始化并将控件添加到DOM中的相应位置。

### 添加客户端动作

客户端动作是一个控制菜单栏下方屏幕部分的控件。 需要时可以有控制面板。定义客户端动作可通过两步完成：实现一个新控件及在动作注册表中注册该控件。

*   实现新客户端动作：

    这通过创建控件来实现：

    ```
    var AbstractAction = require('web.AbstractAction');
    
    var ClientAction = AbstractAction.extend(ControlPanelMixin, {
        ...
    });```
    ```
    
    在不需要时不要添加控制面板mixin。注意需要一些代码（通过mixin所提供的`update_control_panel`方法）来与控制面板进行交互 。
    
*   注册客户端动作：

    通常，我们需要让网页客户端知晓客户端动作和实际类之间的映射：

    ```var core = require('web.core');
core.action_registry.add('my-custom-action', ClientAction);

然后，要在网页客户端中使用客户端动作，我们需要通过相应的`tag`属性创建一个客户端动作记录（`ir.actions.client`模型的记录）：

```
<record id="my_client_action" model="ir.actions.client">
    <field name="name">Some Name</field>
    <field name="tag">my-custom-action</field>
</record>
```



### 使用控制面板mixin

默认AbstractAction类不包含控制面板mixin。这说明客户端动作不显示控制面板。为解决这个问题，需要完成一些步骤。

*   在控件中添加ControlPanelMixin：

    > ```var ControlPanelMixin = require('web.ControlPanelMixin');
    > 
    > var MyClientAction = AbstractAction.extend(ControlPanelMixin, {
    >  ...
    > });
    > ```

*   在需要升级控制面板时调用*update_control_panel*方法。例如：

    > ```
    > var SomeClientAction = Widget.extend(ControlPanelMixin, {
    >  ...
    >  start: function () {
    >      this._renderButtons();
    >      this._updateControlPanel();
    >      ...
    >  },
    >  do_show: function () {
    >       ...
    >       this._updateControlPanel();
    >  },
    >  _renderButtons: function () {
    >      this.$buttons = $(QWeb.render('SomeTemplate.Buttons'));
    >      this.$buttons.on('click', ...);
    >  },
    >  _updateControlPanel: function () {
    >      this.update_control_panel({
    >          cp_content: {
    >             $buttons: this.$buttons,
    >          },
    >   });
    > ```

更多详情，请参见*control_panel.js* 文件。

