---
title: GTE API 文档

language_tabs: # must be one of https://git.io/vQNgJ
  - shell


includes:
  - errors

search: true
---

# 简介
## API简介
##

欢迎使用GTE API！ 你可以使用此 API 获得市场行情数据，进行交易，并且管理你的账户。 

在文档的右侧是代码，目前我们仅提供针对 shell 的代码示例。

# 更新日志

生效时间（北京时间UTC+8） | 接口 | 新增/修改 | 摘要
--------- | ------- | ----------- | ---------
2019.10.25 12：00 |  委托、深度、行情 | 新增 | -新增委托、深度、行情接口

# 基本信息

# Websocket 接口
## 示例
## 连接
### 地址
将ws客户端连接到wss://testnet.gte.com

### 命令
命令的基本形式```{"op": "<command>", "args": ["arg1", "arg2", "arg3"]}```  
\<command\>值见命令目录  
`args`数组在某些命令中是可选项  

命令目录
- 订阅Subscriptions
  - subscribe
  - unsubscribe
- 权鉴Authentication
- 心跳Heartbeating  
  - raw ping
  - cancelAllAfter

## 订阅Subscriptions
连上ws后，订阅行情数据的推送，发送消息 `{"op": "subscribe", "args": [<SubscriptionTopic>]}`  
你可以通过`args`数组同时订阅多个主题  

无需权鉴的主题：  

`"announcement",        // Site announcements  `  
`"chat",                // Trollbox chat  `  
`"connected",           // Statistics of connected users/bots`  
"funding",             // Updates of swap funding rates. Sent every funding interval (usually 8hrs)  
"instrument",          // Instrument updates including turnover and bid/ask  
"insurance",           // Daily Insurance Fund updates  
"liquidation",         // Liquidation orders as they're entered into the book  
"orderBookL2_25",      // Top 25 levels of level 2 order book  
"orderBookL2",         // Full level 2 order book  
"orderBook10",         // Top 10 levels using traditional full book push  
"publicNotifications", // System-wide notifications (used for short-lived messages)  
"quote",               // Top level of the book  
"quoteBin1m",          // 1-minute quote bins  
"quoteBin5m",          // 5-minute quote bins  
"quoteBin1h",          // 1-hour quote bins  
"quoteBin1d",          // 1-day quote bins  
"settlement",          // Settlements  
"trade",               // Live trades  
"tradeBin1m",          // 1-minute trade bins  
"tradeBin5m",          // 5-minute trade bins  
"tradeBin1h",          // 1-hour trade bins  
"tradeBin1d",          // 1-day trade bins  


需要权鉴的主题：  

```  
"affiliate",   // Affiliate status, such as total referred users & payout %
"execution",   // Individual executions; can be multiple per order
"order",       // Live updates on your orders
"margin",      // Updates on your current account balance and margin requirements
"position",    // Updates on your positions
"privateNotifications", // Individual notifications - currently not used
"transact"     // Deposit/Withdrawal updates
"wallet"       // Bitcoin address balance data, including total deposits & withdrawals
```  

`orderBookL2`推送完整的L2订单簿，数据量很大；  
推荐使用`orderBookL2_25`订阅行情，推送截断的25档L2订单簿  
注：L2订单簿中的`id`字段，实际上是price和symbol的组合，并且在给定的价格上是唯一的。它被用于`update`和`delete`等指令。  
`orderBook10`推送10档，但是包含更多数据；

## 权鉴Authentication
- 公开的行情、订单簿、成交列表等数据流无需鉴权；
- 账户的委托、订单、成交等账户相关数据需要鉴权；
- 如果鉴权失败，连接将会关闭。

鉴权方式：
- 在第一次请求的时候，在http head中签名，或者
- 在ws连接建立后，发送`authKeyExpires`命令。  

以上两种方式都是对地址`GET /realtime`的请求。  

```
// signature is hex(HMAC_SHA256(secret, 'GET/realtime' + expires))
// expires must be a number, not a string.
{"op": "authKeyExpires", "args": ["<APIKey>", <expires>, "<signature>"]}
```

## 心跳Heartbeating  
检测连接是否掉线。  
如果客户端使用的库支持`hybi-13`或`ping/pong`，可以发送ping到服务端，服务端返回pong。  
Due to changes in browser power-saving modes, we no longer support expectant pings via the WebSocket API.  
建议的掉线检测方式：
- 收到一条消息后，设定5秒钟的timer；
- 如果在timer到期之前收到新的消息，reset timer；
- 如果timer到期，发送一条ping消息（或者字符串"ping"）；
- 等待服务器返回pong（或字符串"pong"），如果5秒钟内没有收到回复，记录错误并重连。  

