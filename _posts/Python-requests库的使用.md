---
title: Python-requests库的使用
date: 2018-11-14 11:41:55
tags: Python requests
categories: Python lib
---
# Python-requests库的使用
urllib 的用法中有一些不方便的地方，比如处理网页验证和Cooki es 时，需要写 Opener 和 Handler 来处理。 为了更加方便地实现这些操作，就有了更为强大的库requests ，有了它 ， Cookies 、登录验证、代理设置等操作都变得极为简单。

## 一切以官网为主
http://docs.python-requests.org/en/master/

## 基本用法

### 一个简单的例子
urllib库中的 urlopen()法实际上是以GET方式请求网页，而requests中相应的方法就是get()
方法。
```python
import requests

response = requests.get('https://www.baidu.com')
print(type(response))
print(response.status_code)
print(type(response.text))
print(response.text)
print(response.cookies)
```
![enter image description here](https://t1.picb.cc/uploads/2018/11/14/JShz6r.png)

### GET请求
#### 基本实例
``` python
import requests

print('simple test')
response = requests.get('https://httpbin.org/get')
print(response.text)

print('params test')
data = {'name': 'germey', 'age': 22}
response = requests.get('https://httpbin.org/get', params=data)
print(response.text)

# 将json格式解析为字典格式
print('json() test')
response = requests.get('https://httpbin.org/get')
print(type(response.text))
print(response.json())
print(type(response.json()))
```
![enter image description here](https://t1.picb.cc/uploads/2018/11/14/JShDOJ.png)
![enter image description here](https://t1.picb.cc/uploads/2018/11/14/JShYg1.png)
![enter image description here](https://t1.picb.cc/uploads/2018/11/14/JShEP0.png)

#### 抓取网页
``` python
import requests, re

print('抓取网页测试')
headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:63.0) Gecko/20100101 Firefox/63.0'}
response = requests.get('https://www.zhihu.com/explore', headers = headers)
pattern = re.compile('explore-feed.*?question_link.*?>(.*?)</a>', re.S)
title = re.findall(pattern, response.text)
print(title)
```
![enter image description here](https://t1.picb.cc/uploads/2018/11/14/JSvydu.png)

#### 抓取二进制数据
图片、音频、视频这些文件本质上都是由二进制码组成的，由于有特定的保存格式和对应的解析方式， 我们才可以看到这些形形色色的多媒体 。 所以，想要抓取它们，就要拿到它们的二进制码。
``` python
import requests

print('抓取二进制数据测试')
response = requests.get('https://github.com/favicon.ico')
print(response.text)
print(response.content)  # Content of the response, in bytes
with open('favicon.ico', 'wb') as f:
    f.write(response.content)
```
![enter image description here](https://t1.picb.cc/uploads/2018/11/14/JSvJuD.png)

#### 添加headers
``` python
print('添加headers测试')
headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:63.0) Gecko/20100101 Firefox/63.0'}
response = requests.get('https://www.zhihu.com/explore', headers = headers)
print(response.text)
```
![enter image description here](https://t1.picb.cc/uploads/2018/11/14/JSvgsi.png)

### POST请求
``` python
import requests

data = {'name': 'germy', 'age': '22'}
response = requests.post('https://httpbin.org/post', data=data)
print(response.text)
```
![enter image description here](https://t1.picb.cc/uploads/2018/11/14/JSvt6L.png)

### 响应
发送请求后，得到的自然就是响应 。 在上面的实例中，我们使用 text 和 content 获取了响应的内容。 此外，还有很多属性和方法可以用来获取其他信息，比如状态码、响应头、 Cookies 等。
``` python
import requests

print('响应测试')
headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:63.0) Gecko/20100101 Firefox/63.0'}
response = requests.get('https://www.jianshu.com', headers = headers)
print(response.headers)
print(response.cookies)
# print(response.status_code)
exit() if not response.status_code == requests.codes.ok else print('request successfully')
print(response.content)
print(response.text)
```
![enter image description here](https://t1.picb.cc/uploads/2018/11/14/JSx5ST.png)

## 高级用法
### 文件上传
``` python
import requests

files = {'file': open('favicon.ico', 'rb')}
response = requests.post('https://httpbin.org/post', files = files)
print(response.text)
```
![enter image description here](https://t1.picb.cc/uploads/2018/11/14/JSxvT1.png)

### 会话维持
在 requests 中，如果直接利用 get()、post()等方法的确可以做到模拟网页的请求，但是这实际上是相当于不同的会话，也就是说相当于用了两个浏览器打开了不同的页面，其实解决这个问题的主要方法就是维持同一个会话 ， 也就是相当于打开一个新的浏览器选项卡而不是新开一个浏览器 。 但是又不想每次设置 cookies ，那该怎么办呢？这时候就有了新的利器--Session 对象 。利用它，我们可以方便地维护一个会话，而且不用担心cookies的问题，它会帮我们自动处理好。利用 Session ，可以做到模拟同一个会话而不用担心 Cookies 的问题。 它通常用于模拟登录成功之后再进行下一步的操作。
``` python
import requests

print('session测试')
session = requests.Session()
session.get('https://httpbin.org/cookies/set/number/123456789')
response = session.get('https://httpbin.org/cookies')
print(response.text)
```
![enter image description here](https://t1.picb.cc/uploads/2018/11/14/JSLtir.png)

### SSL证书验证
如果请求一个 HTTPS 站点，但是证书验证错误的页面时，就会报这样的错误，那么如何避免这个错误呢？很简单，把 verify 参数设置为False 即可。当然，我们也可以指定一个本地证书用作客户端证书，这可以是单个文件（包含密钥和证书）或一个包含两个文件路径的元组。
``` python
import requests
import logging
from requests.packages import urllib3

urllib3.disable_warnings()  # 忽略警告
logging.captureWarnings(true)  # 捕获警告
response = requests.get(url, **verify = false**)  # 设置verify
response = requests.get(url, cert = (crt_path, key_path))  # 设置证书文件地址
print(response)
```

### 代理设置
对于某些网站，在测试的时候请求几次 ， 能正常获取内容。 但是一旦开始大规模爬取，对于大规模且频繁的请求，网站可能会弹出验证码，或者跳转到登录认证页面 ， 更甚者可能会直接封禁客户端的 IP ，导致一定时间段内无法访问 。那么，为了防止这种情况发生，我们需要设置代理来解决这个问题，这就需要用到 proxies 参数。
``` python
import requests

proxies = {'http': '...'
		   'https': '...'
		   ...
		   }
response = requests.get(url, proxies = proxies)
```

### 超时设置
在本机网络状况不好或者服务器网络响应太慢甚至无响应时，我们可能会等待特别久的时间才可能收到响应，甚至到最后收不到响应而报错。 为了防止服务器不能及时响应，应该设置一个超时时间 ，即超过了这个时间还没有得到响应，那就报错。 这需要用到 timeout 参数。 这个时间的计算是发归请求到服务器返回响应的时间。实际上，请求分两个阶段，即连接()connect )和读取(read )。如果想永久等待，可以直接将 timeout 设置为 None(默认) 。
``` python
import requests

print('有timeout异常捕获')
try:
    response = requests.get('https://www.baidu.com', timeout = 0.01)
    print(response.status_code)
except requests.exceptions.Timeout:
    print('Timeout')
except requests.exceptions.ConnectionError:
    print('ConnextionError')
except requests.exceptions.HTTPError:
    print('HTTPError')

print('无timeout异常捕获')
response = requests.get('https://www.baidu.com', timeout = (0.01, 0.01))
print(response.status_code)
```
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-14/91035917.jpg)

### 身份认证
``` python
import requests
from requests.auth import HTTPBasicAuth

response = requests.get(url, auth=HTTPBasicAuth(username, userpassword))
response = requests,get(url, auth=(username, userpassword))
```

### prepared request
前面介绍 urllib 时，我们可以将请求表示为数据结构，其中各个参数都可以通过一个 Request 对象来表示 。 这在 requests 里同样可以做到，这个数据结构就叫 Prepared Request。有了 Request 这个对象，就可以将请求当作独立的对象来看待，这样在进行队列调度时会非常方便。
``` python
from requests import Request, Session

print('prepared request测试')
url = 'https://httpbin.org/post'
data = {'name': 'germey'}
headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:63.0) Gecko/20100101 Firefox/63.0'}

session = Session()
request = Request('POST', url, data = data, headers = headers)
prepped = session.prepare_request(request)
response = session.send(prepped)
print(response.status_code)
print(response.text)
```
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-14/46350763.jpg)
