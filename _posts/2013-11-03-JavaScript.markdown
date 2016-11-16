---
layout: post
title:  "JavaScript 的BOM"
date:   2013-11-01 18:10:38 +0800
categories: jekyll update
---

<img src="/images/fulls/02.jpg" class="fit image">

一、Javascript 中 BOM编程的简介

	BOM：指的是浏览器对象模型，就是关于浏览器的对象的操作方法

二、Javascript 中 BOM编程的对象的分类

	 window对象（浏览器窗口对象）
	 Location对象（浏览器地址对象）
	 History对象（浏览器历史对象）

三、Javascript 中 BOM编程的window对象

	window.alert()：弹出一个带有确定按钮的对话框
	Window.confirm()：弹出一个带有确认和取消按钮的对话框
	Window.prompt()：弹出一个带有输入框的对话框
	Window.open()：打开一个新的窗口
	Window.close()：关闭浏览器

四、Javascript 中 BOM 编程的location对象

	写法：
	Location.href=’连接地址’ 
	此属性和a标签功能一样，都是进行跳转，这里和 a 标签的 href 属性里面写法一样。
	location.reload()  方法用于重新加载当前文档。

五、history对象的方法

	history.go(-1)：后退
	history.go(0)：刷新
	history.go(1)：前进
	history.back()：后退
	history.forward()：前进

六、Javascript 中的 Array 对象

方式一：var arr=[值1,值2,值3]  隐式创建
方式二：var arr=new Array(值1, 值2, 值3)	// 显示创建或直接实例化
方式三：var arr=new Array(3)  	// 显示创建并指定数组长度

1）掌握Array对象的Length属性和循环方式

	Length：统计数组元素的长度，使用格式：数组名.length
	数组遍历方式一：for 循环遍历

	数组遍历方式二：for (……in……) 循环遍历  语法如下：
	for(变量名  in  对象名)
	{
	    // 循环体
	}

 2）Array对象的基本操作方法

	pop()：移除数组中的最后一个元素并返回该元素。
	shift()：移除数组中的第一个元素并返回该元素。（数字为0索引的）。
	push()：将新元素添加到一个数组中，并返回数组的新长度值。
	unshift()：将指定的元素插入数组开始位置并返回该数组。
	reverse()：该方法用于颠倒数组元素的顺序。
	sort()：该方法用于将数组进行排序。

七、Date 对象的创建方式

	方式一：var date = new Date()	// 创建一个当前时间
	方法二：var date = new Date(2015,9,6)	//  指定年月日创建时间
	方法三： var date = new Date(‘2015 10 01 16:02:30’);// 通过字符串创建对象

Date 对象的基本操作方法

	Date()：返回系统当前的日期和时间。
	getDate()：从日期对象中返回一个月中的某一天。
	getDay()：从日期对象中返回一周中的某一天。
	getMonth()：从日期对象中返回月份。
	getYear()、getHours()、getMinutes()、getSeconds()
	getTime()：返回从 1970年1月1日至今的毫秒数。
	toLocaleString()：根据本地时间格式，把日期对象转换为字符串。

八、定时器

一、定时器分为两种：
1、 setTimeout(‘要执行函数名()’,时间)；只执行一次
2、 setInterval(‘要执行的函数名()’,时间)；

二、清除定时器：
1、clearTimeout(定时器的名字)  只能针对setTimeout()定时器
2、clearInterval(定时器名字)   只能针对setInterval()定时器


