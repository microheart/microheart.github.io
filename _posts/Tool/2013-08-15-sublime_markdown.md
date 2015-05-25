---
layout: post
title: Sublime Text 2 编辑Markdown
category: Tool
tags: sublime
keywords: 
description: 
---

## 1 安装sublime text

* 下载[Sublime Text 2](http://www.sublimetext.com/)，并安装。

* 安装Package Control, 按 Ctrl + `  打开console粘贴以下代码到console并回车


```python
import urllib2,os;pf='Package Control.sublime-package';ipp=sublime.installed_packages_path();os.makedirs(ipp) if not os.path.exists(ipp) else None;open(os.path.join(ipp,pf),'wb').write(urllib2.urlopen('http://sublime.wbond.net/'+pf.replace(' ','%20')).read())
```

* 重启Sublime Text 2

## 2 安装Markdown Preview

* 按`Ctrl + Shift + P` => 输入pci 后回车(Package Control: Install Package) => 输入Markdown Preview 回车


## 3 测试

* 按`Ctrl + N` 新建一个文档
* 按`Ctrl + Shift + P` => 使用Markdown语法编辑文档语法高亮，输入ssm 后回车(Set Syntax: Markdown)
* 向文档中写入内容
* 按`Ctrl + Shift + P` => 输入mp 选择(Markdown Preview: Preview in Browser)，此时就可以在浏览器里看到刚才编辑的文档了

## 注意事项
按上述办法安装Markdown Preview插件可能并没有语法高亮，解决方案如下：

* 打开[https://gist.github.com/CrazyApi/2354062](https://gist.github.com/CrazyApi/2354062)，拷贝markdown.xml的内容.
* 在sublime中，查看`Preferences——Color scheme`，记住你使用哪个scheme，比如我使用Monokai
* `Preferences——Browse Packages`，进入`Color Scheme – Default`目录，找到`Monokai.tmTheme`这个文件，打开编辑，在最后一个</dict>后面, </array>前面，将刚才拷贝的内容粘贴进去，保存
OK，语法高亮就可以了。

附markdown.xml文件内容

	<!-- copy this to YOUR_THEME.tmTheme-->
	<dict>
		<key>name</key>
		<string>diff: deleted</string>
		<key>scope</key>
		<string>markup.deleted</string>
		<key>settings</key>
		<dict>
			<key>background</key>
			<string>#EAE3CA</string>
			<key>fontStyle</key>
			<string></string>
			<key>foreground</key>
			<string>#D3201F</string>
		</dict>
	</dict>
	<dict>
		<key>name</key>
		<string>diff: changed</string>
		<key>scope</key>
		<string>markup.changed</string>
		<key>settings</key>
		<dict>
			<key>background</key>
			<string>#EAE3CA</string>
			<key>fontStyle</key>
			<string></string>
			<key>foreground</key>
			<string>#BF3904</string>
		</dict>
	</dict>
	<dict>
		<key>name</key>
		<string>diff: inserted</string>
		<key>scope</key>
		<string>markup.inserted</string>
		<key>settings</key>
		<dict>
			<key>background</key>
			<string>#EAE3CA</string>
			<key>foreground</key>
			<string>#219186</string>
		</dict>
	</dict>
	<dict>
		<key>name</key>
		<string>Markdown: Linebreak</string>
		<key>scope</key>
		<string>text.html.markdown meta.dummy.line-break</string>
		<key>settings</key>
		<dict>
			<key>background</key>
			<string>#A57706</string>
			<key>foreground</key>
			<string>#E0EDDD</string>
		</dict>
	</dict>
	<dict>
		<key>name</key>
		<string>Markdown: Raw</string>
		<key>scope</key>
		<string>text.html.markdown markup.raw.inline</string>
		<key>settings</key>
		<dict>
			<key>foreground</key>
			<string>#269186</string>
		</dict>
	</dict>

	<dict>
		<key>name</key>
		<string>Markup: Heading</string>
		<key>scope</key>
		<string>markup.heading</string>
		<key>settings</key>
		<dict>
			<key>fontStyle</key>
			<string>bold</string>
			<key>foreground</key>
			<string>#cb4b16</string>
		</dict>
	</dict>
	<dict>
		<key>name</key>
		<string>Markup: Italic</string>
		<key>scope</key>
		<string>markup.italic</string>
		<key>settings</key>
		<dict>
			<key>fontStyle</key>
			<string>italic</string>
			<key>foreground</key>
			<string>#839496</string>
		</dict>

	</dict>
	<dict>
		<key>name</key>
		<string>Markup: Bold</string>
		<key>scope</key>
		<string>markup.bold</string>
		<key>settings</key>
		<dict>
			<key>fontStyle</key>
			<string>bold</string>
			<key>foreground</key>
			<string>#586e75</string>
		</dict>
	</dict>
	<dict>
		<key>name</key>
		<string>Markup: Underline</string>
		<key>scope</key>
		<string>markup.underline</string>
		<key>settings</key>
		<dict>
			<key>fontStyle</key>
			<string>underline</string>
			<key>foreground</key>
			<string>#839496</string>
		</dict>
	</dict>
	<dict>
		<key>name</key>
		<string>Markup: Quote</string>
		<key>scope</key>
		<string>markup.quote</string>
		<key>settings</key>
		<dict>
			<key>fontStyle</key>
			<string>italic</string>
			<key>foreground</key>
			<string>#268bd2</string>
		</dict>
	</dict>
	<dict>
		<key>name</key>
		<string>Markup: List</string>
		<key>scope</key>
		<string>markup.list</string>
		<key>settings</key>
		<dict>
			<key>foreground</key>
			<string>#657b83</string>
		</dict>
	</dict>
	<dict>
		<key>name</key>
		<string>Markup: Raw</string>
		<key>scope</key>
		<string>markup.raw</string>
		<key>settings</key>
		<dict>
			<key>foreground</key>
			<string>#b58900</string>
		</dict>
	</dict>
	<dict>
		<key>name</key>
		<string>Markup: Separator</string>
		<key>scope</key>
		<string>meta.separator</string>
		<key>settings</key>
		<dict>
			<key>background</key>
			<string>#eee8d5</string>
			<key>fontStyle</key>
			<string>bold</string>
			<key>foreground</key>
			<string>#268bd2</string>
		</dict>
	</dict>


> http://www.cnblogs.com/webmoon/p/3240174.html