##	概要



支付接口文档

##	1.0 接口规则

###	协议规则

传输方式：采用HTTP传输(生产环境建议HTTPS)
提交方式：采用POST方式提交
字符编码：UTF-8
签名算法：MD5

###	参数规范

交易金额：默认为人民币交易，单位为分，参数值不能带小数。

###	安全规范

####	签名算法

**签名生成的通用步骤如下**

**第一步：**设所有发送或者接收到的数据为集合M，将集合M内非空参数值的参数按照参数名ASCII码从小到大排序（字典序），使用URL键值对的格式（即key1=value1&key2=value2…）拼接成字符串stringA。
特别注意以下重要规则：
◆ 参数名ASCII码从小到大排序（字典序）；
◆ 如果参数的值为空不参与签名；
◆ 参数名区分大小写；
◆ 验证调用返回或支付中心主动通知签名时，传送的sign参数不参与签名，将生成的签名与该sign值作校验。
◆ 支付中心接口可能增加字段，验证签名时必须支持增加的扩展字段

**第二步：**在stringA最后拼接上key得到stringSignTemp字符串，并对stringSignTemp进行MD5运算，再将得到的字符串所有字符转换为大写，得到sign值signValue。



如请求支付系统参数如下：

```java
    Map signMap = new HashMap<>();
    signMap.put("userId", "test01");
    signMap.put("type", "wechat");
    signMap.put("money", Double.valueOf(2));
    signMap.put("remark", "");
    signMap.put("outTradeNo", "P12312321123");
```



待签名值：money=2.0&outTradeNo=P12312321123&type=wechat&userId=test01&key=EWEFD123RGSRETYDFNGFGFGSHDFGH
签名结果：5E0AA05DD4BB4FE5AB65608123EBA591
最终请求支付系统参数：money=2.0&outTradeNo=P12312321123&type=wechat&userId=test01&sign=5E0AA05DD4BB4FE5AB65608123EBA591

**商户登录商户系统后，通过安全中心查看或修改私钥key



##	1.1 统一下单

###	接口描述

业务通过统一下单接口可以发起任意三方支付渠道的支付订单。业务系统不必关心该如何调用三方支付，统一下单接口会根据业务系统选择的支付渠道ID，选择对应支付渠道的支付产品，发起下单请求，然后响应给业务系统支付请求所需参数。

###	接口链接

URL地址：https://pay.xxx.com/api/pay/create_order

###	**请求参数**

| 字段名              | 变量名     | 必填 | 类型        | 示例值                           | 描述                     |
| ------------------- | ---------- | ---- | ----------- | -------------------------------- | ------------------------ |
| 商户ID              | mchId      | 是   | long        | 20001222                         | 分配的商户号             |
| 应用ID              | appId      | 是   | String(32)  | 0ae8be35ff634e2abe94f5f32f6d5c4f | 该商户创建的应用对应的ID |
| 支付产品ID          | productId  | 是   | int         | 8000，详见文档底部编码表         | 支付产品ID               |
| 商户订单号          | mchOrderNo | 是   | String(30)  | 20160427210604000490             | 商户生成的订单号         |
| 币种                | currency   | 是   | String(3)   | cny                              | 三位货币代码,人民币:cny  |
| 支付金额            | amount     | 是   | int         | 100                              | 支付金额,单位分          |
| 客户端IP            | clientIp   | 否   | String(32)  | 210.73.10.148                    | 客户端IP地址             |
| 设备                | device     | 否   | String(64)  | ios10.3.1                        | 客户端设备               |
| 支付结果前端跳转URL | returnUrl  | 否   | String(128) | http://shop.pay.org/return.htm   | 支付结果回调URL          |
| 支付结果后台回调URL | notifyUrl  | 是   | String(128) | http://shop.pay.org/notify.htm   | 支付结果回调URL          |
| 商品主题            | subject    | 是   | String(64)  | pay测试商品1                     | 商品主题                 |
| 商品描述信息        | body       | 是   | String(256) | pay测试商品描述                  | 商品描述信息             |
| 签名                | sign       | 是   | String(32)  | C380BEC2BFD727A4B6845133519F3AD6 | 签名值，详见签名算法     |

###	返回结果

| 字段名     | 变量名  | 必填 | 类型        | 示例值   | 描述                                                   |
| ---------- | ------- | ---- | ----------- | -------- | ------------------------------------------------------ |
| 返回状态码 | retCode | 是   | String(16)  | SUCCESS  | SUCCESS/FAIL此字段标识是否成功                         |
| 返回信息   | retMsg  | 否   | String(128) | 签名失败 | 返回信息，如非空，为错误原因 签名失败 参数格式校验错误 |

**以下字段在retCode为SUCCESS的时候有返回**

| 字段名     | 变量名     | 必填 | 类型       | 示例值                           | 描述                                                   |
| ---------- | ---------- | ---- | ---------- | -------------------------------- | ------------------------------------------------------ |
| 签名       | sign       | 是   | String(32) | C380BEC2BFD727A4B6845133519F3AD6 | 签名值，详见签名算法                                   |
| 支付订单号 | payOrderId | 是   | String(32) | 20160427210604000490             | 支付中心生成的订单号                                   |
| 支付参数   | payParams  | 是   | JSONObject |                                  | 该字段返回JSON格式数据，多指调起第三方支付所需传递参数 |

