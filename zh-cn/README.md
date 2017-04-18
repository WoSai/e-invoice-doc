##1.接入说明
本文档用于说明商户如何通过交易参数输出来生成电子发票二维码，客户可以通过扫描商户生成的电子发票二维码来实现在线开票的功能。

要实现二维码生成，商户需要完成以下流程：
 > * 第一步：商户准备卖家纳税人识别号、开票卖家抬头、税率、商户简称、商户门店列表(商户门店唯一标识、商户简称、门店开票卖家抬头)；
 > * 第二步：基于第一步向ISV申请appid和密钥secret。注：为确保访问安全，商户应定期更新密钥；
 > * 第三步：在交易发生时基于<3.1申请电子发票>接口构建发票申请http网页地址，再基于此http地址生成二维码打印在收银票据上。

##2.通用说明

###2.1 签到
    为了保证安全，由ISV提供给商户端的密钥应该定期更新，更新通过签到接口完成。
 - 接口地址：https://m.wosai.cn/api/sign/v1
 - 访问方式：post
 - 参数说明：

|名称|含义|类型|必填|备注|
|----|:---|:---|:--:|--------|
|appid|ISV分配给商户唯一标识|varchar(20)|Y||
|sign|参数签名|varchar|Y|详见2.2.如何构造签名|


 - 参数示例：
 
```javascript
{
    "appid":"2200000001",
    "sign":"MDYCGQCNTJhYa4JghYuksPMsE8jO33sq"
}
```


 - 返回说明：

|名称|含义|类型|必填|备注|
|----|:---|:---|:--:|--------|
|code|响应消息代码|varchar(6)|Y|200为成功，其它为异常|
|message|错误消息提示|varchar(6)|N|code为非200时返回|
|data|新的密钥|varchar(32)|N|code为200时返回|



 - 返回示例：
 
```javascript
{
    "code":"200",
    "data":"34719280830192"
}
```

###2.2 如何构造签名
    签名是在进行接口调用或者参数输出生成二维码的过程中，利用密钥和参数进行MD5签名的过程，具体流程如下：
 - 签名说明：
接口调用或者生成二维码所有传递的参数都需要一起进行签名，假设本次参数包括appid、store\_sn、channel、biz\_time、biz\_no等5个参数，并且密钥secret=34719280830192，参数中appid=2200000001，biz\_time=1468780992，channel=alipay，store\_sn=5308，biz\_no=61028309128301298，则签名步骤如下：
```
第一步：参数及密钥拼装成6个元素存入到数组array=['2200000001=appId','1468780992=biz_time','alipay=channel','61028309128301298=biz_no','5308=store_sn','34719280830192=secret']；
第二步：对array数组进行排序得到sortArray=['1468780992=biz_time','2200000001=appid','34719280830192=secret','5308=store_sn','61028309128301298=biz_no','alipay=channel']；
第三步：使用连接符‘&’对数组进行拼接，得到字符串source='1468780992=biz_time&2200000001=appId&34719280830192=secret&5308=store_sn&61028309128301298=biz_no&alipay=channel'；
第四步：对source进行‘md5’32位大写加密，得到sign=Upper(MD5(source))；
```

##3 发票
###3.1 申请电子发票
 - 接口地址：https://m.wosai.cn/api/invoice/apply/v1
 - 访问方式：get
 - 参数说明：

|名称|含义|类型|必填|备注|
|----|:---|:---|:--:|--------|
|appid|ISV分配给商户唯一标识|varchar(20)|Y||
|channel|支付渠道|varchar(10)|N|cash/bank/alipay,用于发票归集|
|payer|用户支付渠道的唯一标识|varchar(10)|N|用于发票归集|
|store_sn|商户门店唯一标识|varchar(20)|Y||
|biz_no|交易流水号|varchar(32)|Y||
|biz_time|交易时间|varchar(10)|Y|以秒为单位的时间戳|
|amount|交易总金额|int|Y|单位为分|
|items|开票商品明细信息|[]|N|参考开票明细信息|
|sign|参数签名|varchar(32)|Y|详见2.2.如何构造签名|


 - 开票商品明细信息

 
|名称|含义|类型|必填|备注|
|----|:---|:---|:--:|--------|
|id|商品唯一标识|varchar(10)|Y|本次交易内唯一，退货时，退货商品的id需要对应该id|
|name|发票项目名称或商品名称|varchar(20)|Y||
|num|商品数量|int|N|折扣行参数不能填，非折扣行必填|
|item_amount|单项商品总价|int|Y|单位为分|


 - 参数示例：
https://m.wosai.cn/api/invoice/apply/v1?appid=2200000001&channel=alipay&store_sn=2200000011&biz_no=22000000012&biz_time=1488262165&amount=1000000&sign=MDYCGQCNTJhYa4JghYuksPMsE8jO33sq&items=[{"id":"1","name":"品类一","num":"1","item_amount":"11200"},{"id":"2","name":"品类二","num":"1","item_amount":"11200"},{"id":"3","name":"品类三","num":"1","item_amount":"11200"},{"id":"4","name":"品类四","num":"1","item_amount":"11200"}]

 - 返回说明：返回网页交互界面，用户在此交互界面确认订单信息和填写抬头。