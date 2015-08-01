
# ebao cloud web服务API开发文档

## 基本简介

本文档主要介绍如何利用ebao cloud开放的web服务API，对系统中的不同保险产品进行保险生命周期（如询价，出单，退保等）的各种操作。

## 快速入门

ebao cloud目前所有的web服务API基于HTTP协议，同时使用JSON的格式封装请求与返回的数据内容。开发者在使用所有的Web服务API之前需要申请开发者的access key和secret key。这两个key用于对API的请求进行签名，以及返回消息的签名验证。有关签名的规则请参考“Web服务API的签名规则”部分。

以下通过使用保单询价的API为示例介绍API的请求的基本步骤，有关询价API的具体细节可以参考“开放的Web服务API”部分对询价API的说明。

询价API的http请求内容如下：

```
POST /pa_web/api/policies/quotations?version=1&accessKey=TqlLeuUB1w&signature=a6e9c0014edee9378b233329d7045a6b HTTP/1.1
Host: localhost:8090
Content-Type: application/json
Cache-Control: no-cache

{
  "channelCode": "AXA_TP",
  "productId": "UbwoOpjMjt:1",
  "effectiveDate": "2015-07-16T03:07:53.970Z",
  "expiredDate": "2016-07-15T03:07:53.970Z",
  "campaigns": [
    {
      "code": "campaign_1",
      "voucher": "abcdef"
    }
  ],
  "policyHolder": {
    "PH001": "Shop.Zhang",
    "PH005": 1,
    "PH006": "132456199001011234",
    "PH015": "Shop.Zhang@123.com",
    "PH013": "15710023568",
    "PH003": "1990-01-01T00:00:00.000Z"
  },
  "policyInsuredPeople": [
    {
      "ISP001": "Shop.Zhang",
      "ISP004": "1",
      "ISP005": "132456199001011234",
      "ISP012": "Shop.Zhang@123.com",
      "ISP011": "15710023568",
      "ISP003": "1990-01-01T00:00:00.000Z"
    }
  ],
  "insuredObjects": [
    {
      "name": "标的相关信息",
      "code": "Veh_Owner",
      "type": "Person",
      "plan_code": "Plan_1",
      "factor_table": {
        "P040": "2",
        "P041": "沪A6U615",
        "AOP_limitAmount_ACC_BUR_WEB": 100000,
        "AOA_unitAmount_PEFF_G": 1000,
        "AOA_numberOfUnit_PEFF_G": 1,
        "AOA_unitType_PEFF_G": "件",
        "AOA_maxUnitAmount_PEFF_G": 2000
      },
      "componentSelection": {
        "Veh_Owner_Guard": true,
        "ACC_BUR_WEB": true,
        "ACC_DEATH": true,
        "ACC_DIS": true,
        "ACC_BURN": true
      }
    }
  ]
}
```

询价的返回结果内容如下：
```
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Content-Type: application/json;charset=UTF-8
Content-Length: 125

Date: Thu, 16 Jul 2015 03:15:45 GMT

{
  "fee": {
    "COMMISSION": 0,
    "APP": 30,
    "AGP": 30,
    "SNP": 30,
    "ANP": 30,
    "SGP": 30
  },
  "quotationNumber": "Q1507160000000010"
}
```

开发者需要利用编程语言相应的HTTP函数库，对询价的请求数据内容进行封装，发送请求到ebao cloud的Web服务API的应用服务器。

在询价API的请求URL中添加了accessKey和signature两个参数。accessKey是开发者身份的唯一标识，而signature是结合secret key对请求内容进行签名的结果。应用服务器接收到询价API的请求后，首先利用access key和signature对请求内容进行签名验证，以此保证请求发送方为access key的合法持有者，以及请求内容在传输过程中没有被篡改。

在签名验证后，应用服务器会将请求路由到对应的服务API处理具体的业务请求。对于返回的业务请求结果，应用服务器对利用开发者的secret key对内容进行签名，并将签名的结果放到HTTP的Response Header中，开发者在获得返回结果后需要对返回的内容进行签名验证，以此保证应用服务器返回的内容未被篡改。在验证成功后，开发者根据业务需求对返回的结果进行相应的处理。

## Web服务API的签名规则

所有Web服务API的基本签名规则是如下：

