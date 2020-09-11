### 彼岸支付是什么？

彼岸支付是一个面向 “**个人开发者**” 量身定制的在线支付能力的 API，拥有企业支付宝的请绕道。

### 使用条件

- 拥有 AppId 和 Key（邮件申请：sanqii@icloud.com）;
- 没有企业支付宝；

- 依赖在线支付回调能力以实现后续的业务逻辑；

- **遵纪守法**！！


### 支持的机构和类型

支付宝支付：**花呗支付、花呗分期，信用卡支付**等任意类型。

### 支付资金安全性

支付回调转账人工无法操作，全权 API 自动实现，响应并完成转账时间 **1s** 内。

### 支付宝支付原理

<p align="center">
  <a href="https://ant.design">
    <img width="200" src="https://gw.alipayobjects.com/os/skylark-tools/public/files/0ba3e82ad37ecf8649ee4219cfe9d16b.png%26originHeight%3D2023%26originWidth%3D2815%26size%3D526149%26status%3Ddone%26width%3D2815">
  </a>
</p>

### 彼岸花支付原理
----
**【请求支付】**

**你方用户调用你的接口 ——> 你的接口调用彼岸花接口 ——> 彼岸花调用支付宝API，返回订单id和支付宝url ——> 你方跳转到该 url ——> 你方用户扫码完成支付**

**【支付回调】**

**你方用户完成扫码支付 ——> 支付宝回调彼岸花 API ——> 彼岸花 API 验签并查询订单是否支付成功
**

（若订单支付不成功，则表示没有交易发生，没有回调）

若订单支付成功：

**支付成功 ——> 支付宝回调彼岸花 API ——> 彼岸花 API POST 携带参数请求你方回调 url，并向你方预留支付宝账户完成转账——> 你方验签（校验参数 key 是否正确）判断是否为彼岸花 API 请求 ——> 处理你的业务逻辑，并向该请求响应 object：`{code:"success"}` 或者 `{code:"failed"} `——> 至此一笔订单交易完成**

注意：
若你方回调地址响应失败或者返回 `{code:"failed"} `，彼岸花API会向支付宝返回 `failed`，支付宝会在一段时间内重复调用彼岸花API，也就意味着彼岸花API会不断到调用你方API直到你方响应` {code:"success"} ` 为止。

> 该回调过程不超过 1s，即你方客户支付的资金流转到彼岸花支付宝中停留时间不会超过 1s。


###请求支付 API 文档
----
**请求 url**

`http://api.sanqi.us/pay/TradePagePay`

**请求方法**

`Post`

**请求参数**

| 参数 |是否必填| 类型 | 描述 | 示例 |
| --------- | -----:|
| appId  |是|string |彼岸花分配的 APPID|d9394b7be88f4a55bfc8770aa8fa0f7d|
| key   |是|   string |彼岸花分配的 key|dade4b7be88f4a55bfc8770aa8fa0f7d|
| data  |是| object |请求业务的具体参数|-|
|└ subject|是|string|订单标题|Iphone6 16G|
|└ totalAmout|是|string|订单金额|9.99|
|└ itemBody|是|string|商品描述信息|特价手机|
|└ returnUrl|是|string|HTTP/HTTPS开头字符串|https://m.alipay.com/Gk8NF23|
|└ otherInfo|是|string|公用回传参数，彼岸花会在异步通知时将该参数原样返回。|merchantBizType%3d3C%26merchantBizNo%3d2016010101111|
|└ qrPayMode|否|string|PC扫码支付的方式，支持前置模式和 跳转模式。 前置模式是将二维码前置到商户的订单确认页的模式。需要商户在自己的页面中以 iframe 方式请求支付宝页面。具体分为以下几种： 0：订单码-简约前置模式，对应 iframe 宽度不能小于600px，高度不能小于300px； 1：订单码-前置模式，对应iframe 宽度不能小于 300px，高度不能小于600px； 3：订单码-迷你前置模式，对应 iframe 宽度不能小于 75px，高度不能小于75px； 4：订单码-可定义宽度的嵌入式二维码，商户可根据需要设定二维码的大小。 跳转模式下，用户的扫码界面是由支付宝生成的，不在商户的域名下。 2：订单码-跳转模式|2|
|└ qrcodeWidth|是|string|自定义二维码宽度，该参数仅在qrPayMode为4时生效|200|

**响应参数**

| 参数 |是否必填| 类型 | 描述 | 示例 |
| --------- | -----:|
|code |是|string|状态码|10000|
|sub_code |否|string|错误原因|isv.insufficient-user-permissions|
|data|否|object|响应参数|-|
|└url|是|string |彼岸花返回的支付url|-|
|└tradeNo|是|string |彼岸花生成的订单id|8b90ed260ea4438789e1e3e0e203f1ff|


###支付成功回调 API 文档
----
**响应参数**

| 参数 |是否必填| 类型 | 描述 | 示例 |
| --------- | -----:|
|orderId|是|string|彼岸花生成的订单id|8b90ed260ea4438789e1e3e0e203f1ff|
|key|是|string |彼岸花下发的key，用于校验是否为彼岸花发出的请求|8b90ed260ea4438789e1e3e0e203f1ff|
|status|是|string |订单状态|TRADE_SUCCESS|
|subject|是|string|订单标题|iphone 6|
|body|是|string |订单描述|低价手机|
|otherInfo|是|string|公用回传参数，彼岸花会在异步通知时将该参数原样返回。|-|
|alipayOrderId|是|string|支付宝交易号|-|
|datetime|是|string|处理时间|2020-10-01 17:20:02|
|amount|是|string|订单金额|9.99|
