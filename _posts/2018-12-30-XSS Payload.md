#	起因
时间 2018年12月29日11:39，今天我做了一次技术分享议题是《My XSS Bypass Cheat sheets》，欢迎各位朋友加入我们的技术分享交流群
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/qrcode.jpg)
XSS我已经写过一篇了，上一个是Payload，遂今天写一篇比较新的思路，也算是给2018年一个交代。

#	案例1
![PIC](https://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/xss/xss1/xss-2.png)
这是一个XSS漏洞挖掘的拓补图，当我们用户进行输入后，可能会进行如上步骤，当我们绕过了所有的防御措施使Poc成功的到了最后一步也就完成了漏洞验证，看起来可能很麻烦其实并没有，我们用一个案例来分析一下，基础的XSS挖掘不说了，来一点奇特的！
`https://blog.compass-security.com/2018/12/xss-worm-a-creative-use-of-web-application-vulnerability/`
先看一下他的twitter的问题描述
![PIC](https://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/xss/xss1/xss-3.png)

```
37 字节以内
不能引用
不能存在<script>
不能有／
```
在他的求助下我们找到了评论里的Poc，我修改之后是这样的 打开会弹出2 和 1
```html
<svg onload=eval(atob(p))>
<svg onload=p+='YWxlcnQoMSk7'>
<svg onload=p='YWxlcnQoMik7'>
```
在t00ls上也看到过关于拼接的XSS 那个时候用的是script标签，这次用的是svg
我们简单的分析一下这个Payload
```
svg onload这就不用提了，可以换换试试，而这个eval也就是执行代码也没有特殊的，我们要看一下atob这个，他是用来解密base64的，所以我们后面的p里的内容也就是base64之后的Poc，因为他说了要限制37个字节，所以进行了拼接。
```
# 案例二
2017
https://medium.com/@maxpasqua/xss-in-oculus-rifts-cdn-f5bac5ec7b9c?tdsourcetag=s_pctim_aiomsg
2018
https://www.amolbaikar.com/xss-on-facebooks-acquisition-oculus-cdn/
这是同一个地方的两处XSS 日期相隔了一年左右，我们先从老的XSS入手分析
他没有写多少内容但是我们可一看一下他YouTube的视频

![PIC](https://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/xss/xss1/xss-4.jpg)

在上传头像的位置进行文件上传
![PIC](https://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/xss/xss1/xss-5.jpg)
修改后缀名为html之后上传，打开返回的URL成功弹窗
![PIC](https://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/xss/xss1/xss-6.jpg)
这是2017年的漏洞
下面我们再看一下2018的这个
```
Remove the "/v/" parameter from the URL and modify the file extension. (".jpg" to ".html")
```
直接上传带有恶意代码的jpg，然后访问url
![PIC](https://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/xss/xss1/xss-7.jpg)
可以看到我们上传之后还是显示jpg的后缀名，是上传之后修改后缀名
然后将`/v/`去掉
原url
`https://scontent.oculuscdn.com/v/t64.5771-25/12410200_1905973632996555_3168227744525844480_n.png?_nc_cat=0&oh=6163326b2eb5e87c16c6949f1e734611&oe=5AD840C8`
最后url变成了
`https://scontent.oculuscdn.com/t64.5771-25/12410200_1905973632996555_3168227744525844480_n.html`
在上传时也可以挖掘类似漏洞

# New Payload
- Payload1
也是在twitter上看到的一些新思路做一个整理
Example:`<!----><script>alert(0);</script>`
利用注释使他们的行为看起来不像是标签所以bypass 原地址
`https://twitter.com/LooseSecurity/status/1076110424913989634`

- Payload2
这个是在portswigger博客上看见的一篇文章

Example:`<link rel="canonical" accesskey="X" onclick="alert(1)" />`
Poc using link elements (Press ALT+SHIFT+X on Windows) (CTRL+ALT+X on OS X)
我们在firefox实验一下输入CTRL+ALT+X触发我们的onclick事件
![PIC](https://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/xss/xss1/xss-8.png)
我这里的是document.cookie
原文链接
`https://portswigger.net/blog/xss-in-hidden-input-fields`
- Payload集合
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/xss/xss1/payload1.jpg)
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/xss/xss1/payload2.jpg)
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/xss/xss1/payload3.jpg)
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/xss/xss1/payload4.jpg)
原文链接
`https://twitter.com/ChefSecure/status/1077644515253587969`


# 总结
思路太多了，总有些姿势等待我们去发现，在这里祝大家新年快乐！
