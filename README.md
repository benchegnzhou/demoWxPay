# Android微信支付集成步骤

## 一、 准备工作

在应用集成微信支付之前，我们在[微信开放平台](https://open.weixin.qq.com/cgi-bin/index?t=home/index&lang=zh_CN&token=79b90f1a82ebf8eecfd25e57c43729df42c5f9b5)必须要个开发者账户

#### 1.注册完之后创建一个移动应用，并获取APPid等可以参考： ####

[http://blog.csdn.net/vroymond/article/details/53422744](http://blog.csdn.net/vroymond/article/details/53422744)

#### 2.申请开通微信支付能力 ####

- 认证开发者资格

![](https://i.imgur.com/Gdm2BHI.png)


- 开通微信支付

![](http://file.service.qq.com/user-files/uploads/201403/b09ce2af0f9153cf0ec2690514ff5fe6.jpg)

#### 3.开通成功后，获取得到商户号并在[商户平台](https://pay.weixin.qq.com/index.php/core/home/login?return_url=%2Findex.php)配置API密钥（生成预支付订单号需要） ####

API密钥配置流程：[https://blog.csdn.net/bencheng06/article/details/81132847](https://blog.csdn.net/bencheng06/article/details/81132847)

#### 4.在项目中添加微信支付依赖 ####

* build文件中


	    //微信支付
	    compile 'com.tencent.mm.opensdk:wechat-sdk-android-with-mta:1.1.6'

![](https://i.imgur.com/ngc394v.png)


 
#### 5.在项目包名下创建一个wxapi的包，并创建一个WXPayEntryActivity的类(微信分享以及登录必须要求，该类继承activity并实现IWXAPIEventHandler接口，用于拿到支付的回调结果)，并在清单文件中注册。 ####


![](https://i.imgur.com/LjytVSp.png)

* 注： WXPayEntryActivity 直接拷贝demo就可 具体代码请移步  [https://blog.csdn.net/bencheng06/article/details/81133218](https://blog.csdn.net/bencheng06/article/details/81133218)

## 二、 调起微信支付

步骤：

#### 1.客户端（APP）提交订单信息给服务端，服务端根据微信接口：统一下单接口，生成预支付Id(prepay_id)返回给客户端。 ####
		
![](https://i.imgur.com/6rib2Rv.png)


#### 2.客户端（APP）根据预支付Id（prepay_id）调起微信支付 ####

![](https://i.imgur.com/Lg5KLr0.png)
![](https://i.imgur.com/eUw0PAp.png)


#### 3. 生成预支付Id（这步在服务端生成完成，切记）


* 根据**[统一下单接口](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=9_1)**文档的规则：

* 接口链接 URL地址：[https://api.mch.weixin.qq.com/pay/unifiedorder](https://api.mch.weixin.qq.com/pay/unifiedorder)




* 服务端需要必须提交的参数字段有以下这些:（POST格式为XML）

|		名称		|	字段		|		字段举例		|解释			|		预留		|
|---------------|-----------------|-----------------------|----------------|-----------------------|
| 应用ID			|		appid        		|  wxe154574854     | 	 微信开放平台审核通过的应用APPID	|
| 商户号			|		mch_id            	| 1530000109 	| 微信支付分配的商户号
| 随机字符串		|		nonce_str	  	 	| 5K8264ILTKCH16CQ2502SI8ZNMTM67VS	| [随机数生成算法](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=4_3)	|
| 商品描述		|		body			 	| 	腾讯充值中心-QQ会员充值|		用户付款界面显示，APP——需传入应用市场上的APP名字-实际商品名称，天天爱消除-游戏充值。		|
| 商户订单号		|		out_trade_no		 | 20150806125346	|   					|
| 总金额			|		total_fee			 | 888				|		以分为单位		|
| 终端IP			|		spbill_create_ip	 | 		123.12.12.123	|
| 通知地址	 	|       notify_url			 | 	http://www.weixin.qq.com/wxpay/pay.php	|  
| 交易类型		|		trade_type			|	APP |			支付类型			|
| 签名			|		sign				|	C380BEC2BFD727A4B6845133519F3AD6|	[签名生成算法（重要）](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=4_3)|
|API接口密码匙	|		API_KEY				|15488845448acb48884544a7488845448acb4a74a7ba|	开发者平台对应有多个APP的，这个值必须登录每个商户号分别设置，商户信息以注册应用成功回执的邮件为准	|

* 详情可看：[统一下单](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=9_1)
* 生成签名后请自行校验  [微信支付接口签名校验工具](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=20_1)

* 最终提交的样例

		<xml>
		   <appid>wx2421b1c4370ec43b</appid>
		   <attach>支付测试</attach>
		   <body>APP支付测试</body>
		   <mch_id>10000100</mch_id>
		   <nonce_str>1add1a30ac87aa2db72f57a2375d8fec</nonce_str>
		   <notify_url>http://wxpay.wxutil.com/pub_v2/pay/notify.v2.php</notify_url>
		   <out_trade_no>1415659990</out_trade_no>
		   <spbill_create_ip>14.23.150.211</spbill_create_ip>
		   <total_fee>1</total_fee>
		   <trade_type>APP</trade_type>
		   <sign>0CB01533B8C1EF103065174F50BCA001</sign>
		</xml>


sign签名生成：

1.把我们所需要提交的参数（除sign外），拼接成URL键值对的格式（即key1=value1&key2=value2…）
![](https://i.imgur.com/6XPn0ez.png)

2.得到拼接后的字符串之后拼接在商户平台生成 API密钥

![](https://i.imgur.com/6XPn0ez.png)

3.拼接完key之后，进行MD5运算，再将得到的字符串所有字符转换为大写，得到sign

![](https://i.imgur.com/D8n8mpg.png)	


#### 提交所有参数 调起统一下单接口 获取预支付Id

![](https://i.imgur.com/K3K4v1C.png)


#### APP客户端调起微信支付

根据微信提供的调起[微信支付](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=9_12&index=2)的规则，APP端需要提交的参数为：

![](https://i.imgur.com/daiaKkh.png)

* 1.sign签名生成

​	sign签名生成步骤跟上面叙述的是一样的（省略）。

* 2.生成完签名，拼接所有支付参数。（PayReq，IWXAPI是微信提供jar包里的类）

 

* 3.调起微信支付

 

* **（注意，运行的应用签名必须跟在微信开放平台的签名需要一致，为了方便调试可以让debug使用relase签名，配置步骤可参考：[http://www.cnblogs.com/niray/p/5242985.html）](http://www.cnblogs.com/niray/p/5242985.html）)**



至此，调起微信支付所有步骤完成



> **源码地址：[ https://github.com/benchegnzhou/demoWxPay]( https://github.com/benchegnzhou/demoWxPay)（此代码只能做参考，已把应用签名以及APPid等删除掉）**

效果图：


 
![](https://i.imgur.com/8Je713G.png)











