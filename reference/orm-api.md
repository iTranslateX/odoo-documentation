# ORM API

本文来自[Odoo 13官方文档之开发者文档](../README.md)系列文章

对象关系模块:

- 等级结构
- 约束一致性及验证
- 对象元数据取决于其状态
- 由复杂查询优化的进程(同时处理多个动作)
- 默认字段值
- 权限优化
- 持久化对象: DB postgresql
- 数据转换
- 多级缓存系统两种不同的继承机制
- 字段类型的丰富集合：
  - 经典类型(varchar, integer, boolean, …)
  - 关联类型 (one2many, many2one, many2many)
  - 函数式类型



## 模型

模型字段在模型本身中以属性来进行定义：

```
from odoo import models, fields
class AModel(models.Model):
    _name = 'a.model.name'

    field1 = fields.Char()
```

### ⚠️警告

这表示你不能使用相同的名称定义字段和方法，后面一个会静默地覆盖掉前面的一个。

默认，字段的标签（用户可见的名称）为字段名的首字母大写，这可通过 `string` 参数来进行重写。

```
field2 = fields.Integer(string="Field Label")
```

字段类型及参数列表可参见[字段指南](#reference-fields)。

默认值是对字段定义的参数，可以是一个值：

```
name = fields.Char(default="a value")
```

也可以调用计算默认值的函数，该函数应返回一个值：

```
def _default_name(self):
    return self.get_value()

name = fields.Char(default=lambda self: self._default_name())
```

### API

###### `*class* odoo.models.BaseModel`

Odoo模型的基类。

Odoo模型通过继承如下之下来进行创建：

- [`Model`](#odoo.models.Model) 用于常规数据库持久化模型
- [`TransientModel`](#odoo.models.TransientModel) 用于临时数据，存储于数据库之中但会不定期地自动清空
- [`AbstractModel`](#odoo.models.AbstractModel) 用于抽象超类，由多个继承模型共享

系统自动对每个数据库进行一次实例化。这些实例代表每个数据库中的可用模型，并取决于该数据库中所安装的模块。每个实例的实际类通过由相应模型创建及继承的Python类所构建。

每个模型实例是都是一个“记录集”，也即一个模型记录的有序集合。记录集由[`browse()`](#odoo.models.Model.browse), [`search()`](#odoo.models.Model.search)这类方法或字段访问所返回。 记录没有显式的体现：一个记录由一条记录的记录集进行体现。

要创建不被实例化的类，可设置 [`_register`](#odoo.models.BaseModel._register) 属性为 False。

###### `_auto *= False*`

用于是否应创建数据表(默认值: `True`)。若设置为 `False`，可重写 `init()` 来创建数据表。

要在创建模型时不创建任何表，可通过继承 [`AbstractModel`](#odoo.models.AbstractModel)。

###### `_table *= None*`

若设置为 [`_auto`](#odoo.models.BaseModel._auto)SQL表名由模型所使用

###### `_sequence *= None*`

用于ID字段的SQL排序

###### `_sql_constraints *= []*`

SQL 约束 [(name, sql_def, message)]

###### `_register *= True*`

在ORM中不可见

###### `_name *= None*`

模型名 (点标记法，模块命名空间)

###### `_description *= None*`

模型的非正式名称

###### `_inherit *= None*`

继承自Python的模型：

类型

str 或 list(str)

- 若设置了[`_name`](#odoo.models.BaseModel._name)，所继承的父级模型的名称
- 若未设置[`_name`](#odoo.models.BaseModel._name) ，该位置所继承单个模型的名称

###### `_inherits *= {}*`

字典 {‘parent_model’: ‘m2o_field’} 映射父级业务对象的 _name到要使用的相应外键字段的名称：

```
_inherits = {
    'a.model': 'a_field_id',
    'b.model': 'b_field_id'
}
```

实现基于组合的继承：新模型暴露所继承模型的所有字段但不对它们进行存储：这些值保持存储在关联的记录中。

### ⚠️警告

如果在 `_inherits`继承的模型中存在多个相同名称的字段，所继承字段会对应（继承列表顺序中的）最后一个。

###### `_rec_name *= None*`

对标签记录所使用的字段，默认值： `name`

###### `_order *= 'id'*`

对搜索结果的默认排序字段

###### `_check_company_auto *= False*`

在写入和创建时，调用`_check_company` 来确保以`check_company=True` 为属性的关联字段的公司一致性。

###### `_parent_name *= 'parent_id'*`

用于父级字段的many2one字段

###### `_parent_store *= False*`

为计算parent_path字段设置为True。

伴随着 [`parent_path`](#odoo.fields.parent_path)字段，设置一个记录树状结构的索引存储，来启用使用`child_of` 及 `parent_of` 域运算符的当前模型记录的更快速等级查询。

###### `_abstract *= True*`

模型是否为抽象

### 参见其它

[`odoo.models.AbstractModel`](#odoo.models.AbstractModel)

###### `_transient *= False*`

模型是否为临时的

### 参见其它

[`odoo.models.TransientModel`](#odoo.models.TransientModel)

###### `_date_name *= 'date'*`

默认日历视图所使用的字段

###### `_fold_name *= 'fold'*`

看板视图中决定折叠分组的字段

### 抽象模型

###### `odoo.models.AbstractModel`

[`odoo.models.BaseModel`](#odoo.models.BaseModel)的别名

### 模型

###### `*class* odoo.models.Model`

常规数据库持久化Odoo模型的主超类。

Odoo模型由通过继承这个类来创建：

```
class user(Model):
    ...
```

系统会我稍后对每个数据库进行一次实例化 (类的模块所安装的数据库)。

### 过渡（临时）模型

###### `*class* odoo.models.TransientModel`

针对临时记录的模型超类，用于临时持久化，会定期进行清空。

TransientModel拥有简化的权限管理，所有的用户可以新建记录，并仅能访问他们自己所创建的记录。超级用户对TransientModel记录的访问不受限。



## 字段

###### `*class* odoo.fields.Field`

字段描述符包含字段定义，并管理权限及对记录中相应字段的分配。以下属性可在初始化字段时进行提供：

参数

- **string** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 用户所能看到的字段标签，如未设置，ORM会接收类（首字母大写）中的字段名。

- **help** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 用户所能看到的字段提示信息

- **readonly** ([`bool`](https://docs.python.org/3/library/functions.html#bool)) –字段是否为只读(默认值： `False`)。仅对 UI 产生影响。代码中的任意字段分配都会起作用(如果字段是一个存储字段且可推导（inversable）).

- **required** ([`bool`](https://docs.python.org/3/library/functions.html#bool)) – 字段值是否为必填 (默认值: `False`)

- **index** ([`bool`](https://docs.python.org/3/library/functions.html#bool)) – 字段在数据库中是否进行了索引。注：对非存储及虚拟字段不产生影响。 (默认值： `False`)

- **default** (`value`` 或 ``callable`) – 字段的默认值；可以是静态值，或接收记录集并返回值的函数；使用 `default=None` 来抛弃字段的默认值。

- states

   (

  `dict`

  ) – 映射状态值到UI属性值对列表的字典；可选的属性有：

  ```
  readonly
  ```

  , 

  ```
  required
  ```

  , 

  ```
  invisible
  ```

  .

  ### ⚠️警告

  任何基于状态的条件都要求有在客户端UI中可用的 `state` 字段值。 这通常通过将其包含在有关联视图中来实现，在与终端用户无关联时可进行隐藏。

- **groups** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 组xml id (字符串)的逗号分隔列表；它限制仅组内的用户才可访问字段。

- **company_dependent** ([`bool`](https://docs.python.org/3/library/functions.html#bool)) –字段值是否依赖于当前公司；值不存储在模型表中。它注册为 `ir.property`。在需要用到company_dependent字段的值时，会搜索关联当前公司（在属性存在的话搜索当前记录）的`ir.property` 。如果值在记录中发生变化，这要么会修改当前记录（若存在）中的已有属性，.要么为当前公司及res_id新建一个属性。如果在公司端发生了值的改变，它会影响值未生改变的所有记录。

- **copy** ([`bool`](https://docs.python.org/3/library/functions.html#bool)) – 在复制记录时是否应拷贝该字段值(默认: 普通字段为`True`， `one2many` 及计算字段，包含属性字段及关联字段，为`False`)

- **store** ([`bool`](https://docs.python.org/3/library/functions.html#bool)) – 字段是否存储在数据库中 (默认:`True`, 对于计算字段为`False`)

### 计算字段

参数

- compute

   (

  `str`

  ) –计算该字段的方法名

  ### 参见其它

  [高级字段/计算字段](#reference-fields-compute)

- **compute_sudo** ([`bool`](https://docs.python.org/3/library/functions.html#bool)) – 字段是否应超越权限以超级用户进行重新计算 (对于存储字段默认为 `True`，对于非存储字段默认为 `False` )

- **inverse** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 推导字段的方法名（可选参数）

- **search** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 实现对字段搜索的方法名 (可选参数)

- related

   (

  `str`

  ) – 字段名的序列

  ### 参见其它

  [高级字段/关联字段](#reference-fields-related)



### 基本字段

###### `*class* odoo.fields.Boolean`

###### `*class* odoo.fields.Char`

基本字符串字段，可限制长度，通常在客户端以单行字符串进行显示。

参数

- **size** ([`int`](https://docs.python.org/3/library/functions.html#int)) – 针对该字段所存储值的最大的大小
- **trim** ([`bool`](https://docs.python.org/3/library/functions.html#bool)) – 描述值是否进行去两边空格的操作 (默认值为 `True`)。注意 trim 操作仅由网页客户端所应用。
- **translate** ([`bool`](https://docs.python.org/3/library/functions.html#bool)` or ``callable`) – 启用对字段值的翻译；使用 `translate=True` 来整体翻译字段值；`translate` 也可由 `translate(callback, value)` 等可调用方法所调用，它通过使用 `callback(term)`来获取词汇的翻译来进行`value`的翻译。

###### `*class* odoo.fields.Float`

属性所给定的精确位数

参数

**digits** ([`tuple`](https://docs.python.org/3/library/stdtypes.html#tuple)`(`[`int`](https://docs.python.org/3/library/functions.html#int)`,`[`int`](https://docs.python.org/3/library/functions.html#int)`) or `[`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 一个(总数, 小数位)对或 或引用 `decimal.precision` 记录的字符串。

###### `*class* odoo.fields.Integer`



### 高级字段

###### `*class* odoo.fields.Html`

###### `*class* odoo.fields.Monetary`

从该字段所接收的小数精度及货币符号

参数

**currency_field** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 存储用于表述这一倾向字段的币种的字段名 (默认为: `'currency_id'`)

###### `*class* odoo.fields.Selection`

参数

- **selection** ([`list`](https://docs.python.org/3/library/stdtypes.html#list)`(`[`tuple`](https://docs.python.org/3/library/stdtypes.html#tuple)`(`[`str`](https://docs.python.org/3/library/stdtypes.html#str)`,`[`str`](https://docs.python.org/3/library/stdtypes.html#str)`)``) 或 ``callable`` 或 `[`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 指定这个字段可能的值。它要么给定一个 `(value, label)`对的列表，要么是一个模型方式或方法名。

- selection_add

   (

  `list`

  ```
  (
  ```

  `tuple`

  ```
  (
  ```

  `str`

  ```
  ,
  ```

  `str`

  ```
  )
  ```

  ```
  )
  ```

  ) –在重载字段的情况下提供对选项的一个扩展。是一个 

  ```
  (value, label)
  ```

   对的列表或单值元组 

  ```
  (value,)
  ```

  ，其中的值必须要出现在重载选项中。新值插入的顺序与重载选项和这个列表是一致的：

  ```
  selection = [('a', 'A'), ('b', 'B')]
  selection_add = [('c', 'C'), ('b',)]
  > result = [('a', 'A'), ('c', 'C'), ('b', 'B')]
  ```

属性`selection` 在`related` 或扩展字段以外的情况下是都是必须的。

###### `*class* odoo.fields.Text`

非常类似于 [`Char`](#odoo.fields.Char) 但用于更长的内容，不带有大小，但通常显示为一个多行文本框。

参数

**translate** ([`bool`](https://docs.python.org/3/library/functions.html#bool)` 或 ``callable`) – 启用对字段值的翻译；使用`translate=True` 来整体翻译字段值；`translate` 也可以为`translate(callback, value)` 这样的可调用方法，它通过使用`callback(term)` 来获取词汇的翻译以翻译`value`。



#### 日期（时间）字段

Dates和Datetime是任何类型业务应用中都非常重要的字段，它们在常用的Odoo应用如物流或会计中都有被重度的使用，并且误用会导致不易察觉且有会很让人痛苦的bug，本部分旨在向 Odoo开发者提供用于避免误用这些字段的知识。

在将一个值赋给Date/Datetime字段时，有如下有效选项：

- `date`或`datetime`对象。
- 针对各字段一个适当的服务端格式字符串，Date字段为 *(YYYY-MM-DD)* ，Datetime字段为*(YYYY-MM-DD HH:MM:SS)*。
- `False` 或 `None`.

Date和Datetime字段类有帮助方法尝试将值转化为兼容的类型： [`to_date()`](#odoo.fields.Date.to_date) 将会转化为一个`datetime.date` 对象，而[`to_datetime()`](#odoo.fields.Datetime.to_datetime) 将转化为 `datetime.datetime`.

### 示例

要解析来自外部的日期/日期时间：

```
fields.Date.to_date(self._context.get('date_from'))
```

Date / Datetime 比较的最佳实践：

- Date字段**仅**能与date对象进行比较。
- Datetime字段**仅**能与datetime对象进行比较。

### ⚠️警告

使用字符串表示的日期和日期时间之间可以进行比较，但结果可能会在预料之外，因为日期时间字符串总是比日期字符串大，因此这种**严重**不鼓励这类操作。

日期及日期时间的常用操作，如加、减或获取一段时间的开始/结束通过 [`Date`](#odoo.fields.Date) 和 [`Datetime`](#odoo.fields.Datetime)对外暴露。这些帮助类也可以通过导入 `odoo.tools.date_utils`来进行使用。

时区

Datetime字段存储于数据库中的`无时区时间戳` 列中，存储的时区为 UTC。这是设计使然，因为这会让Odoo数据库独立于托管服务系统的时区。时间转换完全由客户端进行管理。

###### `*class* odoo.fields.Date`

这个字段类型封装了Python的date对象：date类型:

###### `*static* add(**args*, ***kwargs*)`

返回 `value` 和 `relativedelta`的加总。

参数

- **value** – 初始日期或日期时间。
- **args** – 直接传递给`relativedelta`的位置参数。
- **kwargs** – 直接传递给`relativedelta`的关键字参数。

返回

日期/日期时间的结果。

###### `*static* context_today(*timestamp=None*)`

返回当前日期，它适配了客户端时区的date 字段格式。

这个方法可用于计算默认值。

参数

- **record** – 获取时区的记录集。
- **timestamp** (`datetime`) – 可选的日期时间值，代替当前日期和时间(必须为datetime，普通日期无法在时区间进行转换)。

返回类型

date

###### `*static* end_of(*granularity*)`

从date或datetime中获取一段时间的结束时间。

参数

- **value** – 初始日期或日期时间。
- **granularity** – 字符串形式的时间段类型，可以是年、季、月、周、日或小时。

返回

对应于一个具体时间段的date/datetime对象。

###### `*static* start_of(*granularity*)`

从date或datetime获取一个时间段的起始。

参数

- **value** – 初始日期或日期时间。
- **granularity** – 字符串形式的时间段类型，可以是年、季、月、周、日或小时。

返回

对应于一个具体时间段起始的date/datetime对象。

###### `*static* subtract(**args*, ***kwargs*)`

返回`value`和`relativedelta`之间差。

参数

- **value** – 初始日期或日期时间。
- **args** – 直接传递给`relativedelta`的位置参数。
- **kwargs** – 直接传递给`relativedelta`的关键字参数。

返回

结果date/datetime。

###### `*static* to_date()`

尝试转化 `value` 为一个 `date` 对象。

### ⚠️警告

如果datetime对象给定为一个值，它将转换为一个date对象并且所有的特定datetime信息会丢失(HMS, TZ, …)。

参数

**value** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)` 或 ``date`` 或 ``datetime`) – 待转换的值。

返回

一个表示 `value`的对象。

返回类型

date 或 None

###### `*static* to_string()`

转换`date` 或 `datetime` 对象为字符串。

参数

**value** – 待转换的值。

返回

一个以服务端日期格式所表示的 `value`字符串，如果`value` 是一种`datetime`类型，小时、分钟、秒、时区信息都会被截取掉。

返回类型

[str](https://docs.python.org/3/library/stdtypes.html#str)

###### `*static* today()`

返回ORM所需格式的当天。

这个函数可用于计算默认值。

###### `*class* odoo.fields.Datetime`

这个字段类型封装了python的datetime对象。 :type datetime:

###### `*static* add(**args*, ***kwargs*)`

返回`value`和`relativedelta`的加总。

参数

- **value** – 初始日期或日期时间。
- **args** – 直接传递给 `relativedelta`的位置参数。
- **kwargs** – 直接传递给 `relativedelta`的关键字参数。

返回

返回结果date/datetime。

###### `*static* context_timestamp(*timestamp*)`

将给定时间戳转换为客户端时区返回。

这个方法*不*用作默认初始化器，因为datetime字段会在客户端显示时自动进行转换。设置默认值应使用 [`now()`](#odoo.fields.Datetime.now) 来进行代替。

参数

- **record** – 用于获取时区的记录集。
- **timestamp** (`datetime`) – 用于转换为客户端时区的原生datetime值(通过UTC表示) 。

返回

在上下文时区中将时间戳转换为相应时区的日期时间。

返回类型

datetime

###### `*static* end_of(*granularity*)`

从日期或日期时间中获取一个时间段的结尾。

参数

- **value** – 初始日期或日期时间。
- **granularity** – 字符串形式的时间段类型，可以是年、季、月、周、日或小时。

返回

对应给定时间段起始的date/datetime对象。

###### `*static* now()`

按ORM所需格式返回当前日期和时间。

该函数可用于计算默认值。

###### `*static* start_of(*granularity*)`

通过date或datetime获取时间段的起始。

参数

- **value** – 初始日期或日期时间。
- **granularity** – 字符串形式的时间段类型，可以是年、季、月、周、日或小时。

返回

对应给定时间段起始的date/datetime对象。

###### `*static* subtract(**args*, ***kwargs*)`

返回`value` 和 `relativedelta`之间的差。

参数

- **value** – 初始日期或日期时间。
- **args** – 直接传递给`relativedelta`的位置参数。
- **kwargs** – 直接传递给`relativedelta`的关键字参数。

Returns返回

结果日期/日期时间。

###### `*static* to_datetime()`

转换ORM `value` 为一个 `datetime` 值。

参数

**value** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)` 或 ``date`` 或 ``datetime`) – 待转换的值。

Returns返回

表示`value`的对象。

返回类型

datetime 或 None

###### `*static* to_string()`

转换`datetime` 或 `date` 对象为一个字符串。

参数

**value** (`datetime`` or ``date`) – 待转换的值。

返回

以服务端日期时间格式所表示的 `value` 字符串， 如果 `value` 是一种 `date`类型，时间部分将会是午夜 (00:00:00)。

返回类型

[str](https://docs.python.org/3/library/stdtypes.html#str)

###### `*static* today()`

返回当天午夜时间 (00:00:00)。



#### 关联字段

###### `*class* odoo.fields.Many2one`

这一字段的值是一个大小为0(无记录) 或 1 (单条记录)的记录集。

参数

- **comodel_name** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 目标模型`Mandatory` 除关联或扩展字段以外的名称。
- **domain** – 在客户端（域或字符串）设置备用值的可选域
- **context** ([`dict`](https://docs.python.org/3/library/stdtypes.html#dict)) – 在处理该字段时用于客户端的可选上下文
- **ondelete** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 在引用的记录被删除时所做的操作：可用的值下有： `'set null'`, `'restrict'`, `'cascade'`
- **auto_join** ([`bool`](https://docs.python.org/3/library/functions.html#bool)) – 对该字段的搜索是否有JOIN生成 (默认值：`False`)
- **delegate** ([`bool`](https://docs.python.org/3/library/functions.html#bool)) – 设置为`True` 来让你目标模型的字段可通过当前模型进行访问(对应 `_inherits`)
- **check_company** – 添加默认作用域`['|', ('company_id', '=', False), ('company_id', '=', company_id)]`。 标记该字段在 `_check_company`中进行验证。

###### `*class* odoo.fields.One2many`

One2many字段；该字段的值是 `comodel_name` 中所有记录的记录集，这样`inverse_name`字段等于当前记录。

参数

- **comodel_name** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 目标模型的名称
- **inverse_name** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) –  `comodel_name`中`Many2one` 字段的反向名称
- **domain** – 在客户端（或或字符串）设置备用值的可选域
- **context** ([`dict`](https://docs.python.org/3/library/stdtypes.html#dict)) – 在处理该字段时用于客户端的可选上下文
- **auto_join** ([`bool`](https://docs.python.org/3/library/functions.html#bool)) – 是否在搜索该字段时生成 JOIN语句 (默认值: `False`)
- **limit** ([`int`](https://docs.python.org/3/library/functions.html#int)) – 在读取时的可选限制

属性 `comodel_name` 和 `inverse_name` 在关联字段及字段继承以外的情况下都是必填的。

###### `*class* odoo.fields.Many2many`

Many2many字段，这类字段的值是记录集。

参数

- **comodel_name** – 在关联字段及字段继承以外的情况下必填的目标目标模型（字符串）名
- **relation** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 数据库中存储关联的可选数据表名称
- **column1** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 在 `relation`数据表中引用“这些”记录的可选列名
- **column2** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 在 `relation`数据表中引用“那些”记录的可选列名

属性 `relation`, `column1` 和 `column2` 均为可选。若未指定，会通过模型名自动生成，如 `model_name` 和 `comodel_name` 是不同的名称。

注意ORM中不接受给定模型中使用多个隐式关联参数字段关联相同comodel，因为那些字段使用同一张表。ORM提供两个many2many 字段来使用相同的关联参数，除以下情况：

- 两个字段使用相同的model, comodel，并且关联参数是显式的；或者
- 至少有一个字段属于带有 `_auto = False`的模型。

参数

- **domain** – 对客户端（域或字符串）设置备选值的可选域
- **context** ([`dict`](https://docs.python.org/3/library/stdtypes.html#dict)) – 在处理该字段时对客户端使用的可选上下文
- **check_company** – 添加默认域`['|', ('company_id', '=', False), ('company_id', '=', company_id)]`。标识该字段在 `_check_company`中进行校验。
- **limit** ([`int`](https://docs.python.org/3/library/functions.html#int)) – 用于读取的可选限制

#### 伪关联字段

###### `*class* odoo.fields.Reference`

###### `*class* odoo.fields.Many2oneReference`



#### 计算字段

可使用 `compute` 参数来计算（除直接从数据库中读取外）字段。**必须对该字段分配计算的值。**如果使用其它*字段*值，应当指定这些字段使用 [`depends()`](#odoo.api.depends)。

```
from odoo import api
total = fields.Float(compute='_compute_total')

@api.depends('value', 'tax')
def _compute_total(self):
    for record in self:
        record.total = record.value + record.value * record.tax
```

- 在使用子字段时依赖可以为带点的路径：

  ```
  @api.depends('line_ids.value')
  def _compute_total(self):
      for record in self:
          record.total = sum(line.value for line in record.line_ids)
  ```

- 计算字段默认不存储，而是在请求时进行计算并返回。设置 `store=True` 会将它们存储到数据库中并自动启用搜索。

- 对计算字段的搜索也可通过设置

  ```
  search
  ```

   参数来进行启用。值为返回

  搜索域

  的一个方法名。

  ```
  upper_name = field.Char(compute='_compute_upper', search='_search_upper')
  
  def _search_upper(self, operator, value):
      if operator == 'like':
          operator = 'ilike'
      return [('name', operator, value)]
  ```

  在对模型进行实际搜索前处理域时会调用搜索方法。它必须返回等价于 `域运算符值`条件的域。

- 计算字段默认为只读。要允许对计算字段设置值，应使用 

  ```
  inverse
  ```

   参数。它是一个反向计算及设置关联字段的函数名：

  ```
  document = fields.Char(compute='_get_document', inverse='_set_document')
  
  def _get_document(self):
      for record in self:
          with open(record.get_document_path) as f:
              record.document = f.read()
  def _set_document(self):
      for record in self:
          if not record.document: continue
          with open(record.get_document_path()) as f:
              f.write(record.document)
  ```

- 同一方法可在同时计算多个字段，只需对所有字段使用同一方法并对它们进行设置：

  ```
  discount_value = fields.Float(compute='_apply_discount')
  total = fields.Float(compute='_apply_discount')
  
  @api.depends('value', 'discount')
  def _apply_discount(self):
      for record in self:
          # compute actual discount from discount percentage
          discount = record.value * record.discount
          record.discount_value = discount
          record.total = record.value - discount
  ```



#### 关联字段

计算字段的一种特殊情况是*关联*(代理)字段，它提供当前记录的子字段值。它们通过设置`related` 参数进行定义并可以像普通字段那样进行存储：

```
nickname = fields.Char(related='user_id.partner_id.name', store=True)
```

关联字段的值通过遵循关联字段序列及读取所触达模型的字段来给定。待遍历的完整字段序列由 `related` 属性进行指定。

有些字段属性在未重新定义时从源字段中进行拷贝： `string`, `help`, `readonly`, `required` (仅在要求序列中所有字段时使用), `groups`, `digits`, `size`, `translate`, `sanitize`, `selection`, `comodel_name`, `domain`, `context`。所有无语名的属性都从源字段中拷贝。

默认，关联字段的值不在数据库中存储。和计算字段类似，添加 `store=True` 属性来对其进行存储。关联字段在修改依赖时会自动重新计算。

关联字段在sudo模式下进行计算。



### 自动化字段

###### `odoo.fields.id`

标识符 [`字段`](#odoo.fields.Field)

如果当前记录集的长度为1, 在其中返回唯一记录id。

否则抛出Error。

###### `odoo.fields.create_date`

[`Datetime`](#odoo.fields.Datetime)

###### `odoo.fields.create_uid`

[`Many2one`](#odoo.fields.Many2one)

###### `odoo.fields.write_date`

[`Datetime`](#odoo.fields.Datetime)

###### `odoo.fields.write_uid`

[`Many2one`](#odoo.fields.Many2one)



### 保留字段名

有些字段是为自动化字段之上的预定义行为所预留的。应当在要求关联行为时在模型中进行定义：

###### `odoo.fields.name`

[`_rec_name`](#odoo.models.BaseModel._rec_name)的默认值，用于在需要表述型“naming”时在上下文中显示记录。

[`Char`](#odoo.fields.Char)

###### `odoo.fields.active`

切换记录的全局可见性，如果 `active` 设置为`False`，在大部分搜索和列举中该记录不可见。

[`Boolean`](#odoo.fields.Boolean)

###### `odoo.fields.state`

对象的生命周期阶段，由[`字段`](#odoo.fields.Field)的`states`属性所使用。

[`Selection`](#odoo.fields.Selection)

###### `odoo.fields.parent_id`

[`_parent_name`](#odoo.models.BaseModel._parent_name)的默认值，用于在树状结构中组织记录并在域中启用`child_of` 及 `parent_of` 运算符。

[`Many2one`](#odoo.fields.Many2one)

###### `odoo.fields.parent_path`

在[`_parent_store`](#odoo.models.BaseModel._parent_store) 设置为True时，用于存储反映[`_parent_name`](#odoo.models.BaseModel._parent_name)树状结构的值，并在搜索域中优化 `child_of` 和 `parent_of` 运算符。应对相应运算添加 `index=True` 进行声明。

[`Char`](#odoo.fields.Char)

## 记录集

与模型和记录之间的交互通过记录集执行，它是一个相同模型记录的已排序集合。

> ⚠️**警告**
>
> 和名称字面含义不同，记录集中当前是可以包含重复值的。未来可能会发生改变。

模型中定义的方法对记录集执行，并且它们的`self`是一个记录集：

```
class AModel(models.Model):
    _name = 'a.model'
    def a_method(self):
        # self can be anything between 0 records and all records in the
        # database
        self.do_operation()
```

对记录集的遍历会产生*单*记录（singletons）的新集合，很像遍历Python中单个字符的string字段字符串:

```
def do_operation(self):
    print(self) # => a.model(1, 2, 3, 4, 5)
    for record in self:
        print(record) # => a.model(1), then a.model(2), then a.model(3), ...
```

### 段访问

记录集提供一个“Active Record”接口：模型字段可直接从记录中读取为属性。

在访问潜在多记录的记录集中的非关联字段时，使用`mapped()`：

```
total_qty = sum(self.mapped('qty'))
```

字段值也可像字典项那样进行访问， 对于动态字段名这样会比`getattr()`更为优雅、更为安全。 设置字段值会触发数据库中的更新：

```
>>> record.name
Example Name
>>> record.company_id.name
Company Name
>>> record.name = "Bob"
>>> field = "name"
>>> record[field]
Bob
```

> ⚠️**警告**
>
> 尝试读取多条记录的字段会对非关联字段抛出错误。

访问关联字段（[`Many2one`](#odoo.fields.Many2one), [`One2many`](#odoo.fields.One2many), [`Many2many`](#odoo.fields.Many2many)）*总是*会返回记录集，未进行设置时会返回空。

### 记录缓存和预提取

Odoo对记录值维护一个缓存，这样不是每个字段访问都会进行数据库请求，那样会影响性能。以下示例在第一条语句时仅查询数据库：

```
record.name             # first access reads value from database
record.name             # second access gets value from cache
```

为避免一次只读取一条记录的一个字段，Odoo按照获取良好性能的启发预抓取记录和字段。一旦需要对给定记录进行读取，ORM会实际对更大记录集读取该字段，并返回值给缓存中存储供稍后使用。预抓取的记录集通常是来自遍历的记录。此外所有的简单存储字段 (boolean, integer, float, char, text, date, datetime, selection, many2one) 都同时进行获取；它们分别对应模型数据表中的字段，并在同一条查询中进行有效获取。

看下如下示例，其中 `partners` 是一个拥有1000条记录的记录集。不使用预提取的话，这个循环会对数据库进行 2000 次查询。而使用预提取时仅会做一次查询：

```
for partner in partners:
    print partner.name          # first pass prefetches 'name' and 'lang'
                                # (and other fields) on all 'partners'
    print partner.lang
```

预提取也可用于*二级记录*； 在读取关联字段时，它们的值（即记录）用于未来的预提取。访问这些二级记录中的记录会提取同一模型中的所有二级记录。这样下例中仅会生成两条查询，分别用于partners 和countries:

```
countries = set()
for partner in partners:
    country = partner.country_id        # first pass prefetches all partners
    countries.add(country.name)         # first pass prefetches all countries
```



## 方法装饰器

Odoo API模块定义Odoo环境及方法装饰器。

###### `odoo.api.depends(**args*)`

返回一个指定对“compute”方法（针对新模式函数字段）依赖字段的装饰器。每个参数必须为一个包含点号分隔的字段名序列的字符串：

```
pname = fields.Char(compute='_compute_pname')

@api.depends('partner_id.name', 'partner_id.is_company')
def _compute_pname(self):
    for record in self:
        if record.partner_id.is_company:
            record.pname = (record.partner_id.name or "").upper()
        else:
            record.pname = record.partner_id.name
```

可以传递单个函数来作为参数。这时，依然通过对字段模型的函数调用来给定。

###### `odoo.api.depends_context(**args*)`

返回指定非存储“compute”方法的上下文依赖装饰器。每个参数是上下文字典中的键：

```
price = fields.Float(compute='_compute_product_price')

@api.depends_context('pricelist')
def _compute_product_price(self):
    for product in self:
        if product.env.context.get('pricelist'):
            pricelist = self.env['product.pricelist'].browse(product.env.context['pricelist'])
        else:
            pricelist = self.env['product.pricelist'].get_default_pricelist()
        product.price = pricelist.get_products_price(product).get(product.id, 0.0)
```

所有依赖必须为可哈希的。如下键具有特别的支持：

- `force_company` (上下文中的值或当前公司 id)，
- `uid` (当前用户 id 及超级用户标记)，
- `active_test` (env.context 中的值或field.context中的值)。

###### `odoo.api.constrains(**args*)`

装饰约束检查器。

每个参数必须为用于检查中的字段名：

```
@api.constrains('name', 'description')
def _check_description(self):
    for record in self:
        if record.name == record.description:
            raise ValidationError("Fields name and description must be different")
```

对命名字段进行修改的记录进行调用。

在验证失败时应该会抛出 [`ValidationError`](#odoo.exceptions.ValidationError) 。

### ⚠️警告

`@constrains` 仅支持简单字段名，不支持或忽略 点号字符(关联字段如 `partner_id.customer`) are not supported and will be ignored.

`@constrains` 将仅在装饰方法中声明字段包含 `create` 或 `write` 调用时触发。要保证约束保持触发则需要重载`create` (比如测试值的缺失)。

###### `odoo.api.onchange(**args*)`

返回装饰给定字段onchange方法的装饰器。

在出现字段的表单视图中，会在给定字段进行修改时进行调用。该方法调用对象是一个表单中值所组成的伪记录。对该记录的字段赋值会被发送回客户端。

每个参数必须为一个字段名：

```
@api.onchange('partner_id')
def _onchange_partner(self):
    self.message = "Dear %s" % (self.partner_id.name or "")
return {
    'domain': {'other_id': [('partner_id', '=', partner_id)]},
    'warning': {'title': "Warning", 'message': "What is this?", 'type': 'notification'},
}
```

如果类型设置为notification，警告会在通知中显示。否则会默认在对话框中显示。

> **⚠️警告**
>
> `@onchange` 仅支持简单字段名，不支持并会忽略点号名称 (关联字段的字段，如`partner_id.tz`)。

> **🚫危险**
>
> 因 `@onchange`返回的一个伪记录的记录集， 对前述记录集进行 CRUD (`create()`, `read()`, `write()`, `unlink()`) 方法的调用是一种未定义行为，因为它们可能不存在于数据库中。
>
> 反而，简单地像上例那样设置记录字段或调用 `update()` 方法。

### ⚠️警告

`one2many` 或 `many2many` 字段无法通过onchange来自我修改。这是一种网页客户端的局限 - 参见 [#2693](https://github.com/odoo/odoo/issues/2693)。

###### `odoo.api.returns(*model*, *downgrade=None*, *upgrade=None*)`

对返回`model`实例的方法返回一个装饰器。

参数

- **model** – 模型名，或针对当前模型的`'self'`
- **downgrade** – 一个 `downgrade(self, value, *args, **kwargs)` 函数，用于将记录样式的 `value` 转换为传统样式的输出
- **upgrade** – 一个`upgrade(self, value, *args, **kwargs)` 函数，用于将传统样式的 `value` 转换为记录样式输出

参数 `self`, `*args` 和 `**kwargs` 是以记录样式传递给方法的参数。

装饰器适配方法输出为api样式：传统样式为`id`, `ids` 或 `False` ，记录样式为记录集：

```
@model
@returns('res.partner')
def find_partner(self, arg):
    ...     # return some record

# output depends on call style: traditional vs record style
partner_id = model.find_partner(cr, uid, arg, context=context)

# recs = model.browse(cr, uid, ids, context)
partner_record = recs.find_partner(arg)
```

注意被装饰的方法必须满足这一规则。

这些装饰器自动*被继承：* 重载被装饰的已有方法的方法会会相同的 `@returns(model)`所装饰。

###### `odoo.api.model_create_multi(*method*)`

装饰一个接收字典列表并创建多条记录的方法。该方法可能会通过单个dict或一个字典列表所调用：

```
record = model.create(vals)
records = model.create([vals, ...])
```



## Environment

`Environment` 存储各种由 ORM 所使用的上下文数据：数据库游标(用于数据库查询)、当前用户 (用于访问权限检查) 及当前上下文 (存储任意metadata)。环境中还存储缓存。

所有的记录集都有一个环境，它是不可变的，可使用`env` 进行访问并授权给：

- 当前用户([`user`](#odoo.api.Environment.user))
- 游标 (`cr`)
- 超级用户标记 (`su`)
- 或上下文(`context`)

```
>>> records.env
<Environment object ...>
>>> records.env.user
res.user(3)
>>> records.env.cr
<Cursor object ...)
```

在通过其它记录集创建记录集时，会继承其环境。environment可用于在其它模型中获取空记录集，并查询该模型：

```
>>> self.env['res.partner']
res.partner()
>>> self.env['res.partner'].search([['is_company', '=', True], ['customer', '=', True]])
res.partner(7, 18, 12, 14, 17, 19, 8, 31, 26, 16, 13, 20, 30, 22, 29, 15, 23, 28, 74)
```

###### `Environment.ref(*xml_id*, *raise_if_not_found=True*)`

返回响应指定`xml_id`的记录。

###### `Environment.lang`

返回当前语言代码。

返回类型

[str](https://docs.python.org/3/library/stdtypes.html#str)

###### `Environment.user`

（以实例）返回当前用户。

返回类型

```
res_users
```

###### `Environment.company`

（以实例）返回当前公司。

如未在上下文 (`allowed_company_ids`)中指定， 会使用当前用户的主公司。

抛出

[**AccessError**](#odoo.exceptions.AccessError) – 无效或未授权的 `allowed_company_ids` 上下文键内容。

返回

当前公司 (default=`self.user.company_id`)

返回类型

res.company

> ⚠️**警告**
>
> 在sudo模式下不会进行安全检查！ sudo模式下，用户可以访问任意公司，即使是不允许该用户访问的公司。
>
> 这会导致当前用户即使不对目标公司拥有权限也能触发跨公司修改。

###### `Environment.companies`

返回由用户所启用公司的记录集。

如未在上下文中(`allowed_company_ids`)指定，缺省为当前用户的公司。

抛出

[**AccessError**](#odoo.exceptions.AccessError) – 无效或未授权的 `allowed_company_ids` 上下文键内容。

返回

当前公司 (default=`self.user.company_ids`)

返回类型

res.company

> ⚠️**警告**
>
> 在sudo模式下不会进行安全检查！ sudo模式下，用户可以访问任意公司，即使是不允许该用户访问的公司。
>
> 这会导致当前用户即使不对目标公司拥有权限也能触发跨公司修改。

### 更改environment

###### `Model.with_context(*[context][, **overrides]*) → records`

返回关联到扩展上下文的该记录的新版本。

扩展上下文由`overrides`合并后的 `上下文`所提供，也可由`overrides`合并的*当前*上下文所提供：

```
# current context is {'key1': True}
r2 = records.with_context({}, key2=True)
# -> r2._context is {'key2': True}
r2 = records.with_context(key2=True)
# -> r2._context is {'key1': True, 'key2': True}
```

###### `Model.with_user(*user*)`

在非超级用户模式下返回关联到给定用户的该记录的新版本，除非`user` 为超级用户 (按照规则 ，超级用户总是使用超级用户模式)。

###### `Model.with_env(*env*)`

返回关联到所提供环境的该记录的新版本。

参数

**env** (`Environment`) –

> ⚠️**警告**
>
> 新环境不会从当前环境的数据缓存中获益，因此稍后的数据访问可能会在重新从数据库中提取数据时产生额外延时。返回的数据集应为与 `self`相同的预提取对象。

###### `Model.sudo([*flag=True*])`

根据`flag`的值启动或关闭超级用户模式下返回该记录集的新版本。超级用户模式不会修改当前用户，它只是跳过权限检查。

> ⚠️**警告**
>
> 使用`sudo` 会导致跨越记录规则边界的数据访问，可能会把本应分离的记录进行混合 (例如在多公司环境中来自不同公司的记录)。
>
> 可能在从多条记录中选取一条的方法中会产生直觉以外的结果 -例如获取默认公司，或选择物料清单。

> 因为记录规则和访问控制会进行重新运行，新记录集不会从当前的环境数据缓存中获取好处，因此稍后的数据访问规则可能由从数据库中重新提取而导致额外的延时。返回的记录与`self`有同样的预提取对象。



### SQL执行

环境中的 `cr` 属性是当前数据库事务的游标，允许直接执行SQL，可能是难于使用 ORM 表达的查询 (如复杂的 join)或出于性能原因：

```
self.env.cr.execute("some_sql", param1, param2, param3)
```

因为名模型使用同一个游标并且 `Environment` 存储各种缓存，这些缓存必须在数据库原生 SQL *修改*数据库时置为无效，否则模型的进一步使用可能会产生不一致性。有必要在SQL使用`CREATE`, `UPDATE` 或 `DELETE` 时清除缓存，但无需对 `SELECT`进行此操作 (它只是读取了数据库)。

可执行 [`invalidate_cache()`](#odoo.models.Model.invalidate_cache) 方法清除缓存。

###### `Model.invalidate_cache(*fnames=None*, *ids=None*)`

在修改了某些记录后将其缓存置为无效。如果 `fnames` 和 `ids` 均为 `None`，会清除整个缓存。

参数

- **fnames** – 所修改字段列表， `None` 表示所有字段
- **ids** – 所修改记录 id列表， `None` 表示所有

> ⚠️**警告**
>
> 执行原生SQL会跳过ORM，结果也会跳过Odoo安全规则。请确保在使用用户输入时语句进行过检测，如果不是真的需要使用SQL语句则推荐使用ORM功能。



## 常用ORM方法

### Create/update

###### `Model.create(*vals_list*) → records`

为模型创建新记录。

新记录使用`vals_list`字典列表中的值进行初始化，必要时通过 [`default_get()`](#odoo.models.Model.default_get)。

参数

**vals_list** ([`list`](https://docs.python.org/3/library/stdtypes.html#list)) –模型字段的值，字典列表的形式：

```
[{'field_name': field_value, ...}, ...]
```

为保持向后兼容， `vals_list`可以为字典。它被看作是单值列表t `[vals]`，并返回单条记录。

参见 [`write()`](#odoo.models.Model.write) 了解更多详情

返回

所创建记录

抛出

- **AccessError**

   –

  - 若用户对所请求对象没有创建权限
  - 若用户尝试跳过权限规则对所请求对象进行创建

- [**ValidationError**](#odoo.exceptions.ValidationError) –若用户对字段输入不在选项内的无效值

- [**UserError**](#odoo.exceptions.UserError) – 若在对象等级的操作结果中创建了循环 (比如设置对象为其自身的父级)

###### `Model.copy(*default=None*)`

复制记录 `self` 使用默认值进行更新

参数

**default** ([`dict`](https://docs.python.org/3/library/stdtypes.html#dict)) – 在所复制记录的原始值中重载字段值字典，例如： `{'field_name': overridden_value, ...}`

返回

新记录

###### `Model.default_get(*fields*) → default_values`

为 `fields_list`.中的字段返回默认值。 默认值由上下文、用户缺省及模型本身所决定。

参数

**fields_list** – 字段名列表

返回

如果存在，映射每个字段名到其相应的默认值的字典。

###### `Model.name_create(*name*) → record`

使用所提供值通过调用 [`create()`](#odoo.models.Model.create) 新建一条记录：新记录的显示名。

新记录会通过应用于该模型的默认值或通过上下文所提供的值进行初始化。[`create()`](#odoo.models.Model.create)的常规行为同样适用。

参数

**name** – 待创建记录的显示名

返回类型

[tuple](https://docs.python.org/3/library/stdtypes.html#tuple)

返回

所创建记录的 [`name_get()`](#odoo.models.Model.name_get)对值

###### `Model.write(*vals*)`

通过所提供值更新当前集合中的所有记录。

参数

**vals** ([`dict`](https://docs.python.org/3/library/stdtypes.html#dict)) –要更新的字段及所要设置的值，如：

```
{'foo': 1, 'bar': "Qux"}
```

如果有效会设置 `foo` 为 `1` ，并将字段 `bar` 设为 `"Qux"` (否则会触发报错)。

抛出

- **AccessError**

   –

  - 若用户对所请求对象没有写入权限
  - 若用户尝试跳过权限规则对所请求对象进行写入

- [**ValidationError**](#odoo.exceptions.ValidationError) – 若用户对字段输入不在选项内的无效值

- [**UserError**](#odoo.exceptions.UserError) – 若在对象等级的操作结果中创建了循环 (比如设置对象为其自身的父级)

 

- 对于数值字段 ([`Integer`](#odoo.fields.Integer), [`Float`](#odoo.fields.Float))，值应为相应类型

- 对于[`Boolean`](#odoo.fields.Boolean)，值应为 [`bool`](https://docs.python.org/3/library/functions.html#bool)

- 对于[`Selection`](#odoo.fields.Selection), 值应匹配选项值 (通常为 [`str`](https://docs.python.org/3/library/stdtypes.html#str)，有时为[`int`](https://docs.python.org/3/library/functions.html#int))

- 对于[`Many2one`](#odoo.fields.Many2one)， 值应为所要设置的记录的数据库标识符

- 使用针对值字符串的非关联字段

  ### 🚫危险

  出于历史和兼容性原因， [`Date`](#odoo.fields.Date) 和 [`Datetime`](#odoo.fields.Datetime) 字段使用字符串作为值 (写入和读取)而非 [`date`](https://docs.python.org/3/library/datetime.html#datetime.date) 或 [`datetime`](https://docs.python.org/3/library/datetime.html#datetime.datetime)。这些日期字符串仅为UTC时间并且根据 `odoo.tools.misc.DEFAULT_SERVER_DATE_FORMAT` 和 `odoo.tools.misc.DEFAULT_SERVER_DATETIME_FORMAT`进行格式化

- [`One2many`](#odoo.fields.One2many) 和 [`Many2many`](#odoo.fields.Many2many) 使用特殊“命令”格式来操作存储在该字段或与该字段相关联的记录的集合中。

  这个格式是一个序列执行的三元列表，每个三元元素是一个对一组记录执行的命令。并非所有命令适用所有场景。可用的命令有：

  - `(0, 0, values)`

    新增由所提供的 `value` 字典所创建的记录。

  - `(1, id, values)`

    使用`values`中的值更新已有的 id为 `id`的记录。不能用于 [`create()`](#odoo.models.Model.create)。

  - `(2, id, 0)`

    从集合中移除id为 `id`的记录，然后（从数据库中）删除它。不能在 [`create()`](#odoo.models.Model.create)中使用。

  - `(3, id, 0)`

    集合中移除id为 `id`的记录，但不删除它。不能用于 [`create()`](#odoo.models.Model.create)。

  - `(4, id, 0)`

    向集合添加id为 `id` 的已有记录。

  - `(5, 0, 0)`

    从集合中删除所有记录，等价于对所有记录显式的调用于命令 `3` 。不能用于 [`create()`](#odoo.models.Model.create)。

  - `(6, 0, ids)`

    通过`ids` 列表替换掉集合中所有已有记录list, 等价于对`ids`中的每个`id`执行命令`4`然后执行命令`5`。

###### `Model.flush(*fnames=None*, *records=None*)`

处理所有等待中的重新计算 (或至少是在存在进针对给定字段名 `fnames` ) 并将进行中的更新刷入数据库中。

### Search/Read

###### `Model.browse([*ids*]) → records`

在当前环境中返回以 ids为参数的记录集。

```
self.browse([7, 18, 12])
res.partner(7, 18, 12)
```

参数

**ids** ([`int`](https://docs.python.org/3/library/functions.html#int)` or `[`list`](https://docs.python.org/3/library/stdtypes.html#list)`(`[`int`](https://docs.python.org/3/library/functions.html#int)`) or ``None`) – id(s)

返回

记录集

###### `Model.search(*args[, offset=0][, limit=None][, order=None][, count=False]*)`

根据 `args` [搜索域](#reference-orm-domains)搜索记录。

参数

- **args** – [一个搜索域](#reference-orm-domains)。使用空列表来匹配所有记录。
- **offset** ([`int`](https://docs.python.org/3/library/functions.html#int)) – 要忽略的结果数 (默认值: none)
- **limit** ([`int`](https://docs.python.org/3/library/functions.html#int)) – 返回的最大记录数(默认值: all)
- **order** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 排序字符串
- **count** ([`bool`](https://docs.python.org/3/library/functions.html#bool)) – 若为True, 仅计数并返回所匹配记录数 (默认值: False)

返回

最多`limit`条匹配搜索条件的记录

抛出

[**AccessError**](#odoo.exceptions.AccessError) –

- 若用户尝试跳过权限规则来读取所请求对象。

###### `Model.search_count(*args*) → int`

返回当前模型中匹配[所提供域](#reference-orm-domains)的记录数。

###### `Model.name_search(*name=''*, *args=None*, *operator='ilike'*, *limit=100*) → records`

在与给定`运算符`进行比较时搜索显示名匹配给定`name`样式，而又同时匹配可选搜索域（`args`）的记录。

根据关联字段部分值，用于提供建议的示例。有时看作[`name_get()`](#odoo.models.Model.name_get)函数的反向，但并非一直如此。

这个方法等价于通过基于`display_name`的搜索哉调用 [`search()`](#odoo.models.Model.search) ，然后对搜索结果调用 [`name_get()`](#odoo.models.Model.name_get)。

参数

- **name** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 待匹配的名称样式
- **args** ([`list`](https://docs.python.org/3/library/stdtypes.html#list)) – 可选搜索域(参见针对语法中的 [`search()`](#odoo.models.Model.search) ，指定进行一步限制
- **operator** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 匹配`name`的域运算符，如 `'like'` 或 `'='`.
- **limit** ([`int`](https://docs.python.org/3/library/functions.html#int)) – 待返回的可选最大记录数

返回类型

[list](https://docs.python.org/3/library/stdtypes.html#list)

返回

所有匹配记录的`(id, text_repr)` 对列表。

###### `Model.read([*fields*])`

读取 `self`中记录的所请求字段，低级别/RPC 方法。Python代码中，推荐使用 [`browse()`](#odoo.models.Model.browse)。

参数

**fields** – 待返回的字段名列表(默认为所有字段)

返回

映射字符名及其值的字典的列表，每条记录一个字典

抛出

[**AccessError**](#odoo.exceptions.AccessError) – 若用户对某些给定记录没有读取权限

###### `Model.read_group(*domain*, *fields*, *groupby*, *offset=0*, *limit=None*, *orderby=False*, *lazy=True*)`

获取由指定 `groupby` 字段分组的列表视图中的记录列表。

参数

- **domain** ([`list`](https://docs.python.org/3/library/stdtypes.html#list)) – [一个搜索域](#reference-orm-domains)。使用空列表来匹配所有记录。
- **fields** ([`list`](https://docs.python.org/3/library/stdtypes.html#list)) – 对对象指定的列表视图中的字段列表。每个元素要么是‘field’ (字段名，使用默认聚合)，要么是‘field:agg’ (使用聚合函数‘agg’对字段进行聚合)，要么是‘name:agg(field)’(使用‘agg’进行聚合并返回它作为‘name’)。可能的聚合函数有[PostgreSQL](https://www.postgresql.org/docs/current/static/functions-aggregate.html)所提供的函数及‘count_distinct’,用途和字面意思相同。
- **groupby** ([`list`](https://docs.python.org/3/library/stdtypes.html#list)) –通哪些进行分组的groupby描述列表。groupby描述可为一个字段(然后它会使用该字段进行分组) 或一个字符串‘field:groupby_function’。当前，所支持的函数仅有‘day’, ‘week’, ‘month’, ‘quarter’ 或 ‘year’，并且它们仅对date/datetime字段有意义。
- **offset** ([`int`](https://docs.python.org/3/library/functions.html#int)) – 可选的可跳过的记录数
- **limit** ([`int`](https://docs.python.org/3/library/functions.html#int)) – 可选的返回的最大记录数
- **orderby** ([`list`](https://docs.python.org/3/library/stdtypes.html#list)) – 可选的`order by` 参数，用于重载分组的自然排序，参见 `search()` (当前仅支持many2one字段)
- **lazy** ([`bool`](https://docs.python.org/3/library/functions.html#bool)) – 若为 true，结果仅由第一个进行分组，剩余的groupby放在 __context键中。若为false，所有的groupby都会在一次调用中完成。

返回

字典列表(每条记录一个字典)包含：

- `groupby`参数中字段所分组的字段值
- __domain: 指定搜索条件的元组列表
- __context:带有类似`groupby`参数的字典

返回类型

[{‘field_name_1’: value, ..]

抛出

[**AccessError**](#odoo.exceptions.AccessError) –

- 若用户对所请求对象没有读取权限
- 若用户尝试对所请求对象跳过权限规则读取

#### 字段/视图

###### `Model.fields_get(*[fields][, attributes]*)`

返回每个字段的定义。

返回值是字典中的字典 (由字段名索引)。包含 _inherits的字段。string, help,及 selection (若存在)属性会进行翻译。

参数

- **allfields** – 要记入的字段列表，未提供或为空时表示所有
- **attributes** – 每个字段要返回的描述属性列表，未提供或为空时表示所有

###### `Model.fields_view_get([*view_id | view_type='form'*])`

获取详细的请求视图组合，如字段、模型、视图框架

参数

- **view_id** ([`int`](https://docs.python.org/3/library/functions.html#int)) – 视图 id 或None
- **view_type** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 若view_id 为 None时返回的视图类型 (‘form’, ‘tree’, …)
- **toolbar** ([`bool`](https://docs.python.org/3/library/functions.html#bool)) – true则包含上下文动作
- **submenu** – 已淘汰

返回

所请求视图的组合 (包含所继承视图及扩展)

返回类型

[dict](https://docs.python.org/3/library/stdtypes.html#dict)

抛出

- **AttributeError**

   –

  - 若所继承视图有‘before’, ‘after’, ‘inside’, ‘replace’以外的未知位置进行操作
  - 若在父视图中发现‘position’ 以外的标签

- **Invalid ArchitectureError** – 若视图类型是结构所定义的 form, tree, calendar, search 等以外的类型



#### 搜索作用域

作用域一个条件列表，每个条件由`(field_name, operator, value)`三个元素组成 (可以是 `列表` 或 `元组`) ：

- - `field_name` (`str`)

    当前模型的字段名，或通过[`Many2one`](#odoo.fields.Many2one)使用点号标记的关联遍历，如 `'street'` 或 `'partner_id.country'`

- - `operator` (`str`)

    用于比较 `field_name` 和 `value`的运算符。有效运算符有：`=`等于`!=`不等于`>`大于`>=`大于等于`<`小于`<=`小于等于`=?`未设置或等于 (若`value` 为`None` 或 `False`返回true，否则类似 `=`)`=like`通过`value`模式匹配 `field_name`。样式中的下划线 `_` 表示（匹配）任意单个字符；百分号 `%` 匹配任意包含零个或多个字符的字符串。`like`通过`%value%`模式匹配 `field_name` 。类似于 `=like` ，但在匹配前通过‘%’包裹 `value``not like`不匹配 `%value%` 模式`ilike`不区分大小写的 `like``not ilike`不区分大小写的 `not like``=ilike`不区分大小写的 `=like``in`等于`value`中的任意项， `value` 应为一个列表`not in`不等于 `value`中的所有项`child_of`是 `value` 记录的后代(值可为单项或列表)。考虑模型的句法(如按照`_parent_name`所命名的关联字段)。`parent_of``value` 记录的父级（先辈）(值可为单项或列表)。考虑模型的句法(如按照`_parent_name`所命名的关联字段)。

- - `value`

    变量类型，必须可与命名字段(通过 `运算符`)相比较。

域条件可使用*前缀*形式的逻辑运算符进行合并：

- `'&'`

  逻辑与， 合并相连条件的默认运算。Arity 2 (使用接下来的两个条件或组合)。

- `'|'`

  逻辑或, arity 2.

- `'!'`

  逻辑非, arity 1.大多针对条件组合取反。单个条件通常有一个取反形式(如 `=` -> `!=`, `<` -> `>=`) ，比对正值取负值更为简单。

译者注：函数或运算的arity是指参数数量或函数所接收的运算项，常译为一元、二元...

### 示例

要搜索来自比利时或德国且语言不是英文的名为*ABC*的合作伙伴：

```
[('name','=','ABC'),
 ('language.code','!=','en_US'),
 '|',('country_id.code','=','be'),
     ('country_id.code','=','de')]
```

域的解析如下：

```
    (名称为 'ABC')
AND (语言不是英语)
AND (国家为比利时或德国)
```

### Unlink

###### `Model.unlink()`

删除当前集合的记录

抛出

- **AccessError**

   –

  - 若用户对所请求对象不具有删除权限
  - 若用户尝试跳过权限规则对所请求对象进行删除

- [**UserError**](#odoo.exceptions.UserError) – 若记录为其它记录的默认属性



### 记录（集）信息

###### `Model.ids`

返回对应 `self`的实际记录 id 列表。

###### `odoo.models.env`

返回给定记录集的环境。

类型

```
Environment
```

###### `Model.exists() → records`

返回 `self` 中存在的记录的子集，并在缓存中标记所删除的记录。可用于测试记录：

```
if record.exists():
    ...
```

按照规则，新记录以已有记录进行返回。

###### `Model.ensure_one()`

验证当前记录集存有单条记录。

抛出

**odoo.exceptions.ValueError** – `len(self) != 1`

###### `Model.name_get() → [(id, name), ...]`

返回`self`中记录的文本体现。默认这是 `display_name` 字段的值。

返回

每条记录的 `(id, text_repr)` 对列表

返回类型

[list](https://docs.python.org/3/library/stdtypes.html#list)([tuple](https://docs.python.org/3/library/stdtypes.html#tuple))

###### `Model.get_metadata()`

返回有关给定记录的一些元数据。

返回

每条请求记录的所有杼字典列表

返回类型

带有如下键的字典的列表：

- id: 对象id
- create_uid: 创建记录的用户
- create_date: 记录所创建的日期
- write_uid: 最后修改记录的用户
- write_date: 最后修改记录的日期
- xmlid: 用于引用这条记录的XML ID (若存在)，格式为 `module.name`
- noupdate: 辨识记录是否会更新记录的布尔值



### 运算

记录集是不可变的，但相同模型的集合可使用各种集合运算符进行合并，返回一个新的记录集。

- `record in set` 返回 `record` (必须是一个元素的记录集) 是否在`set`中。`record not in set` 是其反向操作
- `set1 <= set2` 和 `set1 < set2` 返回`set1` 是否为 `set2` 的子集(resp. strict)
- `set1 >= set2` 和 `set1 > set2` 返回 `set1` 是否为 `set2` 的超集(resp. strict)
- `set1 | set2` 返回两个记录集的并集，新记录集中你居然说两个鹰中的所有记录
- `set1 & set2` 返回两个记录集的交集，新记录集中仅包含两个集合中同时出现的记录
- `set1 - set2` 仅返回属于 `set1` 但不属于 `set2`的记录的记录集

记录集是可迭代对象，因此常用的Python工具（([`map()`](https://docs.python.org/3/library/functions.html#map), [`sorted()`](https://docs.python.org/3/library/functions.html#sorted), `itertools.ifilter`, …) ）均可用于转换，但这些会返回一个 [`列表`](https://docs.python.org/3/library/stdtypes.html#list) 或时 [迭代器](https://docs.python.org/3/glossary.html#term-iterator)，去除掉对结果调用方法或使用集合运算的功能。

因此记录集（在可用时）提供如下返回记录集本身的运算：

#### 过滤

###### `Model.filtered(*func*)`

返回 `self` 中满足 `func`的记录。

参数

**func** (`callable 或`[`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 一个函数或点号分隔的字段名序列

返回

满足func的记录的记录集，可以为空。

```
# 仅保留公司为当前用户的记录
records.filtered(lambda r: r.company_id == user.company_id)

# 仅保留合作伙伴为公司的记录
records.filtered("partner_id.is_company")
```

###### `Model.filtered_domain(*domain*)`

#### 映射

###### `Model.mapped(*func*)`

对 `self`中的所有记录应用`func`，返回结果为一个列表或记录集 (若`func` 返回记录集)。后一种情况中，所返回记录集可为任意排序。

参数

**func** (`callable``或`[`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 函数或字段名的点号分隔序列

返回

若func为虚值返回self， func的结果应用于所有 `self` 记录。

返回类型

[列表](https://docs.python.org/3/library/stdtypes.html#list) 或 记录集

```
# 返回对集合中每条记录进行加总的列表
records.mapped(lambda r: r.field1 + r.field2)
```

所提供的函数可为获取字段值的字符串：

```
# 返回名称列表
records.mapped('name')

# 返回伙伴的记录集
record.mapped('partner_id')

# 返回所有伙伴银行的并集，删除重复内容
record.mapped('partner_id.bank_ids')
```

#### 排序

###### `Model.sorted(*key=None*, *reverse=False*)`

返回通过 `key`排序的记录集`self`

参数

- **key** (`callable``或`[`str`](https://docs.python.org/3/library/stdtypes.html#str)`或``None`) – 可为一个参数的返回每条记录键比较的函数、一个字段名或 `None`，这时记录按照默认的模型顺序进行排序。
- **reverse** ([`bool`](https://docs.python.org/3/library/functions.html#bool)) – 若为 `True`，以倒序返回结果

```
# sort records by name
records.sorted(key=lambda r: r.name)
```



## 继承和扩展

Odoo提供三种不同的机制来以模块化的方式继承模型：

- 从已有模型新建一个模型，对复制的模型新增信息，但保留原模块为原样
- 在原处继承模块中定义的模型，替换原来的版本
- 你本来就是该模型的一些字段为它包含的记录

![img](https://www.odoo.com/documentation/13.0/_images/inheritance_methods1.png)

### 经典继承

在共同使用`_inherit` 和 `_name` 属性时， Odoo使用已有模型（通过`_inherit`）作为基础新建一个模型。新模型从基础获取所有字段、方法及元信息(defaults & al) 。

```
class Inheritance0(models.Model):
    _name = 'inheritance.0'
    _description = 'Inheritance Zero'

    name = fields.Char()

    def call(self):
        return self.check("model 0")

    def check(self, s):
        return "This is {} record {}".format(s, self.name)

class Inheritance1(models.Model):
    _name = 'inheritance.1'
    _inherit = 'inheritance.0'
    _description = 'Inheritance One'

    def call(self):
        return self.check("model 1")
```

并使用它们：

```
        a = env['inheritance.0'].create({'name': 'A'})
        b = env['inheritance.1'].create({'name': 'B'})
            a.call()
            b.call()
```

会产生：

```
            "This is model 0 record A"
            "This is model 1 record B"
```

第二个模型从第一个模型的 `check` 方法及`name`字段中继承，但重载 `call` 方法，像使用标准[Python 继承](https://docs.python.org/3/tutorial/classes.html#tut-inheritance)一样。

### 扩展

在使用 `_inherit` 但保留 `_name`时，新模型替换已有模型，基本是在原地进行扩展。这对于向（其它模块中所创建的）已有模型新增字段或方法、或是自定义或重新配置它们（例如修改默认排序）都非常有用：

```
class Extension0(models.Model):
    _name = 'extension.0'
    _description = 'Extension zero'

    name = fields.Char(default="A")

class Extension1(models.Model):
    _inherit = 'extension.0'

    description = fields.Char(default="Extended")
        record = env['extension.0'].create({})
        record.read()[0]
```

会生成：

```
{'name': "A", 'description': "Extended"}
```

除非进行禁用它会生成各种[自动化字段](#reference-fields-automatic) 。

### 代理 

第三种继承机制提供更丰富的灵活性（可在运行时进行修改），但功能更少；使用 `_inherits` 模型代理当前模型中未找到的任意字段查询到“子”模型中。代理通过 [`Reference`](#odoo.fields.Reference) 字段自动在父级模型中设置。

最主要的区别是，在使用代理时，模型拥有它而非是它，转换组合中的关联而非继承：

```
class Screen(models.Model):
    _name = 'delegation.screen'
    _description = 'Screen'

    size = fields.Float(string='Screen Size in inches')

class Keyboard(models.Model):
    _name = 'delegation.keyboard'
    _description = 'Keyboard'

    layout = fields.Char(string='Layout')

class Laptop(models.Model):
    _name = 'delegation.laptop'
    _description = 'Laptop'

    _inherits = {
        'delegation.screen': 'screen_id',
        'delegation.keyboard': 'keyboard_id',
    }

    name = fields.Char(string='Name')
    maker = fields.Char(string='Maker')

    # a Laptop has a screen
    screen_id = fields.Many2one('delegation.screen', required=True, ondelete="cascade")
    # a Laptop has a keyboard
    keyboard_id = fields.Many2one('delegation.keyboard', required=True, ondelete="cascade")
        record = env['delegation.laptop'].create({
            'screen_id': env['delegation.screen'].create({'size': 13.0}).id,
            'keyboard_id': env['delegation.keyboard'].create({'layout': 'QWERTY'}).id,
        })
            record.size
            record.layout
```

会生产结果：

```
            13.0
            'QWERTY'
```

并且可以直接对代理字段进行写入：

```
        record.write({'size': 14.0})
```

> ⚠️**警告**
>
> 在使用代理继承时，方法不会进行继承，仅继承字段

### 字段递增定义

字段在模型类中定义为类属性。如果继承了模型，可以通过对子类重新使用相同名称和相同类型重新定义字段来扩展字段的定义。这时，字段的属性取自父类并由子类所指定的内容重载。

例如，下面的第二个类只会对`state`字段添加提示信息：

```
class First(models.Model):
    _name = 'foo'
    state = fields.Selection([...], required=True)

class Second(models.Model):
    _inherit = 'foo'
    state = fields.Selection(help="Blah blah blah")
```



## 错误管理

Odoo异常模块定义了一些核心异常类型。

这些类型可被RPC层所解释。任意其它异常类在到达 RPC 前会被视为‘服务错误’。

> 如果考虑引入新的异常，请查看 `odoo.addons.test_exceptions` 模块。

###### `*exception* odoo.exceptions.AccessDenied(*message='Access denied'*)[source]`

登录名/密码错误。

> 没有回溯追踪。

> **示例**
>
> 使用错误密码登录时。

###### `*exception* odoo.exceptions.AccessError(*msg*)[source]`

访问权限错误。

> **示例**
>
> 在读取不允许读取的记录时。

###### `*exception* odoo.exceptions.CacheMiss(*record*, *field*)[source]`

在缓存中缺失值。

> **示例**
>
> 在清除的缓存中读取值。

###### `*exception* odoo.exceptions.MissingError(*msg*)[source]`

记录缺失。

> **示例**
>
> 对已删除记录写入时。

###### `*exception* odoo.exceptions.RedirectWarning[source]`

警告并可能会重定向用户而不是简单地显示 警告消息。

> 参数
>
> - **action_id** ([`int`](https://docs.python.org/3/library/functions.html#int)) – 执行重定向处的动作 id
> - **button_text** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 放到会触发重定向的按钮上的文本。

###### `*exception* odoo.exceptions.UserError(*msg*)[source]`

由客户端管理的通用错误。

通常发生在用户对当前状态的记录进行无意义操作时。

###### `*exception* odoo.exceptions.ValidationError(*msg*)[source]`

违反python的约束。

> **示例**
>
> 在使用数据库中已存在的用户名新建用户时。