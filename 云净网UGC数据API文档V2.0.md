云净网UGC数据API文档V2.0
-------------------

## 1. 概述

本文档为云净网平台与外部接入平台的接口描述，通过本文档中的描述进行数据交互。

### 1.1 接口协议

协议说明：

 1. 协议采用http协议的post方式     
 2. 请求消息体采用键值对方式，业务参数放在json_data的键值对中，摘要信息在sign的键值对中
 3. 返回消息直接为json格式，不带消息体摘要信息

### 1.2 申请密钥流程

请联系云净网客服人员获取密钥。

### 1.3 摘要算法

#### 算法：

摘要算法采用HMAC_SHA1摘要，HMAC_SHA1和MD5摘要的区别在于摘要的时候增加了密钥，更加安全。

#### 摘要过程：

 1. 摘要原文内容在各个接口会进行详细定义，例如：数据接入协议的原文内容为：app_id,timestamp连接，用英文半角逗号“,”相连
 2. 使用key对摘要原文内容进行摘要，形成摘要信息，然后对摘要信息进行base64编码并转化为字符串形成sign

#### 举例：

假设明文为testapi20160909，密钥为ICRf0mw4eRcFnVcPSDyS
 1. 获得明文testapi20160909的byte数组A，编码使用utf-8
 2. 使用密钥针对数组A进行加密，得到byte数组B
 3. 针对byte数组B进行base64编码，得到加密后byte数组C
 4. 将byte数组C格式为字符串输出，结果如下：
mYZm2BA8YUrmHxvhhRhXJOLIYt0=

### 1.4 对接流程
 
#### 说明：

 1. 数据检测见步骤1、2，详细协议见数据检测接口协议
 2. 审核结果主动拉取见步骤3、4，详细协议见审核结果拉取接口协议
 3. 云净网主动通知见步骤5、6，详细协议见审核结果实时反馈接口协议

注：2和3是接收审核结果的协议，主动拉取或接收通知，实现一种即可。

## 2 数据检测接口协议

### 2.1 描述

此协议是客户向云净网提出检测请求的接口协议。
用户接入时，调用API须遵循以下规则：

 - 传输方式：采用HTTP传输
 - 提交方式：POST方法提交
 - 数据格式：请求参数见说明；返回参数均为JSON格式
 - 字符编码：UTF-8字符编码
 - 签名算法：用户生成签名字符串，现支持的摘要算法为HMAC_SHA1
 - 签名要求：请求和接收数据均需要签名认证

### 2.2 检测地址

> http://api.yunjingwang.cn:8080/audit

### 2.3 请求描述

#### Http的Post调用参数说明

| 参数名 | 参数类型 | 说明 |
| ---- | ---- | ---- |
| app_id | String(32) | 向云净网申请的appId，每种检测数据类型独立申请201508120001 | 
| version | String | 固定为2.0 |
| timestamp	|String	| 发送请求的时间，格式“yyyy-MM-dd HH:mm:ss” |
| sign | String(256) | 摘要内容：app_id,timestamp连接，用英文半角逗号“,”相连,算法见1.3 |
| json_data | String(Json格式) | json_data为实际需要检测的数据包内容。 |

#### 其中参数json_data的数据说明

| 字段名 | 约束 | 类型 | 说明 |
| ---- | ---- | ---- | ---- |
| id	| 必选 | string(32) | 内容的唯一标识，用于用户标记数据 |
| relation_id | 必选 | string(32) | 关联内容的Id(例如留言、评论、回复等主内容的主键)，用于标记数据 |
| publish_date | 必选 | String(19) | 用户发布内容的时间，格式“yyyy-MM-dd HH:mm:ss” |
| user_id | 可选 | string(32) | 用户的唯一标识，非必选。 |
| user_level | 可选	| String(32) | 用户级别（10-低,20-中,30-高） |
| ip | 可选 | string(32) | 用户发布数据时所使用的公网IP |
| url | 可选 | string(4000) | 文章公网url地址 |
| author | 可选 | string(64) | 作者姓名 |
| contents | 必选 | json-collection | 具体需要校验的内容，是个json对象集合 |

#### contents数据说明

