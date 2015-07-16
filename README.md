
# ebao cloud web服务API开发文档

## 基本简介

本文档主要介绍如何利用ebao cloud开放的web服务API，对系统中的不同保险产品进行保险生命周期（如询价，出单，退保等）的各种操作。

## 快速入门

ebao cloud目前所有的web服务API基于HTTP协议，同时使用JSON的格式封装请求与返回的数据内容。开发者在使用所有的Web服务API之前需要申请开发者的access key和secrect key。这两个key用于对API的请求进行签名，以及返回消息的签名验证。有关签名的规则请参考“Web服务API的签名规则”部分。

以下通过使用保单询价的API为示例介绍API的请求的基本步骤，有关询价API的具体细节可以参考“开放的Web服务API”部分对询价API的说明。

询价API的http请求内容如下：

```
POST /pa_web/api/policies/quotations?accessKey=TqlLeuUB1w&signature=a6e9c0014edee9378b233329d7045a6b HTTP/1.1
Host: localhost:8090
Content-Type: application/json
Cache-Control: no-cache

{
  "channelId": 100003,
  "productId": "UbwoOpjMjt:1",
  "company": "安盛天平",
  "country": "cn",
  "effectiveDate": "2015-07-16T03:07:53.970Z",
  "expiredDate": "2016-07-15T03:07:53.970Z",
  "campaigns": [
    {
      "id": 100000,
      "voucher": "abcdef"
    }
  ],
  "factor_table": {
    "effectiveDate": "2015-07-16T03:07:53.970Z",
    "expiredDate": "2016-07-15T03:07:53.970Z",
    "P040": "2",
    "P041": "沪A6U615",
    "PH001": "Shop.Zhang",
    "PH005": 1,
    "PH006": "132456199001011234",
    "PH015": "Shop.Zhang@123.com",
    "PH013": "15710023568",
    "PH003": "1990-01-01T00:00:00.000Z",
    "PL009": "NO_FREE_INSURANCE",
    "ISP001": "Shop.Zhang",
    "ISP004": "1",
    "ISP005": "132456199001011234",
    "ISP012": "Shop.Zhang@123.com",
    "ISP011": "15710023568",
    "ISP003": "1990-01-01T00:00:00.000Z"
  },
  "calculationModel": {
    "mode": "policy",
    "insuredObjects": [
      {
        "name": "标的相关信息",
        "code": "Veh_Owner",
        "type": "Person",
        "plan_code": "Plan_1",
        "product_id": "UbwoOpjMjt:1",
        "channel_id": 100003,
        "factor_table": {
          "effectiveDate": "2015-07-16T03:07:53.970Z",
          "expiredDate": "2016-07-15T03:07:53.970Z",
          "P040": "2",
          "P041": "沪A6U615",
          "PH001": "Shop.Zhang",
          "PH005": 1,
          "PH006": "132456199001011234",
          "PH015": "Shop.Zhang@123.com",
          "PH013": "15710023568",
          "PH003": "1990-01-01T00:00:00.000Z",
          "PL009": "PAID_BY_INSURER",
          "ISP001": "Shop.Zhang",
          "ISP004": "1",
          "ISP005": "132456199001011234",
          "ISP012": "Shop.Zhang@123.com",
          "ISP011": "15710023568",
          "ISP003": "1990-01-01T00:00:00.000Z",
          "AOP_limitAmount_ACC_BUR_WEB": 100000,
          "AOA_unitAmount_PEFF_G": 1000,
          "AOA_numberOfUnit_PEFF_G": 1,
          "AOA_unitType_PEFF_G": "件",
          "AOA_maxUnitAmount_PEFF_G": 2000
        },
        "selection": {
          "a58df1a5-1c80-441b-bd5b-7cb29418bfbe": true,
          "8ddfce37-0e63-4039-beb3-9f62b7bfcd91": true,
          "3ccb2ffd-cf17-4887-ac5f-1e5ed75d3216": true,
          "83e39830-02a9-4f5e-9cf8-e211ad0cb4a6": true,
          "28e8b1fe-4826-4db3-9279-d2379bf0b1ad": true,
          "70d7a4e7-02e8-4321-98b9-5edc63f87a2e": true
        }
      }
    ]
  }
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

在询价API的请求URL中添加了ccessKey和signature两个参数。accessKey是开发者身份的唯一标识，而signature是结合secrect key对请求内容进行签名的结果。应用服务器接收到询价API的请求后，首先利用access key和signature对请求内容进行签名验证，以此保证请求发送方为access key的合法持有者，以及请求内容在传输过程中没有被篡改。

在签名验证后，应用服务器会将请求路由到对应的服务API处理具体的业务请求。对于返回的业务请求结果，应用服务器对利用开发者的secrect key对内容进行签名，并将签名的结果放到HTTP的Response Header中，开发者在获得返回结果后需要对返回的内容进行签名验证，以此保证应用服务器返回的内容未被篡改。在验证成功后，开发者根据业务需求对返回的结果进行相应的处理。

## Web服务API的签名规则

所有Web服务API的基本签名规则是如下：

1. 获得accessKey对应的secrect key。
2. 将需要签名的内容和secrect key进行字符串拼接。
3. 对于拼接后的内容使用MD5算法进行签名运算。

对于开发者在发送HTTP POST请求之前，需要对HTTP Body的内容与secrect key进行字符串拼接，然后进行MD5运算，运算的结果再添加到请求URL的signature参数上。对于应用服务器在接收到请求后，完成与开发者相同的步骤获取签名的结果，然后将结果与请求URL的signature进行比较。

应用服务器从相应的服务API获得请求的结果后，对要返回的内容与开发者的secrect key进行字符串拼接，然后进行MD5运算，并将运算的结果添加到HTTP Reponse的Header中。开发者从应用服务器获得返回结果后，完成与应用服务器相同的步骤来验证HTTP Response Header中signature的有效性。

## 目前开放的Web服务API

### 询价

HTTP请求路径：**/pa_web/api/policies/quotations?accessKey=[ACCESS_KEY]&signature=[SIGNATURE]**

HTTP请求方法：**POST**

HTTP请求内容：

```
{
  "channelId": 100003,
  "productId": "UbwoOpjMjt:1",
  "company": "安盛天平",
  "country": "cn",
  "effectiveDate": "2015-07-16T03:07:53.970Z",
  "expiredDate": "2016-07-15T03:07:53.970Z",
  "campaigns": [
    {
      "id": 100000,
      "voucher": "abcdef"
    }
  ],
  "factor_table": {
    "effectiveDate": "2015-07-16T03:07:53.970Z",
    "expiredDate": "2016-07-15T03:07:53.970Z",
    "P040": "2",
    "P041": "沪A6U615",
    "PH001": "Shop.Zhang",
    "PH005": 1,
    "PH006": "132456199001011234",
    "PH015": "Shop.Zhang@123.com",
    "PH013": "15710023568",
    "PH003": "1990-01-01T00:00:00.000Z",
    "PL009": "NO_FREE_INSURANCE",
    "ISP001": "Shop.Zhang",
    "ISP004": "1",
    "ISP005": "132456199001011234",
    "ISP012": "Shop.Zhang@123.com",
    "ISP011": "15710023568",
    "ISP003": "1990-01-01T00:00:00.000Z"
  },
  "calculationModel": {
    "mode": "policy",
    "insuredObjects": [
      {
        "name": "标的相关信息",
        "code": "Veh_Owner",
        "type": "Person",
        "plan_code": "Plan_1",
        "product_id": "UbwoOpjMjt:1",
        "channel_id": 100003,
        "factor_table": {
          "effectiveDate": "2015-07-16T03:07:53.970Z",
          "expiredDate": "2016-07-15T03:07:53.970Z",
          "P040": "2",
          "P041": "沪A6U615",
          "PH001": "Shop.Zhang",
          "PH005": 1,
          "PH006": "132456199001011234",
          "PH015": "Shop.Zhang@123.com",
          "PH013": "15710023568",
          "PH003": "1990-01-01T00:00:00.000Z",
          "PL009": "PAID_BY_INSURER",
          "ISP001": "Shop.Zhang",
          "ISP004": "1",
          "ISP005": "132456199001011234",
          "ISP012": "Shop.Zhang@123.com",
          "ISP011": "15710023568",
          "ISP003": "1990-01-01T00:00:00.000Z",
          "AOP_limitAmount_ACC_BUR_WEB": 100000,
          "AOA_unitAmount_PEFF_G": 1000,
          "AOA_numberOfUnit_PEFF_G": 1,
          "AOA_unitType_PEFF_G": "件",
          "AOA_maxUnitAmount_PEFF_G": 2000
        },
        "selection": {
          "a58df1a5-1c80-441b-bd5b-7cb29418bfbe": true,
          "8ddfce37-0e63-4039-beb3-9f62b7bfcd91": true,
          "3ccb2ffd-cf17-4887-ac5f-1e5ed75d3216": true,
          "83e39830-02a9-4f5e-9cf8-e211ad0cb4a6": true,
          "28e8b1fe-4826-4db3-9279-d2379bf0b1ad": true,
          "70d7a4e7-02e8-4321-98b9-5edc63f87a2e": true
        }
      }
    ]
  }
}
```

| 属性  | 类型 | 说明 |
|:------------ |:---------------|:-----|
| channelId | 整型 | 渠道的ID |
| productId | 字符串 | 需要询价的产品ID |
| effectiveDate | 字符串 | 保单生效期 |
| expiredDate | 字符串 | 保单失效期 |
| campaigns | 数组 | 选择的折扣活动 |
| factor_table | key-value | 询价所需的风险因子 |
| insuredObjects | 数组 | 所有的被保对象 |

### 出单
### 退保


