# 错误代码

<aside class="notice">
Errors consist of two parts: an error code and a message.
Codes are universal,but messages can vary.
</aside>

The GTE API uses the following error codes:

```sell
{
  "code": -10012,
  "error": {
    "patternName": "VALIDATE_PARAM_TYPE_ERROR",
    "data": {
      "paramName": "size"
    }
  }
}
```

错误码 | 描述
---------- | -------
-10011 | 参数不能为空异常
-10012 | 参数类型异常
-10013 | 参数值异常
-10051 | api请求签名错误
-10052 | api请求过期
-10053 | api请求太频繁
-10054 | apiKey不存在
-10055 | api请求权限错误