| 字段名 | 约束 | 类型 | 说明 |
| ---- | ---- | ---- | ---- |
| type | 必选 | string(32) | 内容类型；text:文本；image:图片；original_video:直接可以通过浏览器内置播放器播放的视频；warpper_video:通过浏览器可以播放，url返回信息为iframe包装后的窗口；original_audio:直接可以通过浏览器内置播放器播放的音频；wrapper_audio：通过浏览器可以播放，url返回信息为iframe包装后的窗口 |
| content | 必选 | string(32) | 文本内容或图片、视频的url地址，其中视频的url根据传输的类型，如果是标准格式，可以直接给完整的媒体的url；如果不是标准格式，需要给出通过iframe包装后的url地址|
| master | 必选 | int | 1、type为text的，如果传入了多条文本，其中只有一条可以设置主记录，云净网会对其审核，其余文本为审核辅助字段，不进行审核操作。2、type为image或video类型的，此字段无意义。3、值定义：1-主记录文本；0-辅助字段，不审核。 |

#### 请求示例

```
POST /audit HTTP/1.1
Host: api.yunjingwang.cn:8080
Content-Length: 512
Expect: 100-continue
Connection: Keep-Alive

app_id=201608200001&version=2.0&timestamp=2016-08-24 16:45:04&sign=KYqidJA21UdrJBkaP%2ffDn6Oi9fLdIOlBIhDPz2XP77xyrSkq%2b2POtVjWX1AZ4rHYd%2fmYzIyHociqJd9mAEz7RP4sxIJSe5kQOUSXhrSYpBmySfHtNr3BrvT0dKtN5tOnDnX5xRblmMtJKGl1MhCe7z7U8ttoR9Xn%2fKWzRm0cP88%3d&json_data=%7b%22id%22%3a%22d95dbee7aedc4e75a78e3dc09d1320aa%22%2c%22relation_id%22%3a%2212321%22%2c%22publish_date%22%3a%222015-08-24+16%3a45%3a04%22%2c%22user_id%22%3a%228899%22%2c%22ip%22%3a%22127.0.0.2%22%2c%22url%22%3a%22%e4%bd%a0%e5%a5%bd%ef%bc%81%22%7d
```

#### 编码前json_data样例

```
{
        "id": "781ca80b32954dec80c4f1de1dc476ea",
        "relation_id": "99999",
        "publish_date": "2016-09-19 10:41:17",
        "user_id": "shihaitao",
        "ip": "192.168.1.100",
        "contents": [
            {
                "type": "text",
                "content": "测试",
                "master": 1
            },
            {
                "type": "image",
                "content": "http://images.forwallpaper.com/files/images/9/917b/917be8ce/53118/carrie-underwood.jpg",
                "master": 0
            }
        ]
    }
```

### 2.4 应答描述

返回参数为Json数据。

#### Json返回参数说明

| 字段名 | 约束 | 类型 | 说明 |
| ---- | ---- | ---- | ---- |
| code | 必选 | Integer | 1001-操作正常，结果为通过。1002-调用正常，结果为拦截。1003-调用正常，人工审核。2000-不支持的操作。4000-上传参数有误。4005-授权校验失败。5000-服务器繁忙或异常 |
| msg | 必选 | string | code的对应说明。 |
| id | 必选 | String(32) | 数据Id |
| relation_id | 必选 | String(32) | 数据关联Id |
| timestamp | 必选 | string | 云净网服务器的时间，格式“yyyy-MM-dd HH:mm:ss” |
| reason | 必选 | string | 结果原因 |

#### 应答示例

```
HTTP/1.1 200 OK
Content-Length: 311
Server: Microsoft-HTTPAPI/2.0
Date: Mon, 24 Aug 2015 08:45:05 GMT

{
    "code": 1001,
    "msg": "OK-PASS",
    "id": "",
    "relation_id": "",
    "timestamp": "2015-08-24 16:45:05"
}
```

## 3 审核结果实时反馈接口协议

### 3.1 描述

