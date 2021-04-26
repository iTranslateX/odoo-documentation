# 测试Odoo

本文来自[Odoo 13官方文档之开发者文档](https://alanhou.org/odoo-13-developer-documentation/)系列文章</dt>

有很多种测试应用的方法。在Odoo中，我们有三种测试

*   Python 单元测试 (参见[测试Python代码](#testing-python-code))：用于测试模型业务逻辑
*   JS单元测试(参见[测试JS代码](#testing-js-code))：用于分离测试javascript代码
*   导览(参见[集成测试](#integration-testing))：模拟真实场景的导览。它们确保python及 javascript部分正常进行对话。

## 测试Python代码

Odoo提供使用单元测试来测试模块的支持。

编写测试只需在模块中定义一个 `tests` 子包，它会自动查看测试模块。测试模块应有一个以`test_`开头的名称并且会从 `tests/__init__.py`, 进行导入，如：

```
your_module
|-- ...
`-- tests
    |-- __init__.py
    |-- test_bar.py
    `-- test_foo.py
```

且 `__init__.py` 包含：

`from . import test_foo, test_bar`

### ⚠️警告

不是通过 `tests/__init__.py`导入的测试模块将不会运行

测试运行器只是会运行测试类，正如[unittest 官方文档](https://docs.python.org/3/library/unittest.html)所描述的那样， 但 Odoo提供了一些与测试Odoo内容（主要是模块）相关的工具和帮助类：

###### `*class *odoo.tests.common.TransactionCase(*methodName='runTest'*)`

每个TestCase的测试方法以自己的游标在自身的事务中运行。在每次测试完成后会回滚事务并关闭游标。

###### `browse_ref(*xid*)`

为所提供的[外部标识符](https://www.odoo.com/documentation/13.0/glossary.html#term-external-identifier)返回记录对象

参数

**xid** – 完全限定[外部标识符](https://www.odoo.com/documentation/13.0/glossary.html#term-external-identifier)，形式为 `*module*.*identifier*`

在未找到时抛出

ValueError

返回

[`BaseModel`](https://alanhou.org/odoo-13-orm-api/#odoo.models.BaseModel "odoo.models.BaseModel")

###### `ref(*xid*)`

对所提供的[外部标识符](https://www.odoo.com/documentation/13.0/glossary.html#term-external-identifier)返回数据库ID，`get_object_reference`的快捷方式

参数

**xid** – 完全限定 [外部标识符](https://www.odoo.com/documentation/13.0/glossary.html#term-external-identifier)，形式为`*module*.*identifier*`

在未找到时抛出

ValueError

返回

注册的 id

###### `*class *odoo.tests.common.SingleTransactionCase(*methodName='runTest'*)`

所有测试方法的TestCase在同一个事务中运行，事务通过第一个测试方法启动并在最后一个测试方法完成后进行回滚。

###### `browse_ref(*xid*)`

对所提供的[外部标识符](https://www.odoo.com/documentation/13.0/glossary.html#term-external-identifier)返回记录对象

参数

**xid** – 完全限定 [外部标识符](https://www.odoo.com/documentation/13.0/glossary.html#term-external-identifier)，形式为 `*module*.*identifier*`

在未找到时抛出

ValueError

返回

[`BaseModel`](https://alanhou.org/odoo-13-orm-api/#odoo.models.BaseModel "odoo.models.BaseModel")

###### `ref(*xid*)`

对所提供的[外部标识符](https://www.odoo.com/documentation/13.0/glossary.html#term-external-identifier)返回数据库ID，快捷方式为 `get_object_reference`

参数

**xid** – 完全限定 [外部标识符](https://www.odoo.com/documentation/13.0/glossary.html#term-external-identifier)，形式为 `*module*.*identifier*`

在未找到时抛出

ValueError

返回

注册的id

###### `*class *odoo.tests.common.SavepointCase(*methodName='runTest'*)`

类似于 [`SingleTransactionCase`](#odoo.tests.common.SingleTransactionCase "odoo.tests.common.SingleTransactionCase") ，其中所有测试方法都在单个事务中运行，但每个测试用例在回滚的保存点（子事务）内部运行。

用于包含快速测试的测试用例，但具有所有测试用例的常见数据库配置(复杂in-db测试数据)： `setUpClass()` 可用于生成一次数据库测试数据，然后所有测试用例使用不影响其它测试的相同数据， 但也无需重建测试数据。

###### `*class *odoo.tests.common.HttpCase(*methodName='runTest'*)`

带有url_open和Chrome headless帮助方法的事务型HTTP TestCase。

###### `browse_ref(*xid*)`

为所提供的[外部标识符](https://www.odoo.com/documentation/13.0/glossary.html#term-external-identifier)返回记录对象

参数

**xid** – 完全限定[外部标识符](https://www.odoo.com/documentation/13.0/glossary.html#term-external-identifier)，形式为`*module*.*identifier*`

在未找到时返回

ValueError

返回

[`BaseModel`](https://alanhou.org/odoo-13-orm-api/#odoo.models.BaseModel "odoo.models.BaseModel")

###### `phantom_js(*url_path*, *code*, *ready=''*, *login=None*, *timeout=60*, ***kw*)`

在浏览器中运行的Test js代码 - 可日志为‘login’ - 加载url_path给定的页面 - 等待对象就绪 - 页面内部的eval(code)

标识成功测试执行： console.log(‘test successful’)

标识失败时执行: console.error(‘test failed’)

在超时前两者都未执行则测试失败。

###### `ref(*xid*)`

对所提供的[外部标识符](https://www.odoo.com/documentation/13.0/glossary.html#term-external-identifier)返回数据库ID，是 `get_object_reference`的快捷方式

参数

**xid** – 完全限定[外部标识符](https://www.odoo.com/documentation/13.0/glossary.html#term-external-identifier)，形式为 `*module*.*identifier*`

在未找到时抛出

ValueError

返回

注册的id

###### `odoo.tests.common.tagged(**tags*)`

标记BaseCase对象标签的装饰器存储于一个可由‘test_tags’属性访问的集合中。以 ‘-‘ 为标签的前缀会删除该标签，如要删除‘standard’标签。默认所有odoo.tests.common 中的测试类有一个默认为‘standard’及模块技术名称。在使用类继承时，标签未被继承。

默认，测试在相应模块安装后运行一次。测试用例也可配置为所有模块安装后运行，而不是紧接着模块安装后运行：

###### `odoo.tests.common.at_install(*flag*)`

设置测试的在安装状态，该标记是一个指定测试应(`True`)或不应 (`False`) 在模块安装时运行的布尔值。

默认，测试在模块安装之后、开始下一个模块安装之前运行。

自从版本12.0开始淘汰： `at_install` 当前是一个标记，虽然`tagged` 仅在测试类中生效，但你可以使用 [`tagged()`](https://alanhou.org/odoo-13-testing-odoo/#odoo.tests.common.tagged "odoo.tests.common.tagged") 来添加/删除它

###### `odoo.tests.common.post_install(*flag*)`

设置测试的安装后状态。该标记是一个指定测试应或不应在模块安装集合之后运行的布尔值。

默认，在当前安装集合中所有模块的安装测试不运行测试。

从版本12.0开始淘汰： `post_install`现在是一个标记， 虽然`tagged` 仅在测试类中生效，但你可以使用 [`tagged()`](https://alanhou.org/odoo-13-testing-odoo/#odoo.tests.common.tagged "odoo.tests.common.tagged") 来添加/删除它

最常见的情况是使用 [`TransactionCase`](https://alanhou.org/odoo-13-testing-odoo/#odoo.tests.common.TransactionCase "odoo.tests.common.TransactionCase") 并在每个方法中测试模型的属性：

```
class TestModelA(common.TransactionCase):
    def test_some_action(self):
        record = self.env['model.a'].create({'field': 'value'})
        record.some_action()
        self.assertEqual(
            record.field,
            expected_field_value)

    # other tests...
```

测试方法必须以 `test_`开头

###### `*class *odoo.tests.common.Form(*recordp*, *view=None*)`

服务端 (部分)表单视图实现

实现很多的“表单视图”操作流，如服务端测试可更好的反映行为，它将在操作界面时观察到：

*   对“创建”调用相关的 onchange
*   对设置字段调用相关的onchange
*   对在 x2many字段相应地处理默认值 & onchange

保存创建模式下返回的创建记录表单。

常用字段可仅直接分配给表单，对[`Many2one`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Many2one "odoo.fields.Many2one") 字段分配一个singleton记录集：

```
# empty recordset => creation mode
f = Form(self.env['sale.order'])
f.partner_id = a_partner
so = f.save()
```

在编辑记录时，使用表单作为上下文管理器来自动在作用域结束处保存它：

```
with Form(so) as f2:
    f2.payment_term_id = env.ref('account.account_payment_term_15days')
    # f2 is saved here
```

对于 [`Many2many`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Many2many "odoo.fields.Many2many") 字段，字段本身是一个 [`M2MProxy`](#odoo.tests.common.M2MProxy "odoo.tests.common.M2MProxy") 并可通过添加或删除记录来进行修改：

```
with Form(user) as u:
    u.groups_id.add(env.ref('account.group_account_manager'))
    u.groups_id.remove(id=env.ref('base.group_portal').id)
```

最后 [`One2many`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.One2many "odoo.fields.One2many") 具体化为[`O2MProxy`](https://alanhou.org/odoo-13-testing-odoo/#odoo.tests.common.O2MProxy "odoo.tests.common.O2MProxy")。

因 [`One2many`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.One2many "odoo.fields.One2many") 仅通过其父级存在，它更直接地通过[`new()`](#odoo.tests.common.O2MProxy.new "odoo.tests.common.O2MProxy.new") 和 [`edit()`](#odoo.tests.common.O2MProxy.edit "odoo.tests.common.O2MProxy.edit") 方法创建“子表单” 。这些通常将用作上下文管理器，因为它们在父级记录中进行保存：

```
with Form(so) as f3:
    # add support
    with f3.order_line.new() as line:
        line.product_id = env.ref('product.product_product_2')
    # add a computer
    with f3.order_line.new() as line:
        line.product_id = env.ref('product.product_product_3')
    # we actually want 5 computers
    with f3.order_line.edit(1) as line:
        line.product_uom_qty = 5
    # remove support
    f3.order_line.remove(index=0)
    # SO is saved here
```

参数

*   **recordp** ([`odoo.models.Model`](https://alanhou.org/odoo-13-orm-api/#odoo.models.Model "odoo.models.Model")) – 对于singleton记录为空。空记录集会将视图置为“创建”模式并触发对default_get 和 on-load onchange的调用，一个 singleton会将其设置为“编辑”模式并仅加载视图的数据。
*   **view** (`int | str | odoo.model.Model`) – onchange和视图约束所使用的id, xmlid 或实际视图对象。如未提供，仅加载模型的默认视图。

版本12.0中新增。

###### `save()`

保存表单，在可用时返回所创建的记录

*   不保存 `只读` 字段
*   （在编辑时）不保存未修改字段 — 任何赋值或 onchange返回标记字段为已修改，设置为当前值也是如此

抛出

[**AssertionError**](https://docs.python.org/3/library/exceptions.html#AssertionError "(in Python v3.8)") – 若表单没有任何未填写的必填字段

###### `*class *odoo.tests.common.M2MProxy`

作为记录集的`序列`，可进行索引或切片来获取实际底层记录集。

###### `add(*record*)`

向字段添加`记录`，记录必须是已存在的。

所添加的内容仅在保存父级记录时最终生效。

###### `clear()`

删除m2m中的所有已有记录

###### `remove(*id=None*, *index=None*)`

删除指定索引或字段中具有所提供 id 的记录。

###### `*class *odoo.tests.common.O2MProxy`

###### `edit(*index*)`

返回一个 [`表单`](#odoo.tests.common.Form "odoo.tests.common.Form") 来编辑已存在的 [`One2many`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.One2many "odoo.fields.One2many") 记录。

表单通过可编辑的列表视图或通过字段的表单视图进行创建。

抛出

[**AssertionError**](https://docs.python.org/3/library/exceptions.html#AssertionError "(in Python v3.8)") – 若字段不可编辑

###### `new()`

返回一个相应初始化的新 [`One2many`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.One2many "odoo.fields.One2many") 记录的[`表单`](#odoo.tests.common.Form "odoo.tests.common.Form")。

表单通过可编辑的列表视图或通过字段的表单视图进行创建。

抛出

[**AssertionError**](https://docs.python.org/3/library/exceptions.html#AssertionError "(in Python v3.8)") – 若字段不可编辑

###### `remove(*index*)`

删除父级表单中位于 `index`处的记录。

抛出

[**AssertionError**](https://docs.python.org/3/library/exceptions.html#AssertionError "(in Python v3.8)") – 若字段不可编辑

### 运行测试

如果在启动Odoo服务启用了[`--test-enable`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-test-enable)，测试在安装或升级模块时自动运行。

### 测试选择

在Odoo中， Python测试可打标签，来便于在运行测试时选择测试。

`odoo.tests.common.BaseCase` (通过通过 [`TransactionCase`](https://alanhou.org/odoo-13-testing-odoo/#odoo.tests.common.TransactionCase "odoo.tests.common.TransactionCase"), [`SavepointCase`](https://alanhou.org/odoo-13-testing-odoo/#odoo.tests.common.SavepointCase "odoo.tests.common.SavepointCase") 或 [`HttpCase`](https://alanhou.org/odoo-13-testing-odoo/#odoo.tests.common.HttpCase "odoo.tests.common.HttpCase")) 的子类自动使用 `standard`, `at_install` 及默认用它们的源模块的免进行打标签。

#### 调用

[`--test-tags`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-test-tags) 可用于选择/过滤在命令行中运行的测试。

这个选项默认为 `+standard` ，表示测试（显式或隐式地）使用 `standard` 打标签，在使用 [`--test-enable`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-test-enable)启动Odoo时默认运行。

在编写测试时，[`tagged()`](https://alanhou.org/odoo-13-testing-odoo/#odoo.tests.common.tagged "odoo.tests.common.tagged") 装饰器可用于**测试类**来添加或删除标签。

装饰器的参数为标签名，为字符串。

### 🚫危险

[`tagged()`](#odoo.tests.common.tagged "odoo.tests.common.tagged")是一个类装饰器，对于函数或方法没有效果

标签可使用减号 (`-`) 作为前缀，来删除而非添加或选择它们，例：如果你不想要测试默认执行，可以删除 `standard` 标签：

```
from odoo.tests import TransactionCase, tagged

@tagged('-standard', 'nice')
class NiceTest(TransactionCase):
    ...
```

这个测试默认不会选中，要运行它必须显式地选中相关标签：

`$ odoo-bin --test-enable --test-tags nice`

注意只有添加了`nice` 标签的测试会进行执行。要同时运行 `nice` 和 `standard` 测试，为[`--test-tags`](#cmdoption-odoo-bin-test-tags)提供多个值：在命令行中，值是累加的 (你选择带有指定标签中的任意一个的所有测试)

`$ odoo-bin --test-enable --test-tags nice,standard`

配置切换参数还接收 `+` 和 `-` 前缀。 `+` 前缀是暗含的，因此完全可选。即使由其它标签选中， `-` (减号) 前缀用于取消选择具有该前缀标签的测试， 例：如有 `standard` 的测试，还使用 `slow` 添加了标签，那么会运行除 slow 以外的所有标准测试：

`$ odoo-bin --test-enable --test-tags 'standard,-slow'`

在编写不继承 `BaseCase`的测试时， 这个测试不会有默认标签，你需要显式地添加它们来让测试包含在默认测试套件中。这在使用简单的`unittest.TestCase` 时是一个普遍问题，因为它们不会进行运行：

```
import unittest
from odoo.tests import tagged

@tagged('standard', 'at_install')
class SmallTest(unittest.TestCase):
    ...
```

#### 特殊标记

*   `standard`: 所有继承自`BaseCase` 的Odoo测试都隐式地打了standard标签。 [`--test-tags`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-test-tags)默认也为`standard`。这表示未打标签的测试在启用测试时会默认执行。
*   `at_install`: 表示测试会在该模块安装后及其它模块安装前执行。这是默认的隐式标签。
*   `post_install`: 表示测试会在所有模块安装完之后执行。对于大部分时候的HttpCase测试会希望这么做。注意这与`at_install`并不互斥，但通常不会需要两者同时使用，`post_install`通常在对测试类打标签时与 `-at_install` 配对。
*   *module_name*: Odoo测试类继承`BaseCase`，隐式地使用模块的技术名称打标签。这允许在测试时轻易地选择或排除具体模块，例：如果希望仅通过`stock_account`来运行测试：

`$ odoo-bin --test-enable --test-tags stock_account`

#### 示例

### ⚠️重要

测试仅在安装或升级模块时执行。因此模块需要通过 [`-u`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-u) 或 [`-i`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-i) 选项进行选中。为保持简洁，下面的示例中没有指定这些选项。

仅通过sale模块运行测试：

`$ odoo-bin --test-enable --test-tags sale`

通过sale模块但不包含打了 slow 标签的内容运行测试：

`$ odoo-bin --test-enable --test-tags 'sale,-slow'`

仅对stock或打了 slow 标签的运行测试：

`$ odoo-bin --test-enable --test-tags '-standard, slow, stock'`

`-standard` 是隐式的(非必须的)，添加它是为了保持清晰

## 测试JS代码

测试复杂系统是防止退化的重要保障并可确保一些基本功能仍正常运行。因Odoo有大量的Javascript代码，很有必要对其进行测试。在这一小节中，我们将讨论在隔离的状态下测试 JS 代码的实践，这些测试只保留在浏览器层面，而不触达服务端。

### Qunit 测试套件

Odoo框架使用[QUnit](https://qunitjs.com/) 库测试框架来作为测试运行器。QUnit定义了**测试**和**模组**（一组相关测试）的概念，并为我们提供了执行测试的web 界面。

例如，以下是 pyUtils测试运行的状况：

```
QUnit.module('py_utils');

QUnit.test('simple arithmetic', function (assert) {
    assert.expect(2);

    var result = pyUtils.py_eval("1 + 2");
    assert.strictEqual(result, 3, "should properly evaluate sum");
    result = pyUtils.py_eval("42 % 5");
    assert.strictEqual(result, 2, "should properly evaluate modulo operator");
});
```

运行测试套件的主要方式是要有运行中的Odoo服务，然后在浏览器中导航至 `/web/tests`。然后测试套件会由浏览器的.Javascript引擎进行运行。

![image](https://upload-images.jianshu.io/upload_images/14565748-7e4cbfcf6a5f0628.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

网页 UI有很多有用的功能：它可以仅运行一些子模块，或过滤匹配某字符串的测试。它可以显示每个断言，不管成功或失败，重新运行具体测试, …

### ⚠️警告

在运行测试套件时，要确保：

*   焦点在浏览器窗口中，
*   未进行放大/缩放。需要是100%的精确原大小比例。

如果不是如此，有些测试会失败，不返回相当的说明。

### 测试基础框架

以下是对测试框架重要部分的高级别概览：

*   有一个名为[web.js_tests_assets](https://github.com/odoo/odoo/blob/51ee0c3cb59810449a60dae0b086b49b1ed6f946/addons/web/views/webclient_templates.xml#L427)的资源包。 这个包中包含主要代码 (通用资源+ 后台资源), 一些库, QUnit测试运行器, 及一些其它帮助方法代码
*   另一个资源包[web.qunit_suite](https://github.com/odoo/odoo/blob/51ee0c3cb59810449a60dae0b086b49b1ed6f946/addons/web/views/webclient_templates.xml#L509), 包含所有测试(及 js_tests_assets 代码)。几乎所有测试文都应添加到这个包里
*   在网页中有一个[控制器](https://github.com/odoo/odoo/blob/51ee0c3cb59810449a60dae0b086b49b1ed6f946/addons/web/controllers/main.py#L637)，映射到路由 */web/tests*。这个控制器仅渲染 *web.qunit_suite* 模板。
*   要执行测试，我们可以仅将浏览器指向路由 */web/tests*。这种情况下，浏览器会下载所有资源，而 QUnit会接管。
*   在[qunit_config.js](https://github.com/odoo/odoo/blob/51ee0c3cb59810449a60dae0b086b49b1ed6f946/addons/web/static/tests/helpers/qunit_config.js#L49)中有一些代码， 在测试成功或失败时会在控制台中记录一些信息。
*   我们希望runbot也能运行这些测试， 因此(在 [test_js.py](https://github.com/odoo/odoo/blob/51ee0c3cb59810449a60dae0b086b49b1ed6f946/addons/web/tests/test_js.py#L13)中)有一个测试，它只是产生一个浏览器并指向*web/tests* 链接。注意phantom_js 方法并不产生phantom_js，而是用的Chrome headless 。

### 模块化及测试

通过Odoo设计的方式，任何插件可修改系统中其它部分的行为。例如，*voip* 插件可修改 *FieldPhone* 张爱玲的来使用额外的功能。从测试系统的角度来看这并不是好事，因为这表示在插件网页中安装voip插件时测试会失败(注意runbot运行测试时安装了所有的插件)。

同时，我们的测试系统很棒，因为它会监测到其它影响核心功能的模块。对这一问题没有完整的方案。当前我们按具体情况逐一进行解决。

通常修改其它行为不是个好想法。就我们的voip示例来说，添加新的*FieldVOIPPhone组件*及修改需要它的一些视图显然更为清晰。通过这种方式，*FieldPhone* 组件不会受影响，两者均可进行测试。

### 新增测试用例

假设我们在维护一个插件*my_addon*，交且我们希望对一些javascript代码添加测试 (例如，位于*my_addon.utils*中的工具函数 myFunction)。添加u 新测试用例的流程如下：

1.  新建文件 *my_addon/static/tests/utils_tests.js*。该文件包含添加 QUnit 模块 *my_addon > utils*的基本代码。
```
    odoo.define('my_addon.utils_tests', function (require) {
    "use strict";
     
    var utils = require('my_addon.utils');
    
    QUnit.module('my_addon', {}, function () {
     
        QUnit.module('utils');
    
    });
    });
```

2.  在*my_addon/assets.xml*中，将该文件添加到主测试资源中：
```
    <?xml version="1.0" encoding="utf-8"?>
    <odoo>
        <template id="qunit_suite" name="my addon tests" inherit_id="web.qunit_suite">
            <xpath expr="//script[last()]" position="after">
                <script type="text/javascript" src="/my_addon/static/tests/utils_tests.js"/>
            </xpath>
        </template>
    </odoo>
```

3.  重启服务并更新*my_addon*，或通过界面进行 (来确保加载了新测试文件)
4.  在*utils* 子测试套件的定义之后添加一个测试用例：

```
QUnit.test("some test case that we want to test", function (assert) {
    assert.expect(1);
    
   var result = utils.myFunction(someArgument);
    assert.strictEqual(result, expectedResult);
    > });
```

5.  访问*/web/tests/* 来确保执行了测试

### 帮助函数和特殊化断言

不借助帮助，很难测试到Odoo的某些部分。具体的说，视图很复杂，因其与服务端进行通讯并可能会执行一些rpc调用，这需要mock数据。这正是我们开发了一些特定帮助函数的原因，位于 [test_utils.js](https://github.com/odoo/odoo/blob/51ee0c3cb59810449a60dae0b086b49b1ed6f946/addons/web/static/tests/helpers/test_utils.js)中。

*   Mock测试函数：这些函数帮助设置测试环境。最重要的用例是mock由Odoo服务所给定的结果。这些函数使用一个[mock服务](https://github.com/odoo/odoo/blob/51ee0c3cb59810449a60dae0b086b49b1ed6f946/addons/web/static/tests/helpers/mock_server.js)。 这是一个模拟常用模型方法如read, search_read, nameget…的返回结果的 javascript类
*   DOM帮助函数：用于对具体目标模拟事件/动作。例如， testUtils.dom.click 对目标执行点击。注意这样比手动更为安全，因为它还检查目标是否存在及可见。
*   创建帮助函数：它们可能是由[test_utils.js](https://github.com/odoo/odoo/blob/51ee0c3cb59810449a60dae0b086b49b1ed6f946/addons/web/static/tests/helpers/test_utils.js)导出的最为重要的函数。 这些函数用于通过 mock 环境创建组件，以大量尽可能模拟真实条件的小细节。最重要的肯定是 [createView](https://github.com/odoo/odoo/blob/51ee0c3cb59810449a60dae0b086b49b1ed6f946/addons/web/static/tests/helpers/test_utils_create.js#L267)。
*   [qunit断言](https://github.com/odoo/odoo/blob/51ee0c3cb59810449a60dae0b086b49b1ed6f946/addons/web/static/tests/helpers/qunit_asserts.js)： QUnit可通过特定断言进行扩展。对于Odoo，我们经常测试一些DOM 属性。这是我们进行一些断言来进行协助的原因。例如，*containsOnce* 断言接收一个widget/jQuery/HtmlElement 和一个选择器，然后查看目标是否刚好匹配该 css 选择器。

例如，借助这些帮助函数，以下是简单的表单测试的示例：

```
QUnit.test('simple group rendering', function (assert) {
    assert.expect(1);

    var form = testUtils.createView({
        View: FormView,
        model: 'partner',
        data: this.data,
        arch: '<form string="Partners">' +
                '<group>' +
                    '<field name="foo"/>' +
                '</group>' +
            '</form>',
        res_id: 1,
    });

    assert.containsOnce(form, 'table.o_inner_group');

    form.destroy();
});
```

注意使用testUtils.createView帮助函数及containsOnce断言。同时，表单控制在测试结束会进行相应的销毁。

### 最佳实践

以下并未进行排顺序：

*   所有测试文件应在*some_addon/static/tests/*中进行添加
*   对于漏洞修复，确保不进行漏洞修复时测试失败，修复时测试成功。这会确保其真实有效。
*   尝试使用所需的最小化测试代码来让测试有效。
*   通常，两个小测试胜于一个大测试。小测试更易于理解及修复。
*   在测试后保持进行清理。例如，如果测试实例化了一个组件，应在结束时销毁它。
*   无需要进行完整的代码覆盖。但添加一些测试会有所帮助：这让你的代码不会完全崩溃，并在测试修复时，它更易于向已有测试套件添加测试。
*   如果想要查看一些负向断言(例如，HtmlElement 没有具体的 css 类)， 那么尝试在相同测试中添加正向断言 (例如，通过执行一个修改该状态的动作)。这会帮助避免测试在未来无用 (例如在修改了css类时)。

### ℹ️小贴士

*   仅运行一个测试：你可以 (临时！) 修改*QUnit.test(…)* 的定义到 *QUnit.only(…)*。这有助于确保QUnit 仅运行具体的测试。
*   debug 标记：大多数工具函数都有测试模式 (通过 debug: true 参数进行启用)。在这种情况下，目标组件会被放到DOM中而非隐藏的 qunit 具体夹具中，并且会记录更多信息。例如，所有的 mock网络通讯在控制台中都可用。
*   在处理失败的测试时，通常会添加 debug 标记，然后注释测试的结尾 (具体地说是销毁的调用)。通过它，可以直接查看组件的状态，甚至能通过点击/与其交互来操作组件。

## 集成测试

分别Python代码和 JS代码非常有益，但无法证明网页客户端和服务端共同有效。为进行验证，我们可以编写另一种测试：导览。导览是一种有意义的业务流的小型场景。它说明了一系列应该遵循的步骤。然后测试运行器会创建一个phantom_js浏览器，指向相应的url并根据场景模拟点击和输入。

### 在browser_js测试时截屏和录屏

在通过命令行运行使用HttpCase.browser_js的测试时，Chrome浏览器在headless模式下运行。默认在测试失败时，会在失败的瞬间截一张PNGu 并保存在：

```
'/tmp/odoo_tests/{db_name}/screenshots/'
```

自Odoo 13.0开始添加了两个控制这一行为命令行参数： [`--screenshots`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-screenshots) and [`--screencasts`](https://alanhou.org/odoo-13-command-line-interface-odoo-bin/#cmdoption-odoo-bin-screencasts)