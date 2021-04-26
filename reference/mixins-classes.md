# Mixins和有用的类

本文来自[Odoo 13官方文档之开发者文档](../README.md)系列文章

Odoo实现了一些有用的类及mixins来让其易于在你自己的对象中添加常用的行为。本指南将通过示例和用例详细讲解它们。



## 消息功能



### 消息集成

#### 基本消息系统

将消息功能集成到模型中极其容易。只需继承`mail.thread` 模型并在表单视图添加消息字段（及相应的控件）就可以马上配置完成并运行。

### 示例

我们来创建一个表示商务旅行的简化模型。因为组织这类旅行通常包含大量的人和大量的讨论，我们在模型中添加对消息互换的支持。

```
class BusinessTrip(models.Model):
    _name = 'business.trip'
    _inherit = ['mail.thread']
    _description = 'Business Trip'

    name = fields.Char()
    partner_id = fields.Many2one('res.partner', 'Responsible')
    guest_ids = fields.Many2many('res.partner', 'Participants')
```

在表单视图中：

```
<record id="businness_trip_form" model="ir.ui.view">
    <field name="name">business.trip.form</field>
    <field name="model">business.trip</field>
    <field name="arch" type="xml">
        <form string="Business Trip">
            <!-- Your usual form view goes here
            ...
            Then comes chatter integration -->
            <div class="oe_chatter">
                <field name="message_follower_ids" widget="mail_followers"/>
                <field name="message_ids" widget="mail_thread"/>
            </div>
        </form>
    </field>
</record>
```

一旦在模型中添加了chatter支持，用户就可以在模型的任意记录中轻易添加消息或内部记录；每个都会发送通知（消息给所有的关注者，内部记录发送给雇员 (*base.group_user*) 用户）。如若正确地配置了邮件网关及catchall地址，这些消息会通过邮件发送并可通过邮件客户端进行回复；自动路由系统会将回复路由到正确的线程中。

服务端有一些帮助函数可帮助你轻易地发送消息及对记录管理关注者：

##### 提交消息

###### `message_post(*self*, *body=''*, *subject=None*, *message_type='notification'*, *subtype=None*, *parent_id=False*, *attachments=None*, ***kwargs*)`

在已有的线程中提交(post)新消息，返回新的mail.message ID。

参数

