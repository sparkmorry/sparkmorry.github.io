---
layout: post
title: "Cardinal样条曲线的Javascript实现(代码篇)"
date: 2014-01-29 11:08:00 +0800
comments: true
categories: [javascript, computer graph]
---
由上一篇文章得到了Cardinal曲线的矩阵表达式，下面就这个矩阵表达式就可以来对曲线进行插值了。

这里选用了JS来实现，完全是因为之前交作业的时候还不知道怎么在Xcode里建完整的C++OpenGL的项目，所以就用js在浏览器这个最大的跨平台UI上写。。

先看之前得到的Cardinal插值的样条曲线的矩阵，对每一段曲线PkPk+1来说，都可以通过拟合一个被参数化为P(u)函数（0<= u <=1）的图形的形式：</br>
<img src="/images/posts/Cardinal/Cardinal5.png" style="width:220px;"></img>，其中
<img src="/images/posts/Cardinal/Cardinal6.png" style="width:220px;"></img></br>

同样，M*P得到的是系数矩阵[a b c d]的转置，s是引入的表示每个连接点处曲线尖锐程度的变量－张量

所以我们现在的期望是

输入：n个控制点的xy轴坐标数组x[],y[]，张量tension，每两个控制点之间插入的密度grain

输出：插值好的样条曲线上控制点的一个数组

1.构造函数 function CSpline(x, y, grain, tension, n)

拥有的成员变量和成员函数：

	function CSpline(x, y, grain, tension, n) {   
	    var jd = new Array(100);  //用户输入的控制点位置数组，临时变量存储
	    this.n0=n;   //控制点个数
	    this.m = new Array(16);  //Mc矩阵
	    this.Spline = new Array(1024);  //插值好的控制点数组
	    CSpline.prototype.GetCardinalMatrix = function(a1) {}    //通过给定的tension值生成Mc矩阵
	    CSpline.prototype.Matrix = function(a, b, c, d, u){}   //计算P(u)的值，xy分量均相同
	    CSpline.prototype.CubicSpline = function(np, knots, grain, tension) {}   //插值函数

	    //临时函数，建立曲线控制点数组jd，并调用CubicSpline函数计算插值曲线
	    function CSpline(x, y, grain, tension, n)      
	    { 
	        for(var i=1; i<=np; i++){
	        　　jd[i] = new CPT(x[i-1],y[i-1]);   //内部点
	        }
	        jd[0] = new CPT(x[0],y[0]);  //补上隐含的整条曲线起始点
	        jd[np+1] = new CPT(x[np-1],y[np-1]);    //补上隐含的整条曲线终止点
	        np=np+2;   
	        this.CubicSpline(np, jd, grain, tension);
	    }
	}      
这里成员变量通过构造函数分别在实例中存储，而成员函数则在对应的原型函数中存储。

2.计算Mc矩阵函数GetCardinalMatrix = function(a1)

根据输入的平滑度 tension 值计算Mc矩阵

	CSpline.prototype.GetCardinalMatrix = function(a1) {
	    this.m[0]=-a1; this.m[1]=2.0-a1; this.m[2]=a1-2.; this.m[3]=a1;                 
	    this.m[4]=2.*a1; this.m[5]=a1-3.; this.m[8]=-a1; this.m[9]=0.;
	    this.m[12]=0.; this.m[13]=1.; this.m[6]=3.-2*a1; this.m[7]=-a1;
	    this.m[10]=a1; this.m[11]=0.; this.m[14]=0.; this.m[15]=0.;
	} 
这里m是一个4*4的矩阵，a1表示tension值

3.计算P(u)的值

输入4个点Pk-1,Pk,Pk+1,Pk+2的值(x分量或者y分量，以及参数化好的u值)，返回计算得到的P(u)的值

	CSpline.prototype.Matrix = function(p0, p1, p2, p3, u){ 
	    //求解系数
	    var a, b, c, d;
	    a=this.m[0]*p0+this.m[1]*p1+this.m[2]*p2+this.m[3]*p3;    
	    b=this.m[4]*p0+this.m[5]*p1+this.m[6]*p2+this.m[7]*p3; 
	    c=this.m[8]*p0+this.m[9]*p1+this.m[10]*p2+this.m[11]*p3; 
	    d=this.m[12]*p0+this.m[13]*p1+this.m[14]*p2+this.m[15]*p3; 
	    return(d+u*(c+u*(b+u*a))); //au^3+bu^2+cu+d
	}
4.主要绘制函数CSpline.prototype.CubicSpline = function(np, knots, grain, tension)  

	CSpline.prototype.CubicSpline = function(np, knots, grain, tension) {
	    alpha = new Array(50);
	    var k0, kml, k1, k2;
	        //获取Mc矩阵
	        this.GetCardinalMatrix(tension);
	        //对每两个关键点之间的插值点进行参数化到0～1之间
	    for(var i=0; i<grain; i++){
	        alpha[i]=i*1.0/grain;
	    }
	        //从最开始的四个点开始，给第一段曲线插值
	    kml = 0;
	    k0 = 1;
	    k1 = 2;
	    k2 = 3;
	    this.s = 0; //纪录总共插值后的点数
	        //两次循环第一次对输入的控制点遍历，第二次对每两个控制点之间插值，分别计算xy分量上得出的插值后的函数值，k值分别＋1
	    for(var i =1; i<this.n0; i++){
	        for(var j=0; j<grain; j++){
	            cpx = this.Matrix(knots[kml].x, knots[k0].x, knots[k1].x, knots[k2].x, alpha[j]);
	            cpy = this.Matrix(knots[kml].y, knots[k0].y, knots[k1].y, knots[k2].y, alpha[j]);
	            this.Spline[this.s] = new CPT(cpx, cpy);
	            this.s++;
	        }
	        kml++; k0++; k1++; k2++;
	    }
	}
最后的结果截图如下：</br>
<img src="/images/posts/Cardinal/t1.png" style="width:420px;"></img></br>