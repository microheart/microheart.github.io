---
layout: post
title: sublime text2 配置ctags
category: Tool
tags: sublime
keywords: 
description: 
---

## 安装ctags

* 下载ctags  
从[ctags](http://sourceforge.net/projects/ctags/files/ctags/5.8/ctags58.zip/download)下载压缩文件, 解压到到任意目录，如D:\Program Files, 则ctags目录路径为D:\Program Files\ctags58，将路径添加到path路径。  

* 测试ctags是否安装成功  
开始 => cmd =>ctags --help ,显示如Usage: ctags [options] [file(s)]的帮助信息，则正确。  

* 为sublime text2安装ctags插件  
ctrl + shift + p => pi => ctags =>enter(回车)  

* 在工程目录下生成 .tags文件  

	$cd your_code_path  
	$ctags -R -f .tags

* 测试  
在调用函数处或者函数声明上，ctrl+shift+鼠标左键，则会跳转到相应的函数定义处。  

* 快捷键  
![](/public/upload/tool/ctags_shortcut.png)

