---
title: python-urllib库的使用
date: 2018-11-13 12:14:43
tags: python urllib
categories: Python lib
---
# Python urllib库的介绍及使用

@(python)[urllib]

**urllib**库，是 Python 内置的 HTTP 请求库 ，也就是说不需要额外安装即可使用。它包含如下 4 个模块：
- **request** ：最基本的 HTTP 请求模块，可以用来模拟发送请求 。 就像在浏览器里输入网址然后回车一样，只需要给库方法传入 URL 以及额外的参数，就可以模拟实现这个过程；
- **error** ：异常处理模块，如果出现请求错误 ， 我们可以捕获这些异常，然后进行重试或其他操作以保证程序不会意外终止；
- **parse** ：一个工具模块，提供了许多 URL 处理方法，比如拆分、解析 、 合并等。
- **robotparse**：主要是用来识别网站的 robots.txt 文件，然后判断哪些网站可以爬，哪些网站不可以爬，它其实用得比较少；

接下来主要介绍前三个模块：**request**/**error**/**parse**

-------------------

[TOC]

## request--基本请求模块

使用 urllib 的 request 模块 ，我们可以方便地实现请求的发送并得到响应 。 本节就来看下它的具用法。 

### 1. urlopen

#### urlopen API
urllib.request.urlopen(url, data=None, [timeout, ]*, cafile=None, capath=None, cadefault=False, context=None)
##### 参数说明
- url: 具体请求的url
- data：data参数是可选的。如果要添加该参数，并且如果它是字节流编码格式的内容，即bytes类型，则需要通过bytes()方法转化。另外，如果传递了这个参数，则它的请求方式就不再是GET方式，而POST方式。
- timeout：timeout参数用于设置超时时间，单位为秒，如果请求超出了设置的这个时间，还没有得到响应，就会抛出异常。如果不指定该参数，就会使用全局默认时间。它支持HTTP/HTTPS/FTP请求。可以通过设置这个超时时间来控制一个网页如果长时间未响应，就跳过它的抓取 (利用 try - except 语句来实现 )
- cafile：指定CA证书
- capath：指定CA证书路径
- cadefault：弃用，默认为false
- context：必须是ssl.SSLContext类型，用来指定SSL设置
  
``` python
# urlon测试代码
import urllib.request
response = urllib.request.urlopen('http://www.baidu.com')
print(response.read().decode('utf8'))
print(response.status) # 输出响应状态
print(response.getheaders()) # 输出响应头信息
print(response.getheader('Server')) # 输出某响应头一个具体信息，如Server
# data参数测试
data = bytes(urllib.parse.urlencode({'word': 'hello'}), encoding='utf-8')
response = urllib.request.urlopen('http://httpbin.org/post', data = data)
print(response.read().decode('utf-8'))
# timeout参数测试
response = urllib.request.urlopen('http://www.baidu.com', timeout=0.01)
print(response.read().decode('utf8'))
import urllib.error
import socket

try:
    response = urllib.request.urlopen('http://www.baidu.com', timeout=0.01)
except urllib.error.URLError as e:
    if isinstance(e.reason, socket.timeout):
        print('timeout')
```
**输出：**
**获取网页源码/响应状态**
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-13/34725331.jpg)
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-13/72483819.jpg)
**data参数测试**
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-13/23750983.jpg)
**timeout参数测试**
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-13/85235094.jpg)
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-13/67726795.jpg)