云净网向客户实时推送审核结果的接口。客户开发服务端，云净网对客户配置的回调地址发起调用推送数据。客户需按以下规则开发http服务端以便接收云净网的审核结果。
用户调用API须遵循以下规则：

 - 传输方式：采用HTTP传输
 - 提交方式：POST方法提交，一次提交一条业务数据
 - 数据格式：请求参数见说明；返回参数为JSON格式
 - 字符编码：UTF-8字符编码
 - 签名算法：签名算法类型为HMAC_SHA1
 - 签名要求：请求和接收数据均需要签名认证
 - 注意：1）没有配置回调地址的云净网不会实时推送数据。2）接口请求超时时间为15秒，如超时则云净网会再次发起调用。3）如接口调用失败，总计推送3次失败后不再处理。

### 3.2 接口地址

> http://xxx.com/xxx(此地址可在客户的平台设置处进行配置)
 
### 3.3 请求描述

#### Http的Post参数说明

| 参数名称	| 参数类型	| 说明 | 
| ---- | ---- | ---- |
| version | String | 固定为2.0 | 
| sign | String(256) | 摘要内容：app_id,timestamp连接，用英文半角逗号“,”相连,算法见1.3 | 
| json_data	| string | 具体见json_data说明，采用json格式 | 

#### json_data说明

| 字段名 | 约束 | 类型 | 说明 |
| ---- | ---- | ---- | ---- |
| id | 必选 | string(32) | 内容的唯一主键 |
| relation_id | 必选 | string(32) | 关联内容的Id(例如留言、评论、回复等主内容的主键)
| app_id | 必选 | String(19) | 应用标识 |
| audit_date | 可选 | string(32) | 审核时间 |
| result | 可选 | Int | 0-违规；1-正常 |
| op_type | 可选 | Int | 判定类型：1-系统；2-人工 |
| reason | 必选 | string | 结果原因 |

#### 请求示例

```
POST /audit HTTP/1.1
Host: api.yunjingwang.cn:8080
Content-Length: 512
Expect: 100-continue
Connection: Keep-Alive

version=2.0&json_data=%7b%22id%22%3a%228a32158b836642c19e0807d0f13a73c3%22%2c%22relation_id%22%3a%2212321%22%2c&sign=XveWj8HGsjlSdo6%2fX7WTN%2bmh6bkpEt3XBb9VwsPP5V8uPjzBl%2fQNvVFRiJy5egsxBa0DDV20tQPUuN%2begbTR8z457717uHtTc4zYyrcJFeBTaao3au29d5iMUeXHDk%2bHH3ahRjRVOV06X5yaNO7EkAIGGlIqS4akvWlI61MDPV4%3d
```

#### 编码前Json_data样例

```
{"id":"7d46c2fd23b54990b50b78e59adea9d1", //用户业务数据Id
"relation_id":"12321",//用户业务数据(相关)扩展Id
"app_id":"201508200001",//应用标识
"audit_date":"2016-03-02 17:00:52",      //审核时间
"result":0,//0-违规；1-正常
"op_type":1}//判定类型：1-系统；2-人工
```

### 3.4 应答描述

返回参数为Json数据。

#### Json返回参数说明

| 字段名 | 约束 | 类型 | 说明 |
| ---- | ---- | ---- | ---- |
| code | 必选 | Integer | 200-成功。其余-失败。 |
| msg | 必选 | string | 发生错误时code的对应说明。 |

#### 应答示例

```
HTTP/1.1 200 OK
Content-Length: 23
Server: Microsoft-HTTPAPI/2.0
Date: Mon, 24 Aug 2015 08:45:05 GMT

{"code":200,"msg":"OK"}
```

## 4 审核结果拉取接口协议

### 4.1 描述

此协议是客户向云净网获取审核结果数据的接口。
用户调用API须遵循以下规则：

 - 传输方式：采用HTTP传输
 - 提交方式：POST方法提交，一次可返回多条业务数据
 - 数据格式：请求参数见说明；返回参数为JSON格式
 - 字符编码：UTF-8字符编码
 - 签名算法：用户生成签名字符串，现支持的摘要算法类型为HMAC_SHA1
 - 签名要求：请求和接收数据均需要签名认证
 - 审核结果：云净网平台每10分钟汇集一次结果数据，业务平台可以10分钟取一次结果数据。

### 4.2 接口地址

> http://api.yunjingwang.cn:8080/audit-result

### 4.3 请求描述

#### Http的Post调用参数说明

