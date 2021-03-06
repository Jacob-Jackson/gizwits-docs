title: 企业API
---

# 概述
企业API是机智云为接入机智云平台的企业开发者提供的开放API服务，使用企业API的企业将设备接入到机智云平台后，通常还有进一步基于接入机智云设备数据开展企业某个垂直领域的业务需求。企业API为企业提供企业视角全局的设备管理、数据分析等功能，让企业更关注业务管理系统本身，减少不必要的开发成本。



# 协议约定
## 1、请求方式
本文档所定义接口基于HTTP/HTTPS协议进行传输，需要注意协议中标注的请求方式，通过GET、PUT、DELETE等进行不同的操作。
## 2、接口地址
接口地址中用${ }号包含的为变量，需要使用者通过对应的变量替换。例如${product_key}表示需要将获取的product_key值赋予到接口地址中，${token值}表示是需要替换为通过授权接口获取的token，${did} 表示是需要替换为具体的did。
##  3、请求参数
HTTP请求参数的类型一般分为三种。Header表示该参数是在HTTP请求头中；URL表示是通过url传参；Body表示是Request Body，通常Body中都是JSON格式

##  4、HTTP请求头部
本文档协议中设备管理类、设备报表查询类的接口在进行接口访问时，都需要在请求头增加token值，以此校验访问者是否有权访问该接口。token值是通过获取授权接口获得。
请求头格式如下：
```json
Content-Type: application/json
Authorization: token ${token值}
```
注意：${token值}是不包括${}号，只需将从获取token接口获得token值放到token之后。例如：Authorization: token  efbekskdklllsF

## 5、HTTP响应头部
本协议中的接口在返回报文头部会输出如下信息：
```json
X-RateLimit-Limit: 60     //接口允许访问总量
X-RateLimit-Remaining: 56 //接口剩余访问次数
X-RateLimit-Reset: 1372700873 //调用频率限制重置时间，TS类型
```
##  6、Token值的生命周期
Token值有效期为7天， 调用获取token接口返回的expired_at为失效日期时间戳。若现在时间戳 > expired_at时间戳，则需要重新获取token, token调用请见"获取Token"

##  7、接口请求配额说明
企业API每小时默认允许企业API接口每小时调用3600次，超过阈值则抛错，1小时后恢复接口调用。具体抛错请查看5.2错误信息表系统编码5013的相关说明

#  接入场景
## 1、申请企业API服务
在调用企业API接口之前，需要先申请该服务并审批通过后，才可使用企业API。具体申请流程详见 [企业应用开发](./ent_dev.html)
## 2、设备远程控制
### 场景描述
该业务场景是指企业通过企业应用系统去控制接入到机智云平台的设备，管理设备的主要需求就是在业务管理系统中对设备发起控制。
### 接入流程
- 获取token
- 获得设备id（did）：需要通过消息代理的客户端获取到设备状态数据中的did
- 调用远程控制接口：如果产品定义了数据点协议，则可以使用“数据点方式远程控制”去向设备发起控制；如果是数据透传协议，则通过“原始指令方式远程控制”向设备发起控制

## 3、设备数据聚合查询
### 场景描述
该业务场景是指接入企业想通过机智云提供设备数据的聚合查询，在企业应用系统中展现设备的数值型数据的聚合报表，这样企业的业务管理系统就无需存储设备数据并进行较容易出现性能问题的数据查询。

目前设备数据点聚合查询提供单台设备级别的查询以及产品级别的查询。主要能实现以下业务功能：
- 只支持对数值类型的数据点进行求和(sum)、求平均值（avg)、求最大值（max)、求最小值（min)
- 支持按月、周、天、小时四个时间维度计算数据
- 目前支持最大时间范围是最近一个月的数据
- 有特殊时间需求需要向机智云申请


# 接口协议
##  1、获取Token

### [调试地址](http://swagger.gizwits.com/doc/index/debug_enterprise#!/授权/post_v1_products_product_key_access_token)

### 业务功能描述
该接口提供获取企业API接口访问权限的功能
### 接口地址
     http://enterpriseapi.gizwits.com/v1/products/${product_key}/access_token
### 请求方式
     POST
