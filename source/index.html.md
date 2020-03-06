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

# websocket 验证

ws连接地址 wss://td.gte.io/ws

客户端每20秒发送字符串{"op":"ping"},服务端返回{"op":"pong"},服务端2分钟没有收到客户端消息,自动断开客户端

订阅或取消等会返回字符串,{"status":"0","request":"客户端请求字符串"},status等于0,表示成功,小于0
表示错误,具体错误查看错误代码

## auth 验证


**参数说明**

api_key: 用户注册api_key时获取

expires: 当前时间的时间戳(毫秒),推荐当前时间加上3~5秒,防止请求过期

signature: 签名 

**签名说明**

加密方式: hmacSha256（apiSecret,url+expires） 

apiSecret: 注册api_key时获取

url = "GET/ws"   


**验证**

{
  
    "op":"auth_key_expires",

    "args":{
    "api_key":"UVpYHxxxIzdhcsTFFZuuHhIp",
    "expires":"1575365733524",
    "signature":"e36acaa887df38b1527e96234f5c730a56dcfc2a327cd269f2cf4a6e6515e00f"
    }

}

# websocket 永续合约


## instrument 


**说明**

最新的各种价格,行情,交易量等,pc为永续合约,instrument为订阅管道,BTC为资产,BTC_USD为交易对

订阅返回全量数据

推送返回数据变更的字段

action的值partial,update

partial 订阅返回全量数据

update 更新


**订阅**

{
  
    "op":"sub",

    "args":{"instrument_type":"pc","table":"instrument","settle_currency":"BTC","symbol":"BTC_USD"}

}

**取消订阅**

{
    
    "op":"unsub",

    "args":{"instrument_type":"pc","table":"instrument","settle_currency":"BTC","symbol":"BTC_USD"}

}


```shell
# Response
{
    "table":"instrument",
    "action":"partial"
    "key":["change_rate_24h","quote_currency"] //data数据中的字段名
    "data":[
      {
        "instrument_type":"pc",          //交易资产类别,pc永续合约
        "settle_currency":"BTC",    //资产
        "change_rate_24h":"-0.1112",   //24小时价格变化幅度，-0.13代表跌了13%
        "lot_size":"1",              //合约面值,1表示1张合约价格1个quote_currency
        "quote_currency":"USD",        //面值计价货币
        "funding_rate":"-0.0037",      //当期资金费率
        "funding_rate_time":"2",       //当期资金费率到期时间小时
        "indicative_funding_rate":"-0.0037",  //预期资金费率
        "indicative_funding_rate_time":"10",  //预期资金费率到期时间小时
        "index_price":"9374.1",         //指数价格
        "last_price":"9994",                   //最新成交价
        "mark_price":"9365.3",                 //标记价格
        "split_char":"_",                     //交易对分隔符
        "symbol":"BTC_USD",                   //交易对
        "volume_24h":"30022",                 //24小时成交量
        "volume_pos_hold":"269016",            //未平仓的合约持仓量
        "price_precision": "1",           //交易对价格小数位数
      }  
    ],
    "time":"1573022591950"
}
```

## trade 


**说明**

推送最新的成交记录,pc为永续合约,trade为订阅管道,BTC为资产,BTC_USD为交易对


**订阅**

{

     
     "op":"sub",

     "args":{"instrument_type":"pc","table":"trade","settle_currency":"BTC","symbol":"BTC_USD"}

}

**取消订阅**

{
    
    "op":"unsub",

    "args":{"instrument_type":"pc","table":"trade","settle_currency":"BTC","symbol":"BTC_USD"}

}


```shell
# Response
{
    "table":"trade",
    "data": 
        [
          
              {
                "instrument_type":"pc",          //交易资产类别,pc永续合约
                "settle_currency":"BTC",            //资产
                "symbol":"BTC_USD",                //交易对
                "trade_time": "1564295185000",     //成交时间 
                "side": "sell",                    //sell,buy  
                "price": "1000.00",                //成交价格
                "qty": "3"                         //成交量(张数)              
              }
            
        ]
     ,
   "time": "1564295185000"        // 服务器返回数据的时间戳毫秒
}
```


## orderBook 


**说明**

订阅返回全量数据,推送返回增量数据,pc为永续合约,order_book为订阅管道,BTC为资产,BTC_USD为交易对


action的值partial,delete,update,insert

partial 订阅返回全量数据