### 2.Requset
我们知道利用 urlopen()方法可以实现最基本请求的发起，但这几个简单的参数并不足以构建一个完整的请求 。 如果请求中需要加入 Headers 等信息，就可以利用更强大的 Request 类来构建
#### Request API
- urllib. request. Request (ur1, data=None, headers={}, origin_req_host=None,unverifiable=False, method=None)
#### 参数说明：
- **url**：待请求的url
- **data**：data参数是可选的。如果要添加该参数，并且如果它是字节流编码格式的内容，即bytes类型，则需要通过bytes()方法转化。另外，如果传递了这个参数，则它的请求方式就不再是GET方式，而POST方式。
- **headers**：一个字典，它就是请求头，我们可以在构造请求时通过headers参数直接构造，也可通过调用请求实例的add_header()法添加。添加请求头最常用的用法就是通过修改User-Agent来伪装浏览器，默认的User-Agent是Python-urllib，我们可通过修改它来伪装浏览器 。比如要伪装火狐浏览器，可以把它设置为：Mozilla/s.o (X11; U; Linux i686)  Gecko/20071127 Firefox/2.0.0.11
- **origin_req_host:** 指请求方的 host 名称或者 IP 地址。
- **unversiable**：表示这个请求是否是无法验证的，默认是 False ，意思就是说用户没有足够权限来选择接收这个请求的结果。 例如，我们请求一个 HTML 文档中的图片，但是我们没有向动抓取图像的权限，这时 unverifiable 的值就是 True 。
- **method**：指示请求使用的方法，比如 GET 、 POST 和 PUT 等。
``` python
# Requset测试代码
import urllib.request

url = 'http://httpbin.org/post'
headers = {'User-Agent': 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)',
           'Host': 'httpbin.org'}
dict = {'name': 'Germey'}
data = bytes(urllib.parse.urlencode(dict), encoding='utf-8')
req = urllib.request.Request(url = url, data = data, headers = headers, method = 'POST')
response = urllib.request.urlopen(req)
print(response.read().decode('utf-8'))
```
**Request测试**
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-13/20751370.jpg)
### 3.高级用法
在上面的过程中，我们虽然可以构造请求，但是对于一些更高级的操作（比如 Cookies 处理 、 代理设置等），我们该怎么办呢？接下来，就需要更强大的工具 Handler 登场了 。 简而言之，我们可以把它理解为各种处理器，有专门处理登录验证的，有处理 Cookies 的，有处理代理设置的 。利用它们，我们几乎可以做到 HTTP请求中所有的事情。
首先，介绍一下 urllib.request模块里的 BaseHandler 类，它是所有其他 Handler 的父类，它提
供了最基本的方法，例如 default_open()、 protocol_request ()等
接下来，就有各种 Ha ndler 子类继承这个 BaseHandler 类，举例如下：
- **HTTPDefaultErrorHandler**：用于处理HTTP响应错误，错误都会抛出 HTTP Error 类型的异常 
- **HTTPRedirectHandler** ： 用于处理重定向 
- **HTTPCookieProcessor** ：用于处理 Cookies 
- **ProxyHandler** ：用于设置代理 ， 默认代理为空 
- **HTTPPasswordMgr** ：用于管理密码，它维护了用户名和密码的表 
- **HTTPBasicAuthHandler** ：用于管理认证，如果一个链接打开时需要认证，那么可以用它来解决认证问题

另一个比较重要的类就是 **OpenerDirector** ，我们可以称为 **Opener** 。 我们之前用过 urlopen()个方法，实际上它就是 urllib 为我们提供的一个 Opener。之前使用的 Request 和 urlopen()相当于类库为你封装好了极其常用的请求方法，利用它们可以完成基本的请求，但是现在不一样了，我们需要实现更高级的功能，所以需要深入一层进行配置，使用更底层的实例来完成操作，所以这里就用到了 Opener 。Opener 可以使用 open()法，返回的类型和 urlopen()如出一辙 。 那么，它和 Handler 有什么关系呢？简而言之，就是**利用 Handler 来构建 Opener**。
``` python
# 高级用法测试代码

# 验证测试
from urllib.request import HTTPPasswordMgrWithDefaultRealm, HTTPBasicAuthHandler, build_opener
from urllib.error import URLError

username = 'username'
password = 'password'
url = 'http://localhost:5000/'
p = HTTPPasswordMgrWithDefaultRealm()
p.add_password(None, url, username, password)
auth_handler = HTTPBasicAuthHandler(p)
opener = build_opener(auth_handler)
try:
    result = opener.open(url)
    html = result.read().decode('utf-8')
    print(html)
except URLError as e:
    print(e.reason)

# 代理测试
from urllib.request import ProxyHandler, build_opener

proxy_handler = ProxyHandler({'http': 'http://127.0.0.1:9743',
                              'https': 'https://127.0.0.1:9743'})
opener = build_opener(proxy_handler)
try:
    response = opener.open('https://www.baidu.com')
    print(response.read().decode('utf8'))
except URLError as e:
    print(e.reason)

# cookies测试
import http.cookiejar, urllib.request

cookie = http.cookiejar.CookieJar()
handler = urllib.request.HTTPCookieProcessor(cookie)
opener = urllib.request.build_opener(handler)
response = opener.open('http://www.baidu.com')
for item in cookie:
    print(item.name + '=' + item.value)
```
## error--异常处理
urllib 的 error 模块定义了由 req u est 模块产生的异常，如果出现了问题， request 模块便会抛出error 模块中定义的异常
- URLError
URLError类来自 urllib 库的 error 模块，它继承自 OS Error 类，是 error 异常模块的基类，由request模块生的异常都可以通过捕获这个类来处理。它具有一个reason属性，即返回错误的原因。
-HTTPError
它是URLError的子类，专门用来处理HTTP请求错误，比如请求失败等。它具有以下三个属性：
code：返回HTTP状态码-404/200等
reason：返回错误信息
headers：返回请求头
``` python 
# URLError测试代码
print('URLError test:')
from urllib import request, error

try:
    response = request.urlopen('http://hellomrrs.top:8086')
    print(response.status)
except error.URLError as e:
    print(e.reason)
try:
    response = request.urlopen('http://hellomrrs.top:8086/111.html')
    print(response.status)
except error.URLError as e:
    print(e.reason)

# HTTPError测试代码
print('HTTPError test:')
from urllib import request, error

try:
    response = request.urlopen('http://hellomrrs.top:8086/111.html')
except error.HTTPError as e:
    print(e.reason, e.code, e.headers, sep='\n')

# 较好的异常处理机制
from urllib import request, error

try:
    response = request.urlopen('http:hellomrrs.top:8086/111.html')
except error.HTTPError as e:
    print(e.reason, e.code, e.headers, sep='\n')
except error.URLError as e:
    print(e.reason)
```
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-13/26601279.jpg)

