---
layout: post
title:  Csrf的读取—增删改查
date: 2018-11-18 10:28:24.000000000 +08:00
---
第一篇博客 希望几年以后还能在信息安全的道路上，每15天更新一次

+	2018-11月18日1点20分 写给马上十六岁的自己：不忘初心，坚持理想，不退缩，keep real，热爱技术，求知向学


##前面是扫盲后面会讲利用的姿势关于进阶的技巧我会在写一篇
# What is CSRF?
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width="330" height="86" src="//music.163.com/outchain/player?type=2&id=554244265&auto=0&height=66"></iframe>
+	扫盲
`CSRF`(Cross-site request forgery)跨站请求伪造，也被称为“One Click Attack”或者Session Riding，通常缩写为CSRF或者XSRF
这么听起来好像不是那么好理解，我们举个例子
 `小明 和 小红 和 面部解锁 `的故事
一天小明去小红家玩，小红这时在睡觉没有发现小明偷偷拿起了她的手机，因为她的手机有面部解锁，只需要刷一下小红的脸就可以了，机制的小明在小红睡觉的时候吧手机放在她面前。小红在睡梦中受到了这次攻击，手机解锁成功，而我们应该如何防护这一安全问题呢？这时某家手机厂商发布了新的技术，这个技术就是当你解锁的时候验证手机的来源，比如是否能匹配你的手表或者戒指其他东西，来证明是你主动得解锁，也就是证明解锁的人是你，这样小明得利用就比较难了，但是也并不是没有办法，比如使用偷走小红的手表等等，此时手机厂商又推出了新的功能那就是用户的凭证，每一次都会进行凭证的验证，当每一次用户解锁手机的时候手表需要从服务端取回数据，然后再用手表中的作为凭证去做面容解锁，而且每一个数据只能使用一次。这样不能仅仅靠小明的那一双手能够越过的了，但是也不是没有办法，那么故事讲完了我们来分析一下各个事物在`CSRF`攻击中相当于什么东西吧~首先
小明攻击者 小红受害者这里我们把小红的脸部也就是我们的`cookie`或者是一种识别我们用户的标识。手表这样用于验证也就是`Referer` 那么`token`就是服务端生成的数据了

