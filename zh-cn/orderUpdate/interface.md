## 前提
所有的请求和响应 必须都使用 utf-8 字符编码，并且所有请求头中必须包含验签参数sign，生成规则见[验签密钥生成规则](sign.md)

H5开票（订单上传）方案提供两个接口：
1. 上传订单接口
2. 查询订单接口

### 订单上传接口
 - 接口地址: {api_domain}/api/order/commit
 - 访问方式：post
 - 参数说明：
 
名称|含义|类型|必填|备注
----|:---|:---|:--:|--------
terminal_sn|终端号|String|Y|
client_sn|商户订单号|String|Y|可以由字母、数字、下划线组成，长度不超过36位
client_task_sn|商户任务号|String|Y|可以由字母、数字、下划线组成，长度不超过36位
client_time|订单时间|String|Y|timestamp，单位毫秒
order_type|订单类型|String|Y|付款类型订单: "P" ;退款类型订单: "R"
client_original_sn|原始订单号|String|N|对应原始订单号（client_sn），当订单类型（order_type）为R时必填，其他情况为非必填
amount|订单金额|String|Y|以分为单位，eg：100 即 1元
discount|优惠金额|String|Y|以分为单位，eg：100 即 1元
invoice_items|订单明细|[item]|Y|详情见invoice_items说明

 - 订单明细(invoice_items 中的每个 item)

名称|含义|类型|必填|备注
----|:---|:---|:--:|--------
client_sn|商户订单号|String|Y|与外层订单号相同
item_no|商品税收编码|String|Y|
item_name|商品名称|String|Y|一个请求体中的item_name需唯一
quantity|商品数量|String|Y|商品数量不可为零，退款时
row_type|发票行性质|String|Y|注意现阶段仅支持正常行（0），0=正常行，1=折扣行，2=被折扣行。比如充电器单价100元，折扣10元，则明细为2行，充电器行性质为2，折扣行性质为1。如果充电器没有折扣，则值应为0
specification|规格|String|N|
unit|商品单位|String|Y|
amount|上面明细金额|String|Y|以分为单位，eg：100 即 1元
tax_rate|税率|String|Y|eg：0.17
zero_rate_flag|零税率标识|String|N|当税率为零时必传，可选参数：0=出口零税率，1=免税，2=不征收，3=普通零税率

 
### 订单查询接口
- 接口地址: {api_domain}/api/order/query
- 访问方式：post
- 参数说明：

名称|含义|类型|必填|备注
----|:---|:---|:--:|--------
terminal_sn|终端号|String|Y|
client_sn|商户订单号|String|Y|可以由字母、数字、下划线组成，长度不超过36位
