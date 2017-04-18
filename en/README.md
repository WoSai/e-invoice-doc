# [E-Invoice] Wosai API Doc 20170228

标签（空格分隔）： Wosai Api

---

[TOC]

##1. Introduction
This document is used to illustrate how to make a QR code for E-invoice with trading parameters, customers can scan the E-invoice QR code to request an online invoice.

Follow these steps to make a QR code:
 > * Step 1：Prepare informations about bussiness；
 > * Step 2：Use these informations to apply a appid and secret from ISV, secret should be updated periodic.
 > * Step 3：Make a http request url <see 3.1> using trading parameters, generate a QR code using this url and print it on the receipt.

##2. API Description

###2.1 Update Secret
    Update secret periodic to make secure.
 - api url：https://m.wosai.cn/api/sign/v1
 - request type：post
 - parameters：

|name|meaning|type|required|desc|
|----|:---|:---|:--:|--------|
|appid|unique identity from ISV|varchar(20)|Y||
|sign|MD5 signature |varchar|Y|see 2.2|


 - example：
 
```javascript
{
    "appid":"2200000001",
    "sign":"MDYCGQCNTJhYa4JghYuksPMsE8jO33sq"
}
```


 - return parameters：

|name|meaning|type|required|desc|
|----|:---|:---|:--:|--------|
|code|response code|varchar(6)|Y|200 success，otherwise exception|
|message|error message|varchar(6)|N|exist when code is not 200|
|data|new secret|varchar(32)|N|exist when code is 200|



 - example：
 
```javascript
{
    "code":"200",
    "data":"34719280830192"
}
```

###2.2 How to make a MD5 signature
    MD5 signature is used to make api calling and QR code more secure, using secret and parameters to make a MD5 signature, see the follow steps：
 - Signature introduction：
All parameters about api calling and QR code generating are used to make a signature, for example parameters include: appid、store\_sn、channel、biz\_time、biz\_no, and secret=34719280830192，appid=2200000001，biz\_time=1468780992，channel=alipay，store\_sn=5308，biz\_no=61028309128301298，the follow these steps to make a signature：
```
Step 1：store secret and parameters into an array=['2200000001=appId','1468780992=biz_time','alipay=channel','61028309128301298=biz_no','5308=store_sn','34719280830192=secret']；
Step 2：sort this array to get a sortArray=['1468780992=biz_time','2200000001=appid','34719280830192=secret','5308=store_sn','61028309128301298=biz_no','alipay=channel']；
Step：using '&' to connect all these parameters then get a string source='1468780992=biz_time&2200000001=appId&34719280830192=secret&5308=store_sn&61028309128301298=biz_no&alipay=channel'；
Step 4：using MD5 to get a signature, and make the signature combined with capital chars, sign=Upper(MD5(source))；
```

##3 E-Invoice
###3.1 Apply E-Invoice
 - api url：https://m.wosai.cn/api/invoice/apply/v1
 - request type：get
 - parameters：

|name|meaning|type|required|desc|
|----|:---|:---|:--:|--------|
|appid|unique identity from ISV|varchar(20)|Y||
|store_sn|unique store identity|varchar(20)|Y||
|biz_no|order number in trading|varchar(32)|Y||
|biz_time|order time in trading|varchar(10)|Y|time stamp in seconds|
|amount|total price in trading|int|Y|unit in penny|
|items|goods list in invoice|[]|N|see follow introduction|
|sign|MD5 signature|varchar(32)|Y|see 2.2|


 - goods list in invoice

 
|name|meaning|type|required|desc|
|----|:---|:---|:--:|--------|
|id|unique identity for goods|varchar(10)|Y|unique in trading，id in return of goods must be the same|
|name|invoice type name or goods type name|varchar(20)|Y||
|num|goods quantity|int|N|Not exist for discount，otherwist required|
|item_amount|total price for item|int|Y|unit in penny|


 - example：
https://m.wosai.cn/api/invoice/apply/v1?appid=2200000001&store_sn=2200000011&biz_no=22000000012&biz_time=1488262165&amount=1000000&sign=MDYCGQCNTJhYa4JghYuksPMsE8jO33sq&items=[{"id":"1","name":"item1","num":"1","item_amount":"11200"},{"id":"2","name":"item2","num":"1","item_amount":"11200"},{"id":"3","name":"item3","num":"1","item_amount":"11200"},{"id":"4","name":"item4","num":"1","item_amount":"11200"}]

 - return desc：This url can lead customers to a Web page, customers can input invoice informations in this Web page。