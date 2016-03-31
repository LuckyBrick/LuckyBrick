# 定时器性能分析

Firefox 3在整体性能（内存、速度、UI响应，JavaScript等）上做了较大的提升。定时器（setTimeout和setInterval）的性能也有幸得到了提升。为了证明这个，我做了一些性能分析。

我做了一个简单的测试：移动一个div，每次1像素，共100像素，分别用setTimeout、setInterval和Mozilla的内置接口[nsITimer](http://www.xulplanet.com/references/xpcomref/ifaces/nsITimer.html)实现。希望通过测试得到以下几点的精确数据：


1. 两次定时器调用之间的时间间隙是多长？
2. 定时器调用波动有多大？
3. 定时器并发数量范围
4. 定时器最小延迟能达到多少？

## 测试用例：

* [测试效果预览（默认使用setInterval，时间间隔0ms）](http://ejohn.org/apps/timers/)

* [下载测试代码](http://ejohn.org/files/timers.zip)

图表使用[jQuery图表插件Flot](https://code.google.com/p/flot/)渲染 

## 结果（仅在OSX平台）：

**图注：** 横轴是定时器递归调用的深度，纵轴是定时器调用之间的延迟－以毫秒为单位，另外每条线的数值对应于并发执行的定时器的数量。

该图的组织顺序依次为：（从左上，顺时针方向）Firefox2，Safari3，Opera9，Firefox3.
![](http://i1.wp.com/ejohn.org/files/timers.png)

**setTimeout**:[0ms延时](http://ejohn.org/files/timers-0.png)，[10ms延时](http://ejohn.org/files/timers-10.png)，[20ms延时](http://ejohn.org/files/timers-20.png)
![](http://i2.wp.com/ejohn.org/files/intervals.png)

**setInterval**:[0ms延时](http://ejohn.org/files/intervals-0.png),[10ms延时](http://ejohn.org/files/intervals-10.png),[20ms延时](http://ejohn.org/files/intervals-20.png)  
![](http://i0.wp.com/ejohn.org/files/nsitimer.png)  

**nsITimer**:[0ms延时](http://ejohn.org/files/nsitimer-0.png)，[10ms延时](http://ejohn.org/files/nsitimer-10.png),[20ms延时](http://ejohn.org/files/nsitimer-20.png)  
![](http://i0.wp.com/ejohn.org/files/timerfilter-sm.png)  


**修改后的Firefox3：**在运行了这些测试后，我注释掉Firefox3中的[定时器过滤代码](http://lxr.mozilla.org/seamonkey/source/xpcom/threads/TimerThread.cpp#179)，看看结果会怎么样。查看[完整结果](http://ejohn.org/files/timerfilter.png)（从左到右依次是：nsITimer、setInterval、setTimeout，从上到下是：0ms、10ms、20ms延时）

## 结论(仅在OSX平台)

* WebKit内核定时器代码非常完美，它们的测试结果像婴儿的屁股一样光滑，简直难以置信。
    
* 整个测试过程中，Opera都表现的非常混乱。
    
* Firefox2一般情况下比较稳定，但是会出现异常的延时峰值（垃圾收集机制开始起作用时）。
     
* Firefox3较少出现firefox2那种延时很长但是会骤降到0-2ms的情况（这应该深入研究下）。
    
* Firefox2、Opera和Safari都将最小延迟窗口设置为10ms，因为某些原因，Firefox3目前是15ms（这应该深入研究下）。
    
* 一旦定时器并发数量达到64-128范围内，大多数浏览器都不会得到很好的结果。
    
* nsITimer在1-2个并发定时器的情况下表现最好。
    
* Firefox3中nsITimer存在很大的问题。
    
* 删除定时器过滤代码（在Firefox3中）表明定时器有相当一部分逻辑是DOM代码，而不是纯粹的定时器代码。

如上所述，这些测试都是在OSX环境中进行的，如果有人感兴趣，非常欢迎能产出IE、Firefox2、Firefox3和Opera的对比结果。在此期间，Vlad对windows XP环境下的Firefox3（setTimeout、setInterval和nsITimer）做了测试，产出了一些基本结果。注意到这与OSX环境下的结果有很大出入，因此上述结果和结论也许仅仅适用于OSX平台。

我希望借此衍生出一些讨论，以便更好地找出哪些是真正需要解决的bug（特别是在Firefox3中）。Webkit团队则完全不用担心，因为Webkit中不存在这类问题。  
