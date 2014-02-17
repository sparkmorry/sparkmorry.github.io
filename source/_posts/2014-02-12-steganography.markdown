---
layout: post
title: "Steganography"
date: 2013-11-12 00:42:02 +0800
comments: true
categories: [python, image processing, PIL]
---
今天做DAM的作业，做到图片水印的时候，想起来当初小调同学把言页之庭的种子通过图片发给我。看到下面[这个新闻](<http://war.163.com/10/0712/17/6BDLNUB90001123L.html>)真是觉得碉堡了！！

下面分享一下自己用python实现的隐写术，后面如果有时间会再更深入研究一下解码的问题！！

基本思路是：

1.每张图片都有RGB三个通道，每个像素点的值为3个8位的R/G/B值。

2.把每个RGB值的最后两位&11111100(0xFC)移掉，因为最后两位对图片的影响不大，肉眼基本不可见

3.把要隐藏的图片的RGB值都除以85(255/3=85)，这样正好可以通过2bits(00-11,0-3)来表示图片

4.把得到的值填到之前的2个空bit中

5.等解码的时候只要把之前在原图中藏进去的2位提取出来(&0x3)再＊85就可以得到原图信息

还有一种方法是把信息写道Alpha通道里并按RBG格式编码，解码时用L格式。

但是图片会失真一点，因为毕竟相比8位的存储，2位的存储信息会少很多，但是却已经可以看到80%的图片信息了。所以不得不再赞一下二八法则，20%的编码信息里存储了80%的图片信息啊。

	import Image,ImageChops

	def waterMark(originFile,markFile):
    	origin = Image.open(originFile)
    	mark = Image.open(markFile)
    	size = mark.size #record the original size of the marked image
    	mark = mark.resize(origin.size) #set the pics in same size

    	origin.load();
    	mark.load()
    	source1=origin.split() #split the pic in 3 channel
    	source2=mark.split()

    	image=[(),(),()]
    	waterMark=[(),(),()]
    	for x in [0,1,2]: #loop in 3 channel, RGB
        	image[x]=source1[x].point(lambda i:i & 0xFC) #remove the last 2 bits of the original image, 0xFC=11111100
        	waterMark[x]=source2[x].point(lambda i:i/85) #reduce the RGB value of each channel, 255/3=85
        
    	mark=Image.merge("RGB",waterMark)
    	origin=Image.merge("RGB",image)

    	result=ImageChops.add(mark,origin)

    	return result,size

	def deCode(waterMark,size):
    	waterMark.load()
    	source = waterMark.split()
    	originSize=size 

    	mark=[(),(),()]
    	for x in [0,1,2]:
        	mark[x]=source[x].point(lambda i:(i & 0x3)*85) #get back the RGB value of the marked image

    	result=Image.merge("RGB",mark)
    	result = result.resize(originSize)

    	return result

	if __name__ == '__main__':
    	img1,size = waterMark("avatar2.jpg","green.jpg")
    	img2 = deCode(img1,size)

    	img1.show() #original image with watermark
    	img2.show() #embeded image