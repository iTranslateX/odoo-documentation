# 模块翻译

- 本文来自[Odoo 13官方文档之开发者文档](../README.md)系列文章

这部分讲解如何对自己的模块进行翻译。

如果想对Odoo本身进行翻译的藏南，请参见 [Odoo维基页面](https://github.com/odoo/odoo/wiki/Translations)。

## 导出可翻译词汇

你的模块中的词汇数结果是“隐式可翻译”， 即使你没有对翻译进行具体的操作也可以导出模块的可翻译词汇并能查找可操作的内容。

翻译通过登录后台界面并访问Settings ‣ Translations ‣ Import / Export ‣ Export Translations来导出

- 将语言保持为默认 (新语言/空模板)
- 选择[PO File](https://en.wikipedia.org/wiki/Gettext#Translating)格式
- 选择模块
- 点击Export下载文件

[![img](https://www.odoo.com/documentation/13.0/_images/po-export.png)](https://www.odoo.com/documentation/13.0/_images/po-export.png)这会给出一个名为`*yourmodule*.pot` 的文件，应移动到 `*yourmodule*/i18n/` 目录。该文件是一个*PO 模板*，仅列出可翻译字段及通过什么创建实际翻译 (PO 文件)。PO文件可使用[msginit](https://www.gnu.org/software/gettext/manual/gettext.html#Creating)进行创建，有一个类似[POEdit](https://poedit.net/) 专门的翻译工具或者只是将模板拷贝到名为`*language*.po`的新文件。翻译文件应放到`*yourmodule*/i18n/`中，紧邻`*yourmodule*.pot`，并将在安装相应语言时自动（通过Settings ‣ Translations ‣ Languages）由Odoo加载

在安装或更新模块时所有已加载语言的翻译也会进行安装或更新

## 隐式导出

Odoo自动从“data”类型内容导出可翻译字符串：

- 在非QWeb视图中，会导出所有文本节点及`string`, `help`, `sum`, `confirm` 和 `placeholder` 属性的内容

- QWeb模板(服务端及客户端)，导出除 `t-translation="off"`代码块内的所有文本节点， `title`, `alt`, `label` 和 `placeholder`属性的内容也会进行导出

- 对于

  `Field`

  , 除非它们的模型通过 

  ```
  _translate = False
  ```

  进行标记：

  - 其 `string` 和 `help` 属性会进行导出
  - 如出现 `selection` 及列表（或元组），会导出
  - 如若其`translate` 属性设置为 `True`，它们的所有（跨越所有记录的）已存在值会导出

-  `_constraints` 和 `_sql_constraints` 的帮助/错误信息会导出

## 显式导出

在Python或Javascript代码中更“迫切”的场景中， Odoo不能自动导出词，因此它们必须进行显式的标记来导出。这通过在函数调用中封装字面量字符串来实现。

在Python中，封装的函数是`odoo._()`：

```
title = _("Bank Accounts")
```

在JavaScript中，封装的函数通常是 `odoo.web._t()`：

```
title = _t("Bank Accounts");
```

### ⚠️警告

仅有字面量字符串可标记为导出，表达式或变量不行。对于字符串需格式化的场景，这意味着格式字符串而非格式化后的字符串必须进行标记

`_` 和 `_t` 懒翻译版本为python中为 `odoo._lt()` ，在javascript中为 `odoo.web._lt()` 。翻译查找仅在渲染时查找并可用于声明全局变量类方法的可翻译属性。

### 变量

**不要** 提取可能起作用但不会正确地翻译文本：

```
_("Scheduled meeting with %s" % invitee.name)
```

**要** 在翻译查找之外设置动态变量：

```
_("Scheduled meeting with %s") % invitee.name
```

### 代码块

**不要** 在一些代码块或多行中分隔翻译：

```
# 不妥，后缀空格，上下文外的代码块
_("You have ") + len(invoices) + _(" invoices waiting")
_t("You have ") + invoices.length + _t(" invoices waiting");

# 不妥, 多个碎片翻译
_("Reference of the document that generated ") + \
_("this sales order request.")
```

**要** 保持在一个代码块中，将全文本给到翻译器：

```
# 妥, 允许修改在翻译中数量的位置
_("You have %s invoices wainting") % len(invoices)
_.str.sprintf(_t("You have %s invoices wainting"), invoices.length);

# 妥, 全句可理解
_("Reference of the document that generated " + \
  "this sales order request.")
```

### 复数

**不要** 按英文的方式设置词汇的复数：

```
msg = _("You have %s invoice") % invoice_count
if invoice_count > 1:
  msg += _("s")
```

**要** 记住各种语言有不同的复数形式：

```
if invoice_count > 1:
  msg = _("You have %s invoices") % invoice_count
else:
  msg = _("You have %s invoice") % invoice_count
```

### 读取vs运行时间

**不要** 在服务启动时调用翻译查询：

```
ERROR_MESSAGE = {
  # 不妥, 没有用户语言时在服务端启动时运行
  'access_error': _('Access Error'),
  'missing_error': _('Missing Record'),
}

class Record(models.Model):

  def _raise_error(self, code):
    raise UserError(ERROR_MESSAGE[code])
```

**不要** 在读取javascript文件时调用翻译查询：

```
# 不妥, js _t 运行的过早
var core = require('web.core');
var _t = core._t;
var map_title = {
    access_error: _t('Access Error'),
    missing_error: _t('Missing Record'),
};
```

**要** 使用懒翻译查找方法：

```
ERROR_MESSAGE = {
  'access_error': _lt('Access Error'),
  'missing_error': _lt('Missing Record'),
}

class Record(models.Model):

  def _raise_error(self, code):
    # 在错误渲染时执行翻译查找
    raise UserError(ERROR_MESSAGE[code])
```

或 **要** 动态运行可翻译内容：

```
# 妥，在运行时运行
def _get_error_message(self):
  return {
    access_error: _('Access Error'),
    missing_error: _('Missing Record'),
  }
```

**要** 在读取JS文件时翻译查找完成，在使用时用 `_lt` 而非 `_t`来翻译启发式规则：

```
# 妥, js _lt 进行懒运行
var core = require('web.core');
var _lt = core._lt;
var map_title = {
    access_error: _lt('Access Error'),
    missing_error: _lt('Missing Record'),
};
```