### 请求报文
```json
{
  "enterprise_id": "ad7e60f0594247dcba017ba76d2f4275",
  "enterprise_secret": "db2ac98200824181b447b03e4d42f99b",
  "product_secret": "8f11ee69eb9d4269ba0777ca5e7280f5"
}
```
### 应答报文
```json
Http Response Code ： 201
返回报文：
{
  "token": "7cf955ec8b504d6497d83f39ce9b16d2",
  "expired_at": 1372700873   //Expired_at为token过期时间；
}  
```

## 2、原始指令方式远程控制

### [调试地址](http://swagger.gizwits.com/doc/index/debug_enterprise#!/设备管理/post_v1_products_product_key_devices_did_control)

### 业务功能描述
该接口提供通过原始控制指令远程控制在线设备的功能。
### 接口地址
     http://enterpriseapi.gizwits.com/v1/products/${product_key}/devices/${did}/control
### 请求方式
    POST
### 请求报文
1. Header
```json
Content-Type: application/json
Authorization: token ${token值}
```

2. Body
```json
{
    "raw": [<byte>, <byte>, ...]
}  	
```


原始控制指令，以byte数组方式传送，每个byte的范围必须为0~255，十六进制的格式需要转换为十进制。假设要发送的原始控制指令为：01020304ffff（十六进制），那么 raw 的值为 [1, 2, 3, 4, 255, 255]。
若带有数据点，建议直接使用数据点方式发送。 调用方式请见“数据点方式远程控制”。



### 应答报文
```json
Http Response Code ： 200
返回报文：
{}  
```

## 3、数据点方式远程控制

### [调试地址](http://swagger.gizwits.com/doc/index/debug_enterprise#!/设备管理/post_v1_products_product_key_devices_did_control)

### 业务功能描述
该接口提供远程设置设备数据点的功能，完成对在线设备的控制操作
### 接口地址
     http://enterpriseapi.gizwits.com/v1/products/${product_key}/devices/${did}/control

### 请求方式
     POST

### 请求报文
1. Header
```json
Content-Type: application/json
Authorization: token ${token值}
```

2. Body
```json
{
  "attrs": {
     "${数据点标识名}": ${数据点值},    
  }
}  	
```

bool 类型的数据点值设置为 true/false
enum 类型的数据点值设置为枚举的字符串
uint8/uint16/uint32 类型的数据点值设置为数字
binary 类型的数据点值设置为 hex 类型字符串，如发送一串二进制数据 0x01, 0x02, 0x03, 就写成 "010203”
	例如在机智云开发者中心定义了一个产品的数据点例如是开关，标识名为“switch”，类型为bool类型；如果要关闭该设备，则报文为 "attrs": {"switch": true},可同时传送多个数据点值。

### 应答报文
```json
Http Response Code ： 200
返回报文：
{}  
```




## 4、单台设备数据聚合查询

### [调试地址](http://swagger.gizwits.com/doc/index/debug_enterprise#!/设备管理/get_v1_products_product_key_devices_did_agg_data)

### 业务功能描述
该接口提供查询某个时间周期内某个wifi设备数值型数据点的聚合运算，包括求和、平均、最大、最小的计算。
### 接口地址
     http://enterpriseapi.gizwits.com/v1/products/${product_key}/devices/${did}/agg_data?start_ts=${start_ts}&end_ts=${end_ts}&attrs=${数据点标识名}&aggregator=sum&unit=DAYS
### 请求方式
    GET
### 请求报文
|    参数    |  类型   | 必填 | 参数类型 |                            描述                             | 
|:---------- | -------:|:----:|:-------- |:------------------------------------------------------- |
| token      |  String |  是  | header   | 获取授权返回的token值, For example: Authorization:token xxx |    
| did        |  String |  是  | URL      | 设备id                                                   |     
| start_ts   | integer |  是  | URL      | 开始时间戳                                                |     
| end_ts     | integer |  是  | URL      | 结束时间戳                                                |     
| attrs      |  String |  是  | URL      | 只能为数值类型的数据点，对应为数据点标识名                  |      
| aggregator |  String |  是  | URL      | sum,avg,max,min                                             |     
| unit       |  String |  是  | URL      | 展示时间维度的单位，包括：HOURS,DAYS,WEEKS,MONTHS           |      



