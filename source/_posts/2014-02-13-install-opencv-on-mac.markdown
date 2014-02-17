---
layout: post
title: "Install opencv on Mac"
date:  2013-12-10 00:15:32 +0800
comments: true
categories: [opencv, Mac, computer vision]
---
下载好opencv之后

1. 在文件夹下新建一个release或build的文件夹；

2. cmake .

　make

Python中运行：

在该build文件夹下

nano .bash_profile

把python的路径加下去

export PYTHONPATH=/usr/local/lib/python2.7/site-packages/:$PYTHONPATH
 
Xcode中运行，可能碰到的一些问题：

* usr/local/include和usr/local/lib的问题就不说了，编译之前要把头文件和动态库都连进去否则会产生dyld: Library not loaded的错误
* dyld: Library not loaded: cv2.so，有放到cv2.so檔案的, 則是要記得到Build Phases（project target下）內，把選項改為optional
* Undefined symbols for architecture x86_64: 
C++库和opencv库编译不一致，将XCode的C++ Standard Library改为GNU C++ standard library即可正常编译显示图片。Project Build settings -> APPLE LLVM compiler 4.2 - Language -> C++ Standard Library

在这里赞一下这个[网站](http://wiki.opencv.org.cn/index.php/%E9%A6%96%E9%A1%B5)，里面有很多例程参考，每个都有不同语言的版本



