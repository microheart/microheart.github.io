---
layout: post
title: Scrapy Demo
category: python
tags: 爬虫
keywords: 爬虫
description: 
---

## 概览

**demo项目步骤**

* 创建一个新的Scrapy项目
* 定义索要检索的Item对象
* 编写爬虫并检索Item
* 编写Pipeline存储检索的Item

## 创建新项目

	$scrapy startproject demo
	$tree /f
	Folder PATH listing
	Volume serial number is 24C1-0984
	D:.
	└─demo
	    │  scrapy.cfg
	    │
	    └─demo
	        │  items.py
	        │  pipelines.py
	        │  settings.py
	        │  __init__.py
	        │
	        └─spiders
	                __init__.py

### 文件简介
* scrapy.cfg: 项目配置文件
* demo/: 项目python模块
* demo/items.py: Item文件
* demo/pipelines.py: Pipeline文件
* demo/settings.py: 设置文件
* demo/spiders/: 爬虫目录

## 定义Item
爬虫的主要目标之一就是从无结构的数据中检索结构的数据。Item是一个简单的容器，包含将要爬取得数据，相当于简单的python字典。

我们将从`dmoz.org`的页面中获取标题，连接和描述信息。编辑`items.py`，内容如下：

	from scrapy.item import Item, Field

	class DmozItem(Item):
	    title = Field()
	    link = Field()
	    desc = Field()
	    pass

## 爬

爬虫是用户定义的类，用来从一个或者多个域中获取信息。用户自定义的爬虫需继承scrapy.spider.BaseSpider，有如下几个属性：

* name：唯一的爬虫名字
* start_urls：初始检索页面集
* parse()：downloaded Response对象将调用此方法，负责解析数据，包装成Item对象。

在spider目录下新建dmoz_spider.py，内容如下：

	from scrapy.spider import BaseSpider

	class DmozSpider(BaseSpider):
	    name = "dmoz_spider"
	    allowed_domains = ["dmoz.org"]
	    start_urls = [
	        "http://www.dmoz.org/Society/Philosophy/Aesthetics/",
			"http://www.dmoz.org/Sports/Football/"
	    ]

	    def parse(self, response):
	        filename = response.url.split("/")[-2]
	        open(filename, 'wb').write(response.body)

运行爬虫命令

	$scrapy crawl dmoz_spider
	2013-09-05 13:32:01+0800 [scrapy] INFO: Scrapy 0.18.2 started (bot: demo)
	2013-09-05 13:32:01+0800 [scrapy] DEBUG: Optional features available: ssl, http11
	2013-09-05 13:32:01+0800 [scrapy] DEBUG: Overridden settings: {'NEWSPIDER_MODULE ': 'demo.spiders', 'SPIDER_MODULES': ['demo.spiders'], 'BOT_NAME': 'demo'}
	2013-09-05 13:32:01+0800 [scrapy] DEBUG: Enabled extensions: LogStats, TelnetConsole, CloseSpider, WebService, CoreStats, SpiderState
	2013-09-05 13:32:01+0800 [scrapy] DEBUG: Enabled downloader middlewares: HttpAuthMiddleware, DownloadTimeoutMiddleware, UserAgentMiddleware, RetryMiddleware, DefaultHeadersMiddleware, MetaRefreshMiddleware, HttpCompressionMiddleware, RedirectMiddleware, CookiesMiddleware, ChunkedTransferMiddleware, DownloaderStats
	2013-09-05 13:32:01+0800 [scrapy] DEBUG: Enabled spider middlewares: HttpErrorMiddleware, OffsiteMiddleware, RefererMiddleware, UrlLengthMiddleware, DepthMiddleware
	2013-09-05 13:32:01+0800 [scrapy] DEBUG: Enabled item pipelines:
	2013-09-05 13:32:01+0800 [dmoz_spider] INFO: Spider opened
	2013-09-05 13:32:01+0800 [dmoz_spider] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
	2013-09-05 13:32:01+0800 [scrapy] DEBUG: Telnet console listening on 0.0.0.0:6023
	2013-09-05 13:32:01+0800 [scrapy] DEBUG: Web service listening on 0.0.0.0:6080
	2013-09-05 13:32:02+0800 [dmoz_spider] DEBUG: Crawled (200) <GET http://www.dmoz .org/Sports/Football/> (referer: None)
	2013-09-05 13:32:11+0800 [dmoz_spider] DEBUG: Crawled (200) <GET http://www.dmoz .org/Society/Philosophy/Aesthetics/> (referer: None)
	2013-09-05 13:32:11+0800 [dmoz_spider] INFO: Closing spider (finished)
	2013-09-05 13:32:11+0800 [dmoz_spider] INFO: Dumping Scrapy stats:
	        {'downloader/request_bytes': 482,
	         'downloader/request_count': 2,
	         'downloader/request_method_count/GET': 2,
	         'downloader/response_bytes': 14606,
	         'downloader/response_count': 2,
	         'downloader/response_status_count/200': 2,
	         'finish_reason': 'finished',
	         'finish_time': datetime.datetime(2013, 9, 5, 5, 32, 11, 220000),
	         'log_count/DEBUG': 8,
	         'log_count/INFO': 3,
	         'response_received_count': 2,
	         'scheduler/dequeued': 2,
	         'scheduler/dequeued/memory': 2,
	         'scheduler/enqueued': 2,
	         'scheduler/enqueued/memory': 2,
	         'start_time': datetime.datetime(2013, 9, 5, 5, 32, 1, 560000)}
	2013-09-05 13:32:11+0800 [dmoz_spider] INFO: Spider closed (finished)