delete 删除当前价格的深度

update 更新当前价格的深度,量为最终值

insert 新增当前价格的深度

id当前交易对唯一表示,id和price相互反推,symbol_id和tick_size可以在订阅instrument中获取

计算公式
id = (100000000 * symbol_id) - (price / tick_size)
price = ((100000000 * symbol_id) - id) * tick_size

**订阅**

{
    
    "op":"sub",

   "args":{"instrument_type":"pc","table":"order_book","settle_currency":"BTC","symbol":"BTC_USD"}

}

**取消订阅**

{
    
    "op":"sub",

    "args":{"instrument_type":"pc","table":"order_book","settle_currency":"BTC","symbol":"BTC_USD"}

}


```shell
# Response
{
    "action":"partial",
    "table":"order_book",     
    "data":[
     
            {
                "id":"399911110",               //唯一标识
                "instrument_type":"pc",            //交易资产类别,pc永续合约             
                "settle_currency":"BTC",       //资产
                "price":"8889",               //价格
                "side":"sell",                //买卖方向
                "size":"590",                 //量(张数)
                "symbol":"BTC_USD"            //交易对
            },
            {
                "id":"399911200",
                "instrument_type":"pc",       
                "settle_currency":"BTC",
                "price":"8880",
                "side":"buy",
                "size":"677",
                "symbol":"BTC_USD"
            }

        
    ],
    "time":"1573281525515"
}
```


## order 

**需要验证**


**说明**

推送当前用户的活动委托的变更信息



**订阅**

{
  
    "op":"sub",

    "args":{"instrument_type":"pc","table":"order","settle_currency":"BTC"}

}

**取消订阅**

{
    
    "op":"unsub",

    "args":{"instrument_type":"pc","table":"order","settle_currency":"BTC"}

}



```shell
# Response 
{
    "action":"insert",
    "data":[
        {
            "instrument_type":"pc",      //交易类型 pc永续合约
            "avg_price":"0",             //成交均价
            "close_flag":"0",            //0.开仓,1.pc
            "ctime":"1575353429226",     //委托创建时间
            "fee":"0",                   //手续费
            "filled_qty":"0",            //成交张数
            "leverage":"15",             //杠杆
            "order_id":"121976354928411008",  //order_id
            "order_margin":"0.000878",        //委托保证金   
            "order_type":"1",                 //委托类型1.限价委托
            "price":"155",                    //委托价格
            "qty":"2",                        //委托张数
            "settle_currency":"ETH",          //资产
            "side":"1",                       //1.买入,0.卖出
            "status":"1",                     // //委托状态  1:待报 2:已报 4:待撤销 8:已撤销 16:部分成交 32:全部成交  
            "symbol":"ETH_USD",               //交易对
            "client_oid":"123456"             //客户端自己定义的id
        }
    ],
    "keys":[

    ],
    "table":"order",
    "time":"1575353430109"
}
```


## execution 

**需要验证**


**说明**

推送当前用户的委托的执行信息



**订阅**

{
    
    "op":"sub",

    "args":{"instrument_type":"pc","table":"execution","settle_currency":"BTC"}

}

**取消订阅**

{
    
    "op":"unsub",

    "args":{"instrument_type":"pc","table":"execution","settle_currency":"BTC"}
        
}


```shell
# Response 
{
    "action":"insert",
    "data":[
        {
            "instrument_type",:"pc",      //交易类型 pc永续合约
            "ctime":"1575526474231",              //execution时间
            "cum_qty":"2",                        //累计成交量
            "exec_id":"122702112030277888",       //执行id
            "exec_type":"8",                      //执行类型 1.新增,2.未成交,4.成交,8,完成,16.撤销
            "last_qty":"2",                       //上一次推送快照的委托量
            "leaves_qty":"0",                     //委托未成交量
            "maker_flag":"1",                     //1.当前委托执行是maker,0.taker
            "order_id":"122702086658933120",       //委托id
            "order_price":"7500",                 //委托价格
            "order_qty":"2",                      //委托量
            "order_status":"32",                  //委托状态
            "order_type":"1",                     //委托类型
            "settle_currency":"BTC",              //资产
            "side":"1",                           //1.买入,0.卖出
            "close_flag":"1",                     //1.平仓,0.开仓
            "symbol":"BTC_USD",                   //交易对
            "trade_match_id":"d6fb023b-2d31-4db2-8335-b6b0e6bf3570"  //成交撮合id
        }
    ],
    "keys":[

    ],
    "table":"execution",
    "time":"1575526474231"
}
```

