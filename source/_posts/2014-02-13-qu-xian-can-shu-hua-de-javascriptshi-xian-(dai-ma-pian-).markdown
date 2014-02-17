---
layout: post
title: "曲线参数化的Javascript实现（代码篇）"
date: 2014-02-06 22:48:03 +0800
comments: true
categories: [javascript, computer graph]
---
[在曲线参数化的Javascript实现（理论篇）](http://sparkmorry.github.io/blog/2014/01/30/qu-xian-can-shu-hua-de-javascriptshi-xian-%28dai-ma-pian-%29/)中推出了曲线弧长积分的公式，以及用二分法通过弧长s来查找样条曲线上对应的u，再求Q(u)的值。弧长积分函数如下：

<img src="/images/posts/Param/ds.png" style="width:300px;"></img>，其中<img src="/images/posts/Param/ABCDE.png" style="width:200px;"></img>－－－－－公式1

Simpson展开后

<img src="/images/posts/Param/Simpson.png" style="width:300px;"></img>－－－－－－－公式2

这里的f(u)就是上面的弧长s的求值函数，既样条曲线拟合函数Q(u)的积分

这部分具体代码如下：1.计算f(u)的系数ABCDE

    CParametric.prototype.SegCoef = function(CSpline, o)
    {        
        p0.x=this.k[0]*CSpline.Spline[o].x+this.k[1]*CSpline.Spline[o+1].x+this.k[2]*CSpline.Spline[o+2].x+this.k[3]*CSpline.Spline[o+3].x;
        p1.x=this.k[4]*CSpline.Spline[o].x+this.k[5]*CSpline.Spline[o+1].x+this.k[6]*CSpline.Spline[o+2].x+this.k[7]*CSpline.Spline[o+3].x;
        p2.x=this.k[8]*CSpline.Spline[o].x+this.k[9]*CSpline.Spline[o+1].x+this.k[10]*CSpline.Spline[o+2].x+this.k[11]*CSpline.Spline[o+3].x;
        p3.x=this.k[12]*CSpline.Spline[o].x+this.k[13]*CSpline.Spline[o+1].x+this.k[14]*CSpline.Spline[o+2].x+this.k[15]*CSpline.Spline[o+3].x;

        p0.y=this.k[0]*CSpline.Spline[o].y+this.k[1]*CSpline.Spline[o+1].y+this.k[2]*CSpline.Spline[o+2].y+this.k[3]*CSpline.Spline[o+3].y;
        p1.y=this.k[4]*CSpline.Spline[o].y+this.k[5]*CSpline.Spline[o+1].y+this.k[6]*CSpline.Spline[o+2].y+this.k[7]*CSpline.Spline[o+3].y;
        p2.y=this.k[8]*CSpline.Spline[o].y+this.k[9]*CSpline.Spline[o+1].y+this.k[10]*CSpline.Spline[o+2].y+this.k[11]*CSpline.Spline[o+3].y;
        p3.y=this.k[12]*CSpline.Spline[o].y+this.k[13]*CSpline.Spline[o+1].y+this.k[14]*CSpline.Spline[o+2].y+this.k[15]*CSpline.Spline[o+3].y;

        this.A=9*(p0.x*p0.x+p0.y*p0.y);
        this.B=12*(p0.x*p1.x+p0.y*p1.y);
        this.C=6*(p0.x*p2.x+p0.y*p2.y)+4*(p1.x*p1.x+p1.y*p1.y);
        this.D=4*(p1.x*p2.x+p1.y*p2.y);
        this.E=p2.x*p2.x+p2.y*p2.y;
    }
这里的k数组是Cardinal样条曲线的Mc矩阵，具体内容可以参考Cardinal样条曲线的Javascript实现（理论篇）。这里的p0,p1,p2,p3就是公式1里的ax,ay,bx,by,cx,cy,dx,dy。从而求得积分函数系数。

2.计算扩展的Simpson方法展开后得到的积分函数

    CParametric.prototype.ArcLength = function(ustart, uend){
        var h,sum,u;
        var i;
        h=(uend-ustart)*1.0/ITERATION_COUNT; //扩展的Simposon方法的阶数
        sum=0;
        u=ustart+h;

        for(var i=2; i <= ITERATION_COUNT; i++){
            if(!(i&1)){
                sum += 4.0*this.ArcIntergrand(u);
            }
            else{
                sum += 2.0*this.ArcIntergrand(u);
            }
            u += h;
        }
        return(h*this.ArcIntergrand(ustart)+sum+this.ArcIntergrand(uend)/3.0);
    }
这里的ITERATION_COUNT是扩展的Simposon方法的阶数，既公式2中的n，其中的ArcIntergrand(u)就是求fi的值，代码如下：

    CParametric.prototype.ArcIntergrand = function(u)
    {
        var f;
        f=this.A*u*u*u*u+this.B*u*u*u+this.C*u*u+this.D*u+this.E;
        f=Math.sqrt(f);
        return f;
    }
3.建立s－u索引表

上面的积分函数只能得到没一个曲线段上一个u对应s的值，而一整条样条曲线由多个小曲线段组成，比如计算第二段曲线上累加的s的值的时候只要把前面那段曲线的总长作为一个基数加上去就可以了。建立了s－u索引表之后就可以用二分法来查找每个给定的s对应的u的值了。因为这个函数是严格单调递增的，所以这个索引表肯定是升序的，就可以用二分法来查找。具体代码如下：

    CParametric.prototype.SetLengths = function(spline, npoints){
        var no_segments;    //u分割的精度
        var arcLength;
        this.seg_lengths = new Array();

        no_segments = npoints-3;
        arcLength = 0;

        this.seg_lengths[0]=0;
        for(var i=0; i<no_segments; i++){
            this.SegCoef(spline, i);
            arcLength += this.ArcLength(0.,1.);   //ArcLength(0,1)就是前面一个曲线段的总长
            this.seg_lengths[i+1] = arcLength;
        }
        this.total_length = arcLength;
        for(var i=1; i<no_segments; i++){
            this.seg_lengths[i] /= this.total_length;  //s归一化
        }
    }
因为弧长累加的值较大，所以对其进行归一化之后可以得到更标准的弧长值。

4.二分法计算弧长为seg的曲线上的点的位置pt

    CParametric.prototype.ArcLengthPoint = function(spline, seg){
        var uL=0., uR=1., usearch;
        var segment_dist, ssearch;
        var segment_no=0;

        do{
            segment_no++;
        }while(this.seg_lengths[segment_no] < seg);
        segment_no --;
        segment_dist = this.total_length*(s-this.seg_lengths[segment_no]);

        this.SegCoef(spline,segment_no);

        do{
            usearch = (uL+uR)/2;
            ssearch = this.ArcLength(uL, usearch);

            if(segment_dist < ssearch+SEARCH_TOL){
                uR=usearch;
            }
            else{
                uL=usearch;
                segment_dist -= ssearch;
            }
        }while((segment_dist>ssearch+SEARCH_TOL) || (segment_dist<ssearch-SEARCH_TOL));
        this.GetPointOnSpline(usearch);

    };
就是普通的二分查找法，给定一个弧长seg，在函数曲线上找到对应的u的值，返回的usearch就是在指定精度var SEARCH_TOL=0.5（可以任意指定）的情况下得到的u的值。再通过调用this.GetPointOnSpline(usearch)函数来计算u=usearch时对应的Q(u)的值。

    CParametric.prototype.GetPointOnSpline=function(usearch){
        this.pt[pointnum]= new CPT();
        this.pt[pointnum].x=p0.x*usearch*usearch*usearch+p1.x*usearch*usearch+p2.x*usearch+p3.x;
        this.pt[pointnum].y=p0.y*usearch*usearch*usearch+p1.y*usearch*usearch+p2.y*usearch+p3.y;
    }