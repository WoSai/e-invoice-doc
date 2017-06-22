
##1.接入说明
本文档用于说明商户如何通过交易参数输出来生成电子发票二维码，客户可以通过扫描商户生成的电子发票二维码来实现在线开票的功能。

要实现二维码生成，商户需要完成以下流程：
 > * 第一步：商户准备卖家纳税人识别号、开票卖家抬头、税率、商户简称、商户门店列表(商户门店唯一标识、商户简称、门店开票卖家抬头)；
 > * 第二步：基于第一步向ISV申请appid和密钥secret。注：为确保访问安全，商户应定期更新密钥；
 > * 第三步：在交易发生时基于<3.1申请电子发票>接口构建发票申请http网页地址，再基于此http地址生成二维码打印在收银票据上。
 > * 第四步：开发接收通知的接口，订阅收钱吧推送的开票结果信息。

##2.通用说明

###2.1 签到
    为了保证安全，由ISV提供给商户端的密钥应该定期更新，更新通过签到接口完成。
 - 接口地址：https://m.wosai.cn/api/sign/v1
 - 访问方式：post
 - 参数说明：

|名称|含义|类型|必填|备注|
|----|:---|:---|:--:|--------|
|appid|ISV分配给商户唯一标识|varchar(20)|Y| |
|sign|参数签名|varchar(32)|Y|详见2.2.如何构造签名|


 - 参数示例：
 
```javascript
{
    "appid":"2200000001",
    "sign":"MDYCGQCNTJHYA4JGHYUKSPMSE8JO33SQ"
}
```


 - 返回说明：

|名称|含义|类型|必填|备注|
|----|:---|:---|:--:|--------|
|code|响应消息代码|varchar(6)|Y|200为成功，其它为异常|
|message|错误消息提示|varchar(100)|N|code为非200时返回|
|data|新的密钥|varchar(32)|N|code为200时返回|



 - 返回示例：
 
```javascript
{
    "code":"200",
    "data":"9B6210772044610030068CDF2DCE35F3"
}
```

###2.2 如何构造签名
    签名是在进行接口调用或者参数输出生成二维码的过程中，利用密钥和参数进行MD5签名的过程，具体流程如下：
 - 签名说明：
接口调用或者生成二维码所有传递的参数都需要一起进行签名，假设本次参数包括appid、store\_sn、biz\_time、biz\_no、items等5个参数（其中items是数据类型），并且密钥secret=9B6210772044610030068CDF2DCE35F3，参数中appid=2200000001，store\_sn=5308，biz\_time=1468780992，biz\_no=61028309128301298，items=[{"id":"1001","name":"商品一"},{"id":"1002","name":"商品二"}]，则签名步骤如下：
```
第一步：参数及密钥拼装成9个元素存入到数组array= [5308=store_sn, 61028309128301298=biz_no, 2200000001=appid, 1468780992=biz_time, 1002=id[1], 9B6210772044610030068CDF2DCE35F3=secret, 1001=id[0], 商品二=name[1], 商品一=name[0]]；
第二步：对array数组进行排序得到sortArray= [1001=id[0], 1002=id[1], 1468780992=biz_time, 2200000001=appid, 5308=store_sn, 61028309128301298=biz_no, 9B6210772044610030068CDF2DCE35F3=secret, 商品一=name[0], 商品二=name[1]]；
第三步：使用连接符‘&’对数组进行拼接，得到字符串source="1001=id[0]&1002=id[1]&1468780992=biz_time&2200000001=appid&5308=store_sn&61028309128301298=biz_no&9B6210772044610030068CDF2DCE35F3=secret&商品一=name[0]&商品二=name[1]"；
第四步：对source进行‘md5’32位大写加密，得到sign=Upper(MD5(source))；
```

##3 发票
###3.1 申请电子发票
 - 接口地址：https://m.wosai.cn/api/invoice/apply/v1
 - 访问方式：get
 - 参数说明：

|名称|含义|类型|必填|备注|
|----|:---|:---|:--:|--------|
|appid|ISV分配给商户唯一标识|varchar(20)|Y| |
|channel|支付渠道|varchar(10)|N|cash/bank/alipay,用于发票归集|
|payer|用户支付渠道的唯一标识|varchar(10)|N|用于发票归集|
|store_sn|商户门店唯一标识|varchar(20)|Y| |
|biz_no|交易流水号|varchar(32)|Y| |
|biz_time|交易时间|varchar(10)|Y|以秒为单位的时间戳|
|amount|交易总金额|int|Y|单位为分|
|items|开票商品明细信息|[]|N|参考开票明细信息|
|expand|N|varchar(100)|扩展字段|调用通知接口时，该参数会原样返回|
|sign|参数签名|varchar(32)|Y|详见2.2.如何构造签名|


 - 开票商品明细信息

 
|名称|含义|类型|必填|备注|
|----|:---|:---|:--:|--------|
|id|商品唯一标识|varchar(10)|Y|本次交易内唯一，退货时，退货商品的id需要对应该id|
|tax_no|商品税务映射编号|varchar(4)|Y|商户在收钱吧电子发票商户平台配置的商品税率(开票明细名称、单位、税率、默认数量、商品税控税务唯一标识)的编号|
|name|发票项目名称或商品名称|varchar(20)|N|如果传了，以传的值为准，没有传以tax_no对应的开票明细名称为准|
|num|商品数量|int|N|折扣行参数不能填，非折扣行必填|
|item_amount|单项商品总价|int|Y|单位为分|


 - 参数示例：
