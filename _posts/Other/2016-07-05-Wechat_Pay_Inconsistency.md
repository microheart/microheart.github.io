---
layout: post
title: 微信支付API的不一致性
category: Other
tags: 微信, 微信支付
keywords:
description:
---

## 简介
在集成微信支付中，遇到微信支付API的不一致性，一个很小的差别，导致调试起来难以发现。

一方面是自己粗心，没严格按照文档的数据格式发送请求；另外一方面是微信支付API不一致，并且对错误的数据没有任何提示。


## 过程

**微信支付业务流程**

![微信支付业务流程](/public/upload/other/weixin_pay.png)

问题存在第5-7步中。

为了获取prepayid，需要发送相应的xml数据到微信统一支付URL中。例如：

    <xml>
      <nonce_str>1487599986</nonce_str>
      <out_trade_no>1487599986909</out_trade_no>
      <openid>oW0jwjq6bz5lkOuc-FrLoEgzqFi0</openid>
      <appid>wx433752f4f2bb8af2</appid>
      <total_fee>1</total_fee>
      <sign>6BCA5610976DC2CD00E163C4AEB7078E</sign>
      <trade_type>JSAPI</trade_type>
      <mch_id>1357902202</mch_id>
      <body>good</body>
      <notify_url>http://www.example.com/goosee/pay/pay_notify</notify_url>
      <spbill_create_ip>221.178.200.20</spbill_create_ip>
    </xml>

若一不小心将appid写成appId，那么你将无法获得任何数据，微信不会响应任何结果，不会说xml格式错误等，让人疑惑的是，在微信提供的签名校验响应的xml是能够通过校验的。

好，以下是成功获取prepareid

    <xml>
      <return_code><![CDATA[SUCCESS]]></return_code>
      <return_msg><![CDATA[OK]]></return_msg>
      <appid><![CDATA[wx433752f4f2bb8af2]]></appid>
      <mch_id><![CDATA[1357902202]]></mch_id>
      <nonce_str><![CDATA[MPYqWjtpXZSEZztT]]></nonce_str>
      <sign><![CDATA[1296B5AF53F95C0A434DDA86DF76AC05]]></sign>
      <result_code><![CDATA[SUCCESS]]></result_code>
      <prepay_id><![CDATA[wx20170220221307c4effd6a1c0891519009]]></prepay_id>
      <trade_type><![CDATA[JSAPI]]></trade_type>
    </xml>

接下来需要在H5页面中提供响应的参数，在后端传递给前端的Model数据中JS中字段为`appId`，而不是去获取prepay_id的`appid`。

    function onBridgeReady(){
       WeixinJSBridge.invoke(
           'getBrandWCPayRequest', {
               "appId" ： "wx2421b1c4370ec43b",     //公众号名称，由商户传入
               "timeStamp"：" 1395712654",         //时间戳，自1970年以来的秒数
               "nonceStr" ： "e61463f8efa94090b1f366cccfbbb444", //随机串
               "package" ： "prepay_id=wx20170220221307c4effd6a1c0891519009",
               "signType" ： "MD5",         //微信签名方式：
               "paySign" ： "70EA570631E4BB79628FBCA90534C63FF7FADD89" //微信签名
           },
           function(res){
               if(res.err_msg == "get_brand_wcpay_request：ok" ) {}     // 使用以上方式判断前端返回,微信团队郑重提示：res.err_msg将在用户支付成功后返回    ok，但并不保证它绝对可靠。
           }
       );
    }
    if (typeof WeixinJSBridge == "undefined"){
       if( document.addEventListener ){
           document.addEventListener('WeixinJSBridgeReady', onBridgeReady, false);
       }else if (document.attachEvent){
           document.attachEvent('WeixinJSBridgeReady', onBridgeReady);
           document.attachEvent('onWeixinJSBridgeReady', onBridgeReady);
       }
    }else{
       onBridgeReady();
    }

> https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=9_1
	
总结：阅读文档细心，自己做设计也需要考虑全面，API友好，给出较为全面的错误码和错误描述。