**其中，payParams包含如下字段**

| 字段名   | 变量名    | 必填 | 类型   | 示例值                                         | 描述                          |
| -------- | --------- | ---- | ------ | ---------------------------------------------- | ----------------------------- |
| 支付URL  | payUrl    | 是   | String | http://www.abcpay.com?Aid=P2020030175464974641 | 跳转支付wap支付所需的表单信息 |
| 支付方法 | payMethod | 是   | String | formJump                                       | formJump=表单跳转             |

**Tip:当请求参数productId（支付宝类型），返回的JSON格式数据如下：**

```json
{
"payOrderId": "P0020180114172136000000",
"sign": "A1F39B0D7BD15E7A7BB6B99762302C51",
"payParams": {
    "payUrl": "<form name=\"punchout_form\" method=\"post\" action=\"https://openapi.alipay.com/gateway.do?charset=UTF-		  8&method=alipay.trade.wap.pay&sign=DxPbQZwi2Zpnv%2BqC8UdfLgSGMB%2F%2FaspU34ZkRVdi7kzUlAQsDiNoT0MIsyby2c%2FahNXjOx69IrPqhha6oIDUBGFodK6nZ3izfDkagbKyvoSnRVaoQ%2FXKeQowiCmVMiln9xN1SpiWdXE7dsQkCzCAzUD5yeAAbQbn38MkJ2TF6dFXql%2BT0yATgkSzYSuQxlUJVnpKCpDKYeL0NHaf58EbT63pkdOyfODWz%2BD1eUwbspzzul1kY7AYD2ZSHQKyW4zxQS0YzrpUKPhF2olGtgVZo2EEcqQuxIgC4hz1TTqVjD2VK5kj45BJ%2B0xd1DsojLXjxaR5qziFYKqSGU8OiN0yhg%3D%3D&return_url=http%3A%2F%2Fwww.xxpay.org&notify_url=http%3A%2F%2Fpay.t.xxpay.org%2Fnotify%2Fpay%2FaliPayNotifyRes.htm&version=1.0&app_id=2015081500216362&sign_type=RSA2&timestamp=2018-01-14+17%3A21%3A37&alipay_sdk=alipay-sdk-java-dynamicVersionNo&format=json\">\n<input type=\"hidden\" name=\"biz_content\" value=\"{&quot;body&quot;:&quot;XXPAY支付测试&quot;,&quot;out_trade_no&quot;:&quot;P0020180114172136000000&quot;,&quot;product_code&quot;:&quot;QUICK_WAP_PAY&quot;,&quot;subject&quot;:&quot;XXPAY支付测试&quot;,&quot;total_amount&quot;:&quot;0.01&quot;}\">\n<input type=\"submit\" value=\"立即支付\" style=\"display:none\" >\n</form>\n<script>document.forms[0].submit();</script>"
},
"retCode": "SUCCESS"
}
```





##	1.2查询支付订单

###	接口描述

业务系统通过查询支付订单接口获取最新的支付订单状态，并根据状态结果进一步处理业务逻辑。

###	**接口链接**