- **body** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 消息体，通过为会进行清洗的原生HTML
- **message_type** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 参见mail_message.message_type字段
- **parent_id** ([`int`](https://docs.python.org/3/library/functions.html#int)) – 在私有讨论时通过向消息添加父级伙伴来处理对此前消息的回复
- **attachments** ([`list`](https://docs.python.org/3/library/stdtypes.html#list)`(`[`tuple`](https://docs.python.org/3/library/stdtypes.html#tuple)`(`[`str`](https://docs.python.org/3/library/stdtypes.html#str)`,`[`str`](https://docs.python.org/3/library/stdtypes.html#str)`)``)`) – 以 `(name,content)`的形式列出附件元组，内容不进行base64 编码
- ***\*kwargs** – 额外的关键字参数会用作针对新mail.message记录的默认列值

返回

新创建的mail.messageID

返回类型

[int](https://docs.python.org/3/library/functions.html#int)

###### `message_post_with_view(views_or_xmlid, **kwargs):`

发送邮件/提交消息的帮助方法通过ir.qweb引擎使用view_id来渲染。这个方法是独立的，因为模板和composer中不能批量处理视图。这一方法可能会在模板处理ir ui视图时消失。

参数

**or ir.ui.view record** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 应当发送的外部id或视图记录

###### `message_post_with_template(*template_id*, ***kwargs*)`

通过模板发送邮件的帮助方法

参数

- **template_id** – 渲染为创建消息体的模板 id
- ***\*kwargs** – 创建（继承自mail.message的）mail.compose.message向导的参数

##### 接收消息

这些方法在新的邮件由邮件网关处理时调用。这些e-mail可以为新线程 (若通过[别名](#reference-mixins-mail-alias)到达) 或只是对已有线程进行回复。重载它们让你可以根据邮件本身的一些值设置线程记录中的值 (如更新日期或邮件地址，添加 CC地址为关注者等等)。

###### `message_new(*msg_dict*, *custom_values=None*)`

如果消息不属于已有线程，在对给定线程模型接收新消息时由 `message_process` 调用。

默认行为是（根据从消息中提取的一些非常基本的信息）新建相应模型的记录。其它行为可通过重载该方法实现。

参数

- **msg_dict** ([`dict`](https://docs.python.org/3/library/stdtypes.html#dict)) – 包含邮件详情和附件的映射。参见`message_process` 和 `mail.message.parse` 获取详情
- **custom_values** ([`dict`](https://docs.python.org/3/library/stdtypes.html#dict)) – 在创建新的线程记录时传递给create() 的额外字段值的可选字典； 要当心这些值可能会重载来自该消息的其它值

返回类型

[int](https://docs.python.org/3/library/functions.html#int)

返回

新创建的线程对象的 id

###### `message_update(*msg_dict*, *update_vals=None*)`

已有线程收到新消息后由 `message_process` 调用。默认行为是通过进入的email获取的 `update_vals` 更新记录。

其它行为可通过重载该方法实现。

参数

- **msg_dict** ([`dict`](https://docs.python.org/3/library/stdtypes.html#dict)) – 包含email详情和附件的映射；参见 `message_process` 和 `mail.message.parse()` 获取详情。
- **update_vals** ([`dict`](https://docs.python.org/3/library/stdtypes.html#dict)) – 包含值对给定id的更新记录的字典；如字典为None或空，不执行任何写入操作。

返回

True

##### 关注者管理

###### `message_subscribe(*partner_ids=None*, *channel_ids=None*, *subtype_ids=None*, *force=True*)`

将伙伴添加到记录关注者中。

参数

- **partner_ids** ([`list`](https://docs.python.org/3/library/stdtypes.html#list)`(`[`int`](https://docs.python.org/3/library/functions.html#int)`)`) – 将会订阅记录的伙伴的ID
- **channel_ids** ([`list`](https://docs.python.org/3/library/stdtypes.html#list)`(`[`int`](https://docs.python.org/3/library/functions.html#int)`)`) – 将会订阅记录的通道的ID
- **subtype_ids** ([`list`](https://docs.python.org/3/library/stdtypes.html#list)`(`[`int`](https://docs.python.org/3/library/functions.html#int)`)`) – 通道/伙伴会订阅的子类型的ID (在为`None`时默认为默认子类型)
- **force** – 若为True, 使用参数中给定的子类型在新建之前删除已有关注者

返回

成功/失败

返回类型

[bool](https://docs.python.org/3/library/functions.html#bool)

###### `message_unsubscribe(*partner_ids=None*, *channel_ids=None*)`

从记录的关注者中删除伙伴。

参数

- **partner_ids** ([`list`](https://docs.python.org/3/library/stdtypes.html#list)`(`[`int`](https://docs.python.org/3/library/functions.html#int)`)`) – 将会订阅到记录的伙伴的ID
- **channel_ids** ([`list`](https://docs.python.org/3/library/stdtypes.html#list)`(`[`int`](https://docs.python.org/3/library/functions.html#int)`)`) – 将会订阅到记录的通道的ID

返回

True

返回类型

[bool](https://docs.python.org/3/library/functions.html#bool)

###### `message_unsubscribe_users(*user_ids=None*)`

message_subscribe的封装，使用用户。

参数

**user_ids** ([`list`](https://docs.python.org/3/library/stdtypes.html#list)`(`[`int`](https://docs.python.org/3/library/functions.html#int)`)`) – 将会退订记录的用户ID；若为None，则退订当前用户。

返回

True

返回类型

[bool](https://docs.python.org/3/library/functions.html#bool)

#### 记录修改日志

`mail`模块对字段添加了强大的追踪系统，让你可以在记录的chatter中记录具体字段的改变。要追踪某个字段，只需设置追踪属性为True。

### 示例

我们来追踪商务旅行名称和负责人的变更：

```
class BusinessTrip(models.Model):
    _name = 'business.trip'
    _inherit = ['mail.thread']
    _description = 'Business Trip'

    name = fields.Char(tracking=True)
    partner_id = fields.Many2one('res.partner', 'Responsible',
                                 tracking=True)
    guest_ids = fields.Many2many('res.partner', 'Participants')
```

从现在开始，对旅行名称或负责人的每次修改都会记录中进行登记。 (即使名称未进行修改) `name` 字段也会在通知中显示来为通知提供更多的上下文。

#### 子类型

子类型让我们对消息有更细化颗粒度的控制。子系统像是通知的分类系统，让订阅文档者可以自定义他们想要接收的通知的子类型。

子类型在模块中以数据的形式创建；其模型有如下字段：

- `name` (必传) - [`Char`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Char)

  子类型的名称，会在通知的自定义弹窗中显示

- `description` - [`Char`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Char)

  针对这一子类型所会提交的添加到消息中的描述。若为空，会使用名称进行添加

- `internal` - [`Boolean`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Boolean)

  带有内部子类型的消息仅对雇员可见，即 `base.group_user` 组中的成员

- `parent_id` - [`Many2one`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Many2one)

  自动订阅的链接子类型；例如项目子类型通过这一关联链接到了任务子类型。在有人订阅项目时，他会订阅项目中通过父级子类型所查询到的子类型的所有任务f

- `relation_field` - [`Char`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Char)

  举个例子，在链接项目和任务子类型时，关联字段是任务的project_id字段

- `res_model` - [`Char`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Char)

  子类型所应用的模型；若为False，这个子类型会应用到所有的模型

- `default` - [`Boolean`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Boolean)

  在订阅时默认该子类型时否激活

- `sequence` - [`Integer`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Integer)

  用于在通知自定义弹窗中排序子类型

- `hidden` - [`Boolean`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Boolean)

  在通知自定义弹窗中是否隐藏该子类型

与字段追踪交接的子类型允许根据用户所感兴趣的订阅不同类型通知。这样，可以重载 `_track_subtype()` 函数：

###### `_track_subtype(*init_values*)`

根据所更新的值对记录上的变更所触发的给予子类型。

参数

**init_values** ([`dict`](https://docs.python.org/3/library/stdtypes.html#dict)) – 记录的原始值；字典中仅出现所修改的字段

返回

子类型的完整外部id或无子类型触发时为False

### 示例

我们来对示例类添加一个`state` 字段并在字段值发生改变时使用一个特定子类型触发通知。

首先，我们来定义子类型：

```
<record id="mt_state_change" model="mail.message.subtype">
    <field name="name">Trip confirmed</field>
    <field name="res_model">business.trip</field>
    <field name="default" eval="True"/>
    <field name="description">Business Trip confirmed!</field>
</record>
```

然后，需要重载 `track_subtype()`函数。该函数由追踪系统调用来了解根据当前应用的修改使用哪个子类型。在这里，我们要在`state`字段由*draft* 更改 *confirmed*时使用这个全新的子类型：

```
class BusinessTrip(models.Model):
    _name = 'business.trip'
    _inherit = ['mail.thread']
    _description = 'Business Trip'

    name = fields.Char(tracking=True)
    partner_id = fields.Many2one('res.partner', 'Responsible',
                                 tracking=True)
    guest_ids = fields.Many2many('res.partner', 'Participants')
    state = fields.Selection([('draft', 'New'), ('confirmed', 'Confirmed')],
                             tracking=True)

    def _track_subtype(self, init_values):
        # init_values包含修改前变更字段的值 
        #
        # 所应用的值可在在缓存中就绪时在记录中访问
        self.ensure_one()
        if 'state' in init_values and self.state == 'confirmed':
            return self.env.ref('my_module.mt_state_change')
        return super(BusinessTrip, self)._track_subtype(init_values)
```

#### 自定义通知

在将通知发送给关注者时，在模板中添加按钮来允许直接来自e-mail的快速动作非常有用。即使是一个直接链接到记录的表单视图的简单按钮也非常有用，但在大部分情况下会不希望显示这些按钮来传送至用户。

通知系统允许以如下方式自定义通知模板：

- 显示*访问按钮：*这些按钮在通知e-mail的顶部可见，允许接收者直接访问记录的表单视图
- 显示*关注按钮：* 这些按钮允许接收者直接快速订阅记录
- 显示*取关按钮：* 这些按钮允许接收者直接快速取消订阅记录
- 显示*自定义动作按钮：* 这些按钮是对具体路由的调用，允许我们让一些有用的动作直接在e-mai中可用 (如将线索转化为机会，为收支管理器验证收支表单等)

这些按钮设置可用于你自己定义通过重载`_notification_recipients`函数的不同组 。

###### `_notification_recipients(*message*, *groups*)`

根据所更新值对记录中所做修改给定的子类型。

参数

- **message** (`record`) – 当前发送的 `mail.message` 记录

- groups

   (

  `list`

  ```
  (
  ```

  `tuple`

  ```
  )
  ```

  ) – 表单(group_name, group_func,group_data) 的元组列表：

  - group_name

    仅用于能够重载和操纵组的标识符。默认组为e `user` (关联到雇员用户的接收者), `portal` (关联到门户用户的接收者) 和 `customer` (不关联互任何用户的接收者)。重载使用的示例为添加关联到 res.groups的组，像对它们设置具体的动作按钮的Hr Officers。

  - group_func

     是一个接收伙伴记录作为参数的函数指针。该方法将应用于接收者来知晓它们是否属于给定的组。仅保留第一个匹配的组。运行顺序为列表顺序。

  - group_data

    为包含下述keys - values通知email参数的字典：has_button_access是否在email中显示 Access <Document> 。新组中默认为True，门户/客户为False。button_access带有url和按钮标题的字典has_button_follow是否在email中显示 Follow (如接收者当前未 关注线程)。 新组中默认为True，门户/客户为False。button_follow带有url和按钮标题的字典has_button_unfollow是否在email中显示 Unfollow (如接收者当前关注线程)。 新组中默认为True，门户/客户为False。button_unfollow带有url和按钮标题的字典actions在通知email中显示动作按钮的列表。每个动作是一个包含url和按钮标题的字典。

返回

子类型的完整外部id或者在未触发子类型时为False

动作列表中的url可通过调用`_notification_link_helper()` 函数来自动生成：

###### `_notification_link_helper(*self*, *link_type*, ***kwargs*)`

对当前记录的给定类型（或在设置了kwargs `model` 和 `res_id` 时对具体记录）生成链接。

参数

**link_type** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 待生成的关联类型；可为如下任意值：

- `view`

  关联到记录的表单视图

- `assign`

  为记录（若存在）的 `user_id`字段分配已登录用户

- `follow`

  字面含义

- `unfollow`

  字面含义

- `method`

  对记录调用方法；方法名应提供为kwarg `method`

- `new`

  为新记录打开空的表单视图；可以通过在kwarg `action_id`中提供其id（数据库id或完整解析的外部id）来指定具体的动作

返回

为记录所选中的类型链接

返回类型

[str](https://docs.python.org/3/library/stdtypes.html#str)

### 示例

我们来添加**商务旅行**状态修改通知的自定义按钮；这个按钮将重置状态为**草稿**并将仅对（虚拟的）组旅行管理(`business.group_trip_manager`)的成员可见

```
class BusinessTrip(models.Model):
    _name = 'business.trip'
    _inherit = ['mail.thread', 'mail.alias.mixin']
    _description = 'Business Trip'

    # Pevious code goes here

    def action_cancel(self):
        self.write({'state': 'draft'})

    def _notification_recipients(self, message, groups):
        """ Handle Trip Manager recipients that can cancel the trip at the last
        minute and kill all the fun. """
        groups = super(BusinessTrip, self)._notification_recipients(message, groups)

        self.ensure_one()
        if self.state == 'confirmed':
            app_action = self._notification_link_helper('method',
                                method='action_cancel')
            trip_actions = [{'url': app_action, 'title': _('Cancel')}]

        new_group = (
            'group_trip_manager',
            lambda partner: bool(partner.user_ids) and
            any(user.has_group('business.group_trip_manager')
            for user in partner.user_ids),
            {
                'actions': trip_actions,
            })

        return [new_group] + groups
```

注意我本可以在该方法外定义运行函数及定义全局函数来代替lambda，但为保持简洁避免这些文档文件过于逻辑乏味，我选择了了使用前一种方案。

#### 重载默认值

有好几种方法来自定义`mail.thread` 模型行为，包含(但不限于)：

- `_mail_post_access` - [`Model`](https://alanhou.org/odoo-13-orm-api/#odoo.models.Model) 属性

  能够对该模型提交信息的所需访问权限；默认需要`write`权限，也可设置为 `read`

- 上下文键：

  这些上下文键可用于某种控制 `mail.thread` 功能如自动订阅或`create()` 或 `write()` 调用期间的字段追踪 (或任意其它可能有用的方法)。`mail_create_nosubscribe`: 在create 或 message_post时，不要订阅当前用户到记录线程中`mail_create_nolog`: 在创建时，不要记录自动的‘<Document> created’ 消息`mail_notrack`: 在创建和写入时，不要执行值追踪创建消息`tracking_disable`: 在创建和写入时，不执行MailThread功能 (自动订阅、追踪、提交 …)`mail_auto_delete`: 自动删除邮件通知；默认为True`mail_notify_force_send`: 如果发送了50个以内的邮件通知，直接发送它们而不使用队列；默认为 True`mail_notify_user_signature`: 在email通知中添加当前用户签名；默认为True



### Mail别名

别名为关联到在通过e-mail联系将新建记录的具体记录（通常继承`mail.alias.mixin`）的可配置email地址。这是一种让系统可在外部访问、允许用户或客户无需直接连接Odoo在数据库中快速创建记录。

#### 别名 vs. 接收邮件网关

有人使用**接收邮件网关**来实现。仍将需要正确的已配置邮件网关来使用别名，但因所有路由在Odoo内部完成单个catchall域名就足够了。别名较邮件网关有几个优势：

- - 更易于配置

    单个接收网关可由多个别名使用；这避免了对域名做多个邮件配置 (所有的配置在 Odoo中完成)无需系统访问权限来配置别名

- - 更连贯

    对相关记录可配置，而而不是在**设置**子菜单中

- - 更易于重载服务端

    一开始就构建 了Mixin模型来用于继承，允许从接收邮件比邮件网关更容易地提取有用数据。

#### 集成别名支持

别名通常对父级模型进行配置，然后在通过e-mail联络时创建具体的记录。例如，项目有创建任务或issue的别名，销售团队有生成线索的别名。

由别名创建的模型**必须**继承 `mail_thread` 模型。

别名的支持通过继承`mail.alias.mixin`来添加； 这个mixin将为每条所创建父级类的记录生成一个新的 `mail.alias` 记录 (例如，每条 `project.project` 记录在创建时初始化其 `mail.alias` 记录)。

别名也可手动创建并由简单的[`Many2one`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Many2one)字段所支持。本指南假定你想要与别名的自动创建、具体记录默认值等的更完整的集成。

不同于 `mail.thread` 继承， `mail.alias.mixin` **要求**一些具体的重载来正确使用。这些重载将指定已创建别名的值，像它必须创建记录的种类及这些记录依赖于父级对象的一些默认值：

###### `get_alias_model_name(*vals*)`

为别名返回模型名。不对已有记录回复的进入邮件会导致别名模型新记录的创建。该值可能依赖于 `vals`，在创建这个模型的记录时将值的字典传递给 `create` 。

参数

**dict** (`vals`) – 新创建存储别名的记录的值

返回

模型名

返回类型

[str](https://docs.python.org/3/library/stdtypes.html#str)

###### `get_alias_values()`

返回创建别名或在创建后写入别名的值。虽然不完全是自选，它通常要求通过设置别名的中`alias_defaults`字段默认值字典来确保新创建的记录会关联到别名的父级 (即在相应的项目中创建任务) 。

返回

将写入到新别名的值字典

返回类型

[dict](https://docs.python.org/3/library/stdtypes.html#dict)

`get_alias_values()` 重载极为有意思，因为其允许轻易地修改别名的行为。在可对别名进行设置的字段中，有以下相关的内容：

- `alias_name` - [`Char`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Char)

  别名的名称，如在想要捕获<[jobs@example.odoo.com](mailto:jobs@example.odoo.com)>的 email 时为‘jobs’

- `alias_user_id` - [`Many2one`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Many2one) (`res.users`)

  对这一别名所接收的邮件创建记录的所有者；如此字段未设置，系统会尝试根据发送者 (From) 地址查找相应的所有者，或用该地址未查找到系统用户时使用Administrator账户

- `alias_defaults` - [`Text`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Text)

  在对这一别名新建记录时将运行来提供默认值的Python字典

- `alias_force_thread_id` - [`Integer`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Integer)

  即使未对其进行回复，对所有进入消息关联的线程（记录）的可选ID；如进行了设置，会完全禁用新记录的创建

- `alias_contact` - [`Selection`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Selection)

  使用邮件网关对文档提交消息的策略*everyone*: 每个人都可提交*partners*: 仅认证伙伴*followers*: 仅相关文档或关注通道的成员的关注者

注意别名使用[委托继承](https://alanhou.org/odoo-13-orm-api/#reference-orm-inheritance)，表示在别名在另一张表中存储时，需要直接从父级对象访问所有这些字段。这让我们可以在记录的表单视图中轻易地让别名可编辑。

### 示例

我们对商务旅行添加别名来实时通过e-mail创建开支。

```
class BusinessTrip(models.Model):
    _name = 'business.trip'
    _inherit = ['mail.thread', 'mail.alias.mixin']
    _description = 'Business Trip'

    name = fields.Char(tracking=True)
    partner_id = fields.Many2one('res.partner', 'Responsible',
                                 tracking=True)
    guest_ids = fields.Many2many('res.partner', 'Participants')
    state = fields.Selection([('draft', 'New'), ('confirmed', 'Confirmed')],
                             tracking=True)
    expense_ids = fields.One2many('business.expense', 'trip_id', 'Expenses')
    alias_id = fields.Many2one('mail.alias', string='Alias', ondelete="restrict",
                               required=True)

    def get_alias_model_name(self, vals):
    """ Specify the model that will get created when the alias receives a message """
        return 'business.expense'

    def get_alias_values(self):
    """ Specify some default values that will be set in the alias at its creation """
        values = super(BusinessTrip, self).get_alias_values()
        # alias_defaults holds a dictionnary that will be written
        # to all records created by this alias
        #
        # in this case, we want all expense records sent to a trip alias
        # to be linked to the corresponding business trip
        values['alias_defaults'] = {'trip_id': self.id}
        # we only want followers of the trip to be able to post expenses
        # by default
        values['alias_contact'] = 'followers'
        return values

class BusinessExpense(models.Model):
    _name = 'business.expense'
    _inherit = ['mail.thread']
    _description = 'Business Expense'

    name = fields.Char()
    amount = fields.Float('Amount')
    trip_id = fields.Many2one('business.trip', 'Business Trip')
    partner_id = fields.Many2one('res.partner', 'Created by')
```

我们希望通过商务旅行的表单视图可轻易地配置别名，因此在表单视图中添加如下代码：

```
<page string="Emails">
    <group name="group_alias">
        <label for="alias_name" string="Email Alias"/>
        <div name="alias_def">
            <!-- display a link while in view mode and a configurable field
            while in edit mode -->
            <field name="alias_id" class="oe_read_only oe_inline"
                    string="Email Alias" required="0"/>
            <div class="oe_edit_only oe_inline" name="edit_alias"
                 style="display: inline;" >
                <field name="alias_name" class="oe_inline"/>
                @
                <field name="alias_domain" class="oe_inline" readonly="1"/>
            </div>
        </div>
        <field name="alias_contact" class="oe_inline"
                string="Accept Emails From"/>
    </group>
</page>
```

现在我们可通过表单视图直接修改别名地址并改变谁可以对别名发送e-mail。

然后可以对开支模型重载 `message_new()` 来在创建开支时从email获取值:

```
class BusinessExpense(models.Model):
    # Previous code goes here
    # ...

    def message_new(self, msg, custom_values=None):
        """ Override to set values according to the email.

        In this simple example, we simply use the email title as the name
        of the expense, try to find a partner with this email address and
        do a regex match to find the amount of the expense."""
        name = msg_dict.get('subject', 'New Expense')
        # Match the last occurence of a float in the string
        # Example: '50.3 bar 34.5' becomes '34.5'. This is potentially the price
        # to encode on the expense. If not, take 1.0 instead
        amount_pattern = '(\d+(\.\d*)?|\.\d+)'
        expense_price = re.findall(amount_pattern, name)
        price = expense_price and float(expense_price[-1][0]) or 1.0
        # find the partner by looking for it's email
        partner = self.env['res.partner'].search([('email', 'ilike', email_address)],
                                                 limit=1)
        defaults = {
            'name': name,
            'amount': price,
            'partner_id': partner.id
        }
        defaults.update(custom_values or {})
        res = super(BusinessExpense, self).message_new(msg, custom_values=defaults)
        return res
```



### 活动追踪

活动是用户应接收像打电话或组织会议这样的文档的动作。活动自带mail模块，因为它们在Chatter中集成了但未与*mail.thread*打包。 活动是`mail.activity` 类的记录，它拥有类型 (`mail.activity.type`)、名称、描述、计划时间 (等)。进行中的活动在chatter控件中的消息历史内可见。

可以对记录（分别为`mail_activity` 和 `kanban_activity`控件）的对象和具体模块在表单视图和看板视图中（通过`activity_ids`）显示它们使用`mail.activity.mixin` 类集成活动。

### 示例

组织商务旅行是一个枯燥的过程，追踪需要像订机票的活动或是预订机场计程车也很有用。这样我们将需要对模型添加活动mixin并在旅行的消息历史中显示下一个计划活动。

```
class BusinessTrip(models.Model):
    _name = 'business.trip'
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _description = 'Business Trip'

    name = fields.Char()
    # [...]
```

我们修改旅行的表单视图来显示它们的下一个活动：

```
<record id="businness_trip_form" model="ir.ui.view">
    <field name="name">business.trip.form</field>
    <field name="model">business.trip</field>
    <field name="arch" type="xml">
        <form string="Business Trip">
            <!-- Your usual form view goes here -->
            <div class="oe_chatter">
                <field name="message_follower_ids" widget="mail_followers"/>
                <field name="activity_ids" widget="mail_activity"/>
                <field name="message_ids" widget="mail_thread"/>
            </div>
        </form>
    </field>
</record>
```

可以在以下模型中查找具体的集成示例：

- `crm.lead` 在 CRM (*crm*)应用中
- `sale.order` 在Sales (*sale*) 应用中
- `project.task` 在Project (*poject*) 应用中



## 网站功能



### 访客追踪

`utm.mixin` 类可用于追踪通过与指定资源关联中参数的在线营销/通讯活动。mixin 对模型添加了3个字段：

- `campaign_id`: `utm.campaign`对象的[`Many2one`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Many2one)字段 (如 Christmas_Special, Fall_Collection等等)
- `source_id`:  `utm.source` 对象的[`Many2one`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Many2one)字段 (如搜索引擎、邮件列表等)
- `medium_id`: `utm.medium`对象的[`Many2one`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Many2one)字段 (如Snail Mail, e-Mail, 社交网络更新等)

这些模型有一个字段 `name` (只是用于分辨活动但没有任务具体的行为).

在客户通过url (如https://www.odoo.com/?campaign_id=mixin_talk&source_id=www.odoo.com&medium_id=website)中所设置的参数访问网站时，在访客的网站中对这些参数设置了三个cookie。在通过网站创建了继承utm.mixin的对象时 (如线索表单、job应用等)，utm.mixin代码从cookie中获取值来在新记录中设置它们。这么做时，接着可以在定义报表和视图（group by等）时可以使用活动/来源/媒体字段作为其它字段。

要扩展这一行为，只需要在简单模型 (the model should support the 支持*快速创建*的模型(如使用单个`name`值调用`create()` ) 添加关联字段并继承函数 `tracking_fields()`：

```
class UtmMyTrack(models.Model):
    _name = 'my_module.my_track'
    _description = 'My Tracking Object'

    name = fields.Char(string='Name', required=True)


class MyModel(models.Models):
    _name = 'my_module.my_model'
    _inherit = ['utm.mixin']
    _description = 'My Tracked Object'

    my_field = fields.Many2one('my_module.my_track', 'My Field')

    @api.model
    def tracking_fields(self):
        result = super(MyModel, self).tracking_fields()
        result.append([
        # ("URL_PARAMETER", "FIELD_NAME_MIXIN", "NAME_IN_COOKIES")
            ('my_field', 'my_field', 'odoo_utm_my_field')
        ])
        return result
```

这会告诉系统通过url 参数 `my_field`中所查找到的值创建一个名为*odoo_utm_my_field* 的cookie；在通过网站表单调用创建该模型的新记录时，`utm.mixin`的 `create()`方法常用重载会从cookie（及在不存在时实时创建的`my_module.my_track`记录）。

可以在以下模型中查找具体的集成示例：

- `crm.lead` 在CRM (*crm*)应用中
- `hr.applicant` 在Recruitment Process (*hr_recruitment*) 应用中
- `helpdesk.ticket` 在Helpdesk (*helpdesk* - 仅在Odoo企业版中)应用中



### 网站可见性

可以很轻易地对任意记录添加网站可见性切换。虽然这个mixin可以轻易地手动实现，更常使用的是对 `mail.thread`继承；一种对其有用性的验证。这一mixin的典型用例是任意对象有其前端页面；能够控制页面的可见性让我们可以在编辑页面时控制时间并仅在满意时才发布。

要包含这一功能，仅需继承 `website.published.mixin`:

```
class BlogPost(models.Model):
    _name = "blog.post"
    _description = "Blog Post"
    _inherit = ['website.published.mixin']
```

这个mixin对模型添加了2个字段：

- `website_published`: 表示发布状态的[`Boolean`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Boolean) 字段
- `website_url`: 通过访问哪个对象表示URL的[`Char`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Char)

注意后一个是计算字段，必须要在类中实现：

```
def _compute_website_url(self):
    for blog_post in self:
        blog_post.website_url = "/blog/%s" % (log_post.blog_id)
```

一旦这一机制就位，只需调整前台和后台视图来让其可访问。在后台中，在按钮区中添加按钮通常方式如下：

```
<button class="oe_stat_button" name="website_publish_button"
    type="object" icon="fa-globe">
    <field name="website_published" widget="website_button"/>
</button>
```

在前台中，需要一些安全检查来避免向网站访客显示‘编辑’ 按钮：

```
<div id="website_published_button" class="float-right"
     groups="base.group_website_publisher"> <!-- or any other meaningful group -->
    <t t-call="website.publish_management">
      <t t-set="object" t-value="blog_post"/>
      <t t-set="publish_edit" t-value="True"/>
      <t t-set="action" t-value="'blog.blog_post_action'"/>
    </t>
</div>
```

注意必须向模板传递对象作为 `object` 变量；在本例中， `blog.post` 记录通过 `blog_post` 变量传递给 `qweb` 渲染引擎，有必要指定它来发布管理模板。 `publish_edit` 变量允许前台按钮链接后台 (让我们可以轻松地从前台切换到后台，反之亦然)；若进行了设置，必须指定在`action`变量中在后台想要调用动作的完整外部id (注意必须在模型中存在表单视图)。

`website_publish_button` 动作在mixin中定义并对对象适配行为；如果该类有一个有效的`website_url` 计算函数，用户在点击按钮时会重定向到前台；然后用户可通过前台直接发布页面。这确保了线上发布不会意外发生。如果没有compute函数，会触发布尔值 `website_published` 。



### 网站元数据

这一简单的mixin允许我们轻松地在前台页面中注入元数据。

```
class BlogPost(models.Model):
    _name = "blog.post"
    _description = "Blog Post"
    _inherit = ['website.seo.metadata', 'website.published.mixin']
```

该mixin对模型添加了3个字段：

- `website_meta_title`: 允许对页面设置额外标题的[`Char`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Char) 字段
- `website_meta_description`: 包含页面短描述（有时在搜索引擎结果中使用）的[`Char`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Char) 字段
- `website_meta_keywords`: 包含一些有助于在搜索引擎更精准分类页面的关键词的[`Char`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Char) 字段；“Promote”工具有助于我们轻松地选择词法相关的关键词

这些字段通过编辑器工具栏使用“Promote”工具在前台中可编辑。设置这些字段可以有助于搜索引擎更好地收录页面。注意搜索引擎并不是仅根据这些元数据来显示结果；最佳的SEO实践仍应是通过可靠来源的引用。



## 其它



### 客户评分

评分mixin让我们可以给客户发送邮件索要评分，自动对评分在看板中进行转换及数据的聚合。

#### 对模型添加评分

要添加评分的支持，只需继承 `rating.mixin` 模型：

```
class MyModel(models.Models):
    _name = 'my_module.my_model'
    _inherit = ['rating.mixin', 'mail.thread']

    user_id = fields.Many2one('res.users', 'Responsible')
    partner_id = fields.Many2one('res.partner', 'Customer')
```

mixin的行为对模型适配：

- ```
  rating.rating
  ```

   记录会链接到模型的

  ```
  partner_id
  ```

  字段 (若字段存在)。

  - 如果使用`partner_id`以外的字段的话，这一行为可通过`rating_get_partner_id()` 函数进行重载

- ```
  rating.rating
  ```

   记录会链接到模型的

  ```
  user_id
  ```

   字段（若该字段存在）的伙伴 (如所评分的伙伴)

  - 如使用`user_id`以外的字段这一行为可通过 `rating_get_rated_partner_id()` 函数重载 (注意这一函数必须返回一个 `res.partner`， 对于 `user_id` 系统自动获取用户的伙伴)

- （若模型继承自`mail.thread`）chatter历史会显示评分活动

#### 通过e-mail发送评分请求

如果希望发送邮件索要评分，仅需要生成带有评分对象链接的e-mail。最基本的email模板像下面这样：

```
<record id="rating_my_model_email_template" model="mail.template">
            <field name="name">My Model: Rating Request</field>
            <field name="email_from">${object.rating_get_rated_partner_id().email or '' | safe}</field>
            <field name="subject">Service Rating Request</field>
            <field name="model_id" ref="my_module.model_my_model"/>
            <field name="partner_to" >${object.rating_get_partner_id().id}</field>
            <field name="auto_delete" eval="True"/>
            <field name="body_html"><![CDATA[
% set access_token = object.rating_get_access_token()
<p>Hi,</p>
<p>How satsified are you?</p>
<ul>
    <li><a href="/rating/${access_token}/10">Satisfied</a></li>
    <li><a href="/rating/${access_token}/5">Not satisfied</a></li>
    <li><a href="/rating/${access_token}/1">Very unsatisfied</a></li>
</ul>
]]></field>
</record>
```

然后你的客户将接收链接到简单网页的e-mail，让他们可以对与你的用户的交互提供反馈 (包含自由文本反馈消息)。

可以通过为评分定义动作来轻易集成带有表单视图的评分：

```
<record id="rating_rating_action_my_model" model="ir.actions.act_window">
    <field name="name">Customer Ratings</field>
    <field name="res_model">rating.rating</field>
    <field name="view_mode">kanban,pivot,graph</field>
    <field name="domain">[('res_model', '=', 'my_module.my_model'), ('res_id', '=', active_id), ('consumed', '=', True)]</field>
</record>

<record id="my_module_my_model_view_form_inherit_rating" model="ir.ui.view">
    <field name="name">my_module.my_model.view.form.inherit.rating</field>
    <field name="model">my_module.my_model</field>
    <field name="inherit_id" ref="my_module.my_model_view_form"/>
    <field name="arch" type="xml">
        <xpath expr="//div[@name='button_box']" position="inside">
            <button name="%(rating_rating_action_my_model)d" type="action"
                    class="oe_stat_button" icon="fa-smile-o">
                <field name="rating_count" string="Rating" widget="statinfo"/>
            </button>
        </xpath>
    </field>
</record>
```

注意评分存在默认视图 (kanban,pivot,graph)，让我们可以对客户评分有一个快速全景视图。

可以在以下模型中找到具体的集成示例：

- `project.task` 在Project (*rating_project*)应用中
- `helpdesk.ticket` 在Helpdesk (*helpdesk* - 仅在Odoo企业版中存在)应用中