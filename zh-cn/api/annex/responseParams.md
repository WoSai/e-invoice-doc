# 接口响应

### 返回字段列表

字段名 | 字段含义 | 取值 | 备注
--------- | ------ | ----- | -------
result_code | 通讯响应码 |200，400，500|200：通讯成功；400：客户端错误；500:服务端错误
error_code  | 通讯错误码 |见附录《公共错误码》 |通讯 **失败** 的时候才返回
error_message  |通讯错误信息描述 |见附录《公共错误码》 |通讯 **失败** 的时候才返回
biz_response  | 业务响应数据 |数组结构 |通讯 **成功** 的时候才返回
biz\_response.result_code  | 业务执行响应码 |见附录《业务数据响应码列表》|
biz\_response.error_code  | 业务执行结果返回码 |见附录《业执行错误码列表》 |
biz\_response.error_message  |业务执行错误信息 |见附录《业务执行错误码列表》|业务执行成功才返回
biz\_response.data.sn  |收钱吧系统任务号 |16位数字 |业务执行 **成功** 才返回
biz\_response.data.client_sn  | 客户系统任务号 | |业务执行 **成功** 才返回
biz\_response.data.status  | 最新流水状态 |见附录《流水状态列表》 |业务执行 **成功** 才返回
biz\_response.data.task_status  | 最新任务状态 |见附录《任务状态列表》 |业务执行 **成功** 才返回
biz\_response.data.total_amount  | 有效交易总金额 |整数，以 <font color="red">分</font> 为单位 |业务执行 **成功** 才返回
biz\_response.data.finish_time  | 本次业务操作在收钱吧系统中执行完成的时间 |时间戳 |业务执行 **成功**才返回
biz\_response.data.channel\_finish_time  | 本次业务操作在第三方发票通道执行完成的时间 |时间戳|业务执行 **成功**才返回
biz\_response.data.terminal_sn  | 发起本次业务请求的终端ID |字符串| 业务执行 **成功**才返回
biz\_response.data.store_id  | 本次业务执行所属门店 |字符串 |业务执行 **成功**才返回
biz_response.data.data.subject | 本次发票概述 |字符串 |业务执行 **成功**才返回
biz_response.data.description | 本次发票详情 |字符串 |业务执行 **成功**才返回
biz_response.data.reflect | 商户透传参数 |字符串 |业务执行 **成功**才返回
biz_response.data.operator  | 本次业务执行的操作员 |字符串 |业务执行 **成功**才返回