### 应答报文
```json
Http Response Code ： 200
Body：
{
    "query": {
        "start_ts": 1447837618000,
        "end_ts": 1447837828000,
        "attrs": "weight,fat",
        "aggregator": "sum",
        "unit": "DAYS",
        "did": "xxx"
    },
    "data": [{
        "attrs": {
            "weight": 50,
            "fat": 0.2
        }
        "datetime": xxx
    },
    {
        "attrs": {
            "weight": 50,
            "fat": 0.2
        }
        "datetime": xxx
    }]
}
```
返回报文中的 datetime 格式:
HOURS: "2015072010"
DAYS: "20150720"
WEEKS: "201529"
MONTHS: "201507"




## 5、设备历史数据查询

### [调试地址](http://swagger.gizwits.com/doc/index/debug_enterprise#!/设备管理/get_v1_products_product_key_devices_did_data)

### 业务功能描述
该接口提供获取某个产品某台设备的历史数据，如温度值等功能；
### 接口地址
    http://enterpriseapi.gizwits.com/v1/products/${product_key}/devices/${did}/data?start_ts=${start_ts}&end_ts=${end_ts}&limit=20&skip=0
### 请求方式
    GET
### 请求报文
|参数    |类型  |必填    |参数类型     |描述   |
| :-------- | --------:| :--: |:-------- | :-------- |
|did    |String   |是|URL|设备id|
|start_ts   |integer   |是|URL|开始时间戳|
|end_ts   |integer   |是|URL|结束时间戳|
|limit   |integer   |是|URL| |
|skip   |integer   |是|URL| |		|

### 应答报文
```json
Http Response Code ： 200
Body：
	{
  "meta": {
    "total": 100,
    "limit": 20,
    "skip": 0,
    "next": "/v1/products/8f11ee69eb9d4269ba0777ca5e7280f5/devices/gdGn7PzAYf4VrhnVag5x8D/data?start_ts=1372700873&end_ts=1372701873&skip=20&limit=20",
    "previous": null
  },
  "objects": [
    {
      "ts": 1372701873,
      "attrs": {
        "temp": 20,
        "humi": 60
      }
    },
    {
      "ts": 1372701880,
      "attrs": {
        "temp": 20,
        "humi": 60
      }
    }
  ]
}

```



## 6、搜索设备

### [调试地址](http://swagger.gizwits.com/doc/index/debug_enterprise#!/设备管理/get_v1_products_product_key_devices_search)

### 业务功能描述
该接口提供查询产品的设备列表，包括了设备的地理位置信息（如国家，省，市，经纬度），上下线记录，活跃时间，设备是否报警，是否发生故障等。
### 接口地址
     http://enterpriseapi.gizwits.com/v1/products/{product_key}/devices/search

### 请求方式
    GET

### 请求报文

|      参数      |  类型   | 必填 | 参数类型 |            描述            | 
|:-------------- | -------:|:----:|:-------- |:-------------------------- |
| product_key    |  String |  是  | path     |                            |     
| gid            |  String |  否  | query    | 设备组gid                     |     
| country        |  String |  否  | query    | 国家                       |      
| region         |  String |  否  | query    | 省                         |      
| city           |  String |  否  | query    | 城市                       |      
| is_online      | integer |  否  | query    | 是否在线,在线为1,不在线为0 |      
| is_faulty      | integer |  否  | query    | 是否故障,故障为1,无故障为0 |      
| is_alert       | integer |  否  | query    | 是否报警,报警为1,无报警为0 |      
| show_disabled  | integer |  否  | query    | 显示注销为1，过滤注销为0   |     
| liveness_start | integer |  否  | query    | 最近活跃时间戳             |      
| type           |  String |  否  | query    | 可以为 did、mac、uid |  
| val            |  String |  否  | query    | 搜索的值                   |      
| limit          | integer |  否  | query    |                            |      
| skip           | integer |  否  | query    |                            |      



