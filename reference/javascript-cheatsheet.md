# Javascript速查

本文来自[Odoo 13官方文档之开发者文档](https://alanhou.org/odoo-13-developer-documentation/)系列文章

在JavaScript和Odoo中有很多种解决问题的方式。但 Odoo框架的设计是具有可扩展性的 (这一个巨大的约束)，而很多常见的问题有漂亮标准方案。标准方案可能具有易于 Odoo 开发者理解的优势，在 Odoo 修改时可保持继续有效。本文档尝试讲解我们可以解决这些问题的方式。注意这不一个手册。只是一个随机集合，或讲解了如何在某些情况下进行处理。首先，记住通过JS自定义odoo的第一条规则是：*尽量在python中解决*。 这看起来很奇怪，但这个python框架极具扩展性，很多行为只需通过对xml或python.进行少量修改即可实现。 这通常会比操作 JS 的维护成本更低：

*   JS框架会倾向于修改是多内容，因此 JS代码需要进行更频繁的更新
*   如需与服务端通讯并适当集成javascript 框架实现自定义行为经常会更为复杂。有很多由框架处理的小细节的自定义代码也需要重复编写。例如，响应式、更新 URL 或无闪烁显示数据。

本文档并没有真的讲解任何概念。更多的是一个速查小指南。更多详情，请参见javascript手册页面 (见[Javascript手册](https://alanhou.org/odoo-13-javascript-reference/))

## 新建字段组件

这可能是很常见的用例：我们希望在表单视图中以非常具体的（可能取决于业务）方式显示一些信息。 例如，假定我们希望根据一些业务条件来修改文本的颜色。

这可通过三个步骤来实现：新建一个组件、在字段仓库中注册它，然后在表单视图中添加组件到该字段中

*   新建组件：

    可通过继承组件来实现：

```
var FieldChar = require('web.basic_fields').FieldChar;

    var CustomFieldChar = FieldChar.extend({
        _renderReadonly: function () {
            // 在此处实现一些自定义逻辑
        },
    });
```

*   在字段仓库中注册它：

    网页客户端需要知道组件名称和其实际类之间的映射。这通过仓库完成：
```
 var fieldRegistry = require('web.field_registry');

    fieldRegistry.add('my-custom-field', CustomFieldChar);
```

*   向表单视图添加该组件：
```
<field name="somefield" widget="my-custom-field"/>
```

    注意只有表单、列表和看板视图使用这种字段组件仓库。这些视图紧密集成，因为列表和看板视图可以在表单视图中出现。

   

## 修改已有字段组件

另一种用例是我们希望修改已有字段组件。例如，odoo中的voip插件需要修改FieldPhone组件来添加对voip易于调用给定号码的可能性。这通过*包含*FieldPhone组件来实现，因此无需修改已有表单视图。

字段组件 (AbstractField的实例(子类 ))像每个其它组件一样，因此它们可以打[猴子补丁](# "Monkey Patch猴子补丁的含义是指在动态语言中，不去改变源码而对功能进行追加和变更。")。类似下面这样：

```
var basic_fields = require('web.basic_fields');
var Phone = basic_fields.FieldPhone;

Phone.include({
    events: _.extend({}, Phone.prototype.events, {
        'click': '_onClick',
    }),

    _onClick: function (e) {
        if (this.mode === 'readonly') {
            e.preventDefault();
            var phoneNumber = this.value;
            // 对voip调用电话号码...
        }
    },
});
```

注意这里无需添加组件到仓库中，因其已进行了注册。

## 从界面修改主组件

另一个常见用例是需要通过用户界面自定义一些元素。例如，在 home 菜单中添加消息。这种情况下的常用处理还是*包含*组件。这是实现它的唯一方式，因为对这些组件没有仓库。

通常通过下面这样的代码实现：

```
var HomeMenu = require('web_enterprise.HomeMenu');

HomeMenu.include({
    render: function () {
        this._super();
        // do something else here...
    },
});
```

## （全新）新建视图

新建视图是一个更高级的话题。这篇速查仅能重点介绍一些很可能会做的步骤 (排序无先后):

*   在`ir.ui.view`的`type`字段中新增一种视图类型：
```
class View(models.Model):
        _inherit = 'ir.ui.view'

        type = fields.Selection(selection_add=[('map', "Map")])
```

*   在`ir.actions.act_window.view`的`view_mode`字段中新建视图类型：
```
class ActWindowView(models.Model):
        _inherit = 'ir.actions.act_window.view'

        view_mode = fields.Selection(selection_add=[('map', "Map")])
```

* (在JavaScript中)创建4个组成视图的主要部分：

    我们需要一个视图(`AbstractView`的子类，这是工厂类)，一个渲染器(来自 `AbstractRenderer`)，一个控制器 (来看`AbstractController`) 以及一个模型 (来自 `AbstractModel`)。我推荐通过简单继承超类来开始：

 ```
var AbstractController = require('web.AbstractController');
    var AbstractModel = require('web.AbstractModel');
    var AbstractRenderer = require('web.AbstractRenderer');
    var AbstractView = require('web.AbstractView');

    var MapController = AbstractController.extend({});
    var MapRenderer = AbstractRenderer.extend({});
    var MapModel = AbstractModel.extend({});

    var MapView = AbstractView.extend({
        config: {
            Model: MapModel,
            Controller: MapController,
            Renderer: MapRenderer,
        },
    });
 ```

*  在仓库中添加视图：

    和平常一样，视图类型和实际类之间的映射需要进行更新：

```
var viewRegistry = require('web.view_registry');

    viewRegistry.add('map', MapView);
```


*   实现4个主要类：

`View`类需要解析 `arch`字段并设置其它3个类。 `Renderer` 负责在用户界面中展示数据， `Model` 用于和服务端对话、加载数据并处理数据。`Controller` 用于协调、与网页客户端对话, …

*   在数据库中创建一些视图：


```
<record id="customer_map_view" model="ir.ui.view">
        <field name="name">customer.map.view</field>
        <field name="model">res.partner</field>
        <field name="arch" type="xml">
            <map latitude="partner_latitude" longitude="partner_longitude">
                <field name="name"/>
            </map>
        </field>
    </record>
```

## 自定义已有视图

假定我们需要创建一个通用视图的自定义版本。例如，在上方面带有一些额外类似*ribbon*的看板视图（来显示一些具体自定义信息）。在这种情况下，它可通过3个步骤来实现：继承看板视图(可能还表示继承 控制器/渲染器及/或模型)，然后在视图仓库中注册视图，并且最终在看板框架中使用视图 (具体的示例是helpdesk仪表盘)。

*   继承视图：

    以下是其可能会有的样子：

```
var HelpdeskDashboardRenderer = KanbanRenderer.extend({
        ...
    });

    var HelpdeskDashboardModel = KanbanModel.extend({
        ...
    });

    var HelpdeskDashboardController = KanbanController.extend({
        ...
    });

    var HelpdeskDashboardView = KanbanView.extend({
        config: _.extend({}, KanbanView.prototype.config, {
            Model: HelpdeskDashboardModel,
            Renderer: HelpdeskDashboardRenderer,
            Controller: HelpdeskDashboardController,
        }),
    });
```


*  将其添加到视图仓库中：

    和平常一样，我们需要告知网页客户端视图名称和实际类之间的映射。

```
var viewRegistry = require('web.view_registry');
    viewRegistry.add('helpdesk_dashboard', HelpdeskDashboardView);
```


*   在实际视图中使用它：

    现在我们需要告知网页客户端指定的 `ir.ui.view`需要使用我们的新类。注意这是一个网页客户端的具体考虑。从服务端的视角来看，我们还是对应看板视图。这么做的相应方式是通过对框架的根节点使用特殊属性`js_class`(某一天会被重命名为 `widget`，因为这个名称不够好) ：

 ```
<record id="helpdesk_team_view_kanban" model="ir.ui.view" >
        ...
        <field name="arch" type="xml">
            <kanban js_class="helpdesk_dashboard">
                ...
            </kanban>
        </field>
    </record>
 ```

注意： 可以改变视图解析框架结构的方式。但从服务端的视角，这还是同一个基类型的视图，受制于相同的规则 (例如rng验证)。因此你的视图仍需具有有效的arch字段。

## Promise和异步代码

有关promise的良好和完整介绍，请参阅这篇优质文章 [https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/async%20%26%20performance/ch3.md](https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/async%20%26%20performance/ch3.md)

### 新建Promise

*   将常量转换 promise

    Promise有两个静态函数根据常量分别创建已完成（resolved）和已失败（rejected）的promise：

```
var p = Promise.resolve({blabla: '1'}); // 创建一个已完成的promise
    p.then(function (result) {
        console.log(result); // --> {blabla: '1'};
    });

    var p2 = Promise.reject({error: 'error message'}); // 创建一个已失败的promise
    p2.catch(function (reason) {
        console.log(reason); // --> {error: 'error message');
    });
```

注意即使promise的创建已经成功或是失败，`then` 或 `catch` 处理器还是会被异步地调用。

*  基于已有的异步代码

假设在函数你必须执行rpc，在已对this完成设置结果时。`this._rpc` 是一个返回`Promise`的函数。

```
function callRpc() {
        var self = this;
        return this._rpc(...).then(function (result) {
            self.myValueFromRpc = result;
        });
    }
```

*   对于基于回调的函数

    假定你在使用一个接收在完成时调用的回调作为参数的 `this.close` 函数。再假设你在一个方法中执行它且该方法必须在关闭完成时返回一个promise。

```
function waitForClose() {
        var self = this;
        return new Promise (function(resolve, reject) {
            self.close(resolve);
        });
    }
```
    *   第2行: 我们将 `this` 保存到一个变量中，这样在内部函数中，我们可以访问该作用域来作为组件。
    *   第3行: 我们创建并返回了一个新的promise。promise的构造函数接收一个函数作为参数。该函数本身有两个函数，这里我们称之为 `resolve` 和 `reject`
    
        *   `resolve` 是一个在调用时将promise置为resolved状态的函数。
        *   `reject` 是一个在调用时将promise置为rejected状态的函数。这里我们不使用reject，可以将其省略。
    
    *   第4行: 我们调用关闭对象的函数。它接收一个函数作为参数（回调）并在resolve已是一个函数时发生，因此可以直接进行传递。进一步说明，我们可以这样写：

```
return new Promise (function (resolve) {
        self.close(function () {
            resolve();
        });
    });
```

*   创建一个 promise生成器 (按序分别调用不同的promise并等待最一个promise)

    假定你需要对数组进行循环，按顺序执行操作并在最后一个操作完成时返回一个 promise。

```
function doStuffOnArray(arr) {
        var done = Promise.resolve();
        arr.forEach(function (item) {
            done = done.then(function () {
                return item.doSomethingAsynchronous();
            });
        });
        return done;
    }
```

    通过这种方式，我所返回的 promise就是最后那个promise。

*   创建一个promise，然后在其定义的作用域外部解析它（反模式）

    我们不推荐使用它，但有时很有用。先仔细思考下如下的替代做法…

```
...
    var resolver, rejecter;
    var prom = new Promise(function (resolve, reject){
        resolver = resolve;
        rejecter = reject;
    });
    ...

    resolver("done"); // will resolve the promise prom with the result "done"
    rejecter("error"); // will reject the promise prom with the reason "error"
```

### 等待Promise

*   等待数个Promise

    如果你有多个需要等待的promise， 可以将它们转化为单个promise，它将在promise所有使用Promise.all(arrayOfPromises)进行解析时解析。

```
var prom1 = doSomethingThatReturnsAPromise();
    var prom2 = Promise.resolve(true);
    var constant = true;

    var all = Promise.all([prom1, prom2, constant]); // all是一个promise
    // 产生一个数组，单个结果对应它们的索引
    // Promise.all()中调用的promise
    all.then(function (results) {
        var prom1Result = results[0];
        var prom2Result = results[1];
        var constantResult = results[2];
    });
    return all;
```

*   等待promise链中的一部分，另一部分不进行等待


    If you have an asynchronous process that you want to wait to do something, but you also want to return to the caller before that something is done.

```
function returnAsSoonAsAsyncProcessIsDone() {
        var prom = AsyncProcess();
        prom.then(function (resultOfAsyncProcess) {
                return doSomething();
        });
        /* returns prom which will only wait for AsyncProcess(),
           and when it will be resolved, the result will be the one of AsyncProcess */
        return prom;
    }
```

### 错误处理

*   in general in promises

    总体思想是promise在控制流中不应失败，仅在出现错误时失败。在这种情况下，promise应当有多个已完成状态，例如在`then`处理器的状态码及promise链最终的单个 `catch` 处理器。

```
function a() {
        x.y();  // <-- this is an error: x is undefined
        return Promise.resolve(1);
    }
    function b() {
       return Promise.reject(2);
    }

    a().catch(console.log);           // 会把错误记录在a中
    a().then(b).catch(console.log);   // 会把错误记录在a中，then不执行
    b().catch(console.log);           // 会记录b失败的原因 (2)
    Promise.resolve(1)
           .then(b)                   // 执行then，它又执行b
           .then(...)                 // 这个then不被执行
           .catch(console.log);       // 会记录b失败的原因 (2)
```


*   Odoo中具体情况

    在Odoo,中，我们对控制流使用promise的失败状态，类似互拆体和`web.concurrency`中定义的其它并发原语。出于*业务原因*我们还希望执行 catch，但不是在promise或处理器的定义中有代码错误时。因此，我们引入 了`guardedCatch`的概念。它的调用类似 `catch` ，但不在失败的原因为错误时

```
function blabla() {
        if (someCondition) {
            return Promise.reject("someCondition is truthy");
        }
        return Promise.resolve();
    }

    // ...

    var promise = blabla();
    promise.then(function (result) { console.log("everything went fine"); })
    // 在blabla调用返回一个rejected的promise时调用，但不在其报错时
    promise.guardedCatch(function (reason) { console.log(reason); });

    // ...

    var anotherPromise =
            blabla().then(function () { console.log("everything went fine"); })
                    // 若blabla返回一个rejected的promise时调用，
                    // 但不在其报错时
                    .guardedCatch(console.log);
```

```
var promiseWithError = Promise.resolve().then(function () {
        x.y();  // <-- this is an error: x is undefined
    });
    promiseWithError.guardedCatch(function (reason) {console.log(reason);}); // 不会被调用
    promiseWithError.catch(function (reason) {console.log(reason);}); // 会被调用
```


### 测试异步代码

*   在测试中使用promises

    在测试代码中，我们支持Javascript的最新版本， 包含`async` 和 `await`和这样的原语。 这样 promise 的使用和等待都非常容易。大部分帮助方法还返回 promise (通过标记为`async` 或通过直接返回一个promise）。

```
var testUtils = require('web.test_utils');
    QUnit.test("My test", async function (assert) {
        // 让函数变为异步有两大好处：
        // 1) 它总是返回一个promise，这样你无需定义`var done = assert.async()`
        // 2) 它允许你使用 `await`
        assert.expect(1);

        var form = await testUtils.createView({ ... });
        await testUtils.form.clickEdit(form);
        await testUtils.form.click('jquery selector');
        assert.containsOnce('jquery selector');
        form.destroy();
    });

    QUnit.test("My test - no async - no done", function (assert) {
        // 这个函数不是异步的，但它返回一个promise。
        // QUnit将等待这个promise完成。 
        assert.expect(1);

        return testUtils.createView({ ... }).then(function (form) {
            return testUtils.form.clickEdit(form).then(function () {
                return testUtils.form.click('jquery selector').then(function () {
                    assert.containsOnce('jquery selector');
                    form.destroy();
                });
            });
        });
    });

    QUnit.test("My test - no async", function (assert) {
        // 这个函数不是异步的且不返回promise。
        // 我们需要使用done函数来向QUnit发出信息表明测试是异步的且会在异步回调中完成
        assert.expect(1);
        var done = assert.async();

        testUtils.createView({ ... }).then(function (form) {
            testUtils.form.clickEdit(form).then(function () {
                testUtils.form.click('jquery selector').then(function () {
                assert.containsOnce('jquery selector');
                form.destroy();
                done();
                });
            });
        });
    });
```

可以目测到，更好的形式是使用 `async/await` ，因其更为清晰、编写更简短。