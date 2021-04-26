# QWeb

- 本文来自[Odoo 13官方文档之开发者文档](../README.md)系列文章

QWeb是由Odoo[2](#othertemplates)所使用的一种主要[模板](https://en.wikipedia.org/wiki/Template_processor)引擎。它是一种模板引擎[1](#genshif) 并多用于生成 [HTML](https://en.wikipedia.org/wiki/HTML)片断和页面。指定为XML属性的模板指令以 `t-`为前缀，例如 `t-if` 用于[条件判断](#reference-qweb-conditionals)，其元素和其它属性会直接进行渲染。为避免元素渲染，还有 `<t>`占位符元素，它执行其指令但不会在自身及其中生成输出：

```
<t t-if="condition">
    <p>Test</p>
</t>
```

以上会产生的结果：

```
<p>Test</p>
```

如 `condition`为 true, 但使用：

```
<div t-if="condition">
    <p>Test</p>
</div>
```

会产生如下结果：

```
<div>
    <p>Test</p>
</div>
```



## 数据输出

QWeb有一个主输出指令，它自动对其内容进行HTML转义，来在显示用户所提供的内容时降低 [XSS](https://en.wikipedia.org/wiki/Cross-site_scripting) 风险： `esc`.

`esc` 接收一个表达式，运行它并打印出内容：

```
<p><t t-esc="value"/></p>
```

通过将`value` 设置为 `42` 来进行渲染并产生结果：

```
<p>42</p>
```

还有另一个输出指令 `raw` ，它的行为和 `esc` 相同，但*不对输出进行HTML转义。*它可用于显示单独的构建标记 (e.g. 通过函数) 或已清洗的用户提供标记。



## 条件判断

QWeb拥有一个条件指令 `if`，它运行给定为属性值的表达式：

```
<div>
    <t t-if="condition">
        <p>ok</p>
    </t>
</div>
```

如果条件为true的话会渲染元素：

```
<div>
    <p>ok</p>
</div>
```

但如果条件为false，它会从结果中删除：

```
<div>
</div>
```

条件渲染应用于指令的所有者，不必一定为 `<t>`：

```
<div>
    <p t-if="condition">ok</p>
</div>
```

将给出与前例相同的结果。

还存在额外的分支指令 `t-elif` 和 `t-else` ：

```
<div>
    <p t-if="user.birthday == today()">Happy birthday!</p>
    <p t-elif="user.login == 'root'">Welcome master!</p>
    <p t-else="">Welcome!</p>
</div>
```



## 循环

QWeb拥有遍历指令`foreach`，它接收一个表达式，返回待遍历的集合以及第二个用于对遍历的“当前项”提供名称的参数 `t-as` ：

```
<t t-foreach="[1, 2, 3]" t-as="i">
    <p><t t-esc="i"/></p>
</t>
```

将会渲染为：

```
<p>1</p>
<p>2</p>
<p>3</p>
```

类似条件，`foreach` 应用于带有指令属性的元素，并且

```
<p t-foreach="[1, 2, 3]" t-as="i">
    <t t-esc="i"/>
</p>
```

等价于前例。

`foreach` 可对数组 (当前项为当前值)或映射(当前项为当前键)进行遍历。.仍支持对整数的遍历（等价于遍历一个包含0到所提供整数（不含）的数组）但已淘汰。

除通过`t-as`传递名称乐， `foreach`还对不同数据点提供一些其它变量：

### ⚠️警告

`$as`会替换为传递给 `t-as`的名称

- `*$as*_all` (淘汰)

  所进行遍历的对象这个变量仅在JavaScript QWeb中可用，在 Python中不可用。

- `*$as*_value`

  当前遍历值，和列表与整型的 `$as` 相同，但对于映射它提供了值 (其中 `$as`提供了键)

- `*$as*_index`

  当前迭代索引 (遍历中索引为0的第一项)

- `*$as*_size`

  如果可用时为集合的大小

- `*$as*_first`

  当前项是否为遍历的第一项 (等价于 `*$as*_index == 0`)

- `*$as*_last`

  当前项是否为遍历的最后一项(等价于 `*$as*_index + 1 == *$as*_size`)，要求遍历的大小可用

- `*$as*_parity` (淘汰)

   为`"even"` 或 `"odd"`，当前遍历回合索引的奇偶性

- `*$as*_even` (淘汰)

  一个用于表明当前遍历回合为偶数索引的布尔标记

- `*$as*_odd` (淘汰)

  一个用于表明当前遍历回合为奇数索引的布尔标记

这些提供的额外变量及 `foreach`中创建的新变量仅在``foreach``的作用域中可用。若变量存在于`foreach`的上下文之外， foreach末尾的值会拷贝到全局上下文中

```
<t t-set="existing_variable" t-value="False"/>
<!-- existing_variable now False -->

<p t-foreach="[1, 2, 3]" t-as="i">
    <t t-set="existing_variable" t-value="True"/>
    <t t-set="new_variable" t-value="True"/>
    <!-- existing_variable and new_variable now True -->
</p>

<!-- existing_variable always True -->
<!-- new_variable undefined -->
```



## 属性

QWeb可补时计算属性并对输出节点设置计算的结果。这通过 `t-att` (属性)指令完成，存在3种不同形式：

- `t-att-*$name*`

  创建名为 `$name` 的属性，运行属性值并将结果设置为属性的值：`<div t-att-a="42"/>`将会被渲染为：`<div a="42"></div>`

- `t-attf-*$name*`

  与前述相同，但参数是 [格式字符串](https://www.odoo.com/documentation/13.0/glossary.html#term-format-string)而不仅仅是一个表达式，通常用于混合字面量及非字面量字符串 (e.g. 类)：`<t t-foreach="[1, 2, 3]" t-as="item">    <li t-attf-class="row {{ (item_index % 2 === 0) ? 'even' : 'odd' }}">        <t t-esc="item"/>    </li> </t>`将会被渲染为：`<li class="row even">1</li> <li class="row odd">2</li> <li class="row even">3</li>`

- `t-att=mapping`

  如果参数是一个映射，第个 (key, value) 对生成一个新属性及其值：`<div t-att="{'a': 1, 'b': 2}"/>`会被渲染为：`<div a="1" b="2"></div>`

- `t-att=pair`

  如果参数是一个值对 (两个元素的元组或数组)，值对的第一项是属性的名称，第二项是其值：`<div t-att="['a', 'b']"/>`会被渲染为：`<div a="b"></div>`

## 设置变量

QWeb允许在模板内创建变量，来存储（memoize）一个运算 (多次使用它)，提供一个更清晰名称的数据， …

这通过`set`指令完成，它接收待创建变量的名称。设置的值通过两种方式提供：

- 一个包含表达式的 

  ```
  t-value
  ```

   属性，会设置其运行的结果：

  ```
  <t t-set="foo" t-value="2 + 1"/>
  <t t-esc="foo"/>
  ```

  以上将打印 `3`

- 如果没有 

  ```
  t-value
  ```

   属性，节点的内容体会进行渲染并设置为变量的值：

  ```
  <t t-set="foo">
      <li>ok</li>
  </t>
  <t t-esc="foo"/>
  ```

  将会生成 `<li>ok</li>` (因我们使用了 `esc` 指令内容会进行转义)

  使用这一运算的结果是`raw` 指令的一个显著用例。

## 调用子模板

QWeb模板可用于顶层渲染，但它们也可在其它模板之内 (来避免复制或对部分模板给定) 使用 `t-call` 指令：

```
<t t-call="other-template"/>
```

若定义`other_template`为下面的内容，通过父级执行上下文调用命名模板：

```
<p><t t-value="var"/></p>
```

上面的调用会被渲染为 `<p/>` (无内容)，但：

```
<t t-set="var" t-value="1"/>
<t t-call="other-template"/>
```

会被渲染为 `<p>1</p>`.

但这在 `t-call`之外的可见性存在问题。此外，`call` 指令体内的内容会在运行子模板之前调用，并可更改本地内容：

```
<t t-call="other-template">
    <t t-set="var" t-value="1"/>
</t>
<!-- "var" does not exist here -->
```

`call` 指令体可任意复杂 (不仅仅是 `set` 指令)，并且其渲染形式将在所调用模板为魔法 `0` 变量之内可用：

```
<div>
    This template was called with content:
    <t t-raw="0"/>
</div>
```

被调用为：

```
<t t-call="other-template">
    <em>content</em>
</t>
```

会产生结果：

```
<div>
    This template was called with content:
    <em>content</em>
</div>
```

## Python

### 独家指令

#### 资源包

#### “智能记录”字段模式化

`t-field` 指令仅在对“智能”记录（`browse`方法的结果）执行字段访问(`a.b`) 时才可使用。它可以自动根据字段类型格式化，并且在网站的富文本版本中集成。

`t-options` 可用于自定义字段，最常见的选项是 `widget`，其它选项独立于字段或组件。

### 调试

- `t-debug`

  使用PDB的`set_trace` API调用调试器。参数应为模块名，对其调用`set_trace` 方法：`<t t-debug="pdb"/>`等价于 `importlib.import_module("pdb").set_trace()`

### 帮助函数

#### 基于请求

QWeb的大部分Python端使用在控制器（HTTP请求时）中进行，其中存储在数据库中的模板 (作为 [视图](https://alanhou.org/odoo-13-views/#reference-views-qweb)) 可通过调用 [`odoo.http.HttpRequest.render()`](https://alanhou.org/odoo-13-web-controllers/#odoo.http.HttpRequest.render)来进行渲染：

```
response = http.request.render('my-template', {
    'context_value': 42
})
```

这会自动创建一个[`Response`](https://alanhou.org/odoo-13-web-controllers/#odoo.http.Response) 对象，它可在控制器中返回(或进行一步自定义在适配)。

#### 基于视图

比上述帮助函数更深层次的是`ir.ui.view`中的`render`方法：

###### `render(*cr, uid, id[, values][, engine='ir.qweb][, context]*)`

通过数据库 id 或[外部id](https://www.odoo.com/documentation/13.0/glossary.html#term-external-id)渲染QWeb 视图/模板。模板会自动从`ir.ui.view` 记录中进行加载。

在渲染上下文中设置了很多地默认值：

- `request`

  若存在，为当前 [`WebRequest`](https://alanhou.org/odoo-13-web-controllers/#odoo.http.WebRequest) 对象

- `debug`

  当前请求（若有）是否为 `debug` 模式

- [`quote_plus`](https://werkzeug.palletsprojects.com/en/0.16.x/urls/#werkzeug.urls.url_quote_plus)

  url编码工具函数

- [`json`](https://docs.python.org/3/library/json.html#module-json)

  相应的标准库模块

- [`time`](https://docs.python.org/3/library/time.html#module-time)

  相应的标准库模块

- [`datetime`](https://docs.python.org/3/library/datetime.html#module-datetime)

  相应的标准库模块

- [relativedelta](https://labix.org/python-dateutil#head-ba5ffd4df8111d1b83fc194b97ebecf837add454)

  参见模块

- `keep_query`

  `keep_query` 帮助函数

参数

- **values** – 传递给QWeb供渲染的上下文值
- **engine** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 用于渲染的Odoo模型名称，可用于扩展或在本地自定义 QWeb (基于 `ir.qweb` 的修改创建“新”qweb)

## Javascript

### 独家指令

#### 定义模板

`t-name` 指令仅可放在模板文件（文档根目标的直接后代）的顶层：

```
<templates>
    <t t-name="template-name">
        <!-- template code -->
    </t>
</templates>
```

它不接收其它参数，便可与`<t>`元素或任意其它元素共同使用。 通过 `<t>` 元素时， `<t>` 应包含一个子元素。

模板名是一个任意字符串，虽然关联了多个模板 (e.g. 调用了子模板)，可自定义使用点击分隔名称来表明层级关系。

#### 模板继承

模板继承用于在原处更改已有模板，e.g. 向由其它模块创建的模板添加信息。

模板继承通过`t-extend` 指令执行，它接收模板名称来修改为参数。

在 `t-extend` 与 `t-name` 进行合并时，会创建带有给定名称的新模板。在这种情况下所扩展的模板未进行修改，而是由指令定义了如何新建模板。

在这两种情况下修改由任意数量的 `t-jquery` 子指令执行：

```
<t t-extend="base.template">
    <t t-jquery="ul" t-operation="append">
        <li>new element</li>
    </t>
</t>
```

`t-jquery`指令接收一个 [CSS选择器](https://api.jquery.com/category/selectors/)。该选择器用于所继承的模板来选择*上下文节点*来应用所指定的 `t-operation` ：

- `append`

  节点的内容体添加到上下文节点的结束处 (在上下文节点最一个子元素之后)

- `prepend`

  节点的内容体前置到上下文节点 (在上下文节点的第一个子元素之前插入)

- `before`

  节点的内容体插入到上下文节点为之前

- `after`

  节点的内容体插入到上下文节点为之后

- `inner`

  节点的内容体替换上下文节点的子元素

- `replace`

  节点的内容体用于替换上下文节点本身

- `attributes`

  节点的内容体可为任意数量的`attribute` 元素，每个都带有`name` 属性和一些文本内容，上下文节点的命令属性将设置为指定值(如已存在则替换否则进行添加)

- No operation

  如未指定`t-operation` ，模板体会解析为 javascript代码并以 `this`通过上下文节点执行⚠️警告虽然比其它操作强大得多，但这一模式更难调试、更难维护，推荐避免使用它。

### 调试

javascript QWeb实现提供了一些调试钩子：

- `t-log`

  接收表达式参数，在渲染期间运行表达式并通过`console.log`在结果中进行记录：`<t t-set="foo" t-value="42"/> <t t-log="foo"/>`将在控制台中打印出print `42`

- `t-debug`

  在模板渲染时触发调试器断点：`<t t-if="a_test">    <t t-debug=""> </t>`如调试为活跃状态则停止执行 (具体的情况取决于浏览器及其开发工具)

- `t-js`

  节点内容体为在模板渲染时执行的javascript代码。接收一个 `context` 参数，渲染上下文所使用的名称在 `t-js`的内容体中可以获取到：`<t t-set="foo" t-value="42"/> <t t-js="ctx">    console.log("Foo is", ctx.foo); </t>`

### 帮助对象

###### `core.qweb`

(core是 `web.core` 模块) 是加载了所有模块定义的模板的[`QWeb2.Engine()`](#QWeb2.Engine) 实例，并引用标准帮助对象 `_` (下划线), `_t` (翻译函数) 和 [JSON](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON).

`core.qweb.render` can be used to easily render basic module templates



### API

###### `*class* QWeb2.Engine()`

QWeb “渲染器”，处理大部分QWeb逻辑(加载、解析、编译及渲染模板)。

Odoo Web在core模板为用户进行实例化，并将其导出到 `core.qweb`中。它还加载各种模板中的所有模板文件到该QWeb实例中。

[`QWeb2.Engine()`](#QWeb2.Engine) 还作为“模板命名空间”提供服务。

###### `QWeb2.Engine.QWeb2.Engine.render(*template*[, *context*])`

渲染此前加载的模板为字符串，使用 `context` (若提供)来查找在模板渲染期间访问的变量(e.g. 要显示的字符串)。

参数

- **template** (`String`) – 待渲染的模板名
- **context** (`Object`) – 模板渲染要使用的基本命名空间

返回

字符串

该引擎暴露另一个方法，可在一些用例下使用 (e.g. 如需在Odoo Web,中通过看板视图分离模板命名空间来获取它们自己的[`QWeb2.Engine()`](#QWeb2.Engine) 实例，因此它们的模板不与更通用的“模块”模板相冲突)：

###### `QWeb2.Engine.QWeb2.Engine.add_template(*templates*)`

在QWeb实例中加载一个模板文件 (一个模板集合) 。模板可指定为：

- 一个XML字符串

  QWeb会尝试解析其为一个XML文档然后加载它。

- 一个URL

  QWeb会尝试下载 URL内容，然后加载结果XML字符串。

- 一个`Document` 或 `Node`

  QWeb会遍历文档的第一级 (所提供根的子节点)并加载任意命名模板或模板重载。

 

[`QWeb2.Engine()`](#QWeb2.Engine) 还针对自定义行为暴露不同的属性：

###### `QWeb2.Engine.QWeb2.Engine.prefix`

用于在解析期间识别指令的前缀。一个字符串。默认为 `t`。

###### `QWeb2.Engine.QWeb2.Engine.debug`

将引擎置为“调试模式”的布尔标记。通常，QWeb拦截在模板执行期间抛出的任意错误。在调试模式下，它让所有的异常通过不进行拦截。

###### `QWeb2.Engine.QWeb2.Engine.jQuery`

这个 jQuery实例在模板继承处理过程中进行使用。默认为`window.jQuery`。

###### `QWeb2.Engine.QWeb2.Engine.preprocess_node`

一个`函数`。如存在，在编译每个DOM 节点到模板节点之前进行调用。在 Web中，它用于在模板中自动转译文本内容和一些属性。默认为 `null`。

[[1\]](#id2) 类似于[Genshi](https://genshi.edgewall.org/)之中的， 但它不使用 (且不支持) [XML命名空间](https://en.wikipedia.org/wiki/XML_namespace)

[[2\]](#id1) 但出于历史原因或因适合用例还保留一些其它模板。 Odoo 9.0仍依赖于 [Jinja](http://jinja.pocoo.org/) 和 [Mako](https://www.makotemplates.org/)。