URL地址：[ https://pay.xxx.com/api/pay/query_order]( https://pay.xxx.com/api/pay/query_order)

| 字段名       | 变量名        | 必填 | 类型       | 示例值                           | 描述                                                         |
| ------------ | ------------- | ---- | ---------- | -------------------------------- | ------------------------------------------------------------ |
| 商户ID       | mchId         | 是   | String(30) | 1000000010                       | 支付中心分配的商户号                                         |
| 应用ID       | appId         | 是   | String(32) | 0ae8be35ff634e2abe94f5f32f6d5c4f | 该商户创建的应用对应的ID                                     |
| 支付订单号   | payOrderId    | 是   | String(30) | P20160427210604000490            | 支付中心生成的订单号，与mchOrderNo二者传一即可               |
| 商户订单号   | mchOrderNo    | 是   | String(30) | 20160427210604000490             | 商户生成的订单号，与payOrderId二者传一即可                   |
| 是否执行回调 | executeNotify | 否   | Boolean    | true                             | 是否执行回调，如果为true，则支付中心会再次向商户发起一次回调，如果为false则不会发起 |
| 签名         | sign          | 是   | String(32) | C380BEC2BFD727A4B6845133519F3AD6 | 签名值，详见签名算法                                         |

###	返回结果

###	返回结果

| 字段名     | 变量名  | 必填 | 类型        | 示例值   | 描述                                                   |
| ---------- | ------- | ---- | ----------- | -------- | ------------------------------------------------------ |
| 返回状态码 | retCode | 是   | String(16)  | SUCCESS  | SUCCESS/FAIL此字段标识是否成功                         |
| 返回信息   | retMsg  | 否   | String(128) | 签名失败 | 返回信息，如非空，为错误原因 签名失败 参数格式校验错误 |

**以下字段在retCode为SUCCESS的时候有返回**

| 字段名       | 变量名         | 必填 | 类型       | 示例值                                                       | 描述                                                   |
| ------------ | -------------- | ---- | ---------- | ------------------------------------------------------------ | ------------------------------------------------------ |
| 商户ID       | mchId          | 是   | long(30)   | 20001222                                                     | 支付中心分配的商户号                                   |
| 应用ID       | appId          | 是   | String(32) | 0ae8be35ff634e2abe94f5f32f6d5c4f                             | 该商户创建的应用对应的ID                               |
| 支付产品ID   | productId      | 是   | int        | 8001                                                         | 支付产品ID                                             |
| 支付订单号   | payOrderId     | 是   | String(30) | P20160427210604000490                                        | 支付中心生成的订单号                                   |
| 商户订单号   | mchOrderNo     | 是   | String(30) | 20160427210604000490                                         | 商户生成的订单号                                       |
| 支付金额     | amount         | 是   | int        | 100                                                          | 支付金额,单位分                                        |
| 币种         | currency       | 是   | String(3)  | cny                                                          | 三位货币代码,人民币:cny                                |
| 状态         | status         | 是   | int        | 1                                                            | 支付状态,0-订单生成,1-支付中,2-支付成功,3-业务处理完成 |
| 渠道用户ID   | channelUser    | 否   | String(64) | [justhappy@126.com](mailto:justhappy@126.com)                | 渠道测支付时使用的用户ID                               |
| 渠道订单号   | channelOrderNo | 否   | String     | wx20170910211043fb206e92260071822007                         | 对应的第三方支付订单号                                 |
| 渠道数据包   | channelAttach  | 否   | String     | {“bank_type”:”CMB_DEBIT”,”trade_type”:”pay.weixin.micropay”} | 支付渠道数据包                                         |
| 支付成功时间 | paySuccTime    | 否   | Long       | 1505049094262                                                |                                                        |

##	1.3支付结果通知

###	接口链接

该链接是通过统一下单接口提交的参数notifyUrl设置，如果无法访问链接，业务系统将无法接收到支付中心的通知。

| 字段名       | 变量名         | 必填 | 类型       | 示例值                                                       | 描述                                                   |
| ------------ | -------------- | ---- | ---------- | ------------------------------------------------------------ | ------------------------------------------------------ |
| 商户入账     | income         | 是   | int        | 100                                                          | 商户实际入账金额，单位：分                             |
| 支付订单号   | payOrderId     | 是   | String(30) | P20160427210604000490                                        | 支付中心生成的订单号                                   |
| 商户ID       | mchId          | 是   | String(30) | 20001222                                                     | 支付中心分配的商户号                                   |
| 应用ID       | appId          | 是   | String(32) | 0ae8be35ff634e2abe94f5f32f6d5c4f                             | 该商户创建的应用对应的ID                               |
| 支付产品ID   | productId      | 是   | int        | 8001                                                         | 支付产品ID                                             |
| 商户订单号   | mchOrderNo     | 是   | String(30) | 20160427210604000490                                         | 商户生成的订单号                                       |
| 支付金额     | amount         | 是   | int        | 100                                                          | 支付金额,单位分                                        |
| 状态         | status         | 是   | int        | 1                                                            | 支付状态,0-订单生成,1-支付中,2-支付成功,3-业务处理完成 |
| 渠道订单号   | channelOrderNo | 否   | String(64) | wx2016081611532915ae15beab0167893571                         | 三方支付渠道订单号                                     |
| 渠道数据包   | channelAttach  | 否   | String     | {“bank_type”:”CMB_DEBIT”,”trade_type”:”pay.weixin.micropay”} | 支付渠道数据包                                         |
| 扩展参数1    | param1         | 否   | String(64) |                                                              | 支付中心回调时会原样返回                               |
| 扩展参数2    | param2         | 否   | String(64) |                                                              | 支付中心回调时会原样返回                               |
| 支付成功时间 | paySuccTime    | 是   | long       |                                                              | 精确到毫秒                                             |
| 通知类型     | backType       | 是   | int        | 1                                                            | 通知类型，1-前台通知，2-后台通知                       |
| 签名         | sign           | 是   | String(32) | C380BEC2BFD727A4B6845133519F3AD6                             | 签名值，详见签名算法                                   |

###	返回结果

业务系统处理后同步返回给支付中心，返回字符串 success（小写） 则表示成功，返回非success则表示处理失败，支付中心会再次通知业务系统。（通知频率为60/120/180/240/300,单位：秒）



##	2.0 编码表

请联系相关人员获取，以下是举例

| 产品编号 | 产品名称         |
| -------- | ---------------- |
| 8030     | 支付宝H5支付     |
| 8031     | 支付宝扫码支付   |
| 8032     | 支付宝H5转账支付 |



##	3.0 错误码

| 错误码 | 描述     | 原因           | 解决方案                       |
| ------ | -------- | -------------- | ------------------------------ |
| 0010   | 系统错误 | 系统超时或异常 | 系统异常，请用相同参数重新调用 |

