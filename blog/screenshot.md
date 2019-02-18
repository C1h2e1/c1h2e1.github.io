
#	起因
在twitter刷这玩找到了一个有趣的东西
是一个网站
`https://screenshot-v2.now.sh/`


# 正文
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/screenshot/shot1.png)
我们在网站的后面加上我博客看一下
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/screenshot/shot2.png)
这就让我想起了Aquatone的gather 他的功能就是获取网页截图。我想都不想就写个轮子试试


首先我们需要创建初始URl
```python
URL='https://screenshot-v2.now.sh/'
```
经过测试我们可以发现他直接返回一张图片给我。所以我们需要请求并下载
```python
img_URL=URL+target
req1=requests.get(img_URL)
```
在文件中读取target
```python
target_file=sys.argv[1]
with open(target_file,'r') as file:
	for target in file :
         try :
            main(URL,target)
         except :
            print("error")
```
写入文件
```python
def create_html(filename):
    filename1=filename.replace(".jpg","")
    file2=open(filename1+'.html','w')
    moban='''
	<img src="%s.jpg">
	<a href="%s">target</a>
'''%(filename1,target)
    file2.write(moban)
```
这样就完成了主体功能了，因为我怕对这个网站造成伤害，所以我就没有改多线程
下面的是我运行对微博的子域名进行收集的一些截图
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/screenshot/shot3.png)
![PIC](http://c1h2e1.oss-cn-qingdao.aliyuncs.com/image/screenshot/shot4.png)
由于编码问题，所以出现这样的问题

我们的最终脚本如下
```python
import requests
import sys

URL='https://screenshot-v2.now.sh/'

def main(URL,target):
    header={"User-Agent": "Mozilla/5.0 (Linux; U; Android 8.1.0; en-us; Redmi 6A Build/O11019) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/61.0.3163.128 Mobile Safari/537.36 XiaoMi/MiuiBrowser/10.4.2"}
    img_URL=URL+target
    req1=requests.get(img_URL,headers=header)
    filename=target.replace("https://","").replace("http://","").replace("\n","").replace('/',"")+'.jpg'
    with open(filename, 'wb') as file1:
        file1.write(req1.content)
    print(target)
    create_html(filename)


def create_html(filename):
    filename1=filename.replace(".jpg","")
    file2=open(filename1+'.html','w')
    moban='''
	<img src="%s.jpg">
	<a href="%s">target</a>
'''%(filename1,target)
    file2.write(moban)

target_file=sys.argv[1]
with open(target_file,'r') as file:
	for target in file :
         try :
            main(URL,target)
         except :
            print("error")

```

# 总结
眼光放长，思路放开，把所有钱都赚进口袋来  押韵  ：)
