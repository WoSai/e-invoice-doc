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
order_type|订单类型|String|Y|付款类型订单: "P" ;退换款类型订单: "R"
client_original_sn|原始订单号|String|N|对应原始订单号（client_sn），当订单类型（order_type）为R时必填，其他情况为非必填
amount|订单金额|String|Y|以分为单位，eg：100 即 1元
discount|优惠金额|String|N|以分为单位，eg：100 即 1元，优惠金额详细计算规则见下文
invoice_items|订单明细|[item]|Y|详情见invoice_items说明

 - 订单明细(invoice_items 中的每个 item)

名称|含义|类型|必填|备注
----|:---|:---|:--:|--------
client_sn|商户订单号|String|Y|与外层订单号相同
item_no|商品税收编码|String|Y|商品税收编码最长度必须为19位或21位数字
item_name|商品名称|String|Y|一个请求体中的item_name需唯一
quantity|商品数量|String|Y|商品数量不可为零，退款时需为负数
row_type|发票行性质|String|Y|注意现阶段仅支持正常行（0），0=正常行，1=折扣行，2=被折扣行。比如充电器单价100元，折扣10元，则明细为2行，充电器行性质为2，折扣行性质为1。如果充电器没有折扣，则值应为0
specification|规格|String|N|
unit|商品单位|String|Y|
amount|上面明细金额|String|Y|以分为单位，退款时需为负数，eg：100 即 1元
tax_rate|税率|String|Y|eg：0.17
zero_rate_flag|零税率标识|String|N|当税率为零时必传，可选参数：0=出口零税率，1=免税，2=不征收，3=普通零税率


 - 参数实例
 
 ```
    curl -X POST \
      {{domain}}/api/order/commit \
      -H 'content-type: application/json' \
      -H 'sign: 5b8b408fa1adb42e9772b6931cf2df96' \
      -d '{
        "terminal_sn" : "100000000002181948",
        "client_sn" : "test001",
        "client_task_sn" : "test001",
        "client_time" : "1232123213",
        "order_type" : "P",
        "amount" : "8500",
        "discount" : "100",
        "invoice_items" : [
                {
                    "client_sn" : "test001",
                    "item_no" : "1231312",
                    "item_name" : "衣服",
                    "quantity" : "10",
                    "row_type" : "0",
                    "specification" : "X100",
                    "unit" : "件",
                    "amount" : "5500",
                    "tax_rate" : "0.1"
                },
                {
                  "client_sn" : "test001",
                    "item_no" : "1231312",
                    "item_name" : "帽子",
                    "quantity" : "10",
                    "row_type" : "0",
                    "specification" : "X100",
                    "unit" : "件",
                    "amount" : "3000",
                    "tax_rate" : "0.17"
                }
            ]
    }'
 ```
 
 - 返回结果
 ```
    {
        "result_code": "200",
        "biz_response": {
            "result_code": "SYNC_SUCCESS",
            "data": {
                "client_sn": "test001",
                "client_task_sn": "test001",
                "terminal_sn": "100000000002181948",
                "task_sn": "1527140276909522"
            }
        }
    }
 ```
 
### 订单查询接口
- 接口地址: {api_domain}/api/order/query
- 访问方式：post
- 参数说明：

名称|含义|类型|必填|备注
----|:---|:---|:--:|--------
terminal_sn|终端号|String|Y|
client_sn|商户订单号|String|Y|可以由字母、数字、下划线组成，长度不超过36位

 - 参数实例
 
 ```
    curl -X POST \
      {{domain}}/api/order/commit \
      -H 'content-type: application/json' \
      -H 'sign: 5b8b408fa1adb42e9772b6931cf2df96' \
      -d '{
        "terminal_sn" : "100000000002181948",
        "client_sn" : "test001"
    }'
 ```
 
 - 返回结果
 ```
    {
        "result_code": "200",
        "biz_response": {
            "result_code": "SUCCESS",
            "order": {
                "id": "c0a8b00263861ced816386fab2680018",
                "ctime": 1526977967000,
                "mtime": 1526977967000,
                "version": 1,
                "deleted": false,
                "task_sn": "1526977966695426",
                "client_sn": "test001",
                "client_task_sn": "test001",
                "client_time": 1232123213,
                "invoice_amount": 7900,
                "order_amount": 8000,
                "terminal_sn": "100000000002181948",
                "client_original_sn": null,
                "original_terminal_sn": "",
                "discard": false,
                "refund_amount": 0,
                "brand_id": "5562cb5f-54e2-11e8-a5bd-957d23aae337",
                "discount": 100,
                "initial_client_sn": "yxytest3045",
                "discount_rate": null,
                "refund": false
            },
            "orderItems": [
                {
                    "id": "c0a8b00263861ced816386fab2680019",
                    "ctime": 1526977967000,
                    "mtime": 1526977967000,
                    "version": 1,
                    "deleted": false,
                    "client_sn": "test001",
                    "item_name": "衣服",
                    "item_no": "1231312",
                    "order_type": "30",
                    "client_time": null,
                    "amount": 4937,
                    "quantity": 10,
                    "tax_rate": "0.1",
                    "row_type": "0",
                    "unit": "件",
                    "specification": "X100",
                    "terminal_sn": "100000000002181948",
                    "zero_rate_flag": null,
                    "brand_id": "5562cb5f-54e2-11e8-a5bd-957d23aae337",
                    "discard": false,
                    "discount_rate": null
                },
                {
                    "id": "c0a8b00263861ced816386fab268001a",
                    "ctime": 1526977967000,
                    "mtime": 1526977967000,
                    "version": 0,
                    "deleted": false,
                    "client_sn": "test001",
                    "item_name": "帽子",
                    "item_no": "1231312",
                    "order_type": "30",
                    "client_time": null,
                    "amount": 2963,
                    "quantity": 10,
                    "tax_rate": "0.17",
                    "row_type": "0",
                    "unit": "件",
                    "specification": "X100",
                    "terminal_sn": "100000000002181948",
                    "zero_rate_flag": null,
                    "brand_id": "5562cb5f-54e2-11e8-a5bd-957d23aae337",
                    "discard": false,
                    "discount_rate": null
                }
            ]
        }
    }
 ```

## 接口错误返回实例
```json
{
    "error_code": "INVALID_PARAMS",
    "error_message": "参数错误：client_sn(test001) 或 client_task_sn(test001) 已存在",
    "result_code": "400"
}
```

## 错误码列表

error_code|含义
----|:---|:---
INVALID_PARAMS|参数错误
INVALID_TERMINAL|终端错误
AUTHORIZATION_EXPIRED|授权过期
UNKNOWN_SYSTEM_ERROR|未知系统错误
ACCESS_DENIED|没有访问权限
ILLEGAL_SIGN|签名错误




    


