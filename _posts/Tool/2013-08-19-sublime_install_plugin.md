---
layout: post
title: sublime插件安装
category: Tool
tags: sublime
keywords: 
description: 
---


## 安装包控制(Package Control)


打开 Sublime Text 2，按下 Control + ` 调出 Console, 输入以下代码，回车。

    import urllib2,os; pf='Package Control.sublime-package'; ipp = sublime.installed_packages_path(); os.makedirs( ipp ) if not os.path.exists(ipp) else None; urllib2.install_opener( urllib2.build_opener( urllib2.ProxyHandler( ))); open( os.path.join( ipp, pf), 'wb' ).write( urllib2.urlopen( 'http://sublime.wbond.net/' +pf.replace( ' ','%20' )).read()); print( 'Please restart Sublime Text to finish installation')

对于Sublime Text3，请输入以下代码：

    import urllib.request,os; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())


## 安装Markdown Preview插件
Mardown Preview不仅支持在浏览器中预览Markdown文件，还可以导出html代码，这样极大方便导出代码发布到博客上。

* `ctrl + shift + p`(windows下快捷键)调出命令面板，输入mdp
    
# Sublime Text快捷键：

Ctrl+Shift+P：打开命令面板  
Ctrl+P：搜索项目中的文件  
Ctrl+G：跳转到第几行  
Ctrl+W：关闭当前打开文件  
Ctrl+Shift+W：关闭所有打开文件  
Ctrl+Shift+V：粘贴并格式化  
Ctrl+D：选择单词，重复可增加选择下一个相同的单词  
Ctrl+L：选择行，重复可依次增加选择下一行  
;Ctrl+Shift+L：选择多行  
Ctrl+Shift+Enter：在当前行前插入新行  
Ctrl+X：删除当前行 
Ctrl+M：跳转到对应括号  
Ctrl+U：软撤销，撤销光标位置  
Ctrl+J：合并行  
Ctrl+F：查找内容  
Ctrl+Shift+F：查找并替换  
Ctrl+H：替换  
Ctrl+R：前往 method  
Ctrl+N：新建窗口  
Ctrl+F2：设置/删除标记  
/*
Ctrl+/：注释当前行  
Ctrl+Shift+/：当前位置插入注释  
Ctrl+Alt+/：块注释，并Focus到首行，写注释说明用的  
Ctrl+Shift+A：选择当前标签前后，修改标签用的  
*/
F11：全屏  
Shift+F11：全屏免打扰模式，只编辑当前文件  
Alt+F3：选择所有相同的词  
Alt+.：闭合标签  
Alt+Shift+数字：分屏显示  
Alt+数字：切换打开第N个文件  
Shift+右键拖动：光标多不，用来更改或插入列内容  
鼠标的前进后退键可切换Tab文件  
按Ctrl，依次点击或选取，可需要编辑的多个位置  
按Ctrl+Shift+上下键，可替换行  
鼠标右键+shift: 列选择模式
