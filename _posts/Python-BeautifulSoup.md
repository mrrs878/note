---
title: Python-BeautifulSoup
date: 2018-11-16 14:50:41
tags: Python BeautifulSoup
categories: Python lib
---
# Python-BeautifulSoup的使用
BeautiSoup，是借助网页的结构和属性等特性来解析网页的工具，有了它我们不用再去写一些复杂的正则，只需要简单的几条语句就可以完成网页中某个元素的提取。
## BeautifulSoup简介
官方解释如下：
BeautifulSoup提供一些简单的、Python式的函数用来处理导航、搜索、修改分析树等功能。它是一个工具箱，通过解析文档为用户提供需要抓取的数据，因为简单，所以不需要多少代码就可以写出一个完整的应用程序。 BeautifulSoup 自动将输入文档转换为 Unicode 编码，输出文档转换为 utf-8 编码。你不需要考虑编码方式，除非文档没有指定一个编码方式，这时你仅仅需要说明一下原始编码方式就可以了。 BeautifulSoup 已成为和 lxml、html6lib 一样出色的 Python 解释器，为用户灵活地提供不同的解析策略或强劲的速度。
## 解析器
BeautifulSoup 在解析的时候实际上是依赖于解析器的，它除了支持 Python 标准库中的 HTML 解析器，还支持一些第三方的解析器比如 LXML，下面我们对 BeautifulSoup 支持的解析器及它们的一些优缺点做一个简单的对比。
![enter image description here](https://t1.picb.cc/uploads/2018/11/16/JXuhZc.png)

## 基本使用
``` python
from bs4 import BeautifulSoup

html = '''
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>
'''

soup = BeautifulSoup(html, 'lxml') 
print(soup.prettify())  # 把要解析的字符串以标准的缩进格式输出(自动更正格式)
print(soup.title.string)


# 输出
<html>
 <head>
  <title>
   The Dormouse's story
  </title>
 </head>
 <body>
  <p class="title" name="dromouse">
   <b>
    The Dormouse's story
   </b>
  </p>
  <p class="story">
   Once upon a time there were three little sisters; and their names were
   <a class="sister" href="http://example.com/elsie" id="link1">
    <!-- Elsie -->
   </a>
   ,
   <a class="sister" href="http://example.com/lacie" id="link2">
    Lacie
   </a>
   and
   <a class="sister" href="http://example.com/tillie" id="link3">
    Tillie
   </a>
   ;
and they lived at the bottom of a well.
  </p>
  <p class="story">
   ...
  </p>
 </body>
</html>
The Dormouse's story
```
 
## 节点选择器
### 选择元素
``` python
soup = BeautifulSoup(html, 'lxml')
print(soup.title)
print(type(soup.title.string))
print(soup.title.string)
print(soup.head)
print(soup.p)  # 只选择第一个出现的p标签


# 输出
<title>The Dormouse's story</title>
<class 'bs4.element.NavigableString'>
The Dormouse's story
<head><title>The Dormouse's story</title></head>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
```

## 提取信息
#### 获取名称
可以利用 name 属性来获取节点的名称。还是以上面的文本为例，我们选取 title 节点，然后调用 name 属性就可以得到节点名称。
``` python
print(soup.title.name)

# 输出
title
```

#### 获取属性
每个节点可能有多个属性，比如 id，class 等等，我们选择到这个节点元素之后，可以调用 attrs 获取所有属性。
``` python
print(soup.p.attrs)
print(soup.p.attrs['name'])
#更加简便的写法
print(soup.p['name'])
print(soup.p['class'])

# 输出
{'class': ['title'], 'name': 'dromouse'}
dromouse
dromouse
['title']
```

#### 获取内容
可以利用 string 属性获取节点元素包含的文本内容。
``` python
print(soup.p.string)

# 输出
The Dormouse's story
```

### 嵌套选择
在上面的例子中我们知道每一个返回结果都是 bs4.element.Tag 类型，它同样可以继续调用节点进行下一步的选择，比如我们获取了 head 节点元素，我们可以继续调用 head 来选取其内部的 head 节点元素。
``` python
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html, 'lxml')
print(soup.head.title)
print(type(soup.head.title))
print(soup.head.title.string)


# 输出
<title>The Dormouse's story</title>
<class 'bs4.element.Tag'>
The Dormouse's story
```

### 关联选择
我们在做选择的时候有时候不能做到一步就可以选择到想要的节点元素，有时候在选择的时候需要先选中某一个节点元素，然后以它为基准再选择它的子节点、父节点、兄弟节点等等。
#### 子节点和子孙节点
选取到了一个节点元素之后，如果想要获取它的**直接子节点**可以调用 contents 属性。 contents 返回的是列表类型， children 返回的是生成器类型。
``` python
print(soup.p.contents)
print(soup.p.children)
print(soup.p.contents)

print(soup.p.children)
for i, child in enumerate(soup.p.children):
    print(i, child)


# 输出
['Once upon a time there were three little sisters; and their names were\n', <a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>, ',\n', <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>, ' and\n', <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>, ';\nand they lived at the bottom of a well.']

0 Once upon a time there were three little sisters; and their names were

1 <a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>
2 ,

3 <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>
4  and

5 <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>
6 ;
and they lived at the bottom of a well.
```

#### 父节点和祖先节点
如果要获取某个节点元素的直接父节点，可以调用 parent 属性，要想获取所有的祖先节点，可以调用 parents 。
``` python
print(soup.a.parent)
print(soup.a.parents)

# 输出
<p class="story">Once upon a time there were three little sisters; and their names were
<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>,
<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a> and
<a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

[(0, <p class="story">Once upon a time there were three little sisters; and their names were
<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>,
<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a> and
<a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>), (1, <body>
<p class="story">Once upon a time there were three little sisters; and their names were
<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>,
<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a> and
<a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>
</body>), (2, <html><head><title>The Dormouse's story</title></head>
<body>
<p class="story">Once upon a time there were three little sisters; and their names were
<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>,
<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a> and
<a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>
</body></html>), (3, <html><head><title>The Dormouse's story</title></head>
<body>
<p class="story">Once upon a time there were three little sisters; and their names were
<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>,
<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a> and
<a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>
</body></html>)]
```

#### 兄弟节点
``` python
from bs4 import BeautifulSoup
html = """
<html>
    <body>
        <p class="story">
            Once upon a time there were three little sisters; and their names were
            <a href="http://example.com/elsie" class="sister" id="link1">
                <span>Elsie</span>
            </a>
            Hello
            <a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> 
            and
            <a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>
            and they lived at the bottom of a well.
        </p>
"""
soup = BeautifulSoup(html, 'lxml')
print('next_sibling', soup.a.next_sibling)
print('prev_sibling', soup.a.previous_sibling)
print('next_siblings', list(enumerate(soup.a.next_sibling)))
print('prev_siblings', list(enumerate(soup.a.previous_sibling)))


# 输出
next_sibling 
            Hello
            
prev_sibling 
            Once upon a time there were three little sisters; and their names were
            
next_siblings [(0, '\n            Hello\n            '), (1, <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>), (2, ' \n            and\n            '), (3, <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>), (4, '\n            and they lived at the bottom of a well.\n        ')]
prev_siblings [(0, '\n            Once upon a time there were three little sisters; and their names were\n            ')]
```

## 方法选择器
前面讲的选择方法都是通过属性来选择元素的，这种选择方法非常快，但是如果要进行比较复杂的选择的话则会比较繁琐，不够灵活。所以 BeautifulSoup 还提供了一些查询的方法，比如 find_all()、find() 等方法，我们可以调用方法然后传入相应等参数就可以灵活地进行查询。

### find_all--查找所有符合条件的元素
``` python
from bs4 import BeautifulSoup
html='''
<div class="panel">
    <div class="panel-heading">
        <h4>Hello</h4>
    </div>
    <div class="panel-body">
        <ul class="list" id="list-1">
            <li class="element">Foo</li>
            <li class="element">Bar</li>
            <li class="element">Jay</li>
        </ul>
        <ul class="list list-small" id="list-2">
            <li class="element">Foo</li>
            <li class="element">Bar</li>
        </ul>
    </div>
</div>
'''
soup = BeautifulSoup(html, 'lxml')
print(soup.find_all(name = 'ul'))
for ul in soup.find_all(name = 'ul'):
    print(ul.find_all(name = 'li'))
    for li in ul.find_all(name = 'li'):
        print(li.string)

# 输出
[<ul class="list" id="list-1">
<li class="element">Foo</li>
<li class="element">Bar</li>
<li class="element">Jay</li>
</ul>, <ul class="list list-small" id="list-2">
<li class="element">Foo</li>
<li class="element">Bar</li>
</ul>]
[<li class="element">Foo</li>, <li class="element">Bar</li>, <li class="element">Jay</li>]
Foo
Bar
Jay
[<li class="element">Foo</li>, <li class="element">Bar</li>]
Foo
Bar
```

### attrs
根据属性进行查询
``` python
print(soup.find_all(attrs = {'id': 'list-1'}))

# 输出
[<ul class="list" id="list-1">
<li class="element">Foo</li>
<li class="element">Bar</li>
<li class="element">Jay</li>
</ul>]
```

### text
text 参数可以用来匹配节点的文本，传入的形式可以是字符串，可以是正则表达式对象。
``` python
import re
html='''
<div class="panel">
    <div class="panel-body">
        <a>Hello, this is a link</a>
        <a>Hello, this is a link, too</a>
    </div>
</div>
'''
from bs4 import BeautifulSoup
soup = BeautifulSoup(html, 'lxml')
print(soup.find_all(text=re.compile('link')))

# 输出
['Hello, this is a link', 'Hello, this is a link, too']
```

### find()
find方法和find_all类似，只是返回的是第一个匹配的元素

### othres API
- find_parents() find_parent()
find_parents() 返回所有祖先节点，find_parent() 返回直接父节点。

- find_next_siblings() find_next_sibling()
find_next_siblings() 返回后面所有兄弟节点，find_next_sibling() 返回后面第一个兄弟节点。

- find_previous_siblings() find_previous_sibling()
find_previous_siblings() 返回前面所有兄弟节点，find_previous_sibling() 返回前面第一个兄弟节点。

- find_all_next() find_next()
find_all_next() 返回节点后所有符合条件的节点, find_next() 返回第一个符合条件的节点。

- find_all_previous() 和 find_previous()
find_all_previous() 返回节点后所有符合条件的节点, find_previous() 返回第一个符合条件的节点

## CSS选择器
BeautifulSoup 还提供了另外一种选择器，那就是 CSS 选择器。使用 CSS 选择器，只需要调用 select() 方法，传入相应的 CSS 选择器即可。
``` python
from bs4 import BeautifulSoup
html='''
<div class="panel">
    <div class="panel-heading">
        <h4>Hello</h4>
    </div>
    <div class="panel-body">
        <ul class="list" id="list-1">
            <li class="element">Foo</li>
            <li class="element">Bar</li>
            <li class="element">Jay</li>
        </ul>
        <ul class="list list-small" id="list-2">
            <li class="element">Foo</li>
            <li class="element">Bar</li>
        </ul>
    </div>
</div>
'''
soup = BeautifulSoup(html, 'lxml')
print(soup.select('.panel .panel-heading'))
print(soup.select('ul li'))
print(soup.select('#list-2 .element'))
print(type(soup.select('ul')[0]))


# 输出
[<div class="panel-heading">
<h4>Hello</h4>
</div>]
[<li class="element">Foo</li>, <li class="element">Bar</li>, <li class="element">Jay</li>, <li class="element">Foo</li>, <li class="element">Bar</li>]
[<li class="element">Foo</li>, <li class="element">Bar</li>]
<class 'bs4.element.Tag'>
```

### 嵌套选择
select() 方法同样支持嵌套选择，例如我们先选择所有 ul 节点，再遍历每个 ul 节点选择其 li 节点。
``` python
for ul in soup.select('ul'):
    print(ul.select('li'))

# 输出
[<li class="element">Foo</li>, <li class="element">Bar</li>, <li class="element">Jay</li>]
[<li class="element">Foo</li>, <li class="element">Bar</li>]
```
### 获取属性
``` python
for ul in soup.select('ul'):
    print(ul['id'])
    print(ul.attrs['id'])

# 输出
list-1
list-1
list-2
list-2
```
### 获取文本
获取文本可以用前面所讲的 string 属性，还有一个方法那就是 get_text()。
``` python
for li in soup.select('li'):
    print('Get Text:', li.get_text())
    print('String:', li.string)

# 输出
Get Text: Foo
String: Foo
Get Text: Bar
String: Bar
Get Text: Jay
String: Jay
Get Text: Foo
String: Foo
Get Text: Bar
String: Bar
``` 




