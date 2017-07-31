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


### 发票二维码生成方式 （该接口有商户自己实现，为了对应离线二维码打印）
二维码生成接口, 用来根据给定的规则, 生成包含URL信息的二维码

 - 接口地址: {api_domain}/api/invoice/qrcode/v2
 - 访问方式：post
 - 参数说明:

名称|含义|类型|必填|备注
----|:---|:---|:--:|--------
length|二维码的大小|int|Y|正整数,用来控制二维码的大小
payway|支付通道唯一标识|string(20)|N|用于发票归集, 1:支付宝 3:微信 4:百度钱包 5:京东钱包 6:qq钱包 100:现金 101:银联卡 110:银行卡
terminal_sn|终端号|string|Y| 
client_sn|商户系统订单号|string|Y|必须在商户系统内唯一；且长度不超过32字节
client_time|商户系统订单完成时间|int|Y|timestamp,单位毫秒
total_amount|总金额(分)|int|Y|
auth|sn+" "+sign, 其中这里的 sign 是基于vender_sn 和 vender_key 的签名，签名方式可见[签名机制](https://wosai.gitbooks.io/e-invoice-doc/content/zh-cn/api/sign.html)|string|y|vender_sn + " " + sign 
url|{user_api_domain} + {uri_path}|string|Y|用户自己的支撑服务地址例如 `https://www.anycompany.com/invoice/preapply/h5`, 字符长度不超过100


 - 参数示例:

```javascript
{
    "length":200,
    "payway":"1,100",
    "terminal_sn":"10298371039",
    "client_sn": "22000000012",
    "client_time": "1488262165",
    "total_amount":10000,
    "auth":"xxxxxxxxxxxxxxxxxxxx"
    "url":"https://www.anycompany.com/invoice/preapply/h5"
}
```

```
需要注意, 二维码生成规范中的  sn 和 key 是 vender_sn 和 vender_key
```


生成二维码的Content是

```
{url}?terminal_sn={terminal_sn}&client_sn={client_sn}&client_time={client_time}&total_amount={total_amount}&auth={auth}
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

![](../../img/invoice_normal.png?raw=true)

![](../../img/invoice_error_user_return_fail.png?raw=true)

![](../../img/invoice_error_call_user_no_response.png?raw=true)



### 开票接口
 - 接口地址：{api_domain}/api/invoice/apply/v2
 - 访问方式：post
 - 参数说明：

名称|含义|类型|必填|备注
----|:---|:---|:--:|--------
terminal_sn|终端号|string|Y|
notify_url|开票请求回调地址|string|Y|
client_sn|商户系统订单号|string|Y|必须在商户系统内唯一；且长度不超过32字节
client_task_sn|商户系统开票任务流水号|string|Y|必须在商户系统内唯一; 且长度不超过32字节
client_time|商户系统订单完成时间|int|Y|timestamp,单位毫秒
business_type|开票对象业务类型|string(1)|N|默认：0。对于商家对个人开具，为0;对于商家对企业开具，为1;
invoice_type|开票类型|string(10)|Y|blue-蓝票(开蓝票);red-红票(红冲)
payee_name|开票方名称，公司名(如:XX商城)|string(100)|Y|海尔商城
payee_address|开票方地址(新版中为必传)|string(100)|Y|山东省青岛市
payee_bankaccount|开票方银行及 帐号|string(100)|N|92523123213412341234
payer_register_no|付款方税务登记证号。对企业开具电子发票时必填。目前北京地区暂未开放对企业开具电子发票，若北京地区放开后，对于向企业开具的情况，付款方税务登记证号和名称也不能为空,注：根据国家税务总局公告2017年第16号公告，2017年7月1日起，增值税普通发票必须填写纳税人识别号，否则无法作为企业内部报销凭证。|string(20)|N|2015020123123
payee_operator|开票人|string(8)|Y|小张
invoice_amount|开票金额； 当开红票时，该字段为负数, 单位为分 |string|Y|117000
invoice_memo|发票备注，有些省市会把此信息打印到PDF中|string(200)|N|电子发票测试
invoice_time|开票日期, 格式"YYYY-MM-DD HH:SS:MM"|Date|Y|2015-05-21 12:00:00
normal_invoice_code|原发票代码(开红票时传入)|string(12)|N|111100000000
normal_invoice_no|原发票号码(开红票时传入)|string(8)|N|00004349
payee_register_no|收款方税务登记证号|string(20)|Y|20150201321123
payer_address|消费者地址|string(100)|N|浙江省杭州市余杭区文一西路xxx号
payer_bankaccount|付款方开票开户银行及账号|string(100)|N|123412341234
payer_email|消费者电子邮箱|string|N|mytest@xxx.com
payer_name|付款方名称, 对应发票台头|string(100)|Y|付款方名称, 对应发票台头
payer_phone|消费者联系电话|string(20)|N|18234561212
sum_price|合计金额，不含税金额(新版中为必传) 当开红票时，该字段为负数,单位为分|string|Y|100000
sum_tax|合计税额 当开红票时，该字段为负数,单位为分|string|Y|17000
payee_checker|复核人|string(8)|N|小林
payee_receiver|收款人|string(8)|N|小张
payee_phone|收款方电话|string(20)|N|18234561212
invoice_items|开票商品明细信息|[]|N|参考开票明细信息
reflect|反射参数|string(64)|N|任何调用者希望原样返回的信息，可以用于关联商户ERP系统的订单或记录附加订单内容, 比如 { "tips": "200" }
payway|支付通道唯一标识|string(20)|N|用于发票归集, 1:支付宝 3:微信 4:百度钱包 5:京东钱包 6:qq钱包 100:现金 101:银联卡 110:银行卡
payer_uid|指定支付通道对应的唯一标识,比如银行卡号,支付宝id,微信open_id等|string(64)|N|支付通道唯一标识

   - 开票商品明细信息(invoice_items 中的每个 item)

名称|含义|类型|必填|备注
----|:---|:---|:--:|--------
item_no|商品税收编码|string|N|123456
item_name|发票项目名称（或商品名称）|string|Y|电视机
price|单价，单位为分,格式：10000。新版电子发票，折扣行此参数不能传，非折扣行必传；红票、蓝票都为正数|string|N|100.00
quantity|数量, 默认为1。新版电子发票，折扣行此参数不能传，非折扣行必传； 当开红票时，该字段需为负数|string|N|10
row_type|发票行性质。0表示正常行，1表示折扣行，2表示被折扣行。比如充电器单价100元，折扣10元，则明细为2行，充电器行性质为2，折扣行性质为1。如果充电器没有折扣，则值应为0|string|Y|0
specification|规格型号,可选|string|N|X100
sum_price|总价, 不含税金额，格式：100.00； 当开红票时，该字段为负数|string|Y|100000
tax|税额； 当开红票时，该字段为负数, 单位是分|string|Y|17000
tax_rate|税率。税率只能为0或0.03或0.04或0.06或0.11或0.13或0.17|string|Y|0.17
unit|单位。新版电子发票，折扣行不能传，非折扣行必传|string|N|台
amount|价税合计。(等于sumPrice和tax之和) 当开红票时，该字段为负数;单位是分|string|Y|117000
zero_rate_flag|0税率标识，只有税率为0的情况才有值，0=出口零税率，1=免税，2=不征收，3=普通零税率|string|N|1



 - 参数示例：

```javascript
{
    "terminal_sn": "2200000001",
    "client_sn": "22000000012",
    "client_task_sn": "22000009989",
    "client_time": "1488262165",
    "bussiness_type":"0",
    "invoice_amount": "117000",
    "sum_price": "100000",
    "sum_tax": "17000",
    "invoice_time": "2017-01-12 12:00:00",
    "invoice_type": "blue",
    "notify_url":"https://xxx.xxx.xxx/xxx/xxx/xxx",
    "payee_name":"海尔商城",
    "payee_address":"山东省青岛市",
    "payee_operator":"小张",
    "payer_name": "发票抬头",
    "payer_email": "user@example.com",
    "payer_phone": "18268888888",
    "payer_register_no": "9133010060913454XP",
    "invoice_items": [
        {
            "item_no": "1001",
            "item_name": "电视机",
            "price":"10000",
            "sum_price":"100000",
            "tax":"17000",
            "amount": "117000",
            "unit":"台",
            "tax_rate":"0.17",
            "quantity": "10",
            "zero_rate_flag": "1",
            "row_type":"0",
            "specification":"X100"
        }
    ]
}
```

 - 返回说明：

|字段名|字段含义|取值|必要|备注|
|----|:---|:---|:--:|--------|
|task_status|开票状态| |N| |
|client_sn|商户系统订单号|string|Y|必须在商户系统内唯一；且长度不超过32字节|
|client_task_sn|商户系统开票任务流水号|string|Y|必须在商户系统内唯一; 且长度不超过32字节|
|task_sn|开票任务唯一标识| |N|开票申请成功的时返回|
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
            "client_sn":"22000000012",
            "client_task_sn": "22000009989",
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
|client_task_sn|商户系统开票任务流水号|string|Y|必须在商户系统内唯一; 且长度不超过32字节|
|task_sn|开票任务唯一标识|string|N|开票申请成功的时返回|

 - 参数示例

```javascript
{
    "terminal_sn": "2200000001",
    "client_sn": "22000000012",
    "client_task_sn": "22000009989",
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
            "client_task_sn": "22000009989",
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
            "terminal_sn":"2200000001",
            "task_sn":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
            "client_sn": "22000000012",
            "client_task_sn": "22000009989",
            "finish_time": "1492506702864",
            "invoice_date": "2017-01-12",
            "invoice_code": "150003528888",
            "invoice_no": "50877603",
            "anti_fake_code": "CF6B2F6168420008",
            "invoice_amount": "117000",
            "file_path":"demo",
            "payer_name": "发票抬头",
            "payer_mobile": "18268888888",
            "payer_email":"123@qq.com",
            "payer_register_no": "9133010060913454XP",
            "invoice_type":"blue",
            "reflect":reflect_struct
        }
    }
}
```



### 开票结果主动通知, 回调(web hook)规范, (被回调的接口由商户实现并提供)
这里的地址 , 已在 开票接口的 notify_url 中传入

    商户提供接口接收 -> 收钱吧 开票成功（发票信息）或开票失败的通知信息。
    开票完成后，收钱吧调用商户的接口推送开票结果信息，推送频率为（1m/2m/10m/1h/2h/6h/12h/24h）共8次，直到商户返回 SUCCESS 或通知8次为止。

 - 回调接口地址：{user_callback_api_domain}/v2
 - 回调请求类型：post
 - 回调请求参数说明：


|名称|含义|类型|必填|备注|
|----|:---|:---|:--:|--------|
|client_sn|商户系统订单号|string|Y| |
|client_task_sn|商户系统开票任务流水号|string|Y|必须在商户系统内唯一; 且长度不超过32字节|
|task_sn|开票任务号|string|Y|开票的任务号|
|finish_time|通知时间|string|Y|"1492506702864"|
|channel_finish_time|通道完成时间|string|N|"1492506305637"|
|invoice_date|开票日期|string|Y|2017-01-12|
|invoice_code|发票代码|string|N|开票成功必传, 1231231234|
|invoice_no|发票编号|string|N|开票成功必传, 123123|
|device_no|税控设备编号(新版电子发票有)|string|N|sw1231|
|file_path|发票PDF的下载地址(仅在单个查询接口上显示，批量查询不显示)|string|N|demo|
|file_data_type|文件类型(pdf,jpg,png)|string|Y|jpg|
|ciphertext|发票密文，密码区的字符串|string|Y|demosdffsd-32432|
|anti_fake_code|防伪码|string|Y|CF6B2F6168420008|
|qr_code|二维码|string|Y|demo|
|payer_register_no|购买方纳税人识别号|varchar(20)|N| |
|payer_name|发票抬头名称|string(100)|N|开票成功必传|
|payer_mobile|购买方电话|string(16)|N| |
|reflect|反射参数|string(64)|N|任何调用者希望原样返回的信息，可以用于关联商户ERP系统的订单或记录附加订单内容, 比如 { "tips": "200" }|

- 回调参数示例：

```javascript
{
    "result_code": "200",
    "biz_response": {
        "result_code": "INVOICE_SUCCESS",
        "data": {
            "terminal_sn":"2200000001",
            "task_sn":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
            "client_sn": "22000000012",
            "client_task_sn": "22000009989",
            "finish_time": "1492506702864",
            "invoice_date": "2017-01-12",
            "invoice_code": "150003528888",
            "invoice_no": "50877603",
            "anti_fake_code": "CF6B2F6168420008",
            "invoice_amount": "117000",
            "file_path":"demo",
            "payer_name": "发票抬头",
            "payer_mobile": "18268888888",
            "payer_email":"123@qq.com",
            "payer_register_no": "9133010060913454XP",
            "invoice_type":"blue",
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


