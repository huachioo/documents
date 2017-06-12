## 1. 概述

本文档为云净网平台与外部接入平台的直播接口描述，通过本文档中的描述进行直播数据交互。

### 1.1 接口协议

说明：

1. 采用HTTP协议的POST方式；
2. 请求和响应的数据为JSON格式；

### 1.2 申请密钥流程

请联系云净网客服人员获取密钥。

### 1.3 摘要

#### 算法

摘要算法采用HMAC_SHA1，HMAC_SHA1和MD5的区别在于摘要的时候增加了密钥，更加安全。

#### 步骤

1. 摘要原文内容在各个接口会进行详细定义，例如：数据接入协议的原文内容为：app_id,timestamp连接，用英文半角逗号“,”相连；
2. 使用运营平台中生成的密钥对摘要原文内容进行摘要，形成摘要信息，然后对摘要信息进行Base64编码并转化为字符串形成sign；

#### 举例

假设明文为testapi20160909，密钥为ICRf0mw4eRcFnVcPSDyS
输出结果：mYZm2BA8YUrmHxvhhRhXJOLIYt0=

Java代码实现如下：

```java
    /**
     * 使用 HMAC-SHA1 摘要方法对对encryptText进行摘要
     *
     * @param encryptText 被摘要的字符串
     * @param encryptKey  密钥
     */
    public static byte[] hmacSHA1Encrypt(String encryptText, String encryptKey) throws Exception {
        byte[] data = encryptKey.getBytes("UTF-8");
        //根据给定的字节数组构造一个密钥,第二参数指定一个密钥算法的名称
        SecretKey secretKey = new SecretKeySpec(data, "HmacSHA1");
        //生成一个指定 Mac 算法 的 Mac 对象
        Mac mac = Mac.getInstance("HmacSHA1");
        //用给定密钥初始化 Mac 对象
        mac.init(secretKey);

        byte[] text = encryptText.getBytes("UTF-8");
        byte[] enText = mac.doFinal(text);
        return Base64.encodeBase64(enText);
    }
```

## 2. 数据接入协议

### 2.1 描述

此协议是客户向云净网提出检测请求的接口协议。

用户接入时，调用API须遵循以下规则：

* 传输方式：HTTP
* 提交方式：POST
* 数据格式：JSON
* 字符编码：UTF-8
* 摘要算法：用户生成摘要字符串，现支持的摘要算法类型为HMAC_SHA1
* 摘要要求：请求数据均需要摘要认证，也可申请无需摘要认证，但安全性差，不建议使用

### 2.2 接口地址

`http://api.yunjingwang.cn:9999/audit-live`

### 2.3 请求

#### 请求参数

**注意：请求的参数需要URL编码。**

| 参数名 | 约束 | 类型 | 描述 |
| ---- | ---- | ---- | ---- |
|app_id|必传|string|申请的应用ID，每种检测数据类型独立申请|
|version|必传|string|固定为1.0|
|timestamp|必传|string|请求时间，格式为：“yyyy-MM-dd HH:mm:ss”|
|sign|必传|string|摘要内容：app_id,timestamp连接，用英文半角逗号“,”相连，算法见1.3|
|json_data|必传|string|需要检测的业务数据，**JSON格式的字符串**|

#### 业务参数 json_data 说明

| 参数名 | 约束 | 类型 | 描述 |
| ---- | ---- | ---- | ---- |
|anchor_id|必传|string|主播用户ID|
|live_id|必传|string|直播房间ID|
|msg_time|必传|string|消息发送时间，格式为：“yyyy-MM-dd HH:mm:ss”|
|ip_address|必传|string|发消息用户的IP地址|
|user_id|必传|string|发消息用户ID|
|message_type|必传|int|消息类型（0 用户消息、1 系统消息、2 发送礼物、3 图片、 4 转发、5 关注、6 红包、7 私信）|
|msg_content|必传|string|消息内容，如果消息类型是为片，此参数为图片链接|
|nick_name|选传|string|发消息用户昵称|
|star_level|选传|string|发消息用户等级|
|manchine_code|选传|string|发消息用户设备编码|
|imei|选传|string|发消息用户移动设备标识|

