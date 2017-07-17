## 前提
所有的请求和响应 必须都使用 utf-8 字符编码

喔噻平台的功能图
![](../../img/wosai_invoice_platform_arch.png?raw=true)


喔噻开票平台, 提供3种接口,分别为
1. 用户发票二维码生成, 用于访问开票前确认和补完信息的H5页面
2. 开票接口
3. 开票状态被动查询接口

以及 1 种, 主动推送的 Callback (webhook)
4. 开票结果主动通知 Callback


## 接入说明
 > * 第1步: 商户准备卖家纳税人识别号、开票卖家抬头、税率、商户简称、商户门店列表(商户门店唯一标识、商户简称、门店开票卖家抬头)；
 > * 第2步: 根据[对接前准备]("https://wosai.gitbooks.io/e-invoice-doc/content/zh-cn/business.html")开通自己的 开票应用,并获得 app_id
 > * 第3步: 提交商户资料，获取[商户平台]("s.shouqianba.com")登录账号和密码,在[对接前准备]("https://wosai.gitbooks.io/e-invoice-doc/content/zh-cn/business.html")
 通过
 [激活接口]("https://wosai.gitbooks.io/e-invoice-doc/content/zh-cn/api/interface/activate.html")
 [签到接口]("https://wosai.gitbooks.io/e-invoice-doc/content/zh-cn/api/interface/checkin.html")

  ```
  进行获取 terminal_sn 和 terminal_key
  ```

 > * 第4步:

 ```
 如果是接口开票, 直接阅读 开票接口, 不需要阅读 二维码生成接口

 如果需要通过生成二维码进行开票
 应在开票前先基于 <发票二维码生成接口> 构建开票预确认 http 网页地址, 然后基于此 http 地址生成的二维码打印在收银票据上；
 用户可以使用移动终端, 访问扫描这个二维码, 访问网页, 进行开票前确认或补完信息, 然后再通过页面的按钮操作进行开票
 (其中, 二维码中的 "开票预确认" 的网页, 由客户应用层自行开发, 收钱吧仅提供 二维码生成接口)
 ```

 > * 第5步: 开发接收通知的接口，订阅收钱吧推送的开票结果信息。


### 发票二维码生成接口
二维码生成接口, 用来根据给定的规则, 生成包含URL信息的二维码

 - 接口地址: {api_domain}/api/invoice/qrcode/v2
 - 访问方式：post
 - 参数说明:

名称|含义|类型|必填|备注
----|:---|:---|:--:|--------
length|二维码的大小|int|Y|正整数,用来控制二维码的大小
payway|支付通道唯一标识|string(20)|N|用于发票归集, 1:支付宝 3:微信 4:百度钱包 5:京东钱包 6:qq钱包
terminal_sn|终端号|string|Y| 
client_sn|商户系统订单号|string|Y|必须在商户系统内唯一；且长度不超过32字节
client_time|商户系统订单完成时间|int|Y|timestamp,单位毫秒
quantity|数量|int|Y|
url|{user_api_domain} + {uri_path}|string|Y|用户自己的支撑服务地址例如 `https://www.any.com/invoice/preapply/h5`, 字符长度不超过100

 - 参数示例:

```javascript
{
    "length":200,
    "terminal_sn":"10298371039",
    "client_sn": "22000000012",
    "client_time": "1488262165",
    "quantity":8,
    "sign":"xxxxxxxxxxxxxxxxxxxxxxxx"
    "url":"https://www.anycomany.com/invoice/preapply/h5"
}
```

生成二维码的Content是

```
{url}?terminal_sn={terminal_sn}&client_sn={client_sn}&client_time={client_time}&quantity={quantity}&sign={sign}
```

 - 返回示例

```javascript
{
    "result_code": "200",
    "biz_response": {
        "result_code": "SUCCESS",
        "data": {
            "image":"data:image/png;base64,xxxxxxxxxxxxxxxxxxxxx"
        }
    }
}

```



## 开票

开票的时序图
![](../../img/apply_seq_diagram.png?raw=true)


### 开票接口
 - 接口地址：{api_domain}/api/invoice/apply/v2
 - 访问方式：post
 - 参数说明：

