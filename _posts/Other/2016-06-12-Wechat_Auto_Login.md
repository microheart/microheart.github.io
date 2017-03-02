---
layout: post
title: 用户自动登陆微信公众号
category: Other
tags: 微信
keywords:
description:
---

## 简介
微信公众平台开发中，用户关注某个公众号以后，下次通过自定义菜单，进入公众号能够实现自动登陆。

每个用户在不同的公众号中对应一个唯一的openid，可以理解`openid = hash(weixinid, appid)`，自动登录的原理就是能够进入公众号能够自动获取它的openid。

xml交互的消息中虽然包含用户的`openid`，但无法将其保存在session中。

自动登录方式：在公众平台入口处通过静默授权获取用户的openid。

## 策略

1. 第一步：用户同意授权，获取code
2. 第二步：通过code换取网页授权access_token

在确保微信公众账号拥有授权作用域（scope参数）的权限的前提下（服务号获得高级接口后，默认拥有scope参数中的snsapi_base和snsapi_userinfo），引导关注者打开如下页面：

    https://open.weixin.qq.com/connect/oauth2/authorize?appid=APPID&redirect_uri=REDIRECT_URI&response_type=code&scope=SCOPE&state=STATE#wechat_redirect

其中scope=snsapi_base，以snsapi_base为scope发起的网页授权，是用来获取进入页面的用户的openid的，并且是静默授权并自动跳转到回调页的。用户感知的就是直接进入了回调页（往往是业务页面）。
第一步scope=snsapi_base，用户不需要点击同意，即可跳转。
在微信公众号请求用户网页授权之前，开发者需要先到公众平台官网中的开发者中心页配置授权回调域名。请注意，这里填写的是域名（是一个字符串），而不是URL，因此请勿加 http:// 等协议头；

然后在回调url中处理code参数，请求以下链接获取access_token，从而获得`openid`。

    https://api.weixin.qq.com/sns/oauth2/access_token?appid=APPID&secret=SECRET&code=CODE&grant_type=authorization_code

所以我们只需要在用户进入微信公众平台的入口处(通常为自定义菜单栏中的网址)，按照以下url的形式编码，然后通过回调url获得openid。

    https://open.weixin.qq.com/connect/oauth2/authorize?appid=APPID&redirect_uri=REDIRECT_URI&response_type=code&scope=snsapi_base&state=STATE#wechat_redirect