### Dead Man’s Switch (Auto Cancel)
某些意外情况下，例如你在GTE有较大的活动订单，但同时你的机房断电或断网了，你希望你的订单被全部取消。
Via REST at /order/cancelAllAfter or via WebSocket via the example below, one can set a millisecond timeout. This will start a timer. If cancelAllAfter is not called again before time runs out, all of your existing orders on all symbols will be canceled.  

To cancel this operation and keep your orders open, pass a timeout of 0.  

Advanced users of BitMEX should use this operation. A common use pattern is to set a timeout of 60000, and call it every 15 seconds. This gives you sufficient wiggle room to keep your orders open in case of a network hiccup, while still offering significant protection in case of a larger outage. Of course, the parameters are up to you.  

We recommend not setting a timeout of less than 5 seconds to avoid being rate limited or having your orders unexpectedly canceled in case of network congestion.  

For example:  
```
// Places a dead man's switch at 60 seconds.
> {"op": "cancelAllAfter", "args": 60000}
  < {"now":"2015-09-02T14:18:43.536Z","cancelTime":"2015-09-02T14:19:43.536Z",
     "request":{"op":"cancelAllAfter","args":60000}}
// Cancels the switch.
> {"op": "cancelAllAfter", "args": 0}
  < {"now":"2015-09-02T14:19:11.617Z","cancelTime":0,"request":{"op":"cancelAllAfter","args":0}}
```


## 消息格式
## 访问限制

# Http Restful API 

## BaseUrl

* `http://{ip}:port`

* baseUrl/v1/api/pc/asset/query v1为版本号

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

## 查询交易所支持的资产

**查询交易所支持的资产**

**示例**

* GET `baseUrl/v1/api/asset/query`


```shell
# Response
{
  "code": 0,
  "data": {
    "rows": [
      {
        "asset": "BTC"
      },
      {
        "asset": "ETH"
      }
    ]
  },
  "input": null,
  "traceId": "",
  "cost": 0,
  "error": null,
  "msg": null,
  "time": 1573640300898
}
```


## 查询资金账户

**查询资金账户**

**示例**

* GET `baseUrl/v1/api/account/fund/query`

**请求头**

名称 | 类型 | 是否必须 | 描述
----- | ---- | ----- | -----
api-key | string | YES | apiKey
api-expires | string | YES | 当前时间戳毫秒
api-signature | string | YES | api签名


**请求参数**

名称 | 类型 | 是否必须 | 描述
----- | ---- | ----- | -----
asset | string | NO |  资产,不填查询全部资产


```shell
# Response
{
    "code":0,
    "data":{
        "rows":[
            {
                "lock":"0",              //锁定的额度
                "available":"9",     //余额
                "asset":"BTC",           //资产
                "total":"9"          //总额
            }
        ]
    },
    "input":null,
    "traceId":"",
    "cost":0,
    "error":null,
    "msg":null,
    "time":1573639309095
}
```


## 查询永续合约账户

**查询永续合约账户**

**示例**

* GET `baseUrl/v1/api/account/pc/query`

**请求头**

名称 | 类型 | 是否必须 | 描述
----- | ---- | ----- | -----
api-key | string | YES | apiKey
api-expires | string | YES | 当前时间戳毫秒
api-signature | string | YES | api签名


**请求参数**

名称 | 类型 | 是否必须 | 描述
----- | ---- | ----- | -----
asset | string | NO |  资产,不填查询全部资产


```shell
# Response
{
    "code":0,
    "data":{
        "rows":[
            {
                "available":"7",      //可用余额      
                "asset":"BTC",        //资产
                "total":"10",         //总额
                "order_margin":"1",   //委托保证金
                "pos_margin":"2"      //仓位保证金
            }
        ]
    },
    "input":null,
    "traceId":"",
    "cost":0,
    "error":null,
    "msg":null,
    "time":1573639479143
}
```

## 账户划转

**账户划转**

**示例**

* POST `baseUrl/v1/api/account/transfer`

**请求头**

名称 | 类型 | 是否必须 | 描述
----- | ---- | ----- | -----
api-key | string | YES | apiKey
api-expires | string | YES | 当前时间戳毫秒
api-signature | string | YES | api签名


**请求参数**