1. 获得accessKey对应的secret key。
2. 将需要签名的内容和secret key进行字符串拼接。
3. 对于拼接后的内容使用MD5算法进行签名运算。

对于开发者在发送HTTP POST请求之前，需要对HTTP Body的内容与secret key进行字符串拼接，然后进行MD5运算，运算的结果再添加到请求URL的signature参数上。对于应用服务器在接收到请求后，完成与开发者相同的步骤获取签名的结果，然后将结果与请求URL的signature进行比较。

应用服务器从相应的服务API获得请求的结果后，对要返回的内容与开发者的secret key进行字符串拼接，然后进行MD5运算，并将运算的结果添加到HTTP Reponse的Header中。开发者从应用服务器获得返回结果后，完成与应用服务器相同的步骤来验证HTTP Response Header中signature的有效性。

## 目前开放的Web服务API

### 询价

HTTP请求路径：**/pa_web/api/policies/quotations?version=1&accessKey=[ACCESS_KEY]&signature=[SIGNATURE]**

HTTP请求方法：**POST**

HTTP请求内容类型：**application/json**

HTTP请求内容：

```
{
  "channelCode": "AXA_TP",
  "productId": "UbwoOpjMjt:1",
  "effectiveDate": "2015-07-16T03:07:53.970Z",
  "expiredDate": "2016-07-15T03:07:53.970Z",
  "campaigns": [
    {
      "code": "campaign_1",
      "voucher": "abcdef"
    }
  ],
  "policyHolder": {
    "PH001": "Shop.Zhang",
    "PH005": 1,
    "PH006": "132456199001011234",
    "PH015": "Shop.Zhang@123.com",
    "PH013": "15710023568",
    "PH003": "1990-01-01T00:00:00.000Z"
  },
  "policyInsuredPeople": [
    {
      "ISP001": "Shop.Zhang",
      "ISP004": "1",
      "ISP005": "132456199001011234",
      "ISP012": "Shop.Zhang@123.com",
      "ISP011": "15710023568",
      "ISP003": "1990-01-01T00:00:00.000Z"
    }
  ],
  "insuredObjects": [
    {
      "name": "标的相关信息",
      "code": "Veh_Owner",
      "type": "Person",
      "plan_code": "Plan_1",
      "factor_table": {
        "P040": "2",
        "P041": "沪A6U615",
        "AOP_limitAmount_ACC_BUR_WEB": 100000,
        "AOA_unitAmount_PEFF_G": 1000,
        "AOA_numberOfUnit_PEFF_G": 1,
        "AOA_unitType_PEFF_G": "件",
        "AOA_maxUnitAmount_PEFF_G": 2000
      },
      "componentSelection": {
        "Veh_Owner_Guard": true,
        "ACC_BUR_WEB": true,
        "ACC_DEATH": true,
        "ACC_DIS": true,
        "ACC_BURN": true
      }
    }
  ]
}
```

| 属性  | 类型 | 说明 |
|:------------ |:---------------|:-----|
| channelCode | 字符串 | 渠道的Code |
| productId | 字符串 | 需要询价的产品ID |
| effectiveDate | 字符串 | 保单生效期 |
| expiredDate | 字符串 | 保单失效期 |
| campaigns | 数组 | 选中的优惠活动 |
| policyHolder | key-value | 保单持有人相关信息 |
| policyInsuredPeople | 数组 | 所有的被保险人的相关信息 |
| insuredObjects | 数组 | 所有的被保对象的相关信息 |

其中 policyHolder 对象的相关属性

| 属性  | 类型 | 说明 |
|:------------ |:---------------|:-----|
| PH001 | 字符串 | 投保人姓名 |
| PH003 | 字符串 | 投保人出生日期 |
| PH005 | 整数 | 投保人证件类型（1:身份证，2:驾驶证，3:护照，4:军官证，5:出生证，6:其它） |
| PH006 | 字符串 | 投保人证件号码 (校验：当证件类型为身份证（1）时，所录入号码的第7位至第14位需为日期)|
| PH013 | 字符串 | 投保人手机号 |
| PH015 | 字符串 | 投保人邮箱（校验：输入值中需包含@） |

其中 policyInsuredPeople 中每个对象的相关属性