# How to dig csrf Vulnerability ?
+	实例：我们用一个csrf实例来进入我们的讲解过程[wooyun-csrf实例](http://www.anquan.us/static/bugs/wooyun-2015-0164067.html)
![csrf_1]({{ "http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/csrf_1.png"|csrf_1}})
![csrf_2]({{ "http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/csrf_2.png"|csrf_2}})
上面这个例子是我在wooyun找到的以往的一个csrf漏洞，那么我们画一张流程图模拟一下他的挖掘流程和大体思路。
也就是  发现发微博的功能点----尝试利用此功能点发微博----微博发表成功----尝试绕过Referer的限制----绕过referer验证编写Poc进行利用
![csrf_3]({{ "http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/"|csrf_3}})
也就是这样的流程我们下面给大家分开来讲这个流程
+	发现功能点：我想说的是每个点都有可能是csrf的利用点，但敏感程度成了我们的难点，比如我们上面的这个微博的发布，这就算一个比较敏感的功能点了，我给大家列举几个比较明显的点`订单下单处 修改敏感信息处 删除信息处 绑定处 发送信息处 等等`大家这是发现里面有增的操作比如`订单下单`也有改的操作`修改账号密码`也有删的操作`删除信息处`关于查的话我会在后面给大家写读取型的csrf漏洞。那么我们发现完了功能点之后就可以进行下面的操作了
+	尝试使用此功能点：测试此处的时候我们只需要进行模拟的使用这个功能点就好了，但是我们这里就要主要一下token 以及referer的验证了我们把这个内容放在最后讲。我们就假设此处没有任何验证机制我们只需要发送相应数据包就可以完成敏感操作的就好了
+	编写Poc及利用：这里我推荐大家使用burpsuite的csrf POC自动生成的工具，下面我给大家演示一下使用方法![csrf_4]({{ "/assets/images/csrf_1/csrf_4.png"|csrf_4}})我们按照上述步骤点击就可以生成csrf的poc，同时burpsuite还为我们提供了直接的测试功能可以直接在浏览器中打开url进行测试![csrf_5]({{ "/assets/images/csrf_1/csrf_5.png"|csrf_5}})
+	挖掘思路总结：个人觉得csrf的增删改三处操作的挖掘难度不是很大只是有个正常的流程就可以挖到，最后我会去超链接几个关于csrf的案例大家

#### Read type Csrf Vulnerability
+	1.Jsonp劫持 ：首先我们讲一下什么是jsonp [jsonp](https://zh.wikipedia.org/wiki/JSONP)`JSONP（JSON with Padding）是数据格式JSON的一种“使用模式”，可以让网页从别的网域要数据。另一个解决这个问题的新方法是跨来源资源共享。`jsonp的作用也就是做跨域资源共享的，而这里我们又要引入一个函数那就是callback函数[回调函数](https://zh.wikipedia.org/wiki/%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0)这里的解释可能看不懂，其实我们只需要把他理解成一个回调给我们的参数就好了，我们只需要有这个参数在去读取这个参数就可以进行jsonp的劫持了，我们这里还是举一个例子给大家做演示
	+	漏洞案例:![csrf_6]({{ "/assets/images/csrf_1/csrf_6.png"|csrf_6}})我们这个位置的漏洞是劫持用户得一些个人信息我们发现url上有一个参数是`callback=aaa`没错这里就是回调函数，我们就可以根据他来劫持jsonp的数据了那么我们就可以构造如下poc
	```
	<script>
function aaa(json)
{
	alert(JSON.stringify(json))
}
</script>
<script src="https://vip.xxxxx.com/ajax/list/memberPonits.do?callback=aaa
"></script>
	```所以这就是jsonp劫持 思路也是十分简单我们所要做的也就是寻找敏感点-->测试敏感点功能-->构造Poc
+	Cors劫持：首先我们要了解一下什么是cors![csrf_7]({{ "/assets/images/csrf_1/csrf_7.png"|csrf_7}})
![csrf_8]({{ "/assets/images/csrf_1/csrf_8.png"|csrf_8}})和jsonp的作用差不多，只不过是方式有所改变，那么我们在这个漏洞中的关键点就是Orgin这个参数的传递了，有时候我们需要自己添加有时候他有，而有时候他会通过某些参数传递，可能这么说不太理解我还是举个例子
	+	漏洞案例：![csrf_9]({{ "/assets/images/csrf_1/csrf_9.png"|csrf_9}})这里我们还要讲一下他的Poc [cors—Poc](https://github.com/nccgroup/CrossSiteContentHijacking)git 下来然后在本地搭建起来就好了，用法就只需要想像图片中一样讲我们的url和POST DATA传过去就好了。
	+	挖掘思路：添加Ogrin头部信息看返回的数据里面有没有Access-Control-Allow-Orgin这个参数出现如果有尝试让他变成任意的url只要这样就可以进行cors劫持了。

#### GET Rerferer bypass
首先我们假设我们请求的目标主机是`https://dafsec.org`
![csrf_10]({{ "/assets/images/csrf_1/csrf_10.png"|csrf_10}})

+	1.置空：删除referer内容
+	2.子域名：因为开发的正则问题可能存在子域名这种绕过方式，比如`https://dafsec.c1h2e1.org`我们让dafsec变成子域名这样就绕过了比较弱的正则了![csrf_11]({{ "/assets/images/csrf_1/csrf_11.png"|csrf_11}})
+	3.修改域名：`https://adafsec.c1h2e1.org`  `https://0dafsec.c1h2e1.org` `https://Ddafsec.c1h2e1.org`利用这种方法也是绕过referer验证的好方法
+	4.参数： `https://c1h2e1.org/?https://dafsec.org`


#### Last
简单的总结了一下csrf漏洞，第一篇博客经验不足，望见谅