名称 | 类型 | 是否必须 | 描述
----- | ---- | ----- | -----
asset | string | YES |  资产
from | string | YES |  转出账户类型
to | string | YES |  转入账户类型
volume | string | YES |  数量,最小8位小数点


**说明**

账户类型 1.资金账户,2.合约账户


```shell
# Response
{
    "code":0,
    "data":{
        "time":null
    },
    "input":null,
    "traceId":"",
    "cost":0,
    "error":null,
    "msg":null,
    "time":1573639855769
}
```

## 查询永续合约的交易对

**查询永续合约的交易对**

**示例**

* GTE `baseUrl/v1/api/pc/contract/query`


**请求参数**

名称 | 类型 | 是否必须 | 描述
----- | ---- | ----- | -----
asset | string | yes | 资产

```shell
# Response
{
  "code": 0,
  "data": {
    "time": "1573642959704",
    "rows": [
      {
        "symbol": "BTC_USD",    //交易对
        "price_precision": "1"  //价格的最大小数位
      },
      {
        "symbol": "ETH_USD",
        "price_precision": "1"
      },
      {
        "symbol": "EOS_USD",
        "price_precision": "6"
      }
    ]
  }
}
```



## 查询instrument数据

**查询instrument数据**

**示例**

* GTE `baseUrl/v1/api/pc/instrument/query`


**请求参数**

名称 | 类型 | 是否必须 | 描述
----- | ---- | ----- | -----
asset | string | yes | 资产
symbol | string | no | 交易对 不填查询全部交易对

```shell
# Response
{
  "code": 0,
  "data": {
    "time": "1573635077085",
    "rows": [
      {
        "precision": "1",           //交易对小数位
        "asset": "BTC",             //资产
        "symbol": "ETH_USD",        //交易对
        "last_price": "150",        //最新成交价
        "index_price": "186.6",     //指数价格
        "mark_price": "185.9",      //标记价格
        "volume_24h": "230",        //24小时成交量(张数)
        "volume_pos_hold": "172",   //所有持仓量(张数)
        "change_rate_24h": "0.1",   //24小时前的涨幅 10%
        "face_value": "1",          //一张合约的面值
        "quote_currency": "USD",    //合约面值的计价货币
        "funding_rate": "-0.00375", //当期资金费率
        "indicative_funding_rate": "-0.00375",   //下期的资金费率
        "funding_rate_time": "8",                //收取资金费用的时间
        "indicative_funding_rate_time": "16"     //下去收取资金费用的时间
      }
    ]
  },
  "input": null,
  "traceId": "",
  "cost": 0,
  "error": null,
  "msg": null,
  "time": 1573635077085
}
```

## 查询永续合约的k线

**查询永续合约的k线**

**示例**

* GTE `baseUrl/v1/api/pc/candle/query`


**请求参数**

名称 | 类型 | 是否必须 | 描述
----- | ---- | ----- | -----
asset | string | YES | 资产
symbol | string | YES | 交易对
start_time | string | NO | 开始时间,时间戳毫秒
end_time | string | NO | 结束时间,时间戳毫秒
interval | string | NO | k线时间间隔,分钟
size | string | NO | 返回条数

**说明**
interval值 1 5 15 30 60 120 240 360 720 1440 10080 43200
size 最大1440


```shell
# Response
{
  "code": 0,
  "data": {
    "time": "1573641132754",
    "interval": "1",         
    "rows": [
      [
        "1573640940000",    //k线的时间戳毫秒
        "10009",            //开盘价
        "10009",            //最高价
        "10009",            //最低价
        "10009",            //收盘价
        "2"                 //量(张数) 
      ]
    ],
    "asset": "BTC",
    "symbol": "BTC_USD"
  },
  "input": null,
  "traceId": "",
  "cost": 0,
  "error": null,
  "msg": null,
  "time": 1573641132755
}
```


## 查询深度

**查询深度**

**示例**

* GTE `baseUrl/v1/api/pc/order/book/query`


**请求参数**

名称 | 类型 | 是否必须 | 描述
----- | ---- | ----- | -----
asset | string | YES | 资产
symbol | string | YES | 交易对
size | string | NO | 返回数量

**说明**

size 填写15,买卖共30条,最大50

```shell
# Response
{
  "code": 0,
  "data": {
    "time": "1573643118030",
    "bids": [             //bid 买入
      [
        "8888",           //价格
        "136"             //量
      ],
      [
        "8000",
        "96"
      ]
    ],
    "asks": [            //ask 
      [
        "10009",
        "229"
      ],
      [
        "10021",
        "188"
      ]
    ],
    "asset": "BTC",          //资产
    "symbol": "BTC_USD"      //交易对
  },
}
```