| 属性  | 类型 | 说明 |
|:------------ |:---------------|:-----|
| ISP001 | 字符串 | 被保险人姓名 |
| ISP003 | 字符串 | 被保险人出生日期 |
| ISP004 | 整数 | 被保险人证件类型（1:身份证，2:驾驶证，3:护照，4:军官证，5:出生证，6:其它） |
| ISP005 | 字符串 | 被保险人证件号码 （校验：当证件类型为身份证（1）时，所录入号码的第7位至第14位需为日期）|
| ISP011 | 字符串 | 被保险人手机号 |
| ISP012 | 字符串 | 被保险人邮箱（校验：输入值中需包含@）|

其中 campaigns 中每个对象的相关属性

| 属性  | 类型 | 说明 |
|:------------ |:---------------|:-----|
| code | 字符串 | 优惠活动的Code |
| voucher | 字符串 | 优惠码 |

其中 insuredObjects 中每个对象的相关属性

| 属性  | 类型 | 说明 |
|:------------ |:---------------|:-----|
| type | 字符串 | 被保对象的类型 |
| plan_code | 字符串 | 被保对象选中的保险计划代码 |
| factor_table | key-value | 被保对象相关的风险因子，不同产品的风险因子会不相同，需要参考产品说明添加所需的风险因子 |
| componentSelection | key-value | 产品结构中选中的节点（Coverage或Coverage Group） |

询价的返回结果：

```
{
  "fee": {
    "COMMISSION": 0,
    "APP": 30,
    "AGP": 30,
    "SNP": 30,
    "ANP": 30,
    "SGP": 30
  },
  "quotationNumber": "Q1507160000000010"
}
```

| 属性  | 类型 | 说明 |
|:------------ |:---------------|:-----|
| quotationNumber | 字符串 | 询价结果的单号 |
| fee | key-value | 询价的费用结果 |

其中 fee 的属性说明

| 属性  | 类型 | 说明 |
|:------------ |:---------------|:-----|
| COMMISSION | 数字 | 佣金 |
| APP | 数字 | 实际应付保费 |
| AGP | 数字 | 调整后的毛保费 |
| SNP | 数字 | 标准净保费 |
| ANP | 数字 | 调整后的净保费 |
| SGP | 数字 | 标准毛保费 |

### 出单

HTTP请求路径：**/pa_web/api/policies/issuances?version=1&accessKey=[ACCESS_KEY]&signature=[SIGNATURE]**

HTTP请求方法：**POST**

HTTP请求内容类型：**application/json**

HTTP请求内容：
```
{
  "channelCode": "AXA_TP",
  "productId": "UbwoOpjMjt:1",
  "effectiveDate": "2015-07-16T03:07:53.970Z",
  "expiredDate": "2016-07-15T03:07:53.970Z",
  "quotationNumber": "Q1507160000000010",
  "campaigns": [
    {
      "code": "campaign_1",
      "voucher": "abcdef"
    }
  ],
  "policyHolder": {
    "PH001": "Shop.Zhang",
    "PH005": 1,
    "PH006": "132456199001011234",
    "PH015": "Shop.Zhang@123.com",
    "PH013": "15710023568",
    "PH003": "1990-01-01T00:00:00.000Z"
  },
  "policyInsuredPeople": [
    {
      "ISP001": "Shop.Zhang",
      "ISP004": "1",
      "ISP005": "132456199001011234",
      "ISP012": "Shop.Zhang@123.com",
      "ISP011": "15710023568",
      "ISP003": "1990-01-01T00:00:00.000Z"
    }
  ],
  "insuredObjects": [
    {
      "name": "标的相关信息",
      "code": "Veh_Owner",
      "type": "Person",
      "plan_code": "Plan_1",
      "factor_table": {
        "P040": "2",
        "P041": "沪A6U615",
        "AOP_limitAmount_ACC_BUR_WEB": 100000,
        "AOA_unitAmount_PEFF_G": 1000,
        "AOA_numberOfUnit_PEFF_G": 1,
        "AOA_unitType_PEFF_G": "件",
        "AOA_maxUnitAmount_PEFF_G": 2000
      },
      "componentSelection": {
        "Veh_Owner_Guard": true,
        "ACC_BUR_WEB": true,
        "ACC_DEATH": true,
        "ACC_DIS": true,
        "ACC_BURN": true
      }
    }
  ]
}
```