## parse--解析链接
urllib库里还提供了parse模块，它定义了处理URL的标准接口，例如实现URL各部分的抽取、合并以及链接转换。它支持如下协议的 URL 处理：file、ftp、 gopher、hdl、http、 https、imap 等。下面简单介绍一下常用的接口。


观察以下URL: http://www.baidu.com/index/html;user?id=5#comment，urlparse可将其划分为6个部分：*://--*/--*;--*?--*#--*，即http://--www.baidu.com--index.html--user--id=5--comment，这六个部分分别称为：**scheme(协议)**--**netaloc(域名)**--**path(访问路径)**--**params(参数)**--**query(查询条件)**--**fragment(锚点-用来直接定位页面内部的下拉位置)**。所以，可以得出一个标准的链接格式：
**scheme://netaloc/path;params?query#fragment**
- urlparse(urlstring, scheme=”, allow_fragments=True)：该方法可以实现URL的识别和分段
   - **urlstring**：待解析的URL
   - **scheme**：URL的协议类型，只有在urlstring中没有包含协议类型时才起作用
   - **allow_fragments**：是否忽略fragme，如果它被设置为False，fragment 部分就会被忽略，它会被解析为 path 、 parameters 或者 query 的一部分，而 fragment 部分为空。
- urlunparse(parts)：它接受的是参数是一个可迭代对象，但它的长度必须是6
- urlsplit(urlstring, scheme='', allow_fragments=True)：该方法和urlparse方法十分相似，只不过它不再单独解析params这一部分，只返回5个结果。
- urlunsplit(parts)：与 urlunparse类似，它也是将链接各个部分组合成完整链接的方法，传入的参数也是一个可迭代对象，例如列表 、 元组等，唯一的区别是长度必须为 5。
- urljoin(base, url, allow_fragments=True)：base_url提供了三项内容scheme 、 netloc 和 path 。 如果这 3 项在新的链接里不存在，就予以补充；如果新的链接存在，就使用新的链接的部分。而 base_url中的params、query和fragment是不起作用的，最后返回结果 。
- urlencode(query, doseq=False, safe='', encoding=None, errors=None, quote_via=quote_plus)：在构造GET请求参数时非常有用
- parse_qs(qs, keep_blank_values=False, strict_parsing=False)：反序列化，将一串GET参数转回字典
- parse_qsl(qs, keep_blank_values=False, strict_parsing=False)：将参数转化为元组组成的列表
- quote(str)：将内容转化为URL编码的格式。URL 中带有中文参数时，有时可能会导致乱码的问题，此时用这个方法可以将巾文字符转化为 URL 编码。
- unquote(str)：对URL进行解码
``` python
print('urlparse测试')
from urllib.parse import urlparse

response = urlparse('http://www.baidu.com/index.html;user?id=5#comment')
print(type(response), response, sep='\n')


print('\nurlunparse测试')
from urllib.parse import urlunparse

data = ['http', 'www.baidu.com', 'index.html', 'user', 'a=6', 'comment']
print(urlunparse(data))


print('\nurlsplit测试')
from urllib.parse import urlsplit

response = urlsplit('http://www.baidu.com/index.html;user?id=5#comment')
print(response)


print('\nurlunsplit测试')
from urllib.parse import urlunsplit

data = ['http', 'www.baidu.com', 'index.html', 'a=6', 'comment']
print(urlunsplit(data))


print('\nurljoin测试')
from urllib.parse import urljoin

print(urljoin('http://www.baidu.com', 'FAQ.html'))
print(urljoin('http:''www.baidu.com', 'http://hellomrrs.top:8086/index.html'))


print('\nurlencode测试')
from urllib.parse import urlencode

params = {'name': 'germey', 'age': 22}
base_url = 'http://www.baidu.com?'
url = base_url + urlencode(params)
print(url)


print('\nparse_qs测试')
from urllib.parse import parse_qs

query = 'name=germey&age=22'
print(parse_qs(query))


print('\nparse_qsl测试')
from urllib.parse import parse_qsl

query = 'name=germey&age=22'
print(parse_qsl(query))


print('\nquote测试')
from urllib.parse import quote

keyword = '壁纸'
url = 'http://www.baidu.com/s?wd=' + quote(keyword)
print(url)


print('\nunquote测试')
from urllib.parse import unquote

print(unquote('http://www.baidu.com/s?wd=%E5%A3%81%E7%BA%B8'))
```
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-13/32306346.jpg)
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-13/78219128.jpg)














