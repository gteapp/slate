---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - ruby
  - python
  - javascript

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

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
2019.10.24 18：00 | -GTE 委托 | 新增 | -新增委托接口

# 接入说明
## 接入URLs
## 限频规则
## 签名认证
## 请求格式
## 返回格式
## 错误信息
## SDK与代码示例
## 常见问题 Q&A

# 基础信息
# 合约交易
## 下单

**HTTP请求**

* baseUrl/v1/api/pc/order/create

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
## 深度
## 行情
