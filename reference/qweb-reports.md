# QWeb报表

- 本文来自[Odoo 13官方文档之开发者文档](../README.md)系列文章
- 以HTML/QWeb像在Odoo中其它普通视图那样编写报表。 你可以使用通常的[QWeb控制流工作](#reference-qweb) 。PDF渲染本身由 [wkhtmltopdf](https://wkhtmltopdf.org/)执行。如果你想要对某个模型创建报表，会需要定义这个[报表](#reference-reports-report)及其需要使用的[报表模板](#reference-reports-templates)。 如果你想要，也可以为报表指定一个特定的[纸张格式](#reference-reports-paper-formats)。最终，如果需要访问自己的模型之外的内容，可以定义一个给予你访问更多模型及模板中记录的权限的[自定义报表](#reference-reports-custom-reports)类。

## 报表

每个报表必须由 [报表动作](https://alanhou.org/odoo-13-actions/#reference-actions-report)声明。

为进行简化，`<report>`元素快捷方式在定义报表可以使用， 而非需要手动设置 [动作](https://alanhou.org/odoo-13-actions/#reference-actions-report)及其周边内容。这个 `<report>` 可以接收如下属性：

- `id`

  所生成记录的[外部id](https://www.odoo.com/documentation/13.0/glossary.html#term-external-id)

- `name` (必填)

  仅在某种列表中查找内容时用于报表的助记内容/描述

- `model` (必填)

  你的报表会相关的模型

- `report_type` (必填)

  PDF报表的`qweb-pdf` 或HTML的 `qweb-html`

- `report_name`

  你的报表名称(将会是PDF的输出名)

- `groups`

  允许浏览/使用当前报表的组的[`Many2many`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Many2many) 字段

- `attachment_use`

  若设置为True， 使用 `attachment`表达式所生成的名称存储为附件的报表；如果需要仅生成一次报表可以使用它 (例如出于法律原因)

- `attachment`

  定义报表名称的python 表达式；记录可作为 `object`变量访问

- `paperformat`

  希望使用的纸张格式的外部id(如未指定默认为公司的纸张格式)

例：

```
<report
    id="account_invoices"
    model="account.invoice"
    string="Invoices"
    report_type="qweb-pdf"
    name="account.report_invoice"
    file="account.report_invoice"
    attachment_use="True"
    attachment="(object.state in ('open','paid')) and
        ('INV'+(object.number or '').replace('/','')+'.pdf')"
/>
```



## 报表模板

### 最小化可选模板

最小化模板像下面这样：

```
<template id="report_invoice">
    <t t-call="web.html_container">
        <t t-foreach="docs" t-as="o">
            <t t-call="web.external_layout">
                <div class="page">
                    <h2>Report title</h2>
                    <p>This object's name is <span t-field="o.name"/></p>
                </div>
            </t>
        </t>
    </t>
</template>
```

调用`external_layout` 会在报表上添加头部和底部。PDF内容体是 `<div class="page">`内部的内容。 模板的 `id` 必须为在报表声明中指定的名称；例如对上面的报表为 `account.report_invoice` 。因为这是一个QWeb模板，你可以访问由模板接收的`docs`对象的所有字段。

有一些可在报表中访问的具体变量，主要有：

- `docs`

  针对当前报表的记录

- `doc_ids`

  针对 `docs`记录的id列表

- `doc_model`

  针对`docs` 记录的模型

- `time`

  对Python标准库 [`time`](https://docs.python.org/3/library/time.html#module-time)的引用

- `user`

  打印报表的用户的`res.user` 记录

- `res_company`

  当前 `user`的公司的记录

如果你希望访问模板中的记录/模型的其它记录，会需要用到[自定义报表](#reference-reports-custom-reports)。

### 可翻译模板

如果要翻译报表 (如翻译到伙伴的语言)，需要定义两个模板：

- 主报表模板
- 可翻译的文档

然后可以调用通过将属性设置 `t-lang` 为语言代码(例如`fr` 或 `en_US`)或是记录字段的主模板中的可翻译文档。如果使用可翻译的字段（如国家名、销售条件等）还需要通过相应的上下文重新浏览相关记录。

### ⚠️警告

如果报表模板不使用可翻译记录字段，不需要在另一种语言中重新浏览记录且这样会影响性能。

例如，我们来看一下Sale模块中的销售订单报表：

```
<!-- Main template -->
<template id="report_saleorder">
    <t t-call="web.html_container">
        <t t-foreach="docs" t-as="doc">
            <t t-call="sale.report_saleorder_document" t-lang="doc.partner_id.lang"/>
        </t>
    </t>
</template>

<!-- Translatable template -->
<template id="report_saleorder_document">
    <!-- Re-browse of the record with the partner lang -->
    <t t-set="doc" t-value="doc.with_context(lang=doc.partner_id.lang)" />
    <t t-call="web.external_layout">
        <div class="page">
            <div class="oe_structure"/>
            <div class="row">
                <div class="col-6">
                    <strong t-if="doc.partner_shipping_id == doc.partner_invoice_id">Invoice and shipping address:</strong>
                    <strong t-if="doc.partner_shipping_id != doc.partner_invoice_id">Invoice address:</strong>
                    <div t-field="doc.partner_invoice_id" t-options="{&quot;no_marker&quot;: True}"/>
                <...>
            <div class="oe_structure"/>
        </div>
    </t>
</template>
```

主模板调用可翻译模板，将 `doc.partner_id.lang` 作为 `t-lang` 参数，因此它会以伙伴的语言进行渲染。这样，每个订单会以对应客户的语言进行打印。如果希望仅翻译文档内容体，但保留头部和底部为默认语言，需要像下面这样调用报表的外部布局：

```
<t t-call="web.external_layout" t-lang="en_US">
```

注意这样仅在调用外部模板时起作用， 不能通过对`t-call`以外的xml节点设置 `t-lang`属性来翻译文档的部分内容。如果要翻译模板的部分内容，可以使用这一部分模板来创建外部模板并通过带`t-lang`属性的主模板调用它。

### 条形码

条形码为由控制器返回的图像，可轻易地借助QWeb语法来在报表中内嵌(例如参见[属性](#reference-qweb-attributes)):

```
<img t-att-src="'/report/barcode/QR/%s' % 'My text in qr code'"/>
```

可以查询字符串传递更多的参数：

```
<img t-att-src="'/report/barcode/?
    type=%s&value=%s&width=%s&height=%s'%('QR', 'text', 200, 200)"/>
```

### 实用备注

- 可在报表中使用Twitter Bootstrap和FontAwesome类

- 可在模板中直接放置本地CSS

- 可通过继承模板来在主报表中插入全局CSS并插入你自己的CSS：

  ```
  <template id="report_saleorder_style" inherit_id="report.style">
    <xpath expr=".">
      <t>
        .example-css-class {
          background-color: red;
        }
      </t>
    </xpath>
  </template>
  ```

- 如果在 PDF报表中缺失了样式，请查看[这些指南](https://alanhou.org/odoo-13-building-module/#reference-backend-reporting-printed-reports-pdf-without-styles)。



## 纸张格式

纸张格式是`report.paperformat` 记录并可包含如下属性：

- `name` (必传)

  仅用于在某种列表中查看报表的助记内容/描述

- `description`

  格式的简短描述

- `format`

  是预定义格式(A0到A9, B0到 B10, 法律文书, 信件, 公报,…) 或 `自定义`；默认为 A4。你果定义页面大小时不能使用非自定义格式。

- `dpi`

  输出DPI；默认为90

- `margin_top`, `margin_bottom`, `margin_left`, `margin_right`

  单位为mm的边框大小

- `page_height`, `page_width`

  单位为mm的页面大小

- `orientation`

  横向或纵向

- `header_line`

  显示头部线的布尔值

- `header_spacing`

  单位为mm的头部间距

例：

```
<record id="paperformat_frenchcheck" model="report.paperformat">
    <field name="name">French Bank Check</field>
    <field name="default" eval="True"/>
    <field name="format">custom</field>
    <field name="page_height">80</field>
    <field name="page_width">175</field>
    <field name="orientation">Portrait</field>
    <field name="margin_top">3</field>
    <field name="margin_bottom">3</field>
    <field name="margin_left">3</field>
    <field name="margin_right">3</field>
    <field name="header_line" eval="False"/>
    <field name="header_spacing">3</field>
    <field name="dpi">80</field>
</record>
```



## 自定义报表

报表模型有默认的`get_html`函数，会查找名为`report.*module.report_name*`的模型。如若存在，会使用它来调用QWeb引擎；否则会使用一个通用函数。如果希望通过在模板中包含更多内容来自定义报表 (例如像其它模型中记录)，可以定义这个模型，重写函数 `_get_report_values` 并在 `docargs` 字典中传递对象：

```
from odoo import api, models

class ParticularReport(models.AbstractModel):
    _name = 'report.module.report_name'

    @api.model
    def _get_report_values(self, docids, data=None):
        report_obj = self.env['ir.actions.report']
        report = report_obj._get_report_from_name('module.report_name')
        docargs = {
            'doc_ids': docids,
            'doc_model': report.model,
            'docs': self,
        }
        return docargs
```



## 自定义字体

If you want to use如果希望使用自定义字段则需要向`web.reports_assets_common`资源包添加自定义字体相相关联的 less/CSS。向`web.assets_common` 或 `web.assets_backend`添加自定义字体不会让该字体在QWeb可用。

例：

```
<template id="report_assets_common_custom_fonts" name="Custom QWeb fonts" inherit_id="web.report_assets_common">
    <xpath expr="." position="inside">
        <link href="/your_module/static/src/less/fonts.less" rel="stylesheet" type="text/less"/>
    </xpath>
</template>
```

即使在（除`web.reports_assets_common`以外的）其它资源包中使用过，也需要在这个less文件中定义 `@font-face`。

例：

```
@font-face {
    font-family: 'MonixBold';
    src: local('MonixBold'), local('MonixBold'), url(/your_module/static/src/fonts/MonixBold-Regular.otf) format('opentype');
}

.h1-title-big {
    font-family: MonixBold;
    font-size: 60px;
    color: #3399cc;
}
```

在将这一 less添加到资源包中以后就可以在自定义QWeb报表中使用该类了- 配合中为 `h1-title-big`。

## 报表是网页

报表通过report模块动态生成并可直接通过URL进行访问：

例如，可以通过http://<server-address>/report/html/sale.report_saleorder/38来以html模式访问销售订单报表

或者通过http://<server-address>/report/pdf/sale.report_saleorder/38访问pdf版本