|名称|含义|类型|必填|备注|
|----|:---|:---|:--:|--------|
|terminal_sn|终端号|string|Y| |
|notify_url|开票请求回调地址|string|Y| |
|client_sn|商户系统订单号|string|Y|必须在商户系统内唯一；且长度不超过32字节|
|client_time|商户系统订单完成时间|int|Y|timestamp,单位毫秒|
|invoice_amount|交易总金额|int|Y|单位为分|
|type|开票类型|string(2)|Y|B-蓝票(开蓝票);R-红票(红冲)
|items|开票商品明细信息|[]|N|参考开票明细信息|
|title_type|抬头类型|string(2)|N|0-个人;1-企业|
|title_name|抬头名称|string(80)|Y|付款方名称|
|user_mail|消费者邮箱|string(100)|Y|消费者邮箱|
|user_mobile|消费者联系方式|string(16)|Y|消费者电话号码|
|taxpayer_no|购买方纳税人识别号|string(20)|N|付款方名称为企业抬头时建议填写 注：根据国家税务总局公告2017年第16号公告，2017年7月1日起，增值税普通发票必须填写纳税人识别号，否则无法作为企业内部报销凭证。|
|user_bank_name|购买方开户行|string(100)|N|付款方开户行|
|user_bank_account|购买方开户行账户|string(25)|N|付款方开户行账|
|user_address|购买方地址|string(200)|N|付款方地址|
|invoice_remark|发票备注|string(200)|N|部分省份会要求|
|reflect|反射参数|string(64)|N|任何调用者希望原样返回的信息，可以用于关联商户ERP系统的订单或记录附加订单内容, 比如 { "tips": "200" }|
|payway|支付通道唯一标识|string(20)|N|用于发票归集|
|payer_uid|付款人ID|string(64)|N|支付平台（微信，支付宝）上的付款人ID, 样例:"2801003920293239230239"|
|payer_login|指定支付通道对应的唯一标识,比如银行卡号,支付宝账号,微信账号等|string(32)|N| |

   - 交易名细

|名称|含义|类型|必填|备注|
|----|:---|:---|:--:|--------|
|id|商品编号唯一标识|string(10)|Y|本次交易内商品唯一标识,退货时退货商品id需要对应该id|
|tax_no|商品税务映射编号|string(4)|Y|商户在收钱吧电子发票商户平台配置的商品税率(开票明细名称、单位、税率、默认数量、商品税控税务唯一标识)的编号|
|name|发票项目名称或商品名称|string(20)|N|如果传了，以传的值为准，没有传以tax_no对应的开票明细名称为准|
|num|商品数量|int|N|参数为空时，默认为1|
|zero_type|零税率标识|string(2)|N|只有税率为0的情况才有值，0=出口零税率，1=免税，2=不征收，3=普通零税率|
|row_type|发票行性质|string(10)|Y|0表示正常行，1表示折扣行，2表示被折扣行|
|item_amount|单项商品总价|int|Y|明细所有item_amount累加和等于总invoice_amount,单位为分|

 - 开票商品明细信息(items 中的每个 item)
 
|名称|含义|类型|必填|备注|
|----|:---|:---|:--:|--------|
|id|商品唯一标识|string(10)|Y|本次交易内唯一，退货时，退货商品的id需要对应该id|
|tax_no|商品税务映射编号|string(4)|Y|商户在收钱吧电子发票商户平台配置的商品税率(开票明细名称、单位、税率、默认数量、商品税控税务唯一标识)的编号|
|name|发票项目名称或商品名称|string(20)|N|如果传了，以传的值为准，没有传以tax_no对应的开票明细名称为准|
|num|商品数量|int|N|折扣行参数不能填，非折扣行必填|
|item_amount|单项商品总价|int|Y|单位为分|


 - 参数示例：

```javascript
{
    "terminal_sn": "2200000001",
    "client_sn": "22000000012",
    "client_time": "1488262165",
    "invoice_amount": 6000
    "notify_url":"https://xxx.xxx.xxx/xxx/xxx/xxx",
    "type": "B",
    "title_name": "发票抬头",
    "user_email": "user@example.com",
    "user_mobile": "18268888888",
    "taxpayer_no": "9133010060913454XP",
    "items": [
        {
            "id": "1",
            "tax_no": "1001",
            "name": "商品一",
            "num": "1",
            "row_type": "0",
            "item_amount": "4000"
        },
        {
            "id": "2",
            "tax_no": "1002",
            "name": "商品二",
            "num": "3",
            "row_type": "0",
            "item_amount": "2000"
        }
    ]
}
```

 - 返回说明：

|字段名|字段含义|取值|必要|备注|
|----|:---|:---|:--:|--------|
|task_status|开票状态| |N| |
|task_sn|开票任务唯一标识|N|开票申请成功的时返回|
|status|流水状态|见《业务结果码》|N| |
|reflect|反射参数|string(64)|N|任何调用者希望原样返回的信息，可以用于关联商户ERP系统的订单或记录附加订单内容, 比如 { "tips": "200" }|

 - 正常返回示例

