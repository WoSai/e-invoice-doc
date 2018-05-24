# 验签密钥生成规则
```javascript
// 商户订单号
var client_sn = "123456";
// 门店号，该门店号可由终端号获得，具体可咨询相关技术支持人员
var store_sn = "654321";

// 计算验签密钥
var sign = MD5(CONCAT(client_sn + " " + store_sn))
```