## position 

**需要验证**


**说明**

定时推送用户的仓位信息



**订阅**

{
    
    "op":"sub",

    "args":{"instrument_type":"pc","table":"position","settle_currency":"BTC"}

}

**取消订阅**

{
    
    "op":"unsub",

    "args":{"instrument_type":"pc","table":"position","settle_currency":"BTC"}

}


```shell
# Response 
{
    "action":"update",
    "data":[
        {
            "instrument_type",:"pc",      //交易类型 pc永续合约
            "settle_currency":"BTC",                //资产
            "auto_increase_flag":"0",     //当前仓位是否自动追加保证金 1.是,0.否
            "avail_qty":"49",             //仓位可平量
            "entry_price":"6711.6",       //开仓均价
            "leverage":"15",              //仓位杠杆
            "liquidation_price":"6368.2", //强平价
            "side":"1",                    //1.多仓，0.空仓
            "maint_margin_ratio":"0.005", //默认的维持保证金比率
            "margin_mode":"2",            //保证金模式 1.全仓,2.逐仓
            "pnl":"0.0006",               //未实现盈亏
            "pos_margin":"0.000437",      //仓位保证金
            "pos_margin_ratio":"0.1638",  //仓位保证金率
            "pos_pnl_ratio":"0",          //收益率
            "qty":"49",                   //仓位持仓量
            "realised_pnl":"0",           //已实现盈亏
            "symbol":"BTC_USD"            //交易对
        }
    ],
    "keys":[

    ],
    "table":"position",
    "time":"1575528039621"
}
```


## margin 

**需要验证**


**说明**

定时推送用户的永续合约账户信息



**订阅**

{
    
    "op":"sub",

    "args":{"instrument_type":"pc","table":"margin","settle_currency":"BTC"}

}

**取消订阅**

{
    
    "op":"unsub",

    "args":{"instrument_type":"pc","table":"margin","settle_currency":"BTC"}

}


```shell
# Response 
{
    "action":"partial",
    "data":[
        {
            "available":"9.2392817",         //余额
            "instrument_type":"pc",         //交易类型 pc永续合约
            "order_margin":"0",             //委托保证金
            "pos_margin":"0.4583322",       //仓位保证金
            "settle_currency":"BTC",        //资产类别
            "total":"9.6976139"             //总额
        }
    ],
    "keys":[

    ],
    "table":"margin",
    "time":"1576573209743"
}
```

# http 基本信息

## url

*  baseUrl `https://api.gte.io/`

*  /v1/api/pc/asset/query v1为版本号

* 返回数据code等于0,表示请求成功,小于0表示错误,具体清查看错误代码

## API签名说明
* 部分api请求需要在请求头携带签名


* 请求头说明

   * api-key ：交易所用户在api管理页面创建的apiKey

   * api-expires : 当前时间的时间戳(毫秒)

   * api-signature ： api签名


* 签名规则  
    - 构建签名的明文message：分三部分构成  
	    1 api-key:请求头携带

        2 api-expires：请求头携带

        3 请求参数组成的字符串 paras_string:  
         
            * 根据参数名称(key)按照ascii值排序, 例如查询委托,排序后的字符串为 `{"count":"20","end_time":"1570701726000","filter":"{"status":["1"],"order_id":"123000001"}","start_time":"1570788306000","symbol":"BTC_USD"}`  
            * 如果参数count为空或为null,不需要加入排序  
            * 复杂数据类型字段，例如filter字段 {key:value} json类型字符串,filter字段本身参加排序,字段内容不需要排序  
            * 构造字符串paras_string时建议统一去除空格，例如上例中 `...filter":"{"status":["1"],"order_id":"123000001"}...` 和 `...filter":"{"status": ["1"],"order_id": "123000001"}...`(带有空格)将导致不同的签名  
            * 发送给服务端的参数字符串(例如带空格或反斜杠)，和参加签名的重构字符串 paras_string（例如不带空格或反斜杠）不一致，也是签名错误的常见原因  
        
  - 秘钥   

       * apiSecret为创建api-key时显示的对应秘钥  

  - 加密成api签名   

      * message等于api-key+api-expires+请求参数排序后的字符串  

      * hmacSha256（apiSecret,message） 加密成api签名,由api-signature携带   


