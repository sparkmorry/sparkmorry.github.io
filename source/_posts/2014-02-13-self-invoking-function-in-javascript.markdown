---
layout: post
title: "Self-invoking function in Javascript"
date: 2014-02-05 23:31:52 +0800
comments: true
categories: [javascript]
---
在看bootstrap的轮播插件源码的时候发现一种新的自调函数的写法（其实应该不新了），开头的一段就卡住了，原谅我是个菜鸟，就顺便记录一下。stackoverflow上有大神们的回答在这里，轮播的函数形式如下：

	!function ($) {
	   //code here
	}(window.jQuery);
通常自调函数来模仿块级作用域的方式是：

	(function(parameters){
	     //这里是块级作用域
	})();
在function外面的这对括号使里面的匿名函数变成了函数表达式，并在之后马上调用。这个相当于：

	var Name=function(parameters){
	   //这里是块级作用域
	}; 
	Name();  //调用匿名函数
函数表达式是在运行中以引用的方式赋值给一个变量的，如上面的函数创建的方式。而这里函数表达式和函数声明是不一样的，函数声明的通常形式如下：

	function Name(parameters){
	    //code here
	}
函数声明会在程序刚开始运行的时候会被提升（所以可以在执行之后声明），但是并不执行，直到被调用的时候再执行。

所以与括号作用相同的function前面的感叹号，因为括号的优先级高于感叹号，感叹号使得后面的一整串内容变成了一个bool表达式，编译器就会直接运行这个表达式了。这个表达式是个匿名函数，传入了一个参数，这里命名成$，其实无所谓是什么，只是一个形参，并在自调的时候把jQuery对象传进去，这里的jQuery对象是个全局变量，所以是windows.jQuery