### 应答报文
```json
Http Response Code ： 200
Body：
{
  "meta": {
    "total": 0,
    "limit": 0,
    "skip": 0,
    "next": "string",
    "previous": "string"
  },
  "objects": {
    "did": "string",
    "mac": "string",
    "is_online": 0,
    "country": "string",
    "region": "string",
    "city": "string",
    "longitude": "string",
    "latitude": "string",
    "is_faulty": 0,
    "is_alert": 0,
    "latest_online": 0,
    "created_at": 0
  }
}  
```

## 7、获取设备详情

### [调试地址](http://swagger.gizwits.com/doc/index/debug_enterprise#!/设备管理/get_v1_products_product_key_device_detail)

### 业务功能描述
该接口提供查询设备的最新的注册信息。
### 接口地址
     http://enterpriseapi.gizwits.com/v1/products/{product_key}/device_detail

### 请求方式
    GET

### 请求报文

|参数    |类型  |必填    |参数类型    |描述   |
| :-------- | --------:| :--: |:-------- | :-------- | 
|product_key  |String|是|path|
|mac   |String|是|query|mac 地址|



### 应答报文
```json
Http Response Code ： 200
Body：
{
  "product_key": "string",
  "mac": "string",
  "did": "string",
  "is_online": true,
  "is_disabled": true,
  "type": "string"
}
```

## 8、获取设备信息

### [调试地址](http://swagger.gizwits.com/doc/index/debug_enterprise#!/设备管理/get_v1_products_product_key_devices)

### 业务功能描述
该接口提供了通过查询设备的product_key和mac等参数，来获取最新的设备did。
### 接口地址
     http://enterpriseapi.gizwits.com/v1/products/{product_key}/devices

### 请求方式
    GET

### 请求报文

|参数    |类型  |必填    |参数类型    |描述   |
| :-------- | --------:| :--: |:-------- | :-------- |
|product_key  |String|是|path||
|mac   |String|是|query|mac 地址| 



### 应答报文
```json
Http Response Code ： 200	
Body：
{
  "did": "string",
  
}
```

## 9、设备上下线记录

### [调试地址](http://swagger.gizwits.com/doc/index/debug_enterprise#!/设备管理/get_v1_products_product_key_devices_did_online)

### 业务功能描述

**设备上下线记录 ** 
- 按 ts 降序排序。  
- 设备的心跳统计信息，在设备上下线记录的 Payload 中。  
- 设备上线记录中，payload 记录了设备设备指定的心跳时间间隔（KeepAlive），单位为秒，此 KeepAlive 的值等于设备上传的心跳时间间隔再加上5秒。  
- 设备下线记录中，payload 记录了设备的在线时长（duration），以及在线时发送的心跳次数（heartbeat/count），相邻两个心跳的最大间隔时间（heartbeat/max），最小间隔时间（heartbeat/min），平均间隔时间（heartbeat/avg）, 最后一次收到心跳时刻与下线时刻的间隔时间（heartbeat/last），单位为秒  
- 心跳的间隔时间，为m2m收到第n次心跳的时间点（时间戳）与第n-1次心跳的时间点（时间戳）之差，其中n >= 2；当n < 2时，max，min，avg的值固定为0  
- max，min：计算客户端从接到第一次心跳开始，到最后一次接到心跳为止的时间段内，相邻两个心跳时间差的最大，最小值；  
- avg：客户端与云端建立链接开始，到最后一次接到心跳为止的时间段，除以心跳次数；    
- 设备下线记录中，payload记录了设备的下线原因（reason），说明各种reason的意义：
- mqtt_disconnect：设备主动断开与mqtt的连接  
- no_heartbeat：m2m在KeepAlive时段内，没有收到设备心跳  
- tcp_closed：设备主动断开tcp连接  
- ssl_closed：设备主动断开ssl连接  
- offline_force：设备重复上线，原有的连接断开  
- offline_reset：设备注销，断开连接  
- offline_exception：异常断开连接  
- offline_sending_density_overflow：客户端发送信息的频率过大，断开链接  
- offline_sending_data_size_overflow：客户端发送信息的流量过大，断开链接  
**ChangeLog**  
- 0.4.2.1 start_ts 和 end_ts不填，默认查询过去到现在两天以内的通信日志记录  
- 0.4.2.1 start_ts与end_ts之间的间隔秒必须在两天范围以内  
- 0.4.2.1 增加sort排序，默认为降序，asc代表升序，desc代表降序  
### 接口地址
     http://enterpriseapi.gizwits.com/v1/products/{product_key}/devices/{did}/online