顶层demo目录下多了两个文件Football/Aesthetics

## item检索

Scrapy采用XPath选择器检索数据，一中基于XPath表达式的方式，更多资料查看[xpath教程](http://www.w3school.com.cn/xpath/)

例如：
/html/head/title: 选择`<title>`元素
/html/head/title/text(): `<title>`文本元素
//td: 选择所有的td元素

	$scrapy shell http://www.dmoz.org/Computers/Programming/Languages/Python/Books/
	2013-09-05 13:56:01+0800 [scrapy] INFO: Scrapy 0.18.2 started (bot: demo)
	2013-09-05 13:56:01+0800 [scrapy] DEBUG: Optional features available: ssl, http11
	2013-09-05 13:56:01+0800 [scrapy] DEBUG: Overridden settings: {'NEWSPIDER_MODULE': 'demo.spiders', 'SPIDER_MODULES': ['demo.spiders'], 'LOGSTATS_INTERVAL': 0, 'BOT_NAME': 'demo'}
	2013-09-05 13:56:01+0800 [scrapy] DEBUG: Enabled extensions: TelnetConsole, CloseSpider, WebService, CoreStats, SpiderState
	2013-09-05 13:56:01+0800 [scrapy] DEBUG: Enabled downloader middlewares: HttpAuthMiddleware, DownloadTimeoutMiddleware, UserAgentMiddleware, RetryMiddleware, DefaultHeadersMiddleware, MetaRefreshMiddleware, HttpCompressionMiddleware, RedirectMiddleware, CookiesMiddleware, ChunkedTransferMiddleware, DownloaderStats
	2013-09-05 13:56:01+0800 [scrapy] DEBUG: Enabled spider middlewares: HttpErrorMiddleware, OffsiteMiddleware, RefererMiddleware, UrlLengthMiddleware, DepthMiddleware
	2013-09-05 13:56:01+0800 [scrapy] DEBUG: Enabled item pipelines: 
	2013-09-05 13:56:01+0800 [scrapy] DEBUG: Telnet console listening on 0.0.0.0:6023
	2013-09-05 13:56:01+0800 [scrapy] DEBUG: Web service listening on 0.0.0.0:6080
	2013-09-05 13:56:01+0800 [dmoz_spider] INFO: Spider opened
	2013-09-05 13:56:03+0800 [dmoz_spider] DEBUG: Crawled (200) <GET http://www.dmoz.org/Computers/Programming/Languages/Python/Books/> (referer: None)
	[s] Available Scrapy objects:
	[s]   hxs        <HtmlXPathSelector xpath=None data=u'<html>\n<head>\n<meta http-equiv="Content-'>
	[s]   item       {}
	[s]   request    <GET http://www.dmoz.org/Computers/Programming/Languages/Python/Books/>
	[s]   response   <200 http://www.dmoz.org/Computers/Programming/Languages/Python/Books/>
	[s]   settings   <CrawlerSettings module=<module 'demo.settings' from 'demo\settings.pyc'>>
	[s]   spider     <DmozSpider 'dmoz_spider' at 0x4bd7e10>
	[s] Useful shortcuts:
	[s]   shelp()           Shell help (print this help)
	[s]   fetch(req_or_url) Fetch request (or URL) and update local objects
	[s]   view(response)    View response in a browser

	>>> hxs.select('//title')
	[<HtmlXPathSelector xpath='//title' data=u'<title>Open Directory - Computers: Progr'>]
	>>> hxs.select('//title').extract()
	[u'<title>Open Directory - Computers: Programming: Languages: Python: Books</title>']
	>>> hxs.select('//title/text()')
	[<HtmlXPathSelector xpath='//title/text()' data=u'Open Directory - Computers: Programming:'>]
	>>> hxs.select('//title/text()').extract()
	[u'Open Directory - Computers: Programming: Languages: Python: Books']
	>>> hxs.select('//title/text()').re('(\w+):')
	[u'Computers', u'Programming', u'Languages', u'Python']
	>>> 

## 检索存储
将数据存储在我们定义的DmozItem中，修改dmoz_spider.py

	from scrapy.spider import BaseSpider
	from scrapy.selector import HtmlXPathSelector

	from demo.items import DmozItem

	class DmozSpider(BaseSpider):
		name = "dmoz_spider"
		allowed_domains = ["dmoz.org"]
		start_urls = [
			"http://www.dmoz.org/Society/Philosophy/Aesthetics/",
			"http://www.dmoz.org/Sports/Football/"
		]

		def parse(self, response):
			# filename = response.url.split("/")[-2]
			# open(filename, 'wb').write(response.body)
			hxs = HtmlXPathSelector(response)
			sites = hxs.select('//ul/li')
			items = []
			for site in sites:
				item = DmozItem()
				item['title'] = site.select('a/text()').extract()
				item['link'] = site.select('a/@href').extract()
				item['desc'] = site.select('text()').extract()
				items.append(item)
			return items	

运行Scrapy命令，检索的数据将存储在items.json中

	scrapy crawl dmoz_spider -o items.json -t json


> http://doc.scrapy.org/en/0.18/intro/tutorial.html
