这两篇文章写的很乱 我没有删是因为 这是我成长的一个见证 第一篇英语博客 第一次接触新鲜技术
Hello my bro ,long time no see.
this Blog is talk about XS-Search Vulnerability.  
all resource in the END of this page
#Original
yesterday , i look for some case about bug bounty. the `pentester.land` had updated `Hacking NewsLetter 38` so i look that , soon i focus on the Video `https://www.youtube.com/watch?v=HcrQy0C-hEA` then i watch the video immediately .But i can't understand the XS-Search attack. so i decide to do some research about that . here we go!!


#START
Attacker :`Devil`     Victim : `Tom`      Vulnerable server : `test.com`
The Vulnerable server is a Shopping website. hacker Devil find the XS-Search Vulnerability on the order page Search service.this picture is my Hand drawn, i think it maybe little stupid but i wanna show my think to you.
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/timing/timing-2.png)
this is the very easy to understand ,user request exploit code and run it on browser ,then the different response return .The XS-Search attack difficulty lies in how to compared the full response and empty response . the SOP can't allow Devil read the response So this have some challenges .

#XS-Timing Attack
`https://c1h2e1.github.io/2018/12/时序攻击/`
start from one example
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/timing/timing-5.png)
if users login gmail the response will return faster than not login .
use this way you can know the victim Whether to log in `gmail.com`
the timing attack is based on response time
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/timing/timing-4.png)
the true order id only `123` .  you can use the Advanced Search , try different algorithms send lot of request to get true order id !
the timing attack depend on different time!
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/timing/timing-6.png)
###Why is this?
The empty response like thin people ,the full response like fat people ,so the full response can't run as fast as thin
###INFLATION
you can try compared somewhere the empty and full response maybe have no small different ,even they return time same. so the  INFLATION is important .
Make the response size gap larger and the response time will be longer.
**if you can let visitor send any request and measure response time please try this way to get sensitive information**

#XS-Search Attack
always = XS-Timing Attack on search service

#Reference
```
https://www.owasp.org/images/a/a7/AppSecIL2015_Cross-Site-Search-Attacks_HemiLeibowitz.pdf
https://medium.com/@luanherrera/xs-searching-googles-bug-tracker-to-find-out-vulnerable-source-code-50d8135b7549
https://www.youtube.com/watch?v=6JUvS_eLNdI
https://www.youtube.com/watch?v=vzp7JdezZRU
https://www.youtube.com/watch?v=23dGPyitMK4
https://gist.github.com/lbherrera/188a871edbe7645be18545805be036b8
https://www.slideshare.net/RuochunZeung/xssearch
```

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
