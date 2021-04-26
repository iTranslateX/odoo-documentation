# 应用内购买

本文来自[Odoo 13官方文档之开发者文档](../README.md)系列文章

应用内购买(IAP)允许持续通过 Odoo 提供服务的供应商进行持续的服务收费，而不是单独的一次性购买。在这一背景下， Odoo扮演客户和Odoo应用开发者之间的*经纪人*的角色：

- 用户通过Odoo购买服务令牌。
- 服务提供者在用户请求服务时从其Odoo账户中提取令牌。

### ⚠️注意

本文档针对 *服务提供者* 并提供第二项，可直接通过 [JSON-RPC2](https://www.jsonrpc.org/specification) 进行实现或者如对使用Odoo的人提供便捷的帮助类/方法。

## 概览

![img](https://www.odoo.com/documentation/13.0/_images/players.png)

#### 玩家

- 服务提供者 (可能是)读者你自己，你可以按使用收费服务的形式为客户提供价值。
- 客户安装了你的 Odoo应用，并藉此请求服务。
- Odoo从间进行余额控制，客户向账户中充值，然后你在提供服务时从该账户提供金额。
- 外部服务也可以加入：你可以直接提供服务，或者代理实际的服务，在其中作为 Odoo 系统和实际服务之间的桥梁或转换者。

![img](https://www.odoo.com/documentation/13.0/_images/credits.jpg)

#### 积分余额

从**2018年10月**开始积分余额由整数变成了浮点数。仍然保持对整数值的支持。

每个通过IAP平台提供的服务可由客户通过令牌或余额来进行使用。积分余额是一个浮点单位，其货币价值取决所使用服务并由服务商决定。可以是：

- 短信服务： 1 积分 = 1 条短信;
- 广告服务： 1 积分 = 1 条广告; 或
- 邮政服务： 1 积分 = 1张邮票。

积分也可仅与固定金额的钱挂钩来缓解价格的波动 (例如短信和邮票在不同国家的价格会不同)。

积分的价值借助于购买的预付积分包来进行确定，客户可通过[https://iap.odoo.com](https://iap.odoo.com/) (参见 [积分包](#iap-packages))进行购买。

在以下的讨论中我们将忽略外部服务，它们仅是你所提供服务的细节问题。

![img](https://www.odoo.com/documentation/13.0/_images/normal.png)

#### “正常”服务流

如果一切顺利，正常的流程如下：

1. 客户请求某种服务。
2. 服务提供者询问Odoo在该客户账户中是否有足够用于服务的积分，并通过该金额创建交易。
3. 服务提供者提供服务 (通过他们自己或外部服务)。
4. 服务提供者回到Odoo来捕获(若已提供服务)或取消(若无法提供服务) 第2步中所创建的交易。
5. 最后，服务方提供客户服务已提供，（根据服务）可能会在客户的系统中显示或存储该结果。

![img](https://www.odoo.com/documentation/13.0/_images/no-credit.png)

#### 余额不足

但在客户账户里的余额不足支付服务时，流程如下：

1. 客户像之前一样请求服务。
2. 服务提供者向Odoo询问客户账户中是否有足够余额并获得一个否定的回复。
3. 将这一信号返回给客户。
4. 客户被重定向到Odoo账户中进行充值并再次尝试。

## 构建你的服务

本例中，我们将提供的服务- 狗币（dogecoin）挖矿 - 每个积分运行10秒钟CPU。对于你自己的服务，你可以：

- 自己提供在线服务 (如将报价转换为日本的商务传真);
- 自己提供*离线*服务 (如提供会计服务); 或
- 作为其他服务提供者的中介 (如彩信网关的中介)。



### 在Odoo中注册服务

开始实际查询用户账户前的第一步是在 IAP端点中注册服务（生产和/或测试）。要创建服务，进入你在IAP（生产环境为[https://iap.odoo.com](https://iap.odoo.com/) 测试环境为https://iap-sandbox.odoo.com，端点是独立非同步的）的 *Portal* 账户。或者可以进入在Odoo的门户 (https://iap.odoo.com/my/home) 并选择 *In-App Services*。

在生产环境中，服务用于管理真实交易前有一个手动验证的步骤。这一步会在沙盒中自动传递在减轻测试压力。

登录，然后进入My Account ‣ Your In-App Services， 占Create并提供你的服务的相关信息。

服务有7个重要字段：

- `name` - [`ServiceName`](https://alanhou.org/odoo-13-in-app-purchase/#ServiceName): 这是在通过 Odoo 请求交易时需要在客户的 [app](https://alanhou.org/odoo-13-in-app-purchase/#iap-odoo-app) 中进行提供的。(如 `self.env['iap.account].get(name)`)。作为一个良好实践，它应当与你的应用名称相匹配。
- `label` - `Label`: 这个名称在客户端的购买门户上显示。

### ⚠️警告

[`ServiceName`](https://alanhou.org/odoo-13-in-app-purchase/#ServiceName) 和 `Label` 都是唯一的。作为良好实践， [`ServiceName`](https://alanhou.org/odoo-13-in-app-purchase/#ServiceName) 通常应与 Odoo客户端应用名称相匹配。

- `icon` - `Icon`: 在你的[服务包](https://alanhou.org/odoo-13-in-app-purchase/#iap-packages)中作为默认的图标。
- `key` - [`ServiceKey`](https://alanhou.org/odoo-13-in-app-purchase/#ServiceKey): 在IAP中标识你的开发者密钥 (参见 [你的服务](#iap-service)) 并允许从客户端中提取积分。它仅好在创建服务时显示一次，并且可随意进行重新生成。

### 🚫危险

你的 [`ServiceKey`](https://alanhou.org/odoo-13-in-app-purchase/#ServiceKey) 是一个密钥，泄漏服务密钥会导致其他应用开发人员可提供你的服务所赚取的积分。

- `试用积分` - `Float`: 对应你准备为第一次使用应用的用户所提供的积分。注意这类服务仅对有企业合同在身的用户可用。
- `隐私政策` - `PrivacyPolicy`: 这是一个可访问你的服务隐私政策的链接。应明确表明**所收集的信息**，如何**使用信息**，与你的服务的**关联性**，并告知客户他们如何**访问、更新或删除他们的个人信息。**

![img](https://www.odoo.com/documentation/13.0/_images/menu.png)![img](https://www.odoo.com/documentation/13.0/_images/service_list.png)![img](https://www.odoo.com/documentation/13.0/_images/creating_service.png)![img](https://www.odoo.com/documentation/13.0/_images/service_created.png)然后你可以创建客户使用你的服务可购买的 *积分包*。



### 积分包

积分包是一个具有五大特性的产品：

- 名称：包的名称，
- 图标：服务包的具体图标 (如未提供，则回到使用服务图标)，
- 描述：出现在购买页面及发票上的服务包详情，
- 额度：在购买包时客户可获取的积分额度，
- 价格：价格为欧元 (截至目前，有规划对美元的支持)。

Odoo对每个包的销售提取 25%的佣金。相应地调整你的售卖价格。

根据策略不足，每个积分的价格在不同包间可不同。

![img](https://www.odoo.com/documentation/13.0/_images/package.png)



### Odoo应用

第二步是开发一个客户可在 Odoo实例中安装的 [Odoo应用](https://www.odoo.com/apps) 并且通过它他们可以*请求*你所提供的服务。我们的应用将仅向Partner表单提供一个按钮来让用户请求使用 一段时间服务器上的CPU。

首先，我们将根据`iap`创建一个 *odoo 模块* 。IAP是一个标准的 V11 模块并且其依赖确保本地账户进行了相应的配置，我们将可以访问一些必要的视图和有用的帮助信息。

*coalroller/__manifest__.py*

```
{
    'name': "Coal Roller",
    'category': 'Tools',
    'depends': ['iap'],
}
```

*coalroller/__init__.py*

```
# -*- coding: utf-8 -*-
```

然后， 集成的 “本地” 方面。这里我们将仅向伙伴视图添加一个动作按钮，但你完全可以通过你的应用提供本地值及通过远程服务提供额外的部分。

*coalroller/__manifest__.py*

```
    'name': "Coal Roller",
    'category': 'Tools',
    'depends': ['iap'],
    'data': [
        'views/views.xml',
    ],
}
```

*coalroller/views/views.xml*

```
<odoo>
  <record model="ir.ui.view" id="partner_form_coalroll">
    <field name="name">partner.form.coalroll</field>
    <field name="model">res.partner</field>
    <field name="inherit_id" ref="base.view_partner_form" />
    <field name="arch" type="xml">
      <xpath expr="//div[@name='button_box']">
        <button type="object" name="action_partner_coalroll"
                class="oe_stat_button" icon="fa-gears">
          <div class="o_form_field o_stat_info">
            <span class="o_stat_text">Roll Coal</span>
          </div>
        </button>
      </xpath>
    </field>
  </record>
</odoo>
```

![img](https://www.odoo.com/documentation/13.0/_images/button.png)现在我们可以实现动作方法/回调。这将调用我们自己的服务端。

对服务应用和服务端的通讯协议和服务器并没有什么要求，但 `iap` 提供了一个 `jsonrpc()` 帮助方法来调用其它 Odoo 实例上的[JSON-RPC2](https://www.jsonrpc.org/specification) 端点并透明地重新抛出相关Odoo异常 ([`InsufficientCreditError`](https://alanhou.org/odoo-13-in-app-purchase/#odoo.addons.iap.models.iap.InsufficientCreditError), `odoo.exceptions.AccessError` 和 `odoo.exceptions.UserError`).

在这个调用中，我们需要提供：

- 任意相关的客户端参数(此处没有)，
- 由 `iap.account` 模型的 `account_token` 字段提供的当前客户端`token`。你可以通过调用`env['iap.account'].get(*service_name*)` 来获取服务的账户，其中[`service_name`](https://alanhou.org/odoo-13-in-app-purchase/#ServiceName) 是注册在IAP端点的服务名称。

*coalroller/__init__.py*

```
# -*- coding: utf-8 -*-
from . import models
```

*coalroller/models/__init__.py*

```
from . import res_partner
```

*coalroller/models/res_partner.py*

```
# -*- coding: utf-8 -*-
from odoo import api, models
from odoo.addons.iap import jsonrpc, InsufficientCreditError

# whichever URL you deploy the service at, here we will run the remote
# service in a local Odoo bound to the port 8070
DEFAULT_ENDPOINT = 'http://localhost:8070'
class Partner(models.Model):
    _inherit = 'res.partner'
    def action_partner_coalroll(self):
        # fetch the user's token for our service
        user_token = self.env['iap.account'].get('coalroller')
        params = {
            # we don't have any parameter to provide
            'account_token': user_token.account_token
        }
        # ir.config_parameter allows locally overriding the endpoint
        # for testing & al
        endpoint = self.env['ir.config_parameter'].sudo().get_param('coalroller.endpoint', DEFAULT_ENDPOINT)
        jsonrpc(endpoint + '/roll', params=params)
        return True
```

`iap` 自动处理来自动作的 [`InsufficientCreditError`](https://alanhou.org/odoo-13-in-app-purchase/#odoo.addons.iap.models.iap.InsufficientCreditError) 并弹出让用户向自己的账户添加积分。

`jsonrpc()` 替你处理 [`InsufficientCreditError`](https://alanhou.org/odoo-13-in-app-purchase/#odoo.addons.iap.models.iap.InsufficientCreditError) 重新抛出。

### 🚫危险

如果你不使用 `jsonrpc()` ，则*必须*在处理器中谨慎抛出 [`InsufficientCreditError`](https://alanhou.org/odoo-13-in-app-purchase/#odoo.addons.iap.models.iap.InsufficientCreditError) ，否则不会跳出让用户向账户扣积分操作，下一次的调用会以相同的方式失败。



### 服务

虽然并非必须，因为 `iap` 为 [JSON-RPC2](https://www.jsonrpc.org/specification)调用(`jsonrpc()`) 提供了客户端帮助方法并为事务 ([`charge`](https://alanhou.org/odoo-13-in-app-purchase/#odoo.addons.iap.models.iap.charge)) 提供了服务帮助方法，但我们还可以用 Odoo模块实现服务层面：

*coalroller_service/__init__.py*

```
# -*- encoding: utf-8 -*-
```

*coalroller_service/__manifest__.py*

```
{
    'name': "Coal Roller Service",
    'category': 'Tools',
    'depends': ['iap'],
}
```

因为来自客户端的查询以 [JSON-RPC2](https://www.jsonrpc.org/specification) 的形式出现，我们需要对应的控制器，它可以调用[`charge`](https://alanhou.org/odoo-13-in-app-purchase/#odoo.addons.iap.models.iap.charge)并执行其中的服务：

*coalroller_service/controllers/main.py*

```
import time

from passlib import pwd, hash

from odoo import http
from odoo.addons.iap import charge

class CoalBurnerController(http.Controller):
    @http.route('/roll', type='json', auth='none', csrf='false')
    def roll(self, account_token):
        # the service key *is a secret*, it should not be committed in
        # the source
        service_key = self.env['ir.config_parameter'].sudo().get_param('coalroller.service_key')

        # we charge 1 credit for 10 seconds of CPU
        cost = 1
        # TODO: allow the user to specify how many (tens of seconds) of CPU they want to use
        with charge(http.request.env, service_key, account_token, cost):

            # 10 seconds of CPU per credit
            end = time.time() + (10 * cost)
            while time.time() < end:
                # we will use CPU doing useful things: generating and
                # hashing passphrases
                p = pwd.genphrase()
                h = hash.pbkdf2_sha512.hash(p)
        # here we don't have anything useful to the client, an error
        # will be raised & transmitted in case of issue, if no error
        # is raised we did the job
```

*coalroller_service/controllers/__init__.py*

```
# -*- encoding: utf-8 -*-
from . import main
```

*coalroller_service/__init__.py*

```
# -*- encoding: utf-8 -*-
from . import controllers
```

[`charge`](https://alanhou.org/odoo-13-in-app-purchase/#odoo.addons.iap.models.iap.charge) 帮助方法将：

1. 通过具体积分额授权（创建）事务，如果账户没有足够积分，将会报出相关错误
2. 执行 `with`语句体
3. 如果 `with` 体执行成功，在需要时更新交易的价格
4. 捕获（确认）交易
5. 否则，如果从 `with`体中抛出错误，取消交易 (并释放所保留的积分)

### 🚫危险

默认， [`charge`](https://alanhou.org/odoo-13-in-app-purchase/#odoo.addons.iap.models.iap.charge) 连接*生产环境* IAP端点 [https://iap.odoo.com](https://iap.odoo.com/).。而在开发及测试服务时，你可能会希望指向*开发环境* IAP 端点 [https://iap-sandbox.odoo.com](https://iap-sandbox.odoo.com/)。

要实现这点，在Odoo服务中设置`iap.endpoint`配置参数：在调试/开发模式下，访问Setting ‣ Technical ‣ Parameters ‣ System Parameters，仅为 `iap.endpoint` （如尚未存在）端点定义入口。

[`charge`](https://alanhou.org/odoo-13-in-app-purchase/#odoo.addons.iap.models.iap.charge) 帮助方法有两个额外的可选参数，我们可用于让内容对终端用户更为清晰。

- `description`

  是一条消息，它与交易相关联并将在用户的仪表盘中显示，有助于提醒用户为什么存在这笔费用。

- `credit_template`

  是[QWeb](https://alanhou.org/odoo-13-qweb/#reference-qweb) 模板的名称，它将在用户积分不足以支付服务商所请求的金额时进行渲染并显示给用户，其目的是告诉用户为什么他们会对IAP内容感兴趣。

*coalroller_service/controllers/main.py*

```
    def roll(self, account_token):
        # the service key *is a secret*, it should not be committed in
        # the source
        service_key = http.request.env['ir.config_parameter'].sudo().get_param('coalroller.service_key')

        # we charge 1 credit for 10 seconds of CPU
        cost = 1
        # TODO: allow the user to specify how many (tens of seconds) of CPU they want to use
        with charge(http.request.env, service_key, account_token, cost,
                    description="We're just obeying orders",
                    credit_template='coalroller_service.no_credit'):

            # 10 seconds of CPU per credit
            end = time.time() + (10 * cost)
```

*coalroller_service/views/no-credit.xml*

```
<odoo>
  <template id="no_credit" name="No credit warning">
    <div>
      <div class="container-fluid">
        <div class="row">
          <div class="col-md-7 offset-lg-1 mt32 mb32">
            <h2>Consume electricity doing nothing useful!</h2>
            <ul>
              <li>Heat our state of the art data center for no reason</li>
              <li>Use multiple watts for only 0.1€</li>
              <li>Roll coal without going outside</li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </template>
</odoo>
```

*coalroller_service/__manifest__.py*

```
    'name': "Coal Roller Service",
    'category': 'Tools',
    'depends': ['iap'],
    'data': [
        'views/no-credit.xml',
    ],
}
```

## [JSON-RPC2](https://www.jsonrpc.org/specification) 交易API

![img](https://www.odoo.com/documentation/13.0/_images/flow.png)

- IAP 交易 API 不要求在实现服务端网关时使用Odoo，调用是标准的 [JSON-RPC2](https://www.jsonrpc.org/specification)。
- 调用使用不同的*端点* 但对所有端点是相同的*方法* (`call`)。
- 异常以 [JSON-RPC2](https://www.jsonrpc.org/specification) 错误的形式返回，正式的异常名可在程序操作中通过 `data.name` 获取。

### 授权

###### `/iap/1/authorize`

验证用户的账户中至少有可用 `积分` 并对该金额创建一个 *预授权(等待交易) 。*

任意当前由进行中交易所预授权的金额都进一步的授权调用都是不可用的。

返回 [`TransactionToken`](https://alanhou.org/odoo-13-in-app-purchase/#TransactionToken) 标识进行中的交易为可用于捕获 (确认) 或取消所涉及的交易。

参数

- **key** ([`ServiceKey`](https://alanhou.org/odoo-13-in-app-purchase/#ServiceKey)) –
- **account_token** ([`UserToken`](https://alanhou.org/odoo-13-in-app-purchase/#UserToken)) –
- **credit** ([`float`](https://docs.python.org/3/library/functions.html#float)) –
- **description** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 可选，帮助用户知晓账户中扣费的原因
- **dbuuid** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – 可选，让用户在其数据库有效时利用试用积分 (参见 [注册服务](https://alanhou.org/odoo-13-in-app-purchase/#register-service))

若授权成功，返回

[`TransactionToken`](https://alanhou.org/odoo-13-in-app-purchase/#TransactionToken)

若服务令牌无效，抛出

```
AccessError
```

若账户无足够余额，抛出

[`InsufficientCreditError`](https://alanhou.org/odoo-13-in-app-purchase/#odoo.addons.iap.models.iap.InsufficientCreditError)

若`credit`值不为整型或浮点型，抛出

```
TypeError
r = requests.post(ODOO + '/iap/1/authorize', json={
    'jsonrpc': '2.0',
    'id': None,
    'method': 'call',
    'params': {
        'account_token': user_account,
        'key': SERVICE_KEY,
        'credit': 25,
        'description': "Why this is being charged",
    }
}).json()
if 'error' in r:
    # handle authorize error
tx = r['result']

# provide your service here
```

### 捕获

###### `/iap/1/capture`

确定具体的交易，将保留的积分从用户账户转账到服务提供方账户。

捕获调用是幂等的：对已捕获交易执行捕获调用不会有任何效果。

参数

- **token** ([`TransactionToken`](https://alanhou.org/odoo-13-in-app-purchase/#TransactionToken)) –
- **key** ([`ServiceKey`](https://alanhou.org/odoo-13-in-app-purchase/#ServiceKey)) –
- **credit_to_capture** ([`float`](https://docs.python.org/3/library/functions.html#float)) – 可选参数，用于捕获少于预授权的积分的金额

异常后抛出

```
AccessError
  r2 = requests.post(ODOO + '/iap/1/capture', json={
      'jsonrpc': '2.0',
      'id': None,
      'method': 'call',
      'params': {
          'token': tx,
          'key': SERVICE_KEY,
          'credit_to_capture': credit or False,
      }
  }).json()
  if 'error' in r:
      # handle capture error
  # otherwise transaction is captured
```

### 取消

###### `/iap/1/cancel`

取消具体的交易，释放所保留的用户积分。

取消调用是幂等的：对已取消调用执行捕获调用不会产生任何效果。

参数

- **token** ([`TransactionToken`](https://alanhou.org/odoo-13-in-app-purchase/#TransactionToken)) –
- **key** ([`ServiceKey`](https://alanhou.org/odoo-13-in-app-purchase/#ServiceKey)) –

异常抛出

```
AccessError
r2 = requests.post(ODOO + '/iap/1/cancel', json={
    'jsonrpc': '2.0',
    'id': None,
    'method': 'call',
    'params': {
        'token': tx,
        'key': SERVICE_KEY,
    }
}).json()
if 'error' in r:
    # handle cancel error
# otherwise transaction is cancelled
```

### 类型

异常放到一边，这些用于清晰化的*抽象类型*， 无需关心其底层实现。

###### `*class* ServiceName`

字符串标识你在 [https://iap.odoo.com](https://iap.odoo.com/) (生产环境)上的服务以及在客户端数据库中与服务相关联的账户。

###### `*class* ServiceKey`

为提供者的服务所生成的标识符。 每个键（及服务）与一个由服务提供者生成的固定值令牌相匹配。

多种类型的令牌对应多个服务。举个例子，SMS 和 MMS 可以是相同的服务 (一条 MMS 等价于多条SMS) or 或不同价格的不同服务。

### 🚫危险

服务密钥需要*保密，*泄漏服务密钥会让其它应用的开发者可以提取出你的服务所赚得的积分。

###### `*class* UserToken`

用于用户账户的标识符。

###### `*class* TransactionToken`

事务标识符，由授权过程返回并由捕获或取消交易进行消费。

###### `*exception* odoo.addons.iap.models.iap.InsufficientCreditError`

如果在账户中当前没有所请求的积分时在交易授权过程中抛出（没有足够积分或过多待完成交易/已有保留授权）。

###### `*exception* odoo.exceptions.AccessError`

由以下抛出：

- 如果服务令牌无效，对于要求有服务令牌的任意操作；
- 跨服务端调用的任意错误。(经常是在 `jsonrpc()`中)。

###### `*exception* odoo.exceptions.UserError`

在对应用开发者（也就是你）的审查时由任意预期外行为抛出。

### 测试API

为测试已开发应用， 我们推荐使用沙盒平台，这样你可以：

1. 从客户视角测试整个流程 - 可能产生的实际服务和交易。 (当然这要求修改端点，参见[服务](#iap-service)中的危险事项).
2. 测试API。

后者在具体的仅在**IAP沙盒**中生效的令牌中存在。

- 令牌 `000000`: 表示一个不存在的账户。在尝试授权时返回[`InsufficientCreditError`](https://alanhou.org/odoo-13-in-app-purchase/#odoo.addons.iap.models.iap.InsufficientCreditError) 。
- 令牌 `000111`: 表示执行任意服务积分不足的账户。在尝试授权时返回 [`InsufficientCreditError`](https://alanhou.org/odoo-13-in-app-purchase/#odoo.addons.iap.models.iap.InsufficientCreditError) 。
- 令牌 `111111`: 表示执行任意服务时具有足够积分的账户。尝试授权时会返回一个由捕获和取消路由所处理的交易令牌。

- 这些令牌仅在IAP - 沙盒服务端有效。
- 服务密钥在这个流程中完全被忽略。如果想要对服务运行健壮的测试，应当忽略这些令牌。

## Odoo帮助类

为方便起见，如果使用Odoo实现服务， `iap` 模块提供一些让 IAP 流程更为简单的帮助类。



### 收费

###### `*class* odoo.addons.iap.models.iap.charge(*env*, *key*, *account_token*, *credit*[, *dbuuid*, *description*, *credit_template*])`

用于授权及自动捕获或取消后台/代理使用的交易的*上下文管理器*。

运行方式类似于游标（cursor）上下文管理器：

- 立即通过具体的参数授权交易；
- 执行 `with` 体；
- 如果内容体完整执行且未报错，则捕获交易；
- 否则取消交易。

参数

- **env** (`odoo.api.Environment`) – 用于获取 `iap.endpoint` 配置密钥
- **key** ([`ServiceKey`](https://alanhou.org/odoo-13-in-app-purchase/#ServiceKey)) –
- **token** ([`UserToken`](https://alanhou.org/odoo-13-in-app-purchase/#UserToken)) –
- **credit** ([`float`](https://docs.python.org/3/library/functions.html#float)) –
- **description** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) –
- **模板 credit_template** (`Qweb`) –

```
  @route('/deathstar/superlaser', type='json')
  def superlaser(self, user_account,
                 coordinates, target,
                 factor=1.0):
      """
      :param factor: superlaser power factor,
                     0.0 is none, 1.0 is full power
      """
      credits = int(MAXIMUM_POWER * factor)
      description = "We will demonstrate the power of this station on your home planet of Alderaan."
      with charge(request.env, SERVICE_KEY, user_account, credits, description) as transaction:
          # TODO: allow other targets
          transaction.credit = max(credits, 2)
          # Sales ongoing one the energy price,
          # a maximum of 2 credits will be charged/captured.
          self.env['systems.planets'].search([
              ('grid', '=', 'M-10'),
              ('name', '=', 'Alderaan'),
          ]).unlink()
```

### 授权

###### `*class* odoo.addons.iap.models.iap.authorize(*env*, *key*, *account_token*, *credit*[, *dbuuid*, *description*, *credit_template*])`

将授权所有操作。

参数

- **env** (`odoo.api.Environment`) – 用于获取 `iap.endpoint` 配置密钥
- **key** ([`ServiceKey`](https://alanhou.org/odoo-13-in-app-purchase/#ServiceKey)) –
- **token** ([`UserToken`](https://alanhou.org/odoo-13-in-app-purchase/#UserToken)) –
- **credit** ([`float`](https://docs.python.org/3/library/functions.html#float)) –
- **description** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) –
- **template credit_template** (`Qweb`) –

```
  @route('/deathstar/superlaser', type='json')
  def superlaser(self, user_account,
                 coordinates, target,
                 factor=1.0):
      """
      :param factor: superlaser power factor,
                     0.0 is none, 1.0 is full power
      """
      credits = int(MAXIMUM_POWER * factor)
      description = "We will demonstrate the power of this station on your home planet of Alderaan."
      #actual IAP stuff
      transaction_token = authorize(request.env, SERVICE_KEY, user_account, credits, description=description)
      try:
          # Beware the power of this laser
          self.put_galactical_princess_in_sorrow()
      except Exception as e:
          # Nevermind ...
          r = cancel(env,transaction_token, key)
          raise e
      else:
          # We shall rule over the galaxy!
          capture(env,transaction_token, key, min(credits, 2))
```

### 取消

###### `*class* odoo.addons.iap.models.iap.cancel(*env*, *transaction_token*, *key*)`

将取消已授权交易。

参数

- **env** (`odoo.api.Environment`) – 用于获取 `iap.endpoint` 配置密钥
- **transaction_token** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) –
- **key** ([`ServiceKey`](https://alanhou.org/odoo-13-in-app-purchase/#ServiceKey)) –

```
  @route('/deathstar/superlaser', type='json')
  def superlaser(self, user_account,
                 coordinates, target,
                 factor=1.0):
      """
      :param factor: superlaser power factor,
                     0.0 is none, 1.0 is full power
      """
      credits = int(MAXIMUM_POWER * factor)
      description = "We will demonstrate the power of this station on your home planet of Alderaan."
      #actual IAP stuff
      transaction_token = authorize(request.env, SERVICE_KEY, user_account, credits, description=description)
      try:
          # Beware the power of this laser
          self.put_galactical_princess_in_sorrow()
      except Exception as e:
          # Nevermind ...
          r = cancel(env,transaction_token, key)
          raise e
      else:
          # We shall rule over the galaxy!
          capture(env,transaction_token, key, min(credits, 2))
```

### 捕获

###### `*class* odoo.addons.iap.models.iap.capture(*env*, *transaction_token*, *key*, *credit*)`

将对给定交易捕获取 `credit` 金额。

参数

- **env** (`odoo.api.Environment`) – 用于获取 `iap.endpoint` 配置密钥
- **transaction_token** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) –
- **key** ([`ServiceKey`](https://alanhou.org/odoo-13-in-app-purchase/#ServiceKey)) –
- **credit** –

```
  @route('/deathstar/superlaser', type='json')
  def superlaser(self, user_account,
                 coordinates, target,
                 factor=1.0):
      """
      :param factor: superlaser power factor,
                     0.0 is none, 1.0 is full power
      """
      credits = int(MAXIMUM_POWER * factor)
      description = "We will demonstrate the power of this station on your home planet of Alderaan."
      #actual IAP stuff
      transaction_token = authorize(request.env, SERVICE_KEY, user_account, credits, description=description)
      try:
          # Beware the power of this laser
          self.put_galactical_princess_in_sorrow()
      except Exception as e:
          # Nevermind ...
          r = cancel(env,transaction_token, key)
          raise e
      else:
          # We shall rule over the galaxy!
          capture(env,transaction_token, key, min(credits, 2))
```