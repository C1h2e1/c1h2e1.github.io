---
layout: post
title: Timing Attacks
categories: Pentest
description: Timing Attacks
keywords: Pentest
---
还是一波群的广告，欢迎各位朋友加入我们的技术分享交流群
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/qrcode.jpg)

#	起因
国外的一个文档里面的logic漏洞里面有一个timing attack 我google了一下，没找到什么有用的，在知乎的一个问答上看到了相关的内容，所以研究一下

在这里我想说，我研究安全问题处于对技术的热爱，那些喜欢挖各种高危的兄弟不要在说没卵用了！眼光不同互相尊重！

#	What is timing attack

假设我在一个登陆的地方已经知道账号的情况下
我们的正确密码为`Password`当输入密码为`abcde`返回时间为0.0001秒 当输入密码为
`Pbcde`的时候返回时间为0.0002秒，也就是说我们的服务端进行密码判断的时候是根据我们的输入依次判断，如果第一个就错了直接返回错误
所以我们可以根据返回时间进行猜解，这类漏洞在web应用很少有人挖以至于我找了国内的漏洞并没有发现，在twitter上发现了一个演讲，但是没有视频，根据图片上的内容我找到了一篇相关文章

#	Python '==' Vulnerable
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/timing/timing-1.png)
从这个tweet中我开始搜寻python'==' 漏洞
还好我运气比较好找到了这篇文章

`https://thisdata.com/blog/timing-attacks-against-string-comparison/`

```
"hunter2" == "password123" # False. 51μs
 F
"hunter2" == "obama"       # False. 50μs
 F
"hunter2" == "havana"      # False. 73μs
 TF
"hunter2" == "humana"      # False. 90μs
 TTF
...
"hunter2" == "hunter1"     # False. 170μs
 TTTTTTF
"hunter2" == "hunter2"     # True.  190μs
 TTTTTTT
```
这里面就体现了这个思想也就是我们在输入的时候判断的返回时间，根据返回时间我们可以猜出密码，这样就是一次成功的timing attack
我找到了一个youtube的视频
链接如下
`https://www.youtube.com/watch?v=KirTCSAvt9M` blackhat2015年的演讲，开头他提到了web应用的时序攻击，他举的例子是保险企业的社保账号处，根据你输入的，判断是否正确-->返回数据-->根据时间判断-->利用
在我们实际挖掘过程中可能不会挖到这种漏洞，但是写出来是为了记录我所学到的新技术

#	总结
学习 进步
hardwork
