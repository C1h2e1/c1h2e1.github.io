#前言
clay是富二代

#References
(嫌我菜的表哥直接去看别人的，我以后会把链接直接发到上面)
https://www.owasp.org/index.php/Testing_for_HTTP_Parameter_pollution_(OTG-INPVAL-004)
https://medium.com/@logicbomb_1/bugbounty-compromising-user-account-how-i-was-able-to-compromise-user-account-via-http-4288068b901f
https://www.owasp.org/images/b/ba/AppsecEU09_CarettoniDiPaola_v0.8.pdf
https://github.com/EdOverflow/can-i-take-over-xyz
https://0xpatrik.com/subdomain-takeover-ns/
https://dzone.com/articles/what-are-subdomain-takeovers-how-to-test-and-avoid



#HPP-HTTP Parameter Pollution
`Supplying multiple HTTP parameters with the same name may cause an application to interpret values in unanticipated ways. By exploiting these effects, an attacker may be able to bypass input validation, trigger application errors or modify internal variables values. As HTTP Parameter Pollution (in short HPP) affects a building block of all web technologies, server and client side attacks exist. `
如上owasp的介绍已经很明了了。就是多个同名称多参数
`e.g. https://claynb.com/test.php?clay=nb&clay=rich`

因为觉得没什么可以写的所以就做备忘单
[p1](https://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/HPP/HPP1.png)

#Subdomain takeover
关于子域名接管我相信最近大家多多少少都有看到案例，但是我们针对子域名接管的挖掘很少有文章提到，于是我写一篇蹭热度，毕竟相当

##前期侦查
关于子域名接管，首先先得有能让你接管的子域名，所以我们针对子域名要进行收集，关于收集网上的文章多的很，随便找几篇就可以了。那我们找到了子域名之后可以使用`dig` `nslookup` `host`
```
c1h2e1@MST ~$ nslookup c1h2e1.github.io
Server:		192.168.31.1
Address:	192.168.31.1#53

Non-authoritative answer:
Name:	c1h2e1.github.io
Address: 185.199.110.153
Name:	c1h2e1.github.io
Address: 185.199.109.153
Name:	c1h2e1.github.io
Address: 185.199.108.153
Name:	c1h2e1.github.io
Address: 185.199.111.153

c1h2e1@MST ~$ host c1h2e1.github.io
c1h2e1.github.io has address 185.199.109.153
c1h2e1.github.io has address 185.199.108.153
c1h2e1.github.io has address 185.199.111.153
c1h2e1.github.io has address 185.199.110.153

c1h2e1@MST ~$ dig c1h2e1.github.io

; <<>> DiG 9.8.3-P1 <<>> c1h2e1.github.io
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7786
;; flags: qr rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;c1h2e1.github.io.		IN	A

;; ANSWER SECTION:
c1h2e1.github.io.	30	IN	A	185.199.109.153
c1h2e1.github.io.	30	IN	A	185.199.110.153
c1h2e1.github.io.	30	IN	A	185.199.111.153
c1h2e1.github.io.	30	IN	A	185.199.108.153

;; Query time: 78 msec
;; SERVER: 192.168.31.1#53(192.168.31.1)
;; WHEN: Tue Mar 12 23:24:14 2019
;; MSG SIZE  rcvd: 98
```
我们用这样的方法有时可以发现他解析的服务，比如Slack的这个案例
`https://hackerone.com/reports/195350`
首先执行了dig命令发现解析了`redirect.feedpress.me`然而这个feedpress可以导致子域名接管，最终成功的接管了域名。

##可接管服务！
[p1](https://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/HPP/takeover1.png)
[p1](https://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/HPP/takeover2.png)
[p1](https://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/HPP/takeover3.png)
我们需要做的事情就是针对子域名进行搜集，对比指纹，查看是否有接管的可能。

#最后
写的很水，但起码我回过头来能有个资料参考，或许这也是博客存在的原因
