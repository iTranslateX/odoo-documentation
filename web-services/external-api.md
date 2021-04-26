# 外部API

本文来自[Odoo 13官方文档之开发者文档](../README.md)系列文章

Odoo通常通过模型进行内部继承，但它的很多功能和数据也可由外部访问进行外部分析或不同工具的要变成。一部分 [模型引用](https://alanhou.org/odoo-13-orm-api/#reference-orm-model) API 可以轻松地通过 [XML-RPC](https://en.wikipedia.org/wiki/XML-RPC) 访问并可在很多操作语言中使用。

## 连接

### 配置

如果你已经安装了 Odoo服务，可以直接使用其参数。

### ⚠️警告

对于Odoo在线实例(<domain>.odoo.com)，创建的用户没有*本地*密码  (因为是通过Odoo在线认证系统而非实例本身来进行登录的)。要对Odoo在线实例使用XML-RPC，需要对所需的用户账户设置密码：

- 使用管理员账号登录实例
- 访问Settings ‣ Users ‣ Users
- 点击想要使用XML-RPC进行访问的用户
- 点击Change Password按钮
- 设置新密码并点击Change Password

*服务端url* 是实例的域名 (如 *https://mycompany.odoo.com*)，数据库名是实例的名称 (如 *mycompany*)。*username* 通过*Change Password* 所示配置用户的登录信息。

- Python 3

  ```
  url = <insert server URL>
  db = <insert database name>
  username = 'admin'
  password = <insert password for your admin user (default: admin)>
  ```

- Ruby

  ```
  url = <insert server URL>
  db = <insert database name>
  username = "admin"
  password = <insert password for your admin user (default: admin)>
  ```

- PHP

  ```
  $url = <insert server URL>;
  $db = <insert database name>;
  $username = "admin";
  $password = <insert password for your admin user (default: admin)>;
  ```

- Java

  ```
  final String url = <insert server URL>,
                db = <insert database name>,
          username = "admin",
          password = <insert password for your admin user (default: admin)>;
  ```

#### 演示

为让探讨更为简单，可以通过[https://demo.odoo.com](https://demo.odoo.com/) 获取一个测试数据库：

- Python 3

  ```
  import xmlrpc.client
  info = xmlrpc.client.ServerProxy('https://demo.odoo.com/start').start()
  url, db, username, password = \
      info['host'], info['database'], info['user'], info['password']
  ```

- Ruby

  ```
  require "xmlrpc/client"
  info = XMLRPC::Client.new2('https://demo.odoo.com/start').call('start')
  url, db, username, password = \
      info['host'], info['database'], info['user'], info['password']
  ```

- PHP

  ```
  require_once('ripcord.php');
  $info = ripcord::client('https://demo.odoo.com/start')->start();
  list($url, $db, $username, $password) =
    array($info['host'], $info['database'], $info['user'], $info['password']);
  ```

  > 这些示例使用了 [Ripcord](https://code.google.com/p/ripcord/) 库，它提供一种简单的XML-RPC API。 Ripcord要求在PHP安装中 [启用对XML-RPC 支持](https://php.net/manual/en/xmlrpc.installation.php) 。
  >
  > 因为调用通过 [HTTPS](https://en.wikipedia.org/wiki/HTTP_Secure)进行执行，还要求启用 [OpenSSL 插件](https://php.net/manual/en/openssl.installation.php)。

- Java

  ```
  final XmlRpcClient client = new XmlRpcClient();
  
  final XmlRpcClientConfigImpl start_config = new XmlRpcClientConfigImpl();
  start_config.setServerURL(new URL("https://demo.odoo.com/start"));
  final Map<String, String> info = (Map<String, String>)client.execute(
      start_config, "start", emptyList());
  
  final String url = info.get("host"),
                db = info.get("database"),
          username = info.get("user"),
          password = info.get("password");
  ```

  > 这些示例使用了 [Apache XML-RPC库](https://ws.apache.org/xmlrpc/)
  >
  > 示例没有包含导入语句，因其代码无法进行粘贴。

### 登录

Odoo要求API用户首先要进行认证才能查询其大多部分数据。

`xmlrpc/2/common` 端点提供无需认证的meta调用，如验证本身或获取版本信息。要在认证前验证连接信息是否正确，最简单的调用是查看服务端的版本。认证本身是通过`authenticate` 函数来完成，并返回在认证调用时使用的用户标识符 (`uid`) 而非登录信息。

- Python 3

  ```
  common = xmlrpc.client.ServerProxy('{}/xmlrpc/2/common'.format(url))
  common.version()
  {
      "server_version": "8.0",
      "server_version_info": [8, 0, 0, "final", 0],
      "server_serie": "8.0",
      "protocol_version": 1,
  }
  ```

  ```
  uid = common.authenticate(db, username, password, {})
  ```

- Ruby

  ```
  common = XMLRPC::Client.new2("#{url}/xmlrpc/2/common")
  common.call('version')
  {
      "server_version": "8.0",
      "server_version_info": [8, 0, 0, "final", 0],
      "server_serie": "8.0",
      "protocol_version": 1,
  }
  ```

  ```
  uid = common.call('authenticate', db, username, password, {})
  ```

- PHP

  ```
  $common = ripcord::client("$url/xmlrpc/2/common");
  $common->version();
  {
      "server_version": "8.0",
      "server_version_info": [8, 0, 0, "final", 0],
      "server_serie": "8.0",
      "protocol_version": 1,
  }
  ```

  ```
  $uid = $common->authenticate($db, $username, $password, array());
  ```

- Java

  ```
  final XmlRpcClientConfigImpl common_config = new XmlRpcClientConfigImpl();
  common_config.setServerURL(
      new URL(String.format("%s/xmlrpc/2/common", url)));
  client.execute(common_config, "version", emptyList());
  {
      "server_version": "8.0",
      "server_version_info": [8, 0, 0, "final", 0],
      "server_serie": "8.0",
      "protocol_version": 1,
  }
  ```

  ```
  int uid = (int)client.execute(
      common_config, "authenticate", asList(
          db, username, password, emptyMap()));
  ```

> 例如要查看是否可以读取 `res.partner` 模型，我们可以使用位置传递的`operation`关键字传递的`raise_exception`调用`check_access_rights`  (来获取true/false 结果，而非true/error):

## 调用方法

第二个端点是 `xmlrpc/2/object`，用于通过`execute_kw` RPC 函数调用odoo模型的方法。

每个对 `execute_kw` 的调用接收如下参数：

- 要使用的数据库，字符串
- 用户id (通过 `authenticate`获取)，整型
- 用户的密码，字符串
- 模型名，字符串
- 方法名，字符串
- 通过position传递的参数数组/列表
- 通过keyword (可选)传递的参数映射/字典

例如要查看我们是否可读取 `res.partner` 模型我们可以通过传递position给`operation`及传递keyword给`raise_exception`调用 `check_access_rights`  (从而获取true/false的结果，而非 true/error)：

- Python 3

  ```
  models = xmlrpc.client.ServerProxy('{}/xmlrpc/2/object'.format(url))
  models.execute_kw(db, uid, password,
      'res.partner', 'check_access_rights',
      ['read'], {'raise_exception': False})
  
  true
  ```

- Ruby

  ```
  models = XMLRPC::Client.new2("#{url}/xmlrpc/2/object").proxy
  models.execute_kw(db, uid, password,
      'res.partner', 'check_access_rights',
      ['read'], {raise_exception: false})
  
  true
  ```

- PHP

  ```
  $models = ripcord::client("$url/xmlrpc/2/object");
  $models->execute_kw($db, $uid, $password,
      'res.partner', 'check_access_rights',
      array('read'), array('raise_exception' => false));
  
  true
  ```

- Java

  ```
  final XmlRpcClient models = new XmlRpcClient() {{
      setConfig(new XmlRpcClientConfigImpl() {{
          setServerURL(new URL(String.format("%s/xmlrpc/2/object", url)));
      }});
  }};
  models.execute("execute_kw", asList(
      db, uid, password,
      "res.partner", "check_access_rights",
      asList("read"),
      new HashMap() {{ put("raise_exception", false); }}
  ));
  
  true
  ```

### 列出记录

记录可通过`search()`列出并过滤。

`search()` t接收必须的 [domain](https://alanhou.org/odoo-13-orm-api/#reference-orm-domains) 过滤器（可能为空），并返回所有匹配过滤器的记录的数据库标识符。例如要列出客户的公司：

- Python 3

  ```
  models.execute_kw(db, uid, password,
      'res.partner', 'search',
      [[['is_company', '=', True], ['customer', '=', True]]])
  
  [7, 18, 12, 14, 17, 19, 8, 31, 26, 16, 13, 20, 30, 22, 29, 15, 23, 28, 74]
  ```

- Ruby

  ```
  models.execute_kw(db, uid, password,
      'res.partner', 'search',
      [[['is_company', '=', true], ['customer', '=', true]]])
  
  [7, 18, 12, 14, 17, 19, 8, 31, 26, 16, 13, 20, 30, 22, 29, 15, 23, 28, 74]
  ```

- PHP

  ```
  $models->execute_kw($db, $uid, $password,
      'res.partner', 'search', array(
          array(array('is_company', '=', true),
                array('customer', '=', true))));
  
  [7, 18, 12, 14, 17, 19, 8, 31, 26, 16, 13, 20, 30, 22, 29, 15, 23, 28, 74]
  ```

- Java

  ```
  models.execute_kw(db, uid, password,
      'res.partner', 'search',
      [[['is_company', '=', True], ['customer', '=', True]]])
  
  [7, 18, 12, 14, 17, 19, 8, 31, 26, 16, 13, 20, 30, 22, 29, 15, 23, 28, 74]
  ```

#### 分页

默认搜索会返回所有匹配该条件的记录的id，它可以是很大的数量。 `offset` 和 `limit` 参数仅在获取所匹配记录的子集时可用。

- Python 3

  ```
  models.execute_kw(db, uid, password,
      'res.partner', 'search',
      [[['is_company', '=', True], ['customer', '=', True]]],
      {'offset': 10, 'limit': 5})
  
  [13, 20, 30, 22, 29]
  ```

- Ruby

  ```
  models.execute_kw(db, uid, password,
      'res.partner', 'search',
      [[['is_company', '=', true], ['customer', '=', true]]],
      {offset: 10, limit: 5})
  
  [13, 20, 30, 22, 29]
  ```

- PHP

  ```
  $models->execute_kw($db, $uid, $password,
      'res.partner', 'search',
      array(array(array('is_company', '=', true),
                  array('customer', '=', true))),
      array('offset'=>10, 'limit'=>5));
  
  [13, 20, 30, 22, 29]
  ```

- Java

  ```
  asList((Object[])models.execute("execute_kw", asList(
      db, uid, password,
      "res.partner", "search",
      asList(asList(
          asList("is_company", "=", true),
          asList("customer", "=", true))),
      new HashMap() {{ put("offset", 10); put("limit", 5); }}
  )));
  
  [13, 20, 30, 22, 29]
  ```

### 记录计数

不去获取可能很大的记录列表并进行计数，`search_count()` 可用于仅获取匹配查询的记录数。它接收相同的 [domain](https://alanhou.org/odoo-13-orm-api/#reference-orm-domains) 过滤器来作为 `search()`的参数。

- Python 3

  ```
  models.execute_kw(db, uid, password,
      'res.partner', 'search_count',
      [[['is_company', '=', True], ['customer', '=', True]]])
  ```

- Ruby

  ```
  models.execute_kw(db, uid, password,
      'res.partner', 'search_count',
      [[['is_company', '=', true], ['customer', '=', true]]])
  ```

- PHP

  ```
  $models->execute_kw($db, $uid, $password,
      'res.partner', 'search_count',
      array(array(array('is_company', '=', true),
                  array('customer', '=', true))));
  ```

- Java

  ```
  (Integer)models.execute("execute_kw", asList(
      db, uid, password,
      "res.partner", "search_count",
      asList(asList(
          asList("is_company", "=", true),
          asList("customer", "=", true)))
  ));
  ```

```
19
```

### ⚠️警告

调用`search` 然后调用 `search_count` (其反过来) 可能会在其他用户使用服务了时产生连续的结果：存储的数据可能会在调用间存在不同。

### 读取记录

记录数据可通过 `read()` 方法进行访问，它接收一个id列表 (由 `search()`返回) 以及一个可选的要获取的字段列表。默认，它会获取当前用户可以读取的所有字段，并很可能会是很大的数据量。

- Python 3

  ```
  ids = models.execute_kw(db, uid, password,
      'res.partner', 'search',
      [[['is_company', '=', True], ['customer', '=', True]]],
      {'limit': 1})
  [record] = models.execute_kw(db, uid, password,
      'res.partner', 'read', [ids])
  # count the number of fields fetched by default
  len(record)
  ```

- Ruby

  ```
  ids = models.execute_kw(db, uid, password,
      'res.partner', 'search',
      [[['is_company', '=', true], ['customer', '=', true]]],
      {limit: 1})
  record = models.execute_kw(db, uid, password,
      'res.partner', 'read', [ids]).first
  # count the number of fields fetched by default
  record.length
  ```

- PHP

  ```
  $ids = $models->execute_kw($db, $uid, $password,
      'res.partner', 'search',
      array(array(array('is_company', '=', true),
                  array('customer', '=', true))),
      array('limit'=>1));
  $records = $models->execute_kw($db, $uid, $password,
      'res.partner', 'read', array($ids));
  // count the number of fields fetched by default
  count($records[0]);
  ```

- Java

  ```
  final List ids = asList((Object[])models.execute(
      "execute_kw", asList(
          db, uid, password,
          "res.partner", "search",
          asList(asList(
              asList("is_company", "=", true),
              asList("customer", "=", true))),
          new HashMap() {{ put("limit", 1); }})));
  final Map record = (Map)((Object[])models.execute(
      "execute_kw", asList(
          db, uid, password,
          "res.partner", "read",
          asList(ids)
      )
  ))[0];
  // count the number of fields fetched by default
  record.size();
  ```

```
121
```

相反，仅提取三个有趣的字段。

- Python 3

  ```
  models.execute_kw(db, uid, password,
      'res.partner', 'read',
      [ids], {'fields': ['name', 'country_id', 'comment']})
  ```

- Ruby

  ```
  models.execute_kw(db, uid, password,
      'res.partner', 'read',
      [ids], {fields: %w(name country_id comment)})
  ```

- PHP

  ```
  $models->execute_kw($db, $uid, $password,
      'res.partner', 'read',
      array($ids),
      array('fields'=>array('name', 'country_id', 'comment')));
  ```

- Java

  ```
  asList((Object[])models.execute("execute_kw", asList(
      db, uid, password,
      "res.partner", "read",
      asList(ids),
      new HashMap() {{
          put("fields", asList("name", "country_id", "comment"));
      }}
  )));
  ```

```
[{"comment": false, "country_id": [21, "Belgium"], "id": 7, "name": "Agrolait"}]
```

即使没有请求 `id`字段，它也会一直被返回。

### 列出记录字段

`fields_get()` 可用于检查模型的字段并查看哪些字段可能会是所需的。

因为它返回大量的元信息 (也由客户端程序使用)，应在打印前进行过滤，大部分对人类用户有用的内容都是 `string` (字段的标签), `help` (如若存在为帮助文本) 和 `type` (了解要获取的是哪些值或在更新记录时进行发送)：

- Python 3

  ```
  models.execute_kw(
      db, uid, password, 'res.partner', 'fields_get',
      [], {'attributes': ['string', 'help', 'type']})
  ```

- Ruby

  ```
  models.execute_kw(
      db, uid, password, 'res.partner', 'fields_get',
      [], {attributes: %w(string help type)})
  ```

- PHP

  ```
  $models->execute_kw($db, $uid, $password,
      'res.partner', 'fields_get',
      array(), array('attributes' => array('string', 'help', 'type')));
  ```

- Java

  ```
  (Map<String, Map<String, Object>>)models.execute("execute_kw", asList(
      db, uid, password,
      "res.partner", "fields_get",
      emptyList(),
      new HashMap() {{
          put("attributes", asList("string", "help", "type"));
      }}
  ));
  ```

```
{
    "ean13": {
        "type": "char",
        "help": "BarCode",
        "string": "EAN13"
    },
    "property_account_position_id": {
        "type": "many2one",
        "help": "The fiscal position will determine taxes and accounts used for the partner.",
        "string": "Fiscal Position"
    },
    "signup_valid": {
        "type": "boolean",
        "help": "",
        "string": "Signup Token is Valid"
    },
    "date_localization": {
        "type": "date",
        "help": "",
        "string": "Geo Localization Date"
    },
    "ref_company_ids": {
        "type": "one2many",
        "help": "",
        "string": "Companies that refers to partner"
    },
    "sale_order_count": {
        "type": "integer",
        "help": "",
        "string": "# of Sales Order"
    },
    "purchase_order_count": {
        "type": "integer",
        "help": "",
        "string": "# of Purchase Order"
    },
```

### 搜索和读取

因为这是很常见的任务，Odoo提供了一个 `search_read()` 快捷方法，如其名称所表明，这是一个在`search()`后紧接着`read()`的方法，但它会避免执行两次查询，并且保存着 id。

它的参数类似于 `search()`的，但它也可以接收一个`fields`列表 (类似 `read()`，如未提供该列表，它将获取所有匹配字段的记录：

- Python 3

  ```
  models.execute_kw(db, uid, password,
      'res.partner', 'search_read',
      [[['is_company', '=', True], ['customer', '=', True]]],
      {'fields': ['name', 'country_id', 'comment'], 'limit': 5})
  ```

- Ruby

  ```
  models.execute_kw(db, uid, password,
      'res.partner', 'search_read',
      [[['is_company', '=', true], ['customer', '=', true]]],
      {fields: %w(name country_id comment), limit: 5})
  ```

- PHP

  ```
  $models->execute_kw($db, $uid, $password,
      'res.partner', 'search_read',
      array(array(array('is_company', '=', true),
                  array('customer', '=', true))),
      array('fields'=>array('name', 'country_id', 'comment'), 'limit'=>5));
  ```

- Java

  ```
  asList((Object[])models.execute("execute_kw", asList(
      db, uid, password,
      "res.partner", "search_read",
      asList(asList(
          asList("is_company", "=", true),
          asList("customer", "=", true))),
      new HashMap() {{
          put("fields", asList("name", "country_id", "comment"));
          put("limit", 5);
      }}
  )));
  ```

```
[
    {
        "comment": false,
        "country_id": [ 21, "Belgium" ],
        "id": 7,
        "name": "Agrolait"
    },
    {
        "comment": false,
        "country_id": [ 76, "France" ],
        "id": 18,
        "name": "Axelor"
    },
    {
        "comment": false,
        "country_id": [ 233, "United Kingdom" ],
        "id": 12,
        "name": "Bank Wealthy and sons"
    },
    {
        "comment": false,
        "country_id": [ 105, "India" ],
        "id": 14,
        "name": "Best Designers"
    },
    {
        "comment": false,
        "country_id": [ 76, "France" ],
        "id": 17,
        "name": "Camptocamp"
    }
]
```

### 创建记录

模型的记录使用 `create()`来进行创建。该方法将创建单条记录并返回其数据库标识符。

`create()` 接收一个字段对值的映射，用于初始化该记录。对于任意有默认值并未通过映射参数进行设置的字段，会使用字段默认值。

- Python 3

  ```
  id = models.execute_kw(db, uid, password, 'res.partner', 'create', [{
      'name': "New Partner",
  }])
  ```

- Ruby

  ```
  id = models.execute_kw(db, uid, password, 'res.partner', 'create', [{
      name: "New Partner",
  }])
  ```

- PHP

  ```
  $id = $models->execute_kw($db, $uid, $password,
      'res.partner', 'create',
      array(array('name'=>"New Partner")));
  ```

- Java

  ```
  final Integer id = (Integer)models.execute("execute_kw", asList(
      db, uid, password,
      "res.partner", "create",
      asList(new HashMap() {{ put("name", "New Partner"); }})
  ));
  ```

```
78
```

### ⚠️警告

大部分值的类型和预期的一致(integer 对就 `Integer`, string 对应 `Char` 或 `Text`),

- `Date`, `Datetime` 和 `Binary` 字段使用字符串值
- `One2many` 和 `Many2many` 使用一个`针对write方法文档`中详细讲解的特殊命令协议。

### 更新记录

记录可使用 `write()`进行更新，它接收一个待更新的记录列表和类似`create()`的字段对值的映射。

多条记录可同步进行更新，但它们将对所设置的字段获取相值。当前还不可以执行“computed”更新(即值可以根据已的记录值来进行设置)。

- Python 3

  ```
  models.execute_kw(db, uid, password, 'res.partner', 'write', [[id], {
      'name': "Newer partner"
  }])
  # get record name after having changed it
  models.execute_kw(db, uid, password, 'res.partner', 'name_get', [[id]])
  ```

- Ruby

  ```
  models.execute_kw(db, uid, password, 'res.partner', 'write', [[id], {
      name: "Newer partner"
  }])
  # get record name after having changed it
  models.execute_kw(db, uid, password, 'res.partner', 'name_get', [[id]])
  ```

- PHP

  ```
  $models->execute_kw($db, $uid, $password, 'res.partner', 'write',
      array(array($id), array('name'=>"Newer partner")));
  // get record name after having changed it
  $models->execute_kw($db, $uid, $password,
      'res.partner', 'name_get', array(array($id)));
  ```

- Java

  ```
  models.execute("execute_kw", asList(
      db, uid, password,
      "res.partner", "write",
      asList(
          asList(id),
          new HashMap() {{ put("name", "Newer Partner"); }}
      )
  ));
  // get record name after having changed it
  asList((Object[])models.execute("execute_kw", asList(
      db, uid, password,
      "res.partner", "name_get",
      asList(asList(id))
  )));
  ```

```
[[78, "Newer partner"]]
```

### 删除记录

可以通过向 `unlink()`传递 id 来指删除记录。

- Python 3

  ```
  models.execute_kw(db, uid, password, 'res.partner', 'unlink', [[id]])
  # check if the deleted record is still in the database
  models.execute_kw(db, uid, password,
      'res.partner', 'search', [[['id', '=', id]]])
  ```

- Ruby

  ```
  models.execute_kw(db, uid, password, 'res.partner', 'unlink', [[id]])
  # check if the deleted record is still in the database
  models.execute_kw(db, uid, password,
      'res.partner', 'search', [[['id', '=', id]]])
  ```

- PHP

  ```
  $models->execute_kw($db, $uid, $password,
      'res.partner', 'unlink',
      array(array($id)));
  // check if the deleted record is still in the database
  $models->execute_kw($db, $uid, $password,
      'res.partner', 'search',
      array(array(array('id', '=', $id))));
  ```

- Java

  ```
  models.execute("execute_kw", asList(
      db, uid, password,
      "res.partner", "unlink",
      asList(asList(id))));
  // check if the deleted record is still in the database
  asList((Object[])models.execute("execute_kw", asList(
      db, uid, password,
      "res.partner", "search",
      asList(asList(asList("id", "=", 78)))
  )));
  ```

```
[]
```

### 检查和自省

我们此前使用了 `fields_get()` 来查询模型并从一开始就使用了一个任意的模型， Odoo将大部分模型元数据存储在一小部分允许通过XML-RPC即时地查询系统或更改模型或字段（存在一些限制）的元模型中。



#### `ir.model`

通过不同字段提供有关Odoo模型的信息

- `name`

  可供人类阅读的模型描述

- `model`

  系统中每个模型的名称

- `state`

  模型是在Python 代码(`base`) 中 生成或创建一条 `ir.model` 记录 (`manual`)

- `field_id`

  通过对于 [ir.model.fields](#reference-webservice-inspection-fields)的`One2many`的模型字段列表

- `view_ids`

  针对模型定义的[Views](https://alanhou.org/odoo-13-views/#reference-views)的`One2many`

- `access_ids`

   设置于模型中的[Access Control](https://alanhou.org/odoo-13-security-odoo/#reference-security-acl) 的`One2many`关联

`ir.model` 可用于

- 查询系统获取已安装模型 （作为对模型操作的预查询或探查系统的内容）
- 获取有关具体模型的信息 (常通过列出与其关联的字段)
- 通过RPC动态新建模型

### ⚠️警告

- “自定义”模型名称必须以 `x_`开头
- 必须提供`state` 和`manual`，否则模型不会进行加载
- 不能对自定义模型新增*方法*，仅能新增字段

自定义模型一开始仅包含对所有模型可用的“内置”字段：

- Python 3

  ```
  models.execute_kw(db, uid, password, 'ir.model', 'create', [{
      'name': "Custom Model",
      'model': "x_custom_model",
      'state': 'manual',
  }])
  models.execute_kw(
      db, uid, password, 'x_custom_model', 'fields_get',
      [], {'attributes': ['string', 'help', 'type']})
  ```

- PHP

  ```
  $models->execute_kw(
      $db, $uid, $password,
      'ir.model', 'create', array(array(
          'name' => "Custom Model",
          'model' => 'x_custom_model',
          'state' => 'manual'
      ))
  );
  $models->execute_kw(
      $db, $uid, $password,
      'x_custom_model', 'fields_get',
      array(),
      array('attributes' => array('string', 'help', 'type'))
  );
  ```

- Ruby

  ```
  models.execute_kw(
      db, uid, password,
      'ir.model', 'create', [{
          name: "Custom Model",
          model: 'x_custom_model',
          state: 'manual'
      }])
  fields = models.execute_kw(
      db, uid, password, 'x_custom_model', 'fields_get',
      [], {attributes: %w(string help type)})
  ```

- Java

  ```
  models.execute(
      "execute_kw", asList(
          db, uid, password,
          "ir.model", "create",
          asList(new HashMap<String, Object>() {{
              put("name", "Custom Model");
              put("model", "x_custom_model");
              put("state", "manual");
          }})
  ));
  final Object fields = models.execute(
      "execute_kw", asList(
          db, uid, password,
          "x_custom_model", "fields_get",
          emptyList(),
          new HashMap<String, Object> () {{
              put("attributes", asList(
                      "string",
                      "help",
                      "type"));
          }}
  ));
  ```

```
{
    "create_uid": {
        "type": "many2one",
        "string": "Created by"
    },
    "create_date": {
        "type": "datetime",
        "string": "Created on"
    },
    "__last_update": {
        "type": "datetime",
        "string": "Last Modified on"
    },
    "write_uid": {
        "type": "many2one",
        "string": "Last Updated by"
    },
    "write_date": {
        "type": "datetime",
        "string": "Last Updated on"
    },
    "display_name": {
        "type": "char",
        "string": "Display Name"
    },
    "id": {
        "type": "integer",
        "string": "Id"
    }
}
```



#### `ir.model.fields`

提供有关Odoo模型的字段信息并允许无需使用Python字段即添加自定义字段

- `model_id`

   字段所属[ir.model](#reference-webservice-inspection-models) 的`Many2one`

- `name`

  字段的技术名称 (用于`read` 或 `write`)

- `field_description`

  字段的用户可读取标签(如`fields_get`中的 `string` )

- `ttype`

  要创建的字段 [类型](https://alanhou.org/odoo-13-orm-api/#reference-orm-fields)

- `state`

  字段是否通过Python 代码 (`base`) 或 `ir.model.fields` (`manual`)创建

- `required`, `readonly`, `translate`

  对字段启用对应的标记

- `groups`

  [字段级访问控制](https://alanhou.org/odoo-13-security-odoo/#reference-security-fields)， `res.groups`的`Many2many`

- `selection`, `size`, `on_delete`, `relation`, `relation_field`, `domain`

  针对类型的属性和自定义，参见 [字段文档](https://alanhou.org/odoo-13-orm-api/#reference-orm-fields) 了解更多详情

如同自定义模型，仅有通过通过`state="manual"` 新建的字段激活为模型中的实际字段。

### ⚠️警告

计算字段不能通过 `ir.model.fields`进行添加，一些字段元信息 (defaults, onchange) 也无法通过其设置

- Python 3

  ```
  id = models.execute_kw(db, uid, password, 'ir.model', 'create', [{
      'name': "Custom Model",
      'model': "x_custom",
      'state': 'manual',
  }])
  models.execute_kw(
      db, uid, password,
      'ir.model.fields', 'create', [{
          'model_id': id,
          'name': 'x_name',
          'ttype': 'char',
          'state': 'manual',
          'required': True,
      }])
  record_id = models.execute_kw(
      db, uid, password,
      'x_custom', 'create', [{
          'x_name': "test record",
      }])
  models.execute_kw(db, uid, password, 'x_custom', 'read', [[record_id]])
  ```

- PHP

  ```
  $id = $models->execute_kw(
      $db, $uid, $password,
      'ir.model', 'create', array(array(
          'name' => "Custom Model",
          'model' => 'x_custom',
          'state' => 'manual'
      ))
  );
  $models->execute_kw(
      $db, $uid, $password,
      'ir.model.fields', 'create', array(array(
          'model_id' => $id,
          'name' => 'x_name',
          'ttype' => 'char',
          'state' => 'manual',
          'required' => true
      ))
  );
  $record_id = $models->execute_kw(
      $db, $uid, $password,
      'x_custom', 'create', array(array(
          'x_name' => "test record"
      ))
  );
  $models->execute_kw(
      $db, $uid, $password,
      'x_custom', 'read',
      array(array($record_id)));
  ```

- Ruby

  ```
  id = models.execute_kw(
      db, uid, password,
      'ir.model', 'create', [{
          name: "Custom Model",
          model: "x_custom",
          state: 'manual'
      }])
  models.execute_kw(
      db, uid, password,
      'ir.model.fields', 'create', [{
          model_id: id,
          name: "x_name",
          ttype: "char",
          state: "manual",
          required: true
      }])
  record_id = models.execute_kw(
      db, uid, password,
      'x_custom', 'create', [{
          x_name: "test record"
      }])
  models.execute_kw(
      db, uid, password,
      'x_custom', 'read', [[record_id]])
  ```

- Java

  ```
  final Integer id = (Integer)models.execute(
      "execute_kw", asList(
          db, uid, password,
          "ir.model", "create",
          asList(new HashMap<String, Object>() {{
              put("name", "Custom Model");
              put("model", "x_custom");
              put("state", "manual");
          }})
  ));
  models.execute(
      "execute_kw", asList(
          db, uid, password,
          "ir.model.fields", "create",
          asList(new HashMap<String, Object>() {{
              put("model_id", id);
              put("name", "x_name");
              put("ttype", "char");
              put("state", "manual");
              put("required", true);
          }})
  ));
  final Integer record_id = (Integer)models.execute(
      "execute_kw", asList(
          db, uid, password,
          "x_custom", "create",
          asList(new HashMap<String, Object>() {{
              put("x_name", "test record");
          }})
  ));
  
  client.execute(
      "execute_kw", asList(
          db, uid, password,
          "x_custom", "read",
          asList(asList(record_id))
  ));
  ```

```
[
    {
        "create_uid": [1, "Administrator"],
        "x_name": "test record",
        "__last_update": "2014-11-12 16:32:13",
        "write_uid": [1, "Administrator"],
        "write_date": "2014-11-12 16:32:13",
        "create_date": "2014-11-12 16:32:13",
        "id": 1,
        "display_name": "test record"
    }
]
```