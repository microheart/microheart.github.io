---
layout: post
title: Scrapy抓取topit.me
category: python
tags: 爬虫
keywords: 爬虫
description: 
---

## 定义Item for `http://topit.me/`

	class TopitImageItem(Item):
		image_urls = Field()
		images = Field()
		title = Field()
		author = Field()
		image_paths = Field()

![](/public/upload/python/scrapy_img_crawler2.JPG)

## 自定义CrawSpider

	class TopitSpider(CrawlSpider):
		name = 'topit_spider'
		allowed_domains = ['topit.me']
		start_urls = ['http://www.topit.me']

		rules = (			
			Rule(SgmlLinkExtractor(allow = (r'(\?p=(\d)+)+'), deny = ('user', 'album')), follow = True),
			Rule(SgmlLinkExtractor(allow = (r'item/(\d)+$')), callback='parse_item'),
		)

		def parse_start_url(self, response):
			return self.parse_item(response)

		def parse_item(self, response):
			self.log('Hi, item image url: %s' % response.url)
			hxs = HtmlXPathSelector(response)
			item = TopitImageItem()
			item['title'] = hxs.select('//div[@id="content"]/div/h2/text()').extract()
			item['author'] = hxs.select('//div[@id="content"]//p/a//text()').extract()
			item['image_urls'] = hxs.select('//div[@id="content"]/a/img/@src').extract()
			yield item

* 通过rules对连接进行过滤和设定相应的回调函数，我们抓取item页面的大图
* parse_item通过XPath提取页面信息

## Pipeline保存图片和信息

定义两个`Pipeline`，`TopitImagePipeline`保存图片,`JsonWriterPipeline`保存图片相关信息至json文件

	ITEM_PIPELINES = ['demo.pipelines.TopitImagePipeline', 'demo.pipelines.JsonWriterPipeline']
	IMAGES_STORE = 'D:\\info\\data\\imgs\\crawl\\topit'

### TopitImagePipeline

	class TopitImagePipeline(ImagesPipeline):

		def get_media_request(self, item, info):
			for image_url in item['image_urls']:
				yield Request(image_url)

		def item_completed(self, results, item, info):
			image_paths = [x['path'] for ok, x in results if ok]
			if not image_paths:
				raise DropItem('Item contains no images')
			item['image_paths'] = image_paths
			return item

### JsonWriterPipeline

	class JsonWriterPipeline(object):
		def __init__(self):
			self.file = open('topit.json', 'wb')

		def process_item(self, item, spider):
			line = json.dumps(dict(item)) + "\n"
			self.file.write(line)
			return item

将数据保存到`topit.json`