---
layout: post
title: SpringMVC 重定向
category: Java
tags: SpringMVC
keywords:
description:
---

## 简介
Web应用中，当Server处理完POST请求后，一般都会重定向到另外的页面，这样可以防止用户刷新浏览器导致重新执行POST请求，带来业务冲突。

## SpringMVC 重定向方式
SpringMVC 通过'redirect:&lt;url&gt;'的方式，将请求重定向到&lt;url&gt;中，现实情况下，我们通常需要将模型数据发给重定向的url中，一种较为简单的方法是在url后面链接查询参数（k=v），这种方式对一些模型对象的传递较为困难。
SpringMVC支持两种传递数据的方式：

1. 使用URL路径变量或查询参数传递数据
2. 通过flash属性传递数据。

## 实例

先上代码

	@Controller
	@RequestMapping("/r")
	public class RedirectDemoController {
		@RequestMapping("/data")
		public String redictWithData(RedirectAttributes attributes) {
			attributes.addAttribute("code", 1001);
			attributes.addFlashAttribute("msg", "认证失败");
			return "redirect:/r/info";
		}

		@RequestMapping("/info")
		public ModelAndView info(@RequestParam("code") int code) {
			ModelAndView mv = new ModelAndView("/redirect/info");
			mv.addObject("code", code);

			return mv;
		}
	}

发送http://localhost/r/data POST请求，将会重定向到http://localhost/r/info 路径，其中重定向后的url为http://localhost/r/info?code=1001，并携带模型数据msg。
RedirectAttributes#addAttribute()将参数以查询参数形式传递到重定向url中。

通过路径变量和查询参数的形式跨重定向传递数据简单直接，但它只能发送简单的数据，如字符串。

通过请求传递过来的数据，重定向后将会被丢弃，SpringMVC通过RedirectAttributes#addFlashAttribute()方法将模型数据先存入会话中，这些数据到下一次请求中仍然有效，然后再将这些flash模型数据清理。通过flash的方式可以方式复杂的数据对象。


另外，如果我们将用户输入的字符串构建URL或用于SQL查询时，可能导致安全问题，SpringMVC可通过模板的方式对所有不安全的数据进行转义。

	@Controller
	@RequestMapping("/r2")
	public class RedirectDemoController {

		@RequestMapping("/data2")
		public String redictWithData2(RedirectAttributes model) {
			model.addAttribute("code", 1001);
			model.addFlashAttribute("msg", "认证失败");
			return "redirect:/r/info2/{code}";
		}

		@RequestMapping("/info2/{code}")
		public ModelAndView info2(@PathVariable("code") int code) {
			ModelAndView mv = new ModelAndView("/redirect/info");
			mv.addObject("code", code);

			return mv;
		}
	}