https://m.wosai.cn/api/invoice/apply/v1?appid=2200000001&channel=alipay&store_sn=2200000011&biz_no=22000000012&biz_time=1488262165&amount=1000000&sign=MDYCGQCNTJhYa4JghYuksPMsE8jO33sq&items=[{"id":"1",”tax_no”:”2”,"name":"品类一","num":"1","item_amount":"11200"},{"id”:”2”,”tax_no”:”2”,”name":"品类二","num":"1","item_amount":"11200"}]

 - 返回说明：返回网页交互界面，用户在此交互界面确认订单信息和填写抬头。


###3.2 查询订单商品明细接口
    收钱吧调用商户提供的接口，查询订单的商品明细。
	用户提交开票请求时，收钱吧没有开票请求订单的商品明细，因此需要通过商户提供的“查询订单商品明细接口”查询订单商品明细，完成开票。

- 接口地址：{api_domain}/api/invoice/queryItems/v1
- 接口需商户提供
- 访问方式：post
- 请求参数说明：


|名称|类型|含义|必填|备注|
|----|:--:|:---|:---|--------|
|biz_no|varchar(32)|交易流水号|Y|交易流水号|
|sign|varchar(32)|参数签名|Y|参数签名验证|


- 参数示例：

```javascript
{
    "biz_no": "22000000012",
    "sign": "0AD7AE3494B0C16F8B7A5BAE7B21926A"
}
```

- 返回参数：

|名称|类型|含义|必填|备注|
|----|:--:|:---|:---|--------|
|biz_no|varchar(32)|交易流水号|Y|交易流水号|
|original_no|varchar(32)|原始交易流水号|N|原始交易流水号，订单退款时必返回|
|store_sn|varchar(20)|交易门店编号|Y|商户门店唯一标识|
|biz_time|varchar(10)|交易时间|Y|以秒为单位的时间戳|
|amount|int|交易总金额|Y|单位为分|
|type|varchar(2)|交易类型|Y|P-付款；R-退款|
|items|[]|交易商品明细|N|参考交易商品明细|
|remark|varchar(256)|备注|N| |

- 交易商品明细

|名称|含义|类型|必填|备注|
|----|:---|:---|:--:|--------|
|id|商品唯一标识|varchar(10)|Y| |
|tax_no|商品税务映射编号|varchar(4)|Y|商户在收钱吧电子发票商户平台配置的商品税率(开票明细名称、单位、税率、默认数量、商品税控税务唯一标识)的编号|
|name|发票项目名称或商品名称|varchar(20)|N|如果传了，以传的值为准，没有传以tax_no对应的开票明细名称为准|
|num|商品数量|int|N| |
|item_amount|单项商品总价|int|Y|单位为分|

- 返回示例：

```javascript
{
    "appid": "2200000001",
    "biz_no": "22000000012",
    "store_sn": "2200000011",
    "biz_time": "1488262165",
    "amount": "1000000",
    "type": "P",
    "items": [
        {
            "tax_no": "1001",
            "id": "1",
            "name": "商品一",
            "num": "1",
            "item_amount": "400000"
        },
        {
            "tax_no": "1002",
            "id": "2",
            "name": "商品二",
            "num": "1",
            "item_amount": "200000"
        },
        {
            "tax_no": "1003",
            "id": "3",
            "name": "商品三",
            "num": "1",
            "item_amount": "300000"
        },
        {
            "tax_no": "1004",
            "id": "4",
            "name": "商品四",
            "num": "1",
            "item_amount": "100000"
        }
    ]
}
```



###3.3 开票结果通知接口
    开票完成后，收钱吧调用商户的接口推送开票结果信息，推送频率为（1m/2m/10m/1h/2h/6h/12h/24h）共8次，直到商户返回SUCCESS或通知8次为止。

- 接口地址：{api_domain}/api/invoice/notify/v1
- 接口需商户提供
- 访问方式：post
- 请求参数说明：


|名称|类型|含义|必填|备注|
|----|:--:|:---|:---|--------|
|code|varchar(10)|成功与否标识|Y|SUCCESS / FAIL|
|message|varchar(100)|结果描述|Y|开票失败时为错误描述|
|biz_no|varchar(32)|开票交易流水号|Y|开票的交易流水号|
|original_no|varchar(32)|原始交易流水号|N|开红票时必传|
|timestamp|varchar(10)|通知时间|Y|以秒为单位的时间戳|
|einv_code|varchar(20)|发票代码|N|开票成功必传|
|einv_no|varchar(20)|发票编号|N|开票成功必传|
|check_code|varchar(50)|发票校验码|N|开票成功必传|
|title_name|varchar(80)|发票抬头名称|N|开票成功必传|
|user_mobile|varchar(16)|购买方电话|N| |
|user_register_no|varchar(20)|购买方纳税人识别号|N| |
|expand|varchar(100)|扩展字段|N|原样返回申请电子发票接口传的参数|

- 参数示例：

```javascript
{
    "code": "SUCCESS",
    "message": "开票成功",
    "biz_no": "22000000012",
    "timestamp": "1488262167",
    "einv_code": "150003528888",
    "einv_no": "50877603",
    "check_code": "59669422713395768932",
    "title_name": "发票抬头",
    "user_mobile": "18268888888",
    "user_register_no": "9133010060913454XP"
}

```

- 返回说明：

 商户收到通知后应返回字符串 SUCCESS ，收钱吧如果未收到SUCCESS，会重试8次推送通知。

- 返回示例：

```javascript
	SUCCESS 
```