```javascript
{
    "result_code": "200",
    "biz_response": {
        "result_code": "INVOICE_SUCCESS",
        "data": {
            "task_status": "INVOICE_APPLY_SUBMIT_SUCCESS",
            "status":"SUCCESS",
            "task_sn":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
            "reflect":reflect_struct
        }
    }
}

```

### 开票结果查询接口
 - 接口地址：{api_domain}/api/invoice/query/v2
 - 访问方式：post
 - 参数说明：

|名称|含义|类型|必填|备注|
|----|:---|:---|:--:|--------|
|terminal_sn|终端号|string|Y| |
|client_sn|商户系统订单号|string|N|必须在商户系统内唯一；且长度不超过32字节, client_sn和task_sn任意传一个|
|task_sn|开票任务唯一标识|N|开票申请成功的时返回|

 - 参数示例

```javascript
{
    "terminal_sn": "2200000001",
    "client_sn": "22000000012",
    "task_sn":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

}
```

 - 返回结果 进行中

```javascript
{
    "result_code": "200",
    "biz_response": {
        "result_code": "INVOICE_IN_PROGRESS",
        "data": {
            "task_sn":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
            "client_sn": "22000000012",
            "terminal_sn":"2200000001"
        }
    }
}
```

 - 返回结果 成功

```javascript
{
    "result_code": "200",
    "biz_response": {
        "result_code": "INVOICE_SUCCESS",
        "data": {
            "task_sn":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
            "client_sn": "22000000012",
            "timestamp": "1488262167",
            "einv_code": "150003528888",
            "einv_no": "50877603",
            "check_code": "59669422713395768932",
            "invoice_date": "2017-01-12",
            "invoice_amount": "100.00",
            "title_name": "发票抬头",
            "user_mobile": "18268888888",
            "user_email":"123@qq.com",
            "taxpayer_no": "9133010060913454XP",
            "type":"B"
        }
    }
}
```



### 开票结果主动通知, 回调(web hook)规范, (被回调的接口由商户实现并提供)
这里的地址 , 已在 开票接口的 notify_url 中传入

    商户提供接口接收 -> 收钱吧 开票成功（发票信息）或开票失败的通知信息。
    开票完成后，收钱吧调用商户的接口推送开票结果信息，推送频率为（1m/2m/10m/1h/2h/6h/12h/24h）共8次，直到商户返回 SUCCESS 或通知8次为止。

 - 回调接口地址：{user_callback_api_domain}/v2
 - 回调请求累心：post
 - 回调请求参数说明：


|名称|含义|类型|必填|备注|
|----|:---|:---|:--:|--------|
|task_sn|开票任务号|string(32)|Y|开票的任务号|
|client_sn|商户系统订单号|string(32)|Y| |
|finish_time|通知时间|string|Y|"1492506702864"|
|channel_finish_time|通道完成时间|string|Y|"1492506305637"|
|einv_code|发票代码|string(20)|N|开票成功必传|
|einv_no|发票编号|string(20)|N|开票成功必传|
|check_code|发票校验码|string(50)|N|开票成功必传|
|title_name|发票抬头名称|string(80)|N|开票成功必传|
|user_mobile|购买方电话|string(16)|N| |
|user_register_no|购买方纳税人识别号|varchar(20)|N| |
|reflect|反射参数|string(64)|N|任何调用者希望原样返回的信息，可以用于关联商户ERP系统的订单或记录附加订单内容, 比如 { "tips": "200" }|

- 回调参数示例：

```javascript
{
    "result_code": "200",
    "biz_response": {
        "result_code": "INVOICE_SUCCESS",
        "data": {
            "task_sn":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
            "client_sn": "22000000012",
            "finish_time": "1492506702864",
            "channel_finish_time":"1492506305637",
            "einv_code": "150003528888",
            "einv_no": "50877603",
            "check_code": "59669422713395768932",
            "invoice_date": "2017-01-12",
            "invoice_amount": "100.00",
            "title_name": "发票抬头",
            "user_mobile": "18268888888",
            "user_email":"123@qq.com",
            "taxpayer_no": "9133010060913454XP",
            "type":"B",
            "reflect":reflect_struct
        }
    }
}

```

- 回调返回说明：

 商户收到通知后应返回如下结果; 如果收钱吧未收到"成功"结果，或者没有调通回调接口, 会重试8次推送通知

- 回调返回示例：

```javascript
{
    "result_code": "200",
    "biz_response": {
        "result_code": "SUCCESS",
        "data": "接收开票信息成功"
    }
}
```


