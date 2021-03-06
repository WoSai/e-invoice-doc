# 签名机制

## 请求域名

   收钱吧接入域名(api_domain)：`https://einvoice-api.shouqianba.com`
   
   <font color="red">**注：收钱吧的Web API接口是https协议，当发起请求时，会要求检查证书，在发起请求时规避ssl的证书检查或者
  携带证书请求**</font>
   
## 签名所需参数
   * terminal_sn：终端号，由收钱吧提供
   * terminal_key：终端密钥，由收钱吧提供
     
## 签名方法
   如果要正常使用各接口，需要按照以下方式去进行签名验证:
   
   * 开票平台所有的API仅支持JSON格式的请求调用，请务必在HTTP请求头中加入`Content-Type: application/json`。
   * 所有请求的body都需采用`UTF-8`编码，所有响应也会采用相同编码。
   * 支付平台所有的API调用都需要签名验证。
   * 采用应用层签名机制。将HTTP请求body部分的`UTF-8`编码字节流视为被签名的内容，不关心主体的格式。
   * 签名人序列号(sn)和签名值(sign)放在HTTP请求头中，在接入服务中统一校验。
   * 签名算法: sign = MD5( CONCAT( body + terminal_key ) )
   * 签名首部: `Authorization: terminal_sn + " " + sign`
   * 所有返回参数都为 <font color="red">**JSON**</font> 格式，请务必在HTTP请求头中加入Content-Type: application/json。
   * 所有请求的body都需采用UTF-8编码，所有响应也会采用相同编码。
   * 所有返回数据的类型都是 <font color="red">**字符串**</font>。
   * 接口中所有涉及金额的地方都以 <font color="red">**分**</font> 为单位。
   * 所有接口, 无论是收钱吧提供,或是需要用户自己实现的接口 ,http请求都需要使用 `UTF-8` 编码
   