#### 示例

```
POST /audit-live HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Host: api.yunjingwang.cn:9999
Content-Length: 639

app_id=201601010001&version=1.0&timestamp=2016-09-28+17%3A39%3A25&sign=CAQ9aHGBvub7yBxE72t%2B3hyGAlc%3D&json_data=%7B%22anchor_id%22%3A%22600501%22%2C%22live_id%22%3A%221638%22%2C%22msg_time%22%3A%222016-09-28+17%3A39%3A25%22%2C%22ip_address%22%3A%22192.168.0.3%22%2C%22user_id%22%3A%22devUser3%22%2C%22message_type%22%3A%220%22%2C%22msg_content%22%3A%22%5Cu4e00%5Cu4e2a%5Cu7b80%5Cu5355%5Cu7684%5Cu652f%5Cu6301yml%5Cu683c%5Cu5f0f%5Cu9ad8%5Cu4eae%5Cu7684%5Cu6587%5Cu672c%5Cu7f16%5Cu8f91%5Cu5668%22%2C%22nick_name%22%3A%22Love%22%2C%22star_level%22%3A%2257%22%2C%22manchine_code%22%3A%22OBjj6FXpVUwP0Bdb%22%2C%22imei%22%3A%225228651550%22%7D
```

### 2.4 响应

#### 响应参数

| 参数名 | 约束 | 类型 | 描述 |
| ---- | ---- | ---- | ---- |
|json|必传|json|处理结果|

#### 参数 json 说明

| 参数名 | 约束 | 类型 | 描述 |
| ---- | ---- | ---- | ---- |
|code|必传|int|处理结果编码（1001 调用正常：结果为通过、1002 调用正常，结果为禁止、1003 调用正常，需要人工审核、1004 图片审核：结果为处理中、4000 调用失败：客户端参数错误、4001 调用失败：客户端数据包错误、4005 调用失败：appId校验失败、5000 调用失败：服务端异常）|
|timestamp|必传|string|处理时间，格式为：“yyyy-MM-dd HH:mm:ss”|
|msg|必传|string|处理结果描述（1001 OK-passed、1002 OK-denied、1003 OK-pushed manual audit、1004 OK-processing）|
|data_id|必传|string|数据唯一标识，可用于回调或数据一致性验证|
|category|必传|int|分类（0 正常、1 政治、2 色情、3 违法、4 违规）|
|score|必传|float|信度值，针对处理结果（通过、人工审核、禁止）的信度，每个结果的信度值范围为[0,1]，1表示可信度最高|

#### 示例

```
HTTP/1.1 200 OK
Server: yunclean
Content-Type: application/json; charset=utf-8
Connection: close

{"json":{"code":1002,"timestamp":"2016-09-28 10:19:47","msg":"OK-denied","data_id":"8732ccdaacc04d1bb895c564c72c99e2","category":2}}
```

## 3. 回调协议

### 3.1 设置回调地址

用户可在运营平台的`设置—>平台管理`中设置全局回调接口地址，也可在`设置->应用管理`中按应用设置。

### 3.2 回调方式

采用HTTP POST方式请求回调地址

### 3.3 参数

| 参数名 | 类型 | 描述 |
| ---- | ---- | ---- |
|appId|string|应用ID|
|dataId|string|数据唯一标识|
|type|int|类型：1 文本、2 图片|
|content|string|文本内容或图片链接|
|msgTime|string|消息发送时间，格式为：“yyyy-MM-dd HH:mm:ss”|
|result|int|处理结果：0 删除、1 通过|

### 3.4 示例

```
POST /callback HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Host: example.com
Content-Length: 377

appId=201607197244&dataId=739d29ef77cc4fd69ec46f4c5b12127b&msgTime=2016-09-28+15%3A00%3A03&result=0&content=%E8%B7%B3%E5%95%A5%E8%88%9E&type=1
```