---
layout: post
title: "利用crash堆栈信息定位代码崩溃位置"
date: 2015-01-12 18:32:15 +0800
comments: true
categories: 
---
根据程序崩溃的堆栈信息确定代码crash位置,以下是一段crash堆栈信息
	
	........(
	........0...Client.........................0x005c3963.ElongClient. .5712227,
	........0,
	........Client,
	........0x005c3963.Client. .5712227,
	........0x005c3963
	....)
	此crash堆栈信息,需要加入第三方管理,通过后台进行数据统计
	
使用下面的命令符号化,可以准确地定位crash代码的位置

	atos -arch armv7 -o .Dsyms路径 -l 0x52000 0x005c3963

在终端下输入该指令,结果或显示具体方法,具体文件,具体行数

	-[ViewController viewDidLoad] (ViewController.m:156)
	
0x52000是由0x005c3963减去10进制的5712227计算而得,.Dsyms文件是在程序打包的时候一起获得的文件(线上crash版本必须和这个配套)