### 请求方式
    GET

### 请求报文

|参数    |类型  |必填    |参数类型    |描述   |
| :-------- | --------:| :--: |:-------- | :-------- |
|product_key  |String|是|path|
|did    |String|是|query|设备id| 
|start_ts|integer|否|query|开始时间戳| 
|end_ts|intege|否|query|结束时间戳| 
|sort|String|否|query|desc、asc| 
|limit|integer|否|query|
|skip|integer|否|query|



### 应答报文
```json
Http Response Code ： 200	
Body：
{
  "meta": {
    "total": 0,
    "limit": 0,
    "skip": 0,
    "next": "string",
    "previous": "string"
  },
  "objects": [
    {
      "timestamp": 0,
      "type": "string",
      "payload": "string"
    }
  ]
} 
```


## 10、设备通信日志

### [调试地址](http://swagger.gizwits.com/doc/index/debug_enterprise#!/设备管理/get_v1_products_product_key_devices_did_cmd)

### 业务功能描述
设备通信日志  
- 按 ts 降序排序。  
- payload 为二进制进行 base64 编码后的结果。  
ChangeLog  
- 0.4.2.1 start_ts 和 end_ts不填，默认查询过去到现在两天以内的通信日志记录  
- 0.4.2.1 start_ts与end_ts之间的间隔秒必须在两天范围以内  
- 0.4.2.1 增加sort排序，默认为降序，asc代表升序，desc代表降序  

### 接口地址
     http://enterpriseapi.gizwits.com/v1/products/{product_key}/devices/{did}/cmd

### 请求方式
    GET

### 请求报文

|参数    |类型  |必填    |参数类型    |描述   |
| :-------- | --------:| :--: |:-------- | :-------- | 
|product_key  |String|是|path|
|did    |String|是|query|设备id| 
|start_ts|integer|否|query|开始时间戳| 
|end_ts|intege|否|query|开始时间戳| 
|sort|String|否|query|desc、asc| 
|limit|integer|否|query|
|skip|integer|否|query|



### 应答报文
```json
Http Response Code ： 200	
Body：
{
  "meta": {
    "total": 0,
    "limit": 0,
    "skip": 0,
    "next": "string",
    "previous": "string"
  },
  "objects": [
    {
      "timestamp": 0,
      "type": "string",
      "payload": "string"
    }
  ]
} 
```


## 11、获取设备地理位置分布（实时）
### [调试地址](http://swagger.gizwits.com/doc/index/debug_enterprise#!/设备报表/get_v1_products_product_key_devices_locations)

### 业务功能描述
该接口提供了通过查询设备的product_key和gid等参数，来获取最新的设备地理位置分布的功能。
### 接口地址
     http://enterpriseapi.gizwits.com/v1/products/{product_key}/devices/locations

### 请求方式
    GET

### 请求报文

|参数    |类型  |必填    |参数类型    |描述   |
| :-------- | --------:| :--: |:-------- | :-------- | 
|product_key  |String|是|path| 
|gid   |String|否|query|设备组 id| 
|is_online |integer|否|query|是否在线| 
|is_faulty|integer|否|query|是否故障| 
|is_alert|integer|否|query|是否报警| 



### 应答报文
```json
Http Response Code ： 200	
Body：
{
  "gid": "string",
  "total": 0,
  "location": {}
}
```


## 12、用户新增报表
### [调试地址](http://swagger.gizwits.com/doc/index/debug_enterprise#!/用户报表/get_v1_products_product_key_users_report_new)

### 业务功能描述
该接口提供了通过查询设备的product_key和设置起始周期、结束周期等参数，按日、周、月等三种统计方式来获取绑定了该PK下产品的新增用户的功能。
### 接口地址
     http://enterpriseapi.gizwits.com/v1/products/{product_key}/users/report/new

### 请求方式
    GET

### 请求报文

|参数    |类型  |必填    |参数类型    |描述   |
| :-------- | --------:| :--: |:-------- | :-------- | 
|product_key  |String|是|path| 
|gid   |String|否|query|设备组 id| 
|cycle|String|否|query|统计周期，可以为 date、week、month| 
|start|String|是|query|起始周期，为起始日期、起始周或起始月| 
|end|String|是|query|结束周期，为结束日期、结束周或结束月| 



