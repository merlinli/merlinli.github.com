---
layout: post
title: "jekyll在windows下中文问题"
date: 2013-07-25 17:01
comments: true
categories: 
---
首先你的英文编写必须完全正确。
其次你的文件必须是"utf-8 without BOM" 而不仅仅是utf-8.
在windows下utf-8文件都会被加上BOM头。一些编辑软件可以做到这点，比如notepad++,MarkdownPad等。
如果编码正确了，还是会报错：
>Liquid error: incompatible character encodings: UTF-8 and IBM437”

这是因为你的控制台不能使用UTF-8,比如我使用的是win7的powershell.
解决办法是更改控制台编码：
打开powershell 直接输入
>chcp 65001

chcp  ：更改字码页  
65001 ：UTF8

MORE:[代码页 wiki](http://zh.wikipedia.org/wiki/%E4%BB%A3%E7%A0%81%E9%A1%B5 "代码页")


