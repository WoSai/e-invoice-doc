## 前提
所有的请求和响应 必须都使用 utf-8 字符编码

订单回调提供2种方案,分别为
1. 订单开票状态回调接口
2. 开票结果信息回调接口

## 接入说明

  ```
  提供回调地址给收钱吧运营，以及需要哪一种订单回调方案
  ```

### 1. 订单状态回调接口(web hook)规范, (被回调的接口由商户实现并提供)
 - 接口地址：{user_callback_api_domain}
 - 访问方式：post
 - 参数格式：application/json
 - 参数说明：

名称|含义|类型|必填|备注
----|:---|:---|:--:|--------
result_code|返回状态码|string|Y|SUCCESS: 成功; ERROR: 失败

   - 订单信息(order)

名称|含义|类型|必填|备注
----|:---|:---|:--:|--------
client_sn|对应商户订单号|string|Y|-
client_original_sn|对应商户退款前原订单号|string|Y|仅红票回调时存在
client_time|对应商户订单时间|string|Y|-
total_amount|订单总金额|string|Y|-
status|订单开票状态|int|Y|1: 已开蓝票，2: 已开红票

 - 蓝票参数示例：

```json
{
    "result_code":"200",
    "biz_response":{
        "result_code":"SUCCESS",
        "order":{
            "client_sn":"351020180831",
            "client_time":"2018-08-31 16:28:10",
            "total_amount":"192060",
            "status":1
        }
    }
}
```
 - 红票参数示例：

```json
{
    "result_code":"200",
    "biz_response":{
        "result_code":"SUCCESS",
        "order":{
            "client_sn":"351020180831",
			"client_original_sn": "351020180830",
            "client_time":"2018-08-31 16:28:10",
            "total_amount":"192060",
            "status":1
        }
    }
}
```

- 回调返回说明：

 商户收到通知后应返回如下结果; 如果收钱吧未收到"成功"结果，或者没有调通回调接口。 会启用时间序列，尝试间隔一段时间调用商家接口。

- 回调返回示例：

```javascript
{
    "result_code": "200",
    "biz_response": {
        "result_code": "SUCCESS",
        "data": "接收回调信息成功"
    }
}
```

### 2. 开票结果信息回调接口(web hook)规范, (被回调的接口由商户实现并提供)
 - 接口地址：{user_callback_api_domain}
 - 访问方式：post
 - 参数格式：application/json
 - 参数说明：

名称|含义|类型|必填|备注
----|:---|:---|:--:|--------
result_code|返回状态码|string|Y|SUCCESS: 成功; ERROR: 失败

   - 订单信息(order)

名称|含义|类型|必填|备注
----|:---|:---|:--:|--------
client_sn|对应商户订单号|string|Y|-
client_original_sn|对应商户退款前原订单号|string|Y|仅红票回调时存在
client_time|对应商户订单时间|string|Y|-
total_amount|订单总金额|string|Y|-
status|订单开票状态|int|Y|1: 已开蓝票，2: 已开红票

   - 开票结果信息(order 中的 invoice)


名称|含义|类型|必填|备注
----|:---|:---|:--:|--------
task_sn|收钱吧开票任务号|string|Y|-
channel_task_sn|开票渠道任务号|string|Y|-
invoice_time|开票时间|string|Y|-
invoice_code|发票代码|string|Y|-
invoice_no|发票号码|string|Y|-
payee_name|销方名称|string|Y|-
invoice_amount|价税合计|string|Y|-
pdf_url|票面URL|string|Y|电子发票的可访问的图片URL地址

 - 蓝票参数示例：

```json
{
    "result_code":"200",
    "biz_response":{
        "result_code":"SUCCESS",
        "order":{
            "client_sn":"351020180831",
            "client_time":"2018-08-31 16:28:10",
            "total_amount":"192060",
            "status":1,
            "invoice":{
                "task_sn":"10113259247332270",
                "channel_task_sn":"2018090700152001470000011434",
                "invoice_time":"2018-08-31 21:45:23",
                "invoice_code":"982365656091",
                "invoice_no":"39174215",
                "payee_name":"上海喔噻互联网科技有限公司",
                "invoice_amount":"192060",
                "pdf_url":"https://images.wosaimg.com/uinvoice_tickets/prod/c613279fac5d11e8a1261781755664a9.pdf"
            }
        }
    }
}
```
 - 红票参数示例：

```json
{
    "result_code":"200",
    "biz_response":{
        "result_code":"SUCCESS",
        "order":{
            "client_sn":"351020180831",
			"client_original_sn": "351020180830",
            "client_time":"2018-08-31 16:28:10",
            "total_amount":"192060",
            "status":1,
            "invoice":{
                "task_sn":"10113259247332270",
                "channel_task_sn":"2018090700152001470000011434",
                "invoice_time":"2018-08-31 21:45:23",
                "invoice_code":"982365656091",
                "invoice_no":"39174215",
                "payee_name":"上海喔噻互联网科技有限公司",
                "invoice_amount":"192060",
                "pdf_url":"https://images.wosaimg.com/uinvoice_tickets/prod/c613279fac5d11e8a1261781755664a9.pdf"
            }
        }
    }
}
```



- 回调返回说明：

 商户收到通知后应返回如下结果; 如果收钱吧未收到"成功"结果，或者没有调通回调接口。 会启用时间序列，尝试间隔一段时间调用商家接口。

- 回调返回示例：

```javascript
{
    "result_code": "200",
    "biz_response": {
        "result_code": "SUCCESS",
        "data": "接收回调信息成功"
    }
}
```

