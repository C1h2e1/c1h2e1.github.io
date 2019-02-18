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
