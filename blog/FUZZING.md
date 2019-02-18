好久不见，我是衬衫，还有5天开学了，博客的更新频率就要变多了，今天给大家带来的是FUZZING，也算是对寒假的交代了。
现在是2019年2月18日，距离我2月1日的技术分享已经过去了17天
`https://www.bugbank.cn/live/view.html?id=111909`
本文就是对这次的总结与升华，在搭配案例进行分析，那么不多逼逼开始吧
---

#核心思想
在直播中我也有提到核心思想的问题，其实也就是公式。
- 目录Fuzz(漏洞点)
- 参数Fuzz(可利用参数)
- Payload Fuzz(bypass)
很庆幸在这次演讲之后我得到了两个漏洞可以作为典型案例，那我们这里先记下来核心思想我们往下继续
---
#漏洞挖掘与Fuzz之敏感目录

![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/Fuzz/fuzz1.png)
##御剑
关于目录Fuzz从我做技术的开始就开始接触，那个时候我开始使用御剑，那遇见的扫描模式是什么呢？
`http://domain.com/+你的目录字典`那一般我们的目录字典是这样子的
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/Fuzz/fuzz2.png)
那么这种扫描有什么好处呢？就是针对一部分网站可以扫描的全面，只要你的字典足够强大就可以扫描到绝大多部分的目录和文件，来自`Blasting_dictionary`的爆破字典很好，github地址`https://github.com/rootphantomer/Blasting_dictionary`这里的103w+目录字典就很符合御剑的模式，其实也就是看程序员的命名。
##Dirsearch
关于这个工具我没有用过，但是是很好的一款工具，他的扫描模式和dirbuster是差不多的我们直接在dirbuster的时候说吧
##Dirb
我每次都使用`/usr/share/wordlists/dirb/common.txt`这个字典在各种的情景下都是很好用，这是他的字典截图
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/Fuzz/fuzz3.png)
可以看出来他的字典和御剑的不一样的就是没有针对目录和准确的定位到每个文件而是一个一个的目录名，那么这种扫描下是有一些有点的就是先发现目录在进行文件爆破，而且dirb的判断很智能他在你输入目标后会进行计算错误的请求，避免内些返回200的`not found`
`Usage : dirb https://domain.com/`
##wfuzz
wfuzz是一款十分万能的工具我最近的目录爆破全都是使用wfuzz用熟练之后真的非常十分方便，排除一些响应码之后直接baseline这是我目前比较喜欢的用法，后面我会分享一个案例就是wfuzz的成果。
使用参考key师傅wfuzz三部曲,我就是在这学的
```
https://gh0st.cn/archives/2018-10-28/3
https://gh0st.cn/archives/2018-10-28/2
https://gh0st.cn/archives/2018-10-28/1
```
#举栗子
- 案例一
`wfuzz -z file,starter.txt -p 192.168.31.26:1080:SOCKS5 --hs "Cannot" https://foo.domain.com/FUZZ`
当时我fuzz的命令,因为是国外的网站所以加了代理，过滤了返回`Cannot`的返回包当我等待返回结果的时候他返回了`/user/login.action`当我访问的时候返回的具体内容我没有保存，大概意思是`User Not found`然后我停下了wfuzz开始进行`/user/`目录的fuzz这是我fuzz了很久，基本上都是返回这样，然后打开了user目录，发现他直接给我返回了整站的用户数据由于是私有项目而且涉及用户比较多，这里就不放截图了现在回头看我上面的twitter截图，是不是觉得很对？
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/Fuzz/fuzz1.png)
这就是核心思想的体现在寻找漏洞点的目录Fuzz

---


---

#漏洞挖掘与Fuzz之敏感目录可利用参数
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/Fuzz/fuzz7.png)
先从Burpsuite的扩展程序CO2说起，关于CO2的话相信不需要太多介绍了，Sqlmapper模块很好，对于我这种注入菜的很的来说简直就是福音，而`CeWler`的功能是参数提取，比如我们在Http history 里找返回包右键发送到CeWler模块就可以进行参数提取了，在实战中的用处很大，可以把参数提取出来保存做参数字典，这样的字典更高效。

- 案例二
##Jsonp劫持
一次众测中我的目标网站是金融站，在个人中心的很多业务都是json返回，有很多的敏感信息。我尝试Fuzz了callback函数结果没有成功，之后我在另一个子域名进行了参数提取发现了`_cb_`在添加之后发现成功的回调，然后劫持成功。
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/Fuzz/fuzz8.png)

- 案例
##Hidden XSS
这个案例不是我的，原文链接`https://markitzeroday.com/xss/finding/2018/02/03/hidden-xss.html`
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/Fuzz/fuzz4.png)
这里原作者用到了`Nikto`这里他发挥的作用在于目录的爆破其实如果用其他工具也可以代替，返回报告发现了`/test/`目录，这是访问了test目录
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/Fuzz/fuzz5.png)
返回了NULL，这是他进行了进一步的fuzz，也就是参数fuzz
```
$ wfuzz -w /usr/share/wordlists/dirb/common.txt --hh 53 'http://rob-sec-1.com/test/?FUZZ=<script>alert("xss")</script>'
********************************************************
* Wfuzz 2.2.3 - The Web Fuzzer                         *
********************************************************

Target: HTTP://rob-sec-1.com/test/?FUZZ=<script>alert("xss")</script>
Total requests: 4614

==================================================================
ID	Response   Lines      Word         Chars          Payload    
==================================================================

02127:  C=200      9 L	       8 W	     84 Ch	  "item"

Total time: 14.93025
Processed Requests: 4614
Filtered Requests: 4613
Requests/sec.: 309.0369
```
在fuzz过后发现了item参数，这是访问`http://rob-sec-1.com/test/?item=<script>alert("xss")</script>`成功弹窗
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/Fuzz/fuzz6.png)
很好的体现参数Fuzz与目录Fuzz如果这个item参数有过滤那就很完美的体现了所有的核心思想: D

---

#漏洞挖掘与Fuzz之Bypass
