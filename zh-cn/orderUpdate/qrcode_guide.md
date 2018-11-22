### 发票二维码生成方式 （该规范有用户自己实现，为了对应离线二维码打印）
二维码生成规范, 用来根据给定的规则, 生成包含URL信息的二维码

正式环境的接口域名：http://uinvoice-ms-common-backend.shouqianba.com
沙箱环境的接口域名：http://uinvoice-merchant-support-common.test.shouqianba.com

 - 规范

 ```
 {api_domain}/redirect?state={encodeURIComponent(state)}
 ```

 - 为什么需要 encodeURIComponent 操作？

 ```
 因为，一个标准的完整的 URL 的参数中是不允许存在一些特殊字符的， 而按照 ECMAScript 规范，我们可以使用 encodeURIComponent() 函数（或者同样算法的实现）对我们需要使用的参数中的字符进行转义
 这样就能符合 URL 的规范
 ```

 其中 ECMA 定义的 encdoeURIComponent 实现范式文档位于 [文档](https://www.ecma-international.org/ecma-262/5.1/#sec-15.1.3) 当读完ECMA文档的15.1.3后，就能释然
 
 - 参数说明:

 - state 参数说明:

名称|含义|类型|必填|备注
----|:---|:---|:--:|--------
p|支付通道唯一标识(payway)|string(20)|N|用于发票归集, 1:支付宝 3:微信 4:百度钱包 5:京东钱包 6:qq钱包 100:现金 101:银联卡 110:银行卡
ts|终端号(terminal_sn)|string|Y| 
cs|商户系统订单号(client_sn)|string|Y|必须在商户系统内唯一；且长度不超过32字节
ct|商户系统订单完成时间(client_time)|int|Y|timestamp,单位毫秒
ta|总金额(分)(total_amount)|int|Y|
bc|brand_code 品牌编号, 填写商户于 wosai 约定的品牌映射值|string|Y|比如 0 或者 1 分别代表不同的品牌
a|即 auth，值为 client_sn + " " + store_sn ,client_sn为商户订单编号,store_sn为收钱吧门店编号|string|Y|MD5(client_sn + " " + store_sn)

1. 基础 state 参数构造样例，不包含 a（auth）

```javascript
// 基础 state 参数构造样例， 转义前
baseStateParameter="p=1|ts=100010570005554188|cs=28910391282321983|ct=1541975753000|ta=26000|bc=2c896964-545b-11e8-a47c-7d1f5d212442"
```

2. 构造 a(即 auth) 样例

```javascript
// 假设我们的 client_sn 为 28910391282321983
client_sn="28910391282321983"
// 假设我们的 store_sn 为 7f3804007bf8408293b8ebb8157fc0fb
store_sn="7f3804007bf8408293b8ebb8157fc0fb"

// md5 使用 hex_md5
a=MD5(CONCAT(client_sn + " " + store_sn))
// 算出来，signature 的值为 "96afbca1a158e844f75937aa891cb13b"
```

3. 完整的 state 参数构造样例，包含 a(即 auth)

```javascript
// 完整的 state 参数值， 转义前
state=baseStateParameter + "|" + "a=" + a
// 算出来， state 的值是 "p=1|ts=100010570005554188|cs=28910391282321983|ct=1541975753000|ta=26000|bc=2c896964-545b-11e8-a47c-7d1f5d212442|a=96afbca1a158e844f75937aa891cb13b"

// 使用 encodeURIComponent 编码后的的参数为 
encodedState=encodeURIComponent(state)
// 算出来, encodedState 的值是 "p%3d1%7cts%3d100010570005554188%7ccs%3d28910391282321983%7cct%3d1541975753000%7cta%3d26000%7cbc%3d2c896964-545b-11e8-a47c-7d1f5d212442%7ca%3d96afbca1a158e844f75937aa891cb13b"
```

4. 最终生成的二维码的样例 

```javascript
// 最终的二维码的 url 内容为
http://uinvoice-ms-common-backend.shouqianba.com/redirect?state=p%3d1%7cts%3d100010570005554188%7ccs%3d28910391282321983%7cct%3d1541975753000%7cta%3d26000%7cbc%3d2c896964-545b-11e8-a47c-7d1f5d212442%7ca%3d96afbca1a158e844f75937aa891cb13b

```




### 附录
其中 encodeURIComponent() 函数是指需要对参数项，进行编码转义

如果你使用的是 Java ，encodeURIComponent() 函数可以如下定义
```java
public static String encodeURIComponent(String component)   {     
	String result = null;      
	
	try {       
		result = URLEncoder.encode(component, "UTF-8")   
			   .replaceAll("\\%28", "(")                          
			   .replaceAll("\\%29", ")")   		
			   .replaceAll("\\+", "%20")                          
			   .replaceAll("\\%27", "'")   			   
			   .replaceAll("\\%21", "!")
			   .replaceAll("\\%7E", "~");     
	catch (UnsupportedEncodingException e) {       
		result = component;     
	}      
	
	return result;   
}  
```

如果你使用的是 Javascript 的语言， encodeURIComponent() 函数可以这么使用
```javascript
// 如果是在浏览器环境下，可以直接如下使用
var encodedParameter = encodeURIComopnent(oneParameter);
```

而使用 javascript 但又不在浏览器环境下的话
```javascript
// 如果是不在浏览器环境下，比如 node 环境下
// 可参照 ECMA-262 的实现
// ECMA-262 - 15.1.3.4
function URIEncodeComponent(component) {
  var unescapePredicate = function(cc) {
    if (isAlphaNumeric(cc)) return true;
    // !
    if (cc == 33) return true;
    // '()*
    if (39 <= cc && cc <= 42) return true;
    // -.
    if (45 <= cc && cc <= 46) return true;
    // _
    if (cc == 95) return true;
    // ~
    if (cc == 126) return true;

    return false;
  };

  var string = ToString(component);
  return Encode(string, unescapePredicate);
}
```

如果你使用的是 PHP 语言， encodeURIComponent() 函数可以如下定义
```php
function encodeURIComponent($str) {
    $revert = array('%21'=>'!', '%2A'=>'*', '%27'=>"'", '%28'=>'(', '%29'=>')');
    return strtr(rawurlencode($str), $revert);
}
```