| 参数名称 | 参数类型 | 说明 |
| ---- | ---- | ---- |
| platform_id | string | 向云净网申请的平台Id，为整数值 |
| version | String | 固定为2.0 |
| timestamp | String | 发送请求的时间，格式“yyyy-MM-dd HH:mm:ss” |
| sign | String(256) | 摘要内容：platform_id,timestamp连接，用英文半角逗号“,”相连,算法见1.3 |

#### 请求示例

```
POST /audit HTTP/1.1
Host: api.yunjingwang.cn:8080
Content-Length: 512
Expect: 100-continue
Connection: Keep-Alive

platform_id=1&version=2.0&timestamp=2015-09-09 17:08:55&sign=XveWj8HGsjlSdo6%2fX7WTN%2bmh6bkpEt3XBb9VwsPP5V8uPjzBl%2fQNvVFRiJy5egsxBa0DDV20tQPUuN%2begbTR8z457717uHtTc4zYyrcJFeBTaao3au29d5iMUeXHDk%2bHH3ahRjRVOV06X5yaNO7EkAIGGlIqS4akvWlI61MDPV4%3d&json_data=%7b%22id%22%3a%228a32158b836642c19e0807d0f13a73c3%22%2c%22relation_id%22%3a%2212321%22%2c
```

### 4.4 应答描述

返回参数为Json数据。

#### Json返回参数说明

| 字段名 | 约束 | 类型 | 说明 |
| ---- | ---- | ---- | ---- |
| code | 必选 | Integer | 1001-操作正常，返回结果。2000-不支持的操作4000-上传参数有误4005-授权校验失败5000-服务器繁忙或异常 |
| msg | 必选 | string | code的对应说明。 |
| timestamp | 必选 | string | 云净网服务器的时间，格式“yyyy-MM-dd HH:mm:ss” |
| result_list | 必选 | string | Json集合数据，具体见result_list说明 |

#### result_list说明

| 字段名 | 约束 | 类型 | 说明 |
| ---- | ---- | ---- | ---- |
| id | 必选 | Integer | 平台用户传入的id，可用于删除或恢复数据的键值 |
| relation_id | 必选 | string | 平台用户接入时传入的relation_id，可用于删除或恢复数据的关联键值
| audit_date | 必选 | string | 云净网审核时间 |
| app_id | 必选 | string | 平台用户在云净网申请的app_id，用于区别不同业务 |
| result | 必选 | string | 审核结果，0-审核不通过；1-审核通过 |
| reason | 必选 | string | 结果原因 |

#### 应答示例

```
{
    "code": 1001,
    "msg": "OK-PASS",
    "timestamp": "2015-08-2416: 45: 05",
    "result_list": [
        {
            "id": "7d46c2fd23b54990b50b78e59adea9d1",
            "relation_id": "12321",
            "app_id": "201508200001",
            "audit_date": "2015-09-09 17:00:52",
            "result": 0
        },
        {
            "id": "f3c8f018c01c41f5befa7e13303cb61b",
            "relation_id": "12321",
            "app_id": "201508200001",
            "audit_date": "2015-09-09 17:05:56",
            "result": 0
        },
        {
            "id": "dcb805ebf53745a0811dce6955f13ed7",
            "relation_id": "12321",
            "app_id": "201508200002",
            "audit_date": "2015-09-09 17:06:57",
            "result": 0
        }
    ]
}
```

## 5 代码示例

### 5.1 Java代码示例

**示例附件**：[yunclean-api-java-2.0.zip](https://dn-yunclean.qbox.me/documents/docs_guide/yunclean-api-java-V2.0.zip)

### 5.2 PHP代码示例

**示例附件**： [yunclean-api-php-2.0.zip](https://dn-yunclean.qbox.me/documents/docs_guide/yunclean-api-php-2.0.zip)


## 6 对接常见问题

用户在调用“数据接入接口”时可能出现一些问题，主要是一些参数的格式以及签名算法不正确，总结如下。

 1. version：本版本固定为2.0
 2. timestamp：不是时间戳格式，需要传入标准的时间格式“yyyy-MM-dd HH:mm:ss”。
 3. sign：详见1.2节说明（当返回4005时，代表签名计算有误）
 4. contents：是一个json集合格式。即使只有1条数据，也是以[]包含具体的数据，如：

```
"contents": [{
    "type": "text",
    "content": "测试",
    "master": 1
}]
```