| 属性  | 类型 | 说明 |
|:------------ |:---------------|:-----|
| channelCode | 字符串 | 渠道的Code |
| productId | 字符串 | 需要询价的产品ID |
| effectiveDate | 字符串 | 保单生效期 |
| expiredDate | 字符串 | 保单失效期 |
| campaigns | 数组 | 选中的优惠活动 |
| policyHolder | key-value | 保单持有人相关信息 |
| policyInsuredPeople | 数组 | 所有的被保险人的相关信息 |
| insuredObjects | 数组 | 所有的被保对象的相关信息 |

其中 policyHolder 对象的相关属性

| 属性  | 类型 | 说明 |
|:------------ |:---------------|:-----|
| PH001 | 字符串 | 投保人姓名 |
| PH003 | 字符串 | 投保人出生日期 |
| PH005 | 整数 | 投保人证件类型（1:身份证，2:驾驶证，3:护照，4:军官证，5:出生证，6:其它） |
| PH006 | 字符串 | 投保人证件号码（校验：当证件类型为身份证（1）时，所录入号码的第7位至第14位需为日期）|
| PH013 | 字符串 | 投保人手机号 |
| PH015 | 字符串 | 投保人邮箱（校验：输入值中需包含@）|

其中 policyInsuredPeople 中每个对象的相关属性

| 属性  | 类型 | 说明 |
|:------------ |:---------------|:-----|
| ISP001 | 字符串 | 被保险人姓名 |
| ISP003 | 字符串 | 被保险人出生日期 |
| ISP004 | 整数 | 被保险人证件类型（1:身份证，2:驾驶证，3:护照，4:军官证，5:出生证，6:其它） |
| ISP005 | 字符串 | 被保险人证件号码（校验：当证件类型为身份证（1）时，所录入号码的第7位至第14位需为日期）|
| ISP011 | 字符串 | 被保险人手机号 |
| ISP012 | 字符串 | 被保险人邮箱（校验：输入值中需包含@）|

其中 campaigns 中每个对象的相关属性

| 属性  | 类型 | 说明 |
|:------------ |:---------------|:-----|
| code | 字符串 | 优惠活动的Code |
| voucher | 字符串 | 优惠码 |

其中 insuredObjects 中每个对象的相关属性

| 属性  | 类型 | 说明 |
|:------------ |:---------------|:-----|
| type | 字符串 | 被保对象的类型 |
| plan_code | 字符串 | 被保对象选中的保险计划代码 |
| factor_table | key-value | 被保对象相关的风险因子，不同产品的风险因子会不相同，需要参考产品说明添加所需的风险因子 |
| componentSelection | key-value | 产品结构中选中的节点（Coverage或Coverage Group） |

出单的返回结果

```
{
  "policyNumber": "PVeh_Owner_GuardA_TP15000000000106",
  "status":"EFFECTIVE"
}
```

| 属性  | 类型 | 说明 |
|:------------ |:---------------|:-----|
| policyNumber | 字符串 | 出单后的保单号 |

### 退保

HTTP请求路径：**/pa_web/api/endorsement/cancellation?version=1&accessKey=[ACCESS_KEY]&signature=[SIGNATURE]**

HTTP请求方法：**POST**

HTTP请求内容类型：**application/json**

HTTP请求内容：

```
{
  "policyNumber": "P106_B2C15000000003101",
  "effectiveDate": "2015-01-01T07:31:29.738Z",
  "applicationDate": "2015-01-01T07:31:29.738Z",
  "applicationType": "APLY_BY_INSURER"
}
```

| 属性  | 类型 | 说明 |
|:------------ |:---------------|:-----|
| policyNumber | 字符串 | 保单号 |
| effectiveDate | 字符串 | 退保起始日期 |
| applicationDate | 字符串 | 退保申请日期 |
| applicationType | 字符串 | 退保申请类型（APLY_BY_INSURER:由保险公司申请，APLY_BY_POLICY_HOLDER:由投保人申请，APLY_BY_UNDERWRITER:由核保专员申请）|


退保的返回结果
```
{
  "endorsement_number": "E1506160000000003",
  "success": true
}
```

| 属性  | 类型 | 说明 |
|:------------ |:---------------|:-----|
| endorsement_number | 字符串 | 退保单号 |