# http 账户

## 查询资金账户

**限速规则**：10次/1s

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

**限速规则**：10次/1s

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

## 查询币币交易账户

**限速规则**：10次/1s

**示例**

* GET `baseUrl/v1/api/account/bb/query`

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
                "lock":"0",          //锁定的额度
                "available":"9",     //余额
                "asset":"BTC",       //资产
                "total":"9"          //总额
            }
        ]
    },
    "time":1573639479143
}
```


## 账户划转

**限速规则**：1次/1s

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

账户类型 1.资金账户,2.合约账户,3.币币账户


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

# http 永续合约

## 查询交易的资产区

**限速规则**：5次/1s

**示例**

* GET `baseUrl/v1/api/pc/asset/query`


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

## 查询永续合约的交易对

**限速规则**：5次/1s

**示例**

* GET `baseUrl/v1/api/pc/contract/query`


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

**限速规则**：20次/1s

**示例**

* GET `baseUrl/v1/api/pc/instrument/query`


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
        "price_precision": "1",           //交易对价格小数位数
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

**限速规则**：2次/1s

**示例**

* GET `baseUrl/v1/api/pc/candle/query`


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

**限速规则**：10次/1s

**示例**

* GET `baseUrl/v1/api/pc/order/book/query`


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


## 查询用户永续合约的设置

**限速规则**：2次/1s

**示例**

* GET `baseUrl/v1/api/pc/setting/query`

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



```shell
# Response
{
    "code":0,
    "data":{
        "time":"1573720738789",
        "asset":"BTC",              //资产    
        "symbol":"BTC_USD",         //交易对
        "cross_max_leverage": "100",      //满仓最大杠杆
        "bid_max_leverage":"100",     //逐仓做多最大杠杆
        "ask_max_leverage": "66.66",  //逐仓做空最大杠杆
        "bid_leverage": "5",          //逐仓做多杠杆
        "ask_leverage": "5",          //逐仓做空杠杆
        "cross_leverage": "5",        //满仓杠杆
        "margin_mode":"2"           //1.全仓,2.逐仓
    },
    "input":null,
    "traceId":"",
    "cost":0,
    "error":null,
    "msg":null,
    "time":1573720738789
}
```



## 创建活动委托

**限速规则**：20次/1s

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

**限速规则**：20次/1s

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

## 批量撤销委托

**限速规则**：1次/10s

**示例**

* POST `baseUrl/v1/api/pc/order/cancel_batch`

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
filter | string | YES | json格式的字符串


**filter**

* json格式的字符串

* {"order_id":["20001","20002"]}
  
   order_id:数组,最大50个


```shell
# Response
{
  "code": 0,
  "data": {
    "time": "1568639111357",
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

**限速规则**：10次/1s

**示例**

* GET `baseUrl/v1/api/pc/order/query`

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
                "status":"4", //委托状态  1:待报 2:已报 4:待撤销 8:已撤销 16:部分成交 32:全部成交     
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

**限速规则**：10次/1s

**示例**

* GET `baseUrl/v1/api/pc/trade/query`


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

**限速规则**：5次/1s

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
leverage | string | YES | 调整杠杆数,小数点两位



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


## 追加仓位保证金

**限速规则**：5次/1s

**示例**

* POST `baseUrl/v1/api/pc/margin/increase`

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
long_flag | string | YES | 1.多仓,0.空仓
increase_flag | string | YES | 1.增加,0.减少
margin | string | YES | 保证金数量



```shell
# Response
{
    "code":0,
    "data":{
        "time":"1573715942669"
    },
    "input":null,
    "traceId":"",
    "cost":0,
    "error":null,
    "msg":null,
    "time":1573715942669
}
```


## 开关自动追加保证金

**限速规则**：2次/1s

**示例**

* POST `baseUrl/v1/api/pc/margin/auto`

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
long_flag | string | YES | 1.多仓,0.空仓
open_flag | string | YES | 1.开启,0.关闭



```shell
# Response
{
    "code":0,
    "data":{
        "time":"1573716528703"
    },
    "input":null,
    "traceId":"",
    "cost":0,
    "error":null,
    "msg":null,
    "time":1573716528705
}
```


# http 币币交易

## 查询交易的资产区

**限速规则**：5次/1s

**示例**

* GET `baseUrl/v1/api/bb/asset/query`


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
  "time": 1573640300898
}

```

