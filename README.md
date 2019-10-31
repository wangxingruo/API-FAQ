# API-FAQ

## 接入、验签相关

1. 经常断线或者丢数据
> 请确认是否使用 api.huobi.pro 域名访问火币 API
> 请使用日本云服务器

2. 签名认证
https://huobiapi.github.io/docs/spot/v1/cn/#c64cd15fdc
请对比使用Secret Key签名前的字符串与以下字符串的区别
```
GET\n
api.huobi.vn\n
/v1/account/accounts\n
AccessKeyId=rfhxxxxx-950000847-boooooo3-432c0&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2019-10-28T07%3A28%3A38
```
对比时请注意一下几点：
> 1) 签名参数按照ASCII码排序
> 2) 签名串需进行URI编码
> 3) 签名需进行 base64 编码
> 4) Get请求参数需在签名串中
> 5) 时间为UTC时间转换为YYYY-MM-DDTHH:mm:ss
> 6) 检查本机时间与标准时间是否存在偏差（偏差应小于1分钟）
> 7) WebSocket发送验签认证消息时，消息体不需要URI编码
> 8) 签名时所带Host应与请求接口时Host相同
> 9) 若以上检查均正确时，请检查Api Key 与 Secret Key中是否存在隐藏特殊字符，影响签名

3. gateway-internal-error报错
> 1) 请检查account-id是否正确填写
> 2) 请检查是否为网络原因，若因网络原因请稍后重试
> 3) 检查发送数据格式是否正确(标准JSON格式)。
> 4) 检查POST请求头header是否声明为`Content-Type:application/json`

4. login-required
> 检查是否将AccessKeyId参数带入URL中
> 检查参数 account-id 是否是由 GET /v1/account/accounts 接口返回的，而不是填的 UID
> 检查是否 POST 请求是否把业务参数也计算进签名
> 检查 GET 请求是否将参数按照 ASCII 码表顺序排序

## 行情相关
1. 当前盘口数据最快多久更新一次？

> 当前盘口数据数据源这边是一秒更新一次，无论是RESTful查询或是Websocket推送都是一秒一次

2. 24小时行情接口数据接口的成交量会出现负增长嘛？
> /market/detail接口中的成交量、成交金额为24小时滚动数据（平移窗口大小24小时），可能会出现后一个窗口内的累计成交量、累计成交额小于前一窗口的情况

3. K线常见信息获取：
> 涨跌幅：(close价格- 0点的开盘价)/0点的开盘价  
> 开盘价：通过GET /market/history/kline接口读取日线数据，就能知道0点开盘价  
> 最新价：GET /market/detail/merged的close为最新价  
> 买卖盘：GET /market/depth  
> 买一卖一：GET /market/detail/merged的bid和ask  
> 历史K线获取：GET /market/history/kline，返回最大2000条数据  
> 历史K线获取：WebSocket通过market.$symbol$.kline.$period$可获取[1501174800, 2556115200]时间范围内的k线数据  


## 交易相关
1. account-id是什么？
> account-id对应用户不同业务账户的ID，可通过/v1/account/accounts接口获取，并根据account-type区分具体账户。

2. 下单数量、金额、小数限制、精度信息
> 可使用/v1/common/symbols获取相关币对信息， 下单时注意最小下单数量和最小下单金额的区别  
> order-value-min-error: 下单金额小于最小交易额    
> order-orderprice-precision-error : 限价单价格精度错误  
> order-orderamount-precision-error : 下单数量精度错误  
> order-limitorder-price-max-error : 限价单价格高于限价阈值  
> order-limitorder-price-min-error : 限价单价格低于限价阈值  
> order-limitorder-amount-max-error : 限价单数量高于限价阈值  
> order-limitorder-amount-min-error : 限价单数量低于限价阈值  

3. 下单client-order-id
> client-order-id作为下单请求标识的一个参数，类型为字符串，长度为64。 此id为用户自己生成，24小时内有效。

4. WebSocket 订单更新推送主题order.$symbol 和 order.$symbo.update的区别
> order.$symbol 主题作为老的推送主题，会在一段时间后停止主题的维护和使用， 建议使用order.$symbol.update主题，该主题为最新推送，具有更快的推送时效以及更低的时延，并且不会遗漏订单更新.

5. 收到订单推送的消息后进行下单，返回余额不足
> 为保证订单的及时送达以及低延时， 订单推送的结果是在撮合后直接推送，此时订单并未完成资产的结算，建议结合资产推送主题``accounts``同步接收资产变更的消息，确保资金已经完成结算，或保证账户中留有相对充裕的金额。


## 账户充提相关
1. 创建提币时返回api-not-support-temp-addr错误
> API创建提币时仅支持已在提现地址列表中的地址，暂不支持API添加地址至提笔地址管理中，需在网页端或APP端添加地址后才可在API中进行提币操作。
2. USDT提币时返回Invaild-Address错误
> USDT币种为典型的一币多链币种， 创建提币订单时应填写chain参数对应地址类型，若不填写该字段时，使用该币种的默认链作为chain参数，如果提现时未传chain字段（当前默认chain为usdterc20），提现地址输入OMNI地址，则会返回该错误，chain参数可使用值请参考/v2/reference/currencies接口。
3. 创建提币时fee字段应该怎么填？
> 请参考/v2/reference/currencies接口返回值。
 
FAQ for common errors：[wiki](https://github.com/huobiapi/API-FAQ/wiki)