### 应答报文
```json
Http Response Code ： 200	
Body：
{
  "gid": "string",
  "cycle": "string",
  "start": "string",
  "end": "string",
  "data": [
    {
      "date": "string",
      "count": 0,
      "location": {}
    }
  ]
}
```


## 13、搜索用户
### [调试地址](http://swagger.gizwits.com/doc/index/debug_enterprise#!/用户管理/get_v1_products_product_key_users_search)

### 业务功能描述
该接口查询的是绑定用户信息
### 接口地址
     http://enterpriseapi.gizwits.com/v1/products/{product_key}/users/search

### 请求方式
    GET

### 请求报文

|参数    |类型  |必填    |参数类型    |描述   |
| :-------- | --------:| :--: |:-------- | :-------- | 
|product_key  |String|是|path| 
|gid   |String|否|query|设备组 id| 
|type   |String|否|query|可以是uid、username、phone、email、did、mac、device_sn| 
|val|String|否|query|查询条件值| 
|limit|	integer|否|query|每次返回的条数| 
|skip|integer|否|query|每次跳过的条数| 



### 应答报文
```json
Http Response Code ： 200	
Body：
{
  "meta": {
    "total": 0,
    "limit": 0,
    "skip": 0,
    "next": "string",
    "previous": "string"
  },
  "objects": [
    {
      "uid": "string",
      "username": "string",
      "phone": "string",
      "email": "string",
      "name": "string",
      "gender": "string",
      "birthday": "string",
      "address": "string",
      "remark": "string",
      "created_at": 0,
      "is_anomymous": true,
      "auth_src": {}
    }
  ]
}
```



















# 接口错误

## 错误信息格式
```json
{
      "error_code": "5001",
      "error_message": "body json invalid",
      "detail": ""
}
```

## 错误信息表
|HTTP响应编码|	系统错误编码	| 错误信息描述 | 	解决办法 |
| ------------- |:-------------:|:-------------|:-------------|
|400|   5001|   json字串格式错误|   核对json字串|
|400|   5002|   form invalid|   输入数据不对|
|404|   5003|   enterprise id not exist|   Eid不存在，检查是否申请或者Eid输入错误|
|400|   5004|   enterprise secret error|   Esecret校验失败，检查是否是正确的|
|400|   5005|   product secret error|   Product Secret校验失败，检查是否是正确的|
|400|   5006|   product exist devicegroups| |
|404|   5007|   association not exist|   表示Eid没有雨对应的产品Product Key 关联，必须成功关联才能操作|
|400|   5008|   association existed| |
|400|   5009|   token invalid|   请携带token或检查token字段格式,正确的token格式是在http header中输入:token ${token值}，token后面必须空一格后再写入具体的token值|
|400|   5010|   token未匹配|   请核对token|
|400|	5011|	token过期|	请再次获取token|
|400|	5012|	发起关联请求的主机ip无访问权限|   添加ip到ip企业数据访问白名单中|
|403|	5013|	接口使用过于频繁|   等待一定时间后再次使用此接口|
|400|	5014|	Report has not been generated!|	 |
|404|	5015|	Product Key不存在|   核对产品Product Key是否正确|
|403|	5201|	device group not belong to this product|  |
|404|	5202|	parent group not exist| |
|400|	5203|	already has one root group| |
|400|	5204|	group has subgroup|	 |
|400|	5205|	group has device item| |
|404|	5206|	group not exist| |
|404|	5301|	设备不存在|   核对设备ID|
|403|	5302|	产品与设备未绑定|   请先将设备绑定到产品|
|400|	5303|	device not bound| |
|400|	5304|	设备未激活|   激活设备|
|400|	5305|	设备处于下线状态|   上线设备|
|400|	5401|	数据点错误|	核对数据点信息|
|400|	5402|	数据点未定义|   先定义数据点,然后再次尝试|
|400|	5403|	控制命令发送失败|   请再次使用此接口|
|400|	5404|	remote control not allowed|   远程控制操作需要后台开启|
