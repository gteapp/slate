---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell


includes:
  - errors

search: true
---

# 简介
## API简介

欢迎使用GTE API！ 你可以使用此 API 获得市场行情数据，进行交易，并且管理你的账户。

在文档的右侧是代码，目前我们仅提供针对 shell 的代码示例。

# 更新日志

生效时间（北京时间UTC+8） | 接口 | 新增/修改 | 摘要
--------- | ------- | ----------- | ---------
2019.10.25 12：00 |  委托、深度、行情 | 新增 | -新增委托、深度、行情接口

# 基本信息
## BaseUrl

* `http://{ip}:port`

## API签名说明
* 部分api请求需要在请求头携带签名


* 请求头说明

   * api-key ：交易所用户在api管理页面创建的apiKey

   * api-expires : 当前时间的时间戳(毫秒)

   * api-signature ： api签名


* 签名规则

   *  构建签名的明文message分三部分

      1 api-key:请求头携带

      2 api-expires：请求头携带

      3 请求参数组成的字符串

         * 请求参数根据参数名称按照ascii值排序,

             * 例如查询委托,排序后的字符串为 {"count":"20","end_time":"1570701726000","filter":"{"status":"1","order_id":"123000001"}","start_time":"1570788306000","symbol":"BTC_USD"}

             * 如果参数count为空或为null,不需要加入排序
   
             * filter字段 {key:value} json类型字符串,filter字段参加排序,字段内容不需要排序


   * 秘钥

       * apiSecret为创建api-key时显示的对应秘钥

   * 加密成api签名

      * message等于api-key+api-expires+请求参数排序后的字符串

      * hmacSha256（apiSecret,message） 加密成api签名,由api-signature携带
    
  

# Http Restful 

## 行情接口

**查询行情数据**

**示例**

* GTE `baseUrl/v1/api/pc/ticker/query`


**请求参数**

名称 | 类型 | 是否必须 | 描述
----- | ---- | ----- | -----
symbol | string | YES | 交易对

```shell
# Response
{
  "code": 0,
  "data": {
    "time": "1568636015342",
    "symbol": "BTC_USD",           //交易对
    "best_ask": "10001",         //卖一价
    "best_bid": "10000",         //买一价
    "last_price": "10000",       //最新成交价
    "low_24h": "9999",           //24小时最低价
    "high_24h": "10400",         //24小时最高价
    "volume_24h": "0.003997"     //24小时成交量
  },
  "input": null,
  "traceId": "",
  "cost": 0,
  "error": null,
  "msg": null,
  "time": 1568636015343
}
```


## 深度接口

**查询永续合约信息**

**示例**

* GTE `baseUrl/v1/api/pc/depth/query`


**请求参数**

名称 | 类型 | 是否必须 | 描述
----- | ---- | ----- | -----
symbol | string | YES | 交易对

```shell
# Response
{
  "code": 0,
  "data": {
    "time": "1568635734093",
    "ssymbol": "BTC_USD",      //交易对
    "bids": [
      [
        "10000",   //价格
        "2848"     //数量
      ],
      [
        "9999",
        "1"
      ],
      [
        "9998",
        "30"
      ]
    ],
    "asks": [
      [
        "10001",
        "59"
      ],
      [
        "10002",
        "1"
      ],
      [
        "10003",
        "3"
      ]
    ]
  },
  "input": null,
  "traceId": "",
  "cost": 0,
  "error": null,
  "msg": null,
  "time": 1568635734093
}
```

## 创建委托


**创建活动委托**

**示例**

* POST `baseUrl/v1/api/pc/order/create`

**请求头**

名称 | 类型 | 是否必须 | 描述
----- | ---- | ----- | -----
api-key | string | YES | apiKey
api-expires | string | YES | 当前时间戳毫秒
api-signature | string | YES | api签名


**请求参数**

名称 | 类型 | 是否必须 | 描述
----- | ---- | ----- | -----
client_oid | string | NO | 客户端订单Id,由您设置的订单id来唯一标识您的订单,字符串,长度40
symbol | string | YES | 交易对
side | string | YES | 买入1,卖出0
close_flag | string | YES | 0开仓,1平仓
price | string | NO | 价格
qty | string | YES | 张数
order_type | string | YES | 委托类型,1:限价



```shell
# Response
{
  "code": 0,
  "data": {
    "time": "1568639111357",
    "order_id": "93814490360972800"   //委托id
  },
  "input": null,
  "traceId": "",
  "cost": 0,
  "error": null,
  "msg": null,
  "time": 1568639111359
}
```


## 撤销委托


**撤销活动委托**

**示例**

* POST `baseUrl/v1/api/pc/order/cancel`

**请求头**

名称 | 类型 | 是否必须 | 描述
----- | ---- | ----- | -----
api-key | string | YES | apiKey
api-expires | string | YES | 当前时间戳毫秒
api-signature | string | YES | api签名


**请求参数**

名称 | 类型 | 是否必须 | 描述
----- | ---- | ----- | -----
id | string | YES | 委托id
symbol | string | YES | 交易对



```shell
# Response
{
  "code": 0,
  "data": {
    "time": "1568639111357",
    "order_id": "93814490360972800"   //委托id
  },
  "input": null,
  "traceId": "",
  "cost": 0,
  "error": null,
  "msg": null,
  "time": 1568639111359
}
```



## 查询委托

**查询活动委托**

**示例**

* GTE `baseUrl/v1/api/pc/order/query`

**请求头**

名称 | 类型 | 是否必须 | 描述
----- | ---- | ----- | -----
api-key | string | YES | apiKey
api-expires | string | YES | 当前时间戳毫秒
api-signature | string | YES | api签名


**请求参数**

名称 | 类型 | 是否必须 | 描述
----- | ---- | ----- | -----
symbol | string | NO | 交易对
filter | string | NO | 过滤器, json格式的字符串
start_time | string | NO | 时间戳毫秒,不填为当前时间
end_time | string | NO | 时间戳毫秒
reverse | string | NO | 1.时间倒序,0.时间顺序,不填为倒序
count | string | NO | 返回条数最大100条

**查询逻辑**

* reverse 等于1时,按照时间倒序排列, 返回小于等于start_time和大于等于end_time的活动委托

* reverse 等于0时,按照时间程序排列, 返回大于等于start_time和小于等于end_time的活动委托

**filter**

* json格式的字符串

* 支持 key status(委托状态) , 类型 字符串数组

* 例如{"status":["1","8"]}

```shell
# Response
{
    "code":0,
    "data":{
        "time":"1568639823340",
        "orders":[
            {
                "id":"93800123938637312",   //委托id
                "fee":"0",                  //手续费
                "price":"10000",            //价格
                "qty":"2",                  //张数
                "leverage":"10",            //杠杆
                "symbol":"BTC_USD",           //交易对
                "ctime":"1568635692270",    //创建时间
                "client_oid":null,          //客户端id
                "avg_price":"0",            //成交均价
                "filled_qty":"1",           //已经成交张数
                "close_flag":"0",           //0开仓,1平仓        
                "side":"1",                  //买入1,卖出0
                "order_margin":"0.00001",   //委托保证金
                "order_type":"1",            //委托类型
                "status":"2"                //委托状态  1:已创建未匹配 2:新建未成交 4:待取消 8:已取消 16:部分成交 32:全部成交     
            }
        ]
    },
    "input":null,
    "traceId":"",
    "cost":0,
    "error":null,
    "msg":null,
    "time":1568639823342
}
```



