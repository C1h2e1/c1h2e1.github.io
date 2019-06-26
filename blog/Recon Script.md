好久不见各位，我的博客要开始周更3篇，今天带来的是信息收集的脚本打造

#前言
关于信息收集脚本，我确实很喜欢这种收集方式，自动化的无脑可以让我一根烟的功夫就可以拿到信息收集的结果。只有一个爽字可以形容这种自动化了，废话不多说我们开始


#目标
当拿到我们的目标的时候我们需要拿到什么才算是前期的信息收集呢？

- Subdomain 
- FILE 
- Endpoint
- IP

#思考
9012年了我们的信息收集要做到的绝不仅仅是爆破的自动化那样太掉价了，我们要快速且全面的自动化！！！！

#正文
我们的脚本要利用大量的第三方平台，那么我们就针对如上的目标进行分解 全局配置(127.0.0.1:1087是我的代理)

##Subdomain
关于子域名的搜索我们选用的第三方平台很多，首先是`crt.sh`在这里大家可以自行进行测试这个网站的玩法。我们就直接给出我们的脚本 (原谅我正则写的不好～～)
```
curl -x 127.0.0.1:1087 https://crt.sh/?q=%.$1 | grep "$1" | cut -d '>' -f2 | cut -d '<' -f1 | grep -v " " | sort -u 
```
利用curl发送数据包，`-x`使用代理 然后我们后面的进行正则，这样我们就可以获得一大堆子域名
![p1](/assets/images/recon/subdomain1.png)

```
curl -x 127.0.0.1:1087 https://www.threatcrowd.org/searchApi/v2/domain/report/\?domain=$1 |jq .subdomains |grep -o '\w.*$1'
```
![p1](/assets/images/recon/subdomain2.png)

```
curl -x 127.0.0.1:1087 https://api.hackertarget.com/hostsearch/\?q\=$1 | grep -o '\w.*$1'
```
![p1](/assets/images/recon/subdomain3.png)

```
 curl -x 127.0.0.1:1087 https://certspotter.com/api/v0/certs?domain=baidu.com | grep  -o '\[\".*\"\]' |grep -o '\w.*.baidu.com'
```

```
curl -x 127.0.0.1:1087 --connect-timeout 1 -m 20 "http://dns.bufferover.run/dns?q=.$1" | jq -r '.FDNS_A[],.RDNS[]' | awk -F ',' '{print $2}' | sort -u
```

##FILE&Endpoint
关于文件和端点的搜索主要是Commoncrawl 和wayback medicine 

```
curl -x 127.0.0.1:1087 http://web.archive.org/cdx/search/cdx/search/cds?url=baidu.com/*&output=text&fl=original&collapse=urlkey
```
如果只针对JS可以使用可以吧.js替换成想寻找的任何后缀名
```
curl -x 127.0.0.1:1087 http://web.archive.org/cdx/search/cdx/search/cds?url=baidu.com/*&output=text&fl=original&collapse=urlkey |grep -o '.*.js'
```
关于commoncrawl
```
curl -x 127.0.0.1:1087 -sX GET "http://index.commoncrawl.org/CC-MAIN-2018-22-index?url=*.$1&output=json" | jq -r .url | sort -u 
```

在寻找到JS之后我们可以使用LINKFINDER来进行正则匹配Endpoint，由于LINKFINDER不能很好的支持文件的批量导入所以我们要使用一个脚本吧每一个JS都进行匹配
```
#!/bin/bash
file_lines=$(cat $1)

for line in $file_lines
do
       sudo python linkfinder.py -i `echo $line` -o $(echo `pwd`)"/"$(echo `echo "$line"` | md5sum | sed -e 's/^\(.\{32\}\).*/\1/')".html" &>/dev/null

done
```
如上的脚本的`sudo python linkfinder.py`要根据你的环境来修改. Linkfinder github 地址如下
`https://github.com/GerbenJavado/LinkFinder`
同样的我们还可以利用Endpoint 来提取JS。这里有一个脚本，叫Subjs`https://github.com/lc/subjs` 这里的subjs不支持代理也就是不能对国外的网站进行提取，所以尽量把subjs装在服务器上，如果不想用subjs也可以使用命令行 

```
curl -x 127.0.0.1:1087 js地址 | grep -Eo "(http|https)://[a-zA-Z0-9./?=_-]*"
```
这样就可以输出端点了～～～


#整理
我们吧所有的端点整理到一起使用就可以获得我们想要的子域名端点js了～～～

晚安晚安2019.6.26