## 创建活动委托


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
asset | string | YES | 资产
symbol | string | YES | 交易对
side | string | YES | 买入1,卖出0
close_flag | string | YES | 0开仓,1平仓
price | string | NO | 价格
qty | string | YES | 张数
order_type | string | YES | 委托类型,1:限价

**说明**
order_type等于1,price必填


```shell
# Response
      
{
    "code":0,
    "data":{
        "time":"1573645810154",
        "order_id":"114813904143597952"   //委托id
    },
    "input":null,
    "traceId":"",
    "cost":0,
    "error":null,
    "msg":null,
    "time":1573645810155
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
asset | string | YES | 资产
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

**查询委托**

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
asset | string | YES | 资产
symbol | string | NO | 交易对
filter | string | NO | 过滤器, json格式的字符串
gt_order_id | string | NO | order_id,请求大于order_id的数据,gt和lt都填,以gt为准
lt_order_id | string | NO | order_id,请求小于order_id的数据
count | string | NO | 返回条数最大100条


**filter**

* json格式的字符串

* 支持 status(委托状态) , 类型 字符串数组

* 例如{"status":["1","8"]}

```shell
# Response
{
    "code":0,
    "data":{
        "time":"1568639823340",
        "rows":[
            {    
                "order_id":"114813904143597952", //委托id
                "status":"4", //委托状态  1:已创建未匹配 2:新建未成交 4:待取消 8:已取消 16:部分成交 32:全部成交     
                "fee":"0.023",    //手续费
                "price":"8500",   //价格
                "qty":"1",        //张数
                "leverage":"5",   //杠杆
                "ctime":"1573645767887",     //订单创建时间
                "client_oid":null,           //客户端id
                "avg_price":"0",             //成交均价
                "filled_qty":"0",            //已成交张数
                "close_flag":"0",            //0开仓,1平仓       
                "side":"1",                  //买入1,卖出0
                "asset":"BTC",               //资产
                "symbol":"BTC_USD",          //交易对
                "order_margin":"0.000023",   //委托保证金
                "order_type":"1"             //委托类型
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

## 查询成交记录

**查询成交记录**

**示例**

* GTE `baseUrl/v1/api/pc/order/query`


**请求参数**

名称 | 类型 | 是否必须 | 描述
----- | ---- | ----- | -----
asset | string | YES | 资产
symbol | string | YES | 交易对
gt_trade_id | string | NO | 成交记录id,请求大于trade_id的数据,gt和lt都填,以gt为准
lt_trade_id | string | NO | 成交记录id,请求小于trade_id的数据
count | string | NO | 返回条数,最大100条



```shell
# Response
{
  "code": 0,
  "data": {

    "rows": [
      {
        "price": "8999",    //成交价格
        "qty": "4",         //成交数量张数
        "trade_id": "115025316845580800", //成交记录id
        "asset": "BTC",                   //资产    
        "symbol": "BTC_USD",              //交易对
        "trade_time": "1573696172596"     //交易时间毫秒
      }
    ]
  },
  "input": null,
  "traceId": "",
  "cost": 0,
  "error": null,
  "msg": null,
  "time": 1573703825189
}
```

## 调整杠杆


### 调整杠杆111

**调整杠杆**

**示例**

* POST `baseUrl/v1/api/pc/leverage/change`

**请求头**

名称 | 类型 | 是否必须 | 描述
----- | ---- | ----- | -----
api-key | string | YES | apiKey
api-expires | string | YES | 当前时间戳毫秒
api-signature | string | YES | api签名


**请求参数**

名称 | 类型 | 是否必须 | 描述
----- | ---- | ----- | -----
asset | string | YES | 资产
symbol | string | YES | 交易对
mode | string | YES | 仓位模式,1.全仓,2.逐仓
long_flag | string | NO | 1.多仓,0.空仓,全仓模式可以不填
leverage | string | YES | 调整杠杆数



```shell
# Response
{
  "code": 0,
  "data": {

    "rows": [
      {
        "asset": "BTC",             //资产    
        "symbol": "BTC_USD",        //交易对
        "mode": "1",     //仓位模式,1.全仓,2.逐仓
        "leverage":"3"   //杠杆

      }
    ]
  },
  "input": null,
  "traceId": "",
  "cost": 0,
  "error": null,
  "msg": null,
  "time": 1573703825189
}
```

