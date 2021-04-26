# 会计本地化

完整文档：[Odoo 13官方文档之开发者文档](../README.md)

### ⚠️警告

本教程要求已了解有关在Odoo中构建模块的相关知识(参见[构建模块](https://alanhou.org/odoo-13-building-module/) )。

## 创建本地化模块

在安装 `accounting` 模块时，本地化模块对应公司自动安装的国家代码。如未设置国家代码或未找到本地化模块，默认会安装`l10n_generic_coa` (US) 本地化模块。

例如，`l10n_be` 会在公司的国家设置为`Belgium`时进行安装。

这一行为通过使用包含如下代码的 *.xml* 文件来进行获取：

```
<function model="account.chart.template" name="try_loading_for_current_company">
   <value eval="[ref(module.template_xmlid)]"/>
</function>
```

其中 `module.template_xmlid` 是相应模板的**完全限定**xmlid 。

通常位于 `data` 文件夹下，它必须在 `__manifest__.py` 文件的最后进行加载。

### 🚫危险

如果不存在该 *.xml* 文件，则不会及时加载正确的会计科目表！

## 配置自己会计科目表？

首先，在继续之前，我们需要讨论下模板。模板是一条允许自己被复制的记录。这一机制在操作多公司时需要用到。例如，新账户的创建通过 `account.account.template` 模型完成。但是，每个使用这个会计科目表的公司会被链接到一个以 `account.account` 作为模型的复制内容上。因此，该模板不会直接在公司中进行使用。

然后，如果需要安装会计科目表，所有依赖于它的模板会创建一个复制并将这个新生成的记录链接到公司用户。它表示所有这种模板必须以相同的方式链接到会计科目表上。要实现这点，每个必须使用 `chart_template_id` 字段引用所需要的会计科目表。出于这个原因，我们需要在创建其模板前定义一个 `account.chart.template` 模型的实例。

```
<record id="..." model="account.chart.template">
    <!-- [Required] Specify the name to display for this CoA. -->
    <field name="name">...</field>

    <!-- [Required] Specify the currency. E.g. "base.USD". -->
    <field name="currency_id" ref="..."/>

    <!-- [Required] Specify a prefix of the bank accounts. -->
    <field name="bank_account_code_prefix">...</field>

    <!-- [Required] Specify a prefix of the cash accounts. -->
    <field name="cash_account_code_prefix">...</field>

    <!-- [Optional] Define a parent chart of accounts that will be installed just before this one. -->
    <field name="parent_id" ref="..."/>

    <!-- [Optional] Define the number of digits of account codes. By default, the value is 6. -->
    <field name="code_digits">...</field>

    <!-- [Optional] Boolean to show or not this CoA on the list. By default, the CoA is visible.
     This field is mostly used when this CoA has some children (see parent_id field). -->
    <field name="visible" eval="..."/>

    <!-- [Optional] Boolean to enable the Anglo-Saxon accounting. By default, this field is False. -->
    <field name="use_anglo_saxon" eval="..."/>

    <!-- [Optional] Boolean to enable the complete set of taxes. By default, this field is True.
    This boolean helps you to choose if you want to propose to the user to encode the sale and purchase rates or choose from list of taxes.
    This last choice assumes that the set of tax defined on this template is complete. -->
    <field name="complete_tax_set" eval="..."/>

    <!-- [Optional] Specify the spoken languages.
    /!\ This option is only available if your module depends of l10n_multilang.
    You must provide the language codes separated by ';', e.g. eval="'en_US;ar_EG;ar_SY'". -->
    <field name="spoken_languages" eval="..."/>
</record>
```

例如，让我们来看一下比利时的会计科目表。

```
<record id="l10nbe_chart_template" model="account.chart.template">
    <field name="name">Belgian PCMN</field>
    <field name="currency_id" ref="base.EUR"/>
    <field name="bank_account_code_prefix">550</field>
    <field name="cash_account_code_prefix">570</field>
    <field name="spoken_languages" eval="'nl_BE'"/>
</record>
```

既然已创建了会计科目表，我们可以聚焦到模板创建身上了。如前所述，每条记录必须通过 `chart_template_id` 字段引用这条记录。若未引用，则模板会被忽略。下面的部分显示如何创建这些模板的详情。

### 向自己的会计科目表新增账户

是时候创建我们自己的账户了。它存在于 `account.account.template` 类型的创建记录中。每个 `account.account.template` 可以为每个公司创建一个 `account.account` 。

```
<record id="..." model="account.account.template">
    <!-- [Required] Specify the name to display for this account. -->
    <field name="name">...</field>

    <!-- [Required] Specify a code. -->
    <field name="code">...</field>

    <!-- [Required] Specify a type. -->
    <field name="user_type_id">...</field>

    <!-- [Required] Set the CoA owning this account. -->
    <field name="chart_template_id" ref="..."/>

    <!-- [Optional] Specify a secondary currency for each account.move.line linked to this account. -->
    <field name="currency_id" ref="..."/>

    <!-- [Optional] Boolean to allow the user to reconcile entries in this account. True by default. -->
    <field name="reconcile" eval="..."/>

    <!-- [Optional] Specify a group for this account. -->
    <field name="group_id" ref="...">

    <!-- [Optional] Specify some tags. -->
    <field name="tag_ids" eval="...">
</record>
```

以上所述的一些字段需要做更进一步的讲解。

`user_type_id`字段要求一个类型为`account.account.type`的值。虽然可以在本地化模块中创建一些其它类型，但我们推荐使用[account/data/data_account_type.xml](https://github.com/odoo/odoo/blob/13.0/addons/account/data/data_account_type.xml) 文件中的已有类型。这使用这些通用类型可确保你在本地化模块中创建的报表以外的通用报表可正确运行。

### ⚠️警告

避免使用流动资金 `account.account.type`！确实，银行及现金账户在安装本地化模块时直接创建，然后链接到 `account.journal`。

### ⚠️警告

仅一个 payable/receivable类型账户就够了。

虽然 `tag_ids` 字段为可选，但它是一个很强大的功能。其实它让你可以为账户定义不同的标签并在报表中进行正确的传播。例如，假定你想要创建一个带有多行的财务报告，但又没有方法去查找一个按照 `code` 或 `name`分配账户的规则 。解决方案就是使用标签，每个报表线有一个标签，按照需要进行传播和聚合。

和其它记录相似，标签可通过如下xml 结构进行创建：

```
<record id="..." model="account.account.tag">
    <!-- [Required] Specify the name to display for this tag. -->
    <field name="name">...</field>

    <!-- [Optional] Define a scope for this applicability.
    The available keys are 'accounts' and 'taxes' but 'accounts' is the default value. -->
    <field name="applicability">...</field>
</record>
```

你可能会想到，这个功能也可以用于计税。

`l10n_be` 模块中有如下示例：

```
<record id="a4000" model="account.account.template">
    <field name="name">Clients</field>
    <field name="code">4000</field>
    <field name="user_type_id" ref="account.data_account_type_receivable"/>
    <field name="chart_template_id" ref="l10nbe_chart_template"/>
</record>
```

### ⚠️警告

不要创建过多的账户：200-300个足够了。

### 向自己的会计科目表新增税务

要创建税务记录，只需要按照创建账户相同的流程就好了。唯一的不同是你必须使用 `account.tax.template` 模型。

```
<record id="..." model="account.tax.template">
    <!-- [Required] Specify the name to display for this tax. -->
    <field name="name">...</field>

    <!-- [Required] Specify the amount.
    E.g. 7 with fixed amount_type means v + 7 if v is the amount on which the tax is applied.
     If amount_type is 'percent', the tax amount is v * 0.07. -->
    <field name="amount" eval="..."/>

    <!-- [Required] Set the CoA owning this tax. -->
    <field name="chart_template_id" ref="..."/>

    <!-- [Required/Optional] Define an account if the tax is not a group of taxes. -->
    <field name="account_id" ref="..."/>

    <!-- [Required/Optional] Define an refund account if the tax is not a group of taxes. -->
    <field name="refund_account_id" ref="..."/>

    <!-- [Optional] Define the tax's type.
    'sale', 'purchase' or 'none' are the allowed values. 'sale' is the default value.
    Note: 'none' means a tax can't be used by itself, however it can still be used in a group. -->
    <field name="type_tax_use">...</field>

    <!-- [Optional] Define the type of amount:
    'group' for a group of taxes, 'fixed' for a tax with a fixed amount or 'percent' for a classic percentage of price.
    By default, the type of amount is percentage. -->
    <field name="amount_type" eval="..."/>

    <!-- [Optional] Define some children taxes.
    /!\ Should be used only with an amount_type with 'group' set. -->
    <field name="children_tax_ids" eval="..."/>

    <!-- [Optional] The sequence field is used to define order in which the tax lines are applied.
    By default, sequence = 1. -->
    <field name="sequence" eval="..."/>

    <!-- [Optional] Specify a short text to be displayed on invoices.
    For example, a tax named "15% on Services" can have the following label on invoice "15%". -->
    <field name="description">...</field>

    <!-- [Optional] Boolean that indicates if the amount should be considered as included in price. False by default.
    E.g. Suppose v = 132 and a tax amount of 20.
    If price_include = False, the computed amount will be 132 * 0.2 = 26.4.
    If price_include = True, the computed amount will be 132 - (132 / 1.2) = 132 - 110 = 22. -->
    <field name="price_include" eval="..."/>

    <!-- [Optional] Boolean to set to include the amount to the base. False by default.
     If True, the subsequent taxes will be computed based on the base tax amount plus the amount of this tax.
     E.g. suppose v = 100, t1, a tax of 10% and another tax t2 with 20%.
     If t1 doesn't affects the base,
     t1 amount = 100 * 0.1 = 10 and t2 amount = 100 * 0.2 = 20.
     If t1 affects the base,
     t1 amount = 100 * 0.1 = 10 and t2 amount = 110 * 0.2 = 22.  -->
    <field name="include_base_amount" eval="..."/>

    <!-- [Optional] Boolean false by default.
     If set, the amount computed by this tax will be assigned to the same analytic account as the invoice line (if any). -->
    <field name="analytic" eval="..."/>

    <!-- [Optional] Specify some tags.
    These tags must have 'taxes' as applicability.
    See the previous section for more details. -->
    <field name="tag_ids" eval="...">

    <!-- [Optional] Define a tax group used to display the sums of taxes in the invoices. -->
    <field name="tax_group_id" ref="..."/>

    <!-- [Optional] Define the tax exigibility either based on invoice ('on_invoice' value) or
    either based on payment using the 'on_payment' key.
    The default value is 'on_invoice'. -->
    <field name="tax_exigibility">...</field>

    <!-- [Optional] Define a cash basis account in case of tax exigibility 'on_payment'. -->
    <field name="cash_basis_account" red="..."/>
</record>
```

`l10n_pl` 模块中有如下示例：

```
<record id="vp_leas_sale" model="account.tax.template">
    <field name="chart_template_id" ref="pl_chart_template"/>
    <field name="name">VAT - leasing pojazdu(sale)</field>
    <field name="description">VLP</field>
    <field name="amount">1.00</field>
    <field name="sequence" eval="1"/>
    <field name="amount_type">group</field>
    <field name="type_tax_use">sale</field>
    <field name="children_tax_ids" eval="[(6, 0, [ref('vp_leas_sale_1'), ref('vp_leas_sale_2')])]"/>
    <field name="tag_ids" eval="[(6,0,[ref('l10n_pl.tag_pl_21')])]"/>
    <field name="tax_group_id" ref="tax_group_vat_23"/>
</record>
```

### 在会计科目表中新增账务状况

如需要了解有有关财务状况的更多详情以及其在Odoo中的运行方式，请参见please refer to [如何根据我的客户的状态及本地化调整计税](https://www.odoo.com/documentation/user/online/accounting/others/taxes/application.html)。

要新增财务状况，仅需要使用 `account.fiscal.position.template` 模型：

```
<record id="..." model="account.fiscal.position.template">
    <!-- [Required] Specify the name to display for this fiscal position. -->
    <field name="name">...</field>

    <!-- [Required] Set the CoA owning this fiscal position. -->
    <field name="chart_template_id" ref="..."/>

    <!-- [Optional] Add some additional notes. -->
    <field name="note">...</field>
</record>
```

### 向自己的会计科目表添加属性

在生成整个账户时，可以通过添加具体状况中所用的默认账户相对应的属性来重载新生成的会计科目表。必须在账户创建完成后 ，才能将每个账户与会计科目表进行关联。

```
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_xx_chart_template" model="account.chart.template">

        <!-- Define receivable/payable accounts. -->
        <field name="property_account_receivable_id" ref="..."/>
        <field name="property_account_payable_id" ref="..."/>

        <!-- Define categories of expense/income account. -->
        <field name="property_account_expense_categ_id" ref="..."/>
        <field name="property_account_income_categ_id" ref="..."/>

        <!-- Define input/output accounts for stock valuation. -->
        <field name="property_stock_account_input_categ_id" ref="..."/>
        <field name="property_stock_account_output_categ_id" ref="..."/>

        <!-- Define an account template for stock valuation. -->
        <field name="property_stock_valuation_account_id" ref="..."/>

        <!-- Define loss/gain exchange rate accounts. -->
        <field name="expense_currency_exchange_account_id" ref="..."/>
        <field name="income_currency_exchange_account_id" ref="..."/>

        <!-- Define a transfer account. -->
        <field name="transfer_account_id" ref="..."/>
    </record>
</odoo>
```

例如，让我们回归到比利时PCMN（Plan Comptable Minimum Normalisé标准化最低会计计划）。会计科目表以添加一些属性的方式进行重载。

```
<record id="l10nbe_chart_template" model="account.chart.template">
    <field name="property_account_receivable_id" ref="a4000"/>
    <field name="property_account_payable_id" ref="a440"/>
    <field name="property_account_expense_categ_id" ref="a600"/>
    <field name="property_account_income_categ_id" ref="a7010"/>
    <field name="expense_currency_exchange_account_id" ref="a654"/>
    <field name="income_currency_exchange_account_id" ref="a754"/>
    <field name="transfer_account_id" ref="trans"/>
</record>
```

## 如何新建一个银行操作模型？

Odoo中的银行操作模型如何运作? 参见[配置各方的模型](https://www.odoo.com/documentation/user/online/accounting/bank/reconciliation/configure.html)。

从 `V10`开始，在银行对账表组件中新增了一个功能：银行操作模型。这让用户可以通过单次点击预填写一些计账方。`account.reconcile.model.template` 记录的创建非常容易：

```
<record id="..." model="account.reconcile.model.template">
     <!-- [Required] Specify the name to display. -->
     <field name="name">...</field>

     <!-- [Required] Set the CoA owning this. -->
     <field name="chart_template_id" ref="..."/>

     <!-- [Optional] Set a sequence number defining the order in which it will be displayed.
     By default, the sequence is 10. -->
     <field name="sequence" eval="..."/>

     <!-- [Optional] Define an account. -->
     <field name="account_id" ref="..."/>

     <!-- [Optional] Define a label to be added to the journal item. -->
     <field name="label">...</field>

     <!-- [Optional] Define the type of amount_type, either 'fixed' or 'percentage'.
     The last one is the default value. -->
     <field name="amount_type">...</field>

     <!-- [Optional] Define the balance amount on which this model will be applied to (100 by default).
     Fixed amount will count as a debit if it is negative, as a credit if it is positive. -->
     <field name="amount">...</field>

     <!-- [Optional] Define eventually a tax. -->
     <field name="tax_id" ref="..."/>

     <!-- [Optional] The sames fields are available twice.
     To enable a second journal line, you can set this field to true and
     fill the fields accordingly. -->
     <field name="has_second_line" eval="..."/>
     <field name="second_account_id" ref="..."/>
     <field name="second_label">...</field>
     <field name="second_amount_type">...</field>
     <field name="second_amount">...</field>
     <field name="second_tax_id" ref="..."/>
</record>
```

## 如何新建一个动态报表？

如果需要对本地添加一些报表，要新建一个名为**l10n_xx_reports**的模块。此外，这模块必须放在 `enterprise` 仓库中，并且必须有两个依赖，一个是将所有内容加到你的本地化模块中，另一个是 `account_reports`，用于设计动态报表 。

```
'depends': ['l10n_xx', 'account_reports'],
```

一旦完成，就可以开始创建报表账单了。 文档请参见如下[s幻灯片](https://www.odoo.com/slides/slide/how-to-create-custom-accounting-report-415)。