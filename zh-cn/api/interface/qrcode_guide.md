### 发票二维码生成方式 （该规范有由户自己实现，为了对应离线二维码打印）
二维码生成规范, 用来根据给定的规则, 生成包含URL信息的二维码

 - 规范URL: {merchant_app_service_domain} + {uri_path}
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
url|{merchant_app_service_domain} + {uri_path}|string|Y|用户自己的支撑服务地址例如 `https://www.anycompany.com/invoice/preapply/h5`, 字符长度不超过300

```
与开票验签不同的是, 二维码生成规范中的  sn 和 key 是 vender_sn 和 vender_key
```


生成二维码的Content是

```
// 而需要注意，sign 生成的内容部分不包含 "&auth={auth}"
{url}?terminal_sn={terminal_sn}&client_sn={client_sn}&client_time={client_time}&total_amount={total_amount}&auth={auth}
```
