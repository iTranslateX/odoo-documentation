# 数据库升级

本文来自[Odoo 13官方文档之开发者文档](../README.md)系列文章

## 引言

本文档讲解有关将Odoo数据库升级为更高版本的API。

允许在[https://upgrade.odoo.com](https://upgrade.odoo.com/)无需助还原html表单升级数据库。但数据库会按照表单中所述相同的过程进行处理。

所要求的步骤为：

- [ 创建请求](#upgrade-api-create-method)
- [上传数据库导出文件](#upgrade-api-upload-method)
- [运行升级处理](#upgrade-api-process-method)
- [获取数据库请求的状态](#upgrade-api-status-method)
- [下载已升级数据库导出文件](#upgrade-api-download-method)

## 方法



### 创建一个数据库升级请求

这个动作通过如下信息创建一个数据库请求：

- 你的合同备案号
- 你的邮箱地址
- 目标版本 （想要升级到的Odoo版本）
- 请求的目的（测试或生产）
- 数据库导出文件名（必须但仅用于提供信息）
- 可选服务端时区 (针对 Odoo 源版本 < 6.1)

#### `create` 方法

###### `https://upgrade.odoo.com/database/v1/create`

创建一个数据库升级请求

参数

- **contract** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – (必填) 你的企业合同备案号
- **email** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – (必填) 你的邮箱地址
- **target** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – (必填) 你想要升级到的Odoo版本。有效选项： 11.0, 12.0, 13.0
- **aim** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – (必填) 你升级数据库请求的目的。有效选项： test, production.
- **filename** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – (必填) 针对数据库导出文件的信息提示名称
- **timezone** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – (可选) 你的服务器所使用的时区。仅针对源版本< 6.1

返回 请求结果

返回类型 JSON字典

*create* 方法返回一个包含如下键的JSON字典：



##### `failures`

错误列表。

一个字典列表，每字典给出一个具体错误的相关信息。每个字典可包含依赖于不同类型错误的不同键，但其中总会包含 `reason` 和 `message` 这两个键：

- `reason`: 错误类型
- `message`: 易于阅读的消息

一些可能出现的键：

- `code`: 一个错误码
- `value`: 一个错误值
- `expected`: 一个错误值列表

参见如下示例：

- JSON

```
{
  "failures": [
    {
      "expected": [
        "11.0",
        "12.0",
        "13.0",
      ],
      "message": "Invalid value \"5.0\"",
      "reason": "TARGET:INVALID",
      "value": "5.0"
    },
    {
      "code": "M123456-abcxyz",
      "message": "Can not find contract M123456-abcxyz",
      "reason": "CONTRACT:NOT_FOUND"
    }
  ]
}
```

##### `request`

如果 *create* 方法执行成功的话，与 *request* 键相关联的键会是包含与创建请求相关的各种信息的字典：

最重要的键有：

- `id`: 请求id
- `key`: 这个请求的私钥

这两个值将会由其它方法（upload, process和status）所请求。

其它键会在描述 [status方法](#upgrade-api-status-method)的部分中进行讲解。

##### 示例脚本

这里有两个数据升级请求的示例，使用：

- 一个使用Python语言编写，用到了requests库
- 一个使用bash语言编写，用到了 [curl](https://curl.haxx.se/) (使用http传输数据的工具) and [jq](https://stedolan.github.io/jq) (JSON处理器)：

- Python 2

  ```
  import requests
  
  CREATE_URL = "https://upgrade.odoo.com/database/v1/create"
  CONTRACT = "M123456-abcdef"
  AIM = "test"
  TARGET = "12.0"
  EMAIL = "john.doe@example.com"
  FILENAME = "db_name.dump"
  
  fields = dict([
      ('aim', AIM),
      ('email', EMAIL),
      ('filename', DB_SOURCE),
      ('contract', CONTRACT),
      ('target', TARGET),
  ])
  
  r = requests.get(CREATE_URL, data=fields)
  print(r.text)
  ```

- Bash

  ```
  CONTRACT=M123456-abcdef
  AIM=test
  TARGET=12.0
  EMAIL=john.doe@example.com
  FILENAME=db_name.dump
  CREATE_URL="https://upgrade.odoo.com/database/v1/create"
  URL_PARAMS="contract=${CONTRACT}&aim=${AIM}&target=${TARGET}&email=${EMAIL}&filename=${FILENAME}"
  curl -sS "${CREATE_URL}?${URL_PARAMS}" > create_result.json
  
  # check for failures
  failures=$(cat create_result.json | jq -r '.failures[]')
  if [ "$failures" != "" ]; then
    echo $failures | jq -r '.'
    exit 1
  fi
  ```

   



### 上传数据库导出文件

有两种上传数据库导出文件的方法：

- `upload` 方法使用的是HTTPS协议
- `request_sftp_access` 方法使用的是SFTP协议

#### `upload` 方法

这是上传数据库导出文件最简单直接的方式。 它使用 HTTPS协议。

###### `https://upgrade.odoo.com/database/v1/upload`

上传一个数据库导出文件

参数

- **key** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – (必填) 你的私钥
- **request** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – (必填) 你的请求 id

返回 请求结果

返回类型 JSON字典

请求id和私钥通过使用 [create方法](#upgrade-api-create-method)获取

结果是一个包含 `failures`列表的JSON字典，如果一切正常的话它应当为空。

- Python 2

  ```
  import requests
  
  UPLOAD_URL = "https://upgrade.odoo.com/database/v1/upload"
  DUMPFILE = "/tmp/dump.sql"
  
  fields = dict([
      ('request', '10534'),
      ('key', 'Aw7pItGVKFuZ_FOR3U8VFQ=='),
  ])
  headers = {"Content-Type": "application/octet-stream"}
  
  with open(DUMPFILE, 'rb') as f:
      requests.post(UPLOAD_URL, data=f, params=fields, headers=headers)
  ```

- Bash

  ```
  UPLOAD_URL="https://upgrade.odoo.com/database/v1/upload"
  DUMPFILE="openchs.70.cdump"
  KEY="Aw7pItGVKFuZ_FOR3U8VFQ=="
  REQUEST_ID="10534"
  URL_PARAMS="key=${KEY}&request=${REQUEST_ID}"
  HEADER="Content-Type: application/octet-stream"
  curl -H $HEADER --data-binary "@${DUMPFILE}" "${UPLOAD_URL}?${URL_PARAMS}"
  ```

   



#### `request_sftp_access` 方法

本方法推荐对大的数据库导出文件使用。它使用SFTP协议并支持断点续传。

这将创建一个临时的SFTP服务器，可以对其进行连接并允许你使用SFTP客户端上传数据库导出文件。

###### `https://upgrade.odoo.com/database/v1/request_sftp_access`

创建一个SFTP服务端

参数

- **key** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – (必传) 你的私钥
- **request** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – (必传) 你的请求id
- **ssh_keys** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – (必传) 包含你想要使用的ssh公钥的文件路径

返回 请求结果

返回类型 JSON字典

请求id和私钥通过使用 [create方法](#upgrade-api-create-method)获取

包含ssh公钥的文件应当类似于标准的 `authorized_keys` 文件。该文件应仅包含公钥、空行或注释 (以 `#` 字符开头的行)

你的数据库升级请求应当会处于 `draft`（草稿）状态。

- Python 2

  ```
  import requests
  
  UPLOAD_URL = "https://upgrade.odoo.com/database/v1/request_sftp_access"
  SSH_KEY = "$HOME/.ssh/id_rsa.pub"
  SSH_KEY_CONTENT = open(SSH_KEY,'r').read()
  
  fields = dict([
      ('request', '10534'),
      ('key', 'Aw7pItGVKFuZ_FOR3U8VFQ=='),
      ('ssh_keys', SSH_KEY_CONTENT)
  ])
  
  r = requests.post(UPLOAD_URL, params=fields)
  print(r.text)
  ```

- Bash

  ```
  REQUEST_SFTP_ACCESS_URL="https://upgrade.odoo.com/database/v1/request_sftp_access"
  SSH_KEYS=/path/to/your/authorized_keys
  KEY="Aw7pItGVKFuZ_FOR3U8VFQ=="
  REQUEST_ID="10534"
  URL_PARAMS="key=${KEY}&request=${REQUEST_ID}"
  
  curl -sS "${REQUEST_SFTP_ACCESS_URL}?${URL_PARAMS}" -F ssh_keys=@${SSH_KEYS} > request_sftp_result.json
  
  # check for failures
  failures=$(cat request_sftp_result.json | jq -r '.failures[]')
  if [ "$failures" != "" ]; then
    echo $failures | jq -r '.'
    exit 1
  fi
  ```

`request_sftp_access` 方法返回一个包含如下键的 JSON 字典：

##### `failures`

错误列表。参见 [错误](#upgrade-api-json-failure) 部分获取有关在错误时返回的JSON 字典。

##### `request`

如果调用成功，与 *request* 键相关联的值将会是一个包含SFTP 连接参数的字典：

- `hostname`: 待连接的主机地址
- `sftp_port`: 待连接的端口
- `sftp_user`: 用于连接的SFTP用户
- `shared_file`: 你需要使用的文件名 (和在[create方法](#upgrade-api-create-method)中创建请求时所使用的 `filename` 值相同。)
- `request_id`: 相关联的升级请求id (仅提供信息，在进行连接时并非必须要有)
- `sample_command`:使用‘sftp’客户端的示例命令

通常应该能够按照原示例命令来进行连接。

你将仅能访问 `shared_file`。其它文件不可访问并且在SFTP服务器的共享环境下你将无法新建文件。

###### 使用‘sftp’客户端

一旦使用SFTP客户端进行了成功连接，你就可以上传数据库导出文件了。这里有一个使用‘sftp’ 客户端的示例会话：

```
$ sftp -P 2200 user_10534@upgrade.odoo.com
Connected to upgrade.odoo.com.
sftp> put /path/to/openchs.70.cdump openchs.70.cdump
Uploading /path/to/openchs.70.cdump to /openchs.70.cdump
sftp> ls -l openchs.70.cdump
-rw-rw-rw-    0 0        0          849920 Aug 30 15:58 openchs.70.cdump
```

如果你的连接被打断，你可以继续使用`-a` 命令行开关进行文件传输：

```
sftp> put -a /path/to/openchs.70.cdump openchs.70.cdump
Resuming upload of /path/to/openchs.70.cdump to /openchs.70.cdump
```

如果不想要手动输入命令，而是使用脚本自动化数据库升级，你可以使用一个批处理文件或者使用管道符将命令重定向到‘sftp’：

```
echo "put /path/to/openchs.70.cdump openchs.70.cdump" | sftp -b - -P 2200 user_10534@upgrade.odoo.com
```

`-b` 参数接收一个文件名。如果文件名为 `-`，它会从标准输入中读取命令。



### 要求处理请求

这个动作要求**升级平台**来处理你的数据库文件。

#### `process` 方法

###### `https://upgrade.odoo.com/database/v1/process`

处理数据库文件

参数

- **key** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – (必传) 你的私钥
- **request** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – (必传) 你的请求id

返回 请求结果

返回类型 JSON字典

请求id和私钥通过使用 [create方法](#upgrade-api-create-method)获取

结果是一个包含 `failures`列表的JSON字典，如果一切正常的话它应当为空。

- Python 2

  ```
  import requests
  
  PROCESS_URL = "https://upgrade.odoo.com/database/v1/process"
  
  fields = dict([
      ('request', '10534'),
      ('key', 'Aw7pItGVKFuZ_FOR3U8VFQ=='),
  ])
  
  r = requests.get(PROCESS_URL, data=fields)
  print(r.text)
  ```

- Bash

  ```
  PROCESS_URL="https://upgrade.odoo.com/database/v1/process"
  KEY="Aw7pItGVKFuZ_FOR3U8VFQ=="
  REQUEST_ID="10534"
  URL_PARAMS="key=${KEY}&request=${REQUEST_ID}"
  curl -sS "${PROCESS_URL}?${URL_PARAMS}"
  ```

   



### 请求跳过测试

这个动作要求**升级平台**对你的请求跳过测试。如果你不想要Odoo来测试和验证迁移，可以跳过测试步骤、直接获取迁移导出文件。

#### `skip_test` 方法

###### `https://upgrade.odoo.com/database/v1/skip_test`

跳过测试，传输已升级导出文件，并设置状态为‘delivered’（已投递）

参数

- **key** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – (必传) 你的私钥
- **request** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – (必传) 你的请求id

返回 请求结果

返回类型 JSON字典

请求id和私钥通过使用 [create方法](#upgrade-api-create-method)获取

结果是一个包含 `failures`列表的JSON字典，它应该在一切正常是为空。

- Python 2

  ```
  import requests
  
  PROCESS_URL = "https://upgrade.odoo.com/database/v1/skip_test"
  
  fields = dict([
      ('request', '10534'),
      ('key', 'Aw7pItGVKFuZ_FOR3U8VFQ=='),
  ])
  
  r = requests.get(PROCESS_URL, data=fields)
  print(r.text)
  ```

- Bash

  ```
  PROCESS_URL="https://upgrade.odoo.com/database/v1/skip_test"
  KEY="Aw7pItGVKFuZ_FOR3U8VFQ=="
  REQUEST_ID="10534"
  URL_PARAMS="key=${KEY}&request=${REQUEST_ID}"
  curl -sS "${PROCESS_URL}?${URL_PARAMS}"
  ```

   



### 获取请求状态

这个动作要求获取你的数据库升级请求的状态。

#### `status` 方法

###### `https://upgrade.odoo.com/database/v1/status`

获取数据库升级请求的状态

参数

- **key** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – (必传) 你的私钥
- **request** ([`str`](https://docs.python.org/3/library/stdtypes.html#str)) – (必传) 你的请求 id

返回 请求结果

返回类型 JSON字典

请求id和私钥通过使用 [create方法](#upgrade-api-create-method)获取

结果是一个包含有关数据库升级请求不同信息的JSON字典。

- Python 2

  ```
  import requests
  
  PROCESS_URL = "https://upgrade.odoo.com/database/v1/status"
  
  fields = dict([
      ('request', '10534'),
      ('key', 'Aw7pItGVKFuZ_FOR3U8VFQ=='),
  ])
  
  r = requests.get(PROCESS_URL, data=fields)
  print(r.text)
  ```

- Bash

  ```
  STATUS_URL="https://upgrade.odoo.com/database/v1/status"
  KEY="Aw7pItGVKFuZ_FOR3U8VFQ=="
  REQUEST_ID="10534"
  URL_PARAMS="key=${KEY}&request=${REQUEST_ID}"
  curl -sS "${STATUS_URL}?${URL_PARAMS}"
  ```

   

#### 示例输出

`request` 键包含有关请求的各种有用信息：

- `id`

  请求id

- `key`

  你的私钥

- `email`

  在创建请求时你所提供的email地址

- `target`

  在创建请求时你所提供的目标Odoo版本

- `aim`

  在创建请求时你所提供的数据库升级请求的目的（测试，生产）

- `filename`

  在创建请求时你所提供的文件名

- `timezone`

  在创建请求时你所提供的时区

- `state`

  你的请求的状态

- `issue_stage`

  在Odoo主服务器上我们所创建的事项阶段

- `issue`

  在Odoo主服务器上我们所创建的事项id

- `status_url`

  访问你的数据库升级请求html页面的URL

- `notes_url`

  获取有关数据库升级记录的URL

- `original_sql_url`

  用于获取你的已上传（非已升级）数据库来作为SQL流的URL

- `original_dump_url`

  用于获取你的已上传（非已升级）数据库来作为存档文件的URL

- `upgraded_sql_url`

  用于获取你的已升级数据库来作为SQL流的URL

- `upgraded_dump_url`

  用于获取你的已上升级数据库来作为存档文件的URL

- `modules_url`

  用于获取你的自定义模块的URL

- `filesize`

  你的已上传数据文件的大小

- `database_uuid`

  你的数据库的唯一ID

- `created_at`

  你创建请求时的日期

- `estimated_time`

  升级数据库所花费时间的预估

- `processed_at`

  数据库升级开启时的时间

- `elapsed`

  升级数据库所花费的时间

- `filestore`

  你的附件转化为了文件存储

- `customer_message`

  有关你的请求的重要消息

- `database_version`

  猜测的你所上传的（非升级后）Odoo版本。

- `postgresql`

  猜测的你所上传的（非升级后） PostgreSQL 版本

- `compressions`

  由你的已上传数据库所使用的压缩方法

- JSON

```
{
  "failures": [],
  "request": {
    "id": 10534,
    "key": "Aw7pItGVKFuZ_FOR3U8VFQ==",
    "email": "john.doe@example.com",
    "target": "12.0",
    "aim": "test",
    "filename": "db_name.dump",
    "timezone": null,
    "state": "draft",
    "issue_stage": "new",
    "issue": 648398,
    "status_url": "https://upgrade.odoo.com/database/eu1/10534/Aw7pItGVKFuZ_FOR3U8VFQ==/status",
    "notes_url": "https://upgrade.odoo.com/database/eu1/10534/Aw7pItGVKFuZ_FOR3U8VFQ==/upgraded/notes",
    "original_sql_url": "https://upgrade.odoo.com/database/eu1/10534/Aw7pItGVKFuZ_FOR3U8VFQ==/original/sql",
    "original_dump_url": "https://upgrade.odoo.com/database/eu1/10534/Aw7pItGVKFuZ_FOR3U8VFQ==/original/archive",
    "upgraded_sql_url": "https://upgrade.odoo.com/database/eu1/10534/Aw7pItGVKFuZ_FOR3U8VFQ==/upgraded/sql",
    "upgraded_dump_url": "https://upgrade.odoo.com/database/eu1/10534/Aw7pItGVKFuZ_FOR3U8VFQ==/upgraded/archive",
    "modules_url": "https://upgrade.odoo.com/database/eu1/10534/Aw7pItGVKFuZ_FOR3U8VFQ==/modules/archive",
    "filesize": "912.99 Kb",
    "database_uuid": null,
    "created_at": "2018-09-09 07:13:49",
    "estimated_time": null,
    "processed_at": null,
    "elapsed": "00:00",
    "filestore": false,
    "customer_message": null,
    "database_version": null,
    "postgresql": "9.4",
    "compressions": [
      "pgdmp_custom",
      "sql"
    ]
  }
}
```



### 下载数据库导出文件

除使用由[status方法](#upgrade-api-status-method)所提供的URL下载迁移后数据库外，你还可以按照[request_sftp_access 方法](#upgrade-api-request-sftp-access-method)中所述那样使用 SFTP协议

不同之处在于你将仅能下载迁移后的数据库。无法进行上传。

你的数据库升级请求应当处于 `done` 状态。

一旦你成功地使用SFTP 客户端进行了连接，就可以下载数据库导出文件。以下是一个使用‘sftp’ 客户端的示例会话：

```
$ sftp -P 2200 user_10534@upgrade.odoo.com
Connected to upgrade.odoo.com.
sftp> get upgraded_openchs.70.cdump /path/to/upgraded_openchs.70.cdump
Downloading /upgraded_openchs.70.cdump to /path/to/upgraded_openchs.70.cdump
```