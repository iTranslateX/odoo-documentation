# 移动端JavaScript

本文来自[Odoo 13官方文档之开发者文档](../README.md)系列文章

## 导言

在Odoo 10.0中我们发布了移动端应用，可通过它访问所胡**Odoo app** (甚至是你自己的自定义模块)。

该应用是**Odoo Web** 和 **Native原生移动组件**的融合。换句话说这是在原生、移动、mobile容器中加载的Odoo Web实例。

本文讲解如何通过Odoo Web (借助JavaScript)访问移动原生组件如相机、振动、通知和Toast。为些，你不需要是一个移动开发者，如果了解Odoo JavaScript API就可以获取所有的移动端功能。

### ⚠️警告

这些功能仅在**Odoo 企业版10.0+** 中可使用

## 工作原理是什么？

移动端应用的内容工作原理如下：

![img](https://www.odoo.com/documentation/13.0/_images/mobile_working.jpg)当然，这是一个在移动原生Web容器中加载的网页。但集成的方式是你可以通过网页端JavaScript访问原生资源。

WebPages (Odoo Web)在每层的上部，其中第二层是Odoo Web (JS)和原生移动组件之间的桥梁。

在触发任意来自JavaScript的调用时，它通过桥梁传递并且桥梁将其传递给原生调用者来执行该动作。

在原生组件完成其任务时，它再次传递给桥梁并且你可以获取到JavaScript中的输出。

原生组件处理花费的时间取决于从原生资源所请求的内容。如相机或GPS定位。

## 如何使用？

类似于Odoo Web框架，可通过从**web_mobile.rpc**获取对象来在任意地方使用移动端API

![img](https://www.odoo.com/documentation/13.0/_images/odoo_mobile_api.png)移动端RPC对象提供一个可用的方法列表 (仅配合移动端应用使用)。

查看方法是否可用，然后执行它。

### 方法

每个方法返回一个JQuery延时对象，它返回一个数据JSON字典

#### 在设备中显示Toast

###### `showToast()`

参数

- **args** (`object`) – 显示的**消息**文本

toast提供有关小弹窗中操作的简单反馈。它仅填充消息所要求的空间且当前活动保持可见和交互性。

```
mobile.methods.showToast({'message': 'Message sent'});
```

![img](https://www.odoo.com/documentation/13.0/_images/toast.png)

#### 振动设备

###### `vibrate()`

参数

- **args** (`object`) – 在指定时间（按毫秒）内持续振动。

按照给定时长振动移动设备。

```
mobile.methods.vibrate({'duration': 100});
```

#### 显示带有动作的snackbar

###### `showSnackBar()`

参数

- **args** (`object`) – (*必传*)  在snackbar中显示的Snackbar及在Snackbar中的动作**按钮标签**（可选）

返回

若用户点击动作则为`True`，若在一段时间后SnackBar自动消失则为 `False` 。

Snackbar对操作提供轻量的反馈。它们在移动端屏幕的底部或在更大设备的左下角显示简短消息。Snackbar位于屏幕上所有其它元素的之上且一次只能显示一个。

```
mobile.methods.showSnackBar({'message': 'Message is deleted', 'btn_text': 'Undo'}).then(function(result){
        if(result){
                // 执行 undo 操作
        }else{
                // 取消Snack Bar
        }
});
```

![img](https://www.odoo.com/documentation/13.0/_images/snackbar.png)

#### 显示通知

###### `showNotification()`

参数

- **args** (`object`) – 标准通知中通知的**标题**(第一行) ，通知的**信息**(第二行)。

通知是一个在应用的常规UI之外对用户显示的信息。 在告知系统发布通知时，它首先在通知区以一个图标形式出现。要查看通知的详情，用户打开通知列表。通知区和通知列表都是系统控制的区域，用户可在任意时刻查看。

```
mobile.showNotification({'title': 'Simple Notification', 'message': 'This is a test for a simple notification'})
```

![img](https://www.odoo.com/documentation/13.0/_images/mobile_notification.png)

#### 在设备中创建联系人

###### `addContact()`

参数

- **args** (`object`) – 带有联系人详情的字典。可能的键有 (姓名, 手机号, 电话, 传真, email, 网址, 街道, 街道2, 国家id, 州id, 城市, 邮编, parent_id, 函数和图像)

使用给定的联系人详情新建一个设备联系人。

```
var contact = {
        'name': 'Michel Fletcher',
        'mobile': '9999999999',
        'phone': '7954856587',
        'fax': '765898745',
        'email': 'michel.fletcher@agrolait.example.com',
        'website': 'http://www.agrolait.com',
        'street': '69 rue de Namur',
        'street2': false,
        'country_id': [21, 'Belgium'],
        'state_id': false,
        'city': 'Wavre',
        'zip': '1300',
        'parent_id': [8, 'Agrolait'],
        'function': 'Analyst',
        'image': '<<BASE 64 Image Data>>'
}

mobile.methods.addContact(contact);
```

![img](https://www.odoo.com/documentation/13.0/_images/mobile_contact_create.png)

#### 扫描条形码

###### `scanBarcode()`

返回

通过任意条形码扫码`code`

条形码API实时在设备上监测任意方向的条形码。

条形码API可读取如下条形码格式：

- 1维条形码: EAN-13, EAN-8, UPC-A, UPC-E, Code-39, Code-93, Code-128, ITF, Codabar
- 2D条形码: 二维友, 数据矩阵, PDF-417, AZTEC

```
mobile.methods.scanBarcode().then(function(code){
        if(code){
                // 使用所扫描的编码执行操作
        }
});
```

#### 在设备中切换账户

###### `switchAccount()`

使用switchAccount在设备上从一个账户切换到另一个账户。

```
mobile.methods.switchAccount();
```

![img](https://www.odoo.com/documentation/13.0/_images/mobile_switch_account.png)