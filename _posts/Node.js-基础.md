---
title: Node.js-基础
date: 2020-1-14 14:11:45
tags: Node.js
categories: Node.js
---

# [Buffer](http://nodejs.cn/api/buffer.html)
JS自身只能只有字符串数据类型，没有二进制数据类型，因此nodejs提供了一个与string对等的全局构造函数Buffer来提供对二进制数据的操作
- `Buffer.form` 创建一个buffer
    ``` javascript
    Buffer.form([ 0x68, 0x65, 0x6c, 0x6c, 0x6f ]).toString()
    // hello
    ```
- `Burrer`与`String`的一个重要区别就是字符串时只读的，并且对于字符串的任何修改得到的都是一个新的字符串，源字符串保持不变。至于`Buffer`，更像是可以做指针操作的C语言数组，例如，可以使用`[index]`方式直接修改某个位置的字节

# [Stream](http://nodejs.cn/api/stream.html)
- 当内存中无法一次装下需要处理的数据时，或者一边读取一边处理更加高效时，我们就需要用到数据流。NodeJS中通过各种`Stream`来提供对数据流的操作。
  ``` javascript
  const rs = require('fs');

  function copy (src, dst) {
      fs.createReadStream(src).pipe(fs.createWriteSreeam(dst));
  }

  function main (argv) {
      copy(argv[0], argv[1])
  }

  main(process.argv.slice(2));
  ```
- `Stream`基于事件机制工作，所有`Stream`的实例都继承于NodeJS提供的[`EventEmitter`](http://nodejs.cn/api/events.html#events_class_eventemitter)

# [File System](http://nodejs.cn/api/fs.html)
NodeJS通过`fs`内置模块提供对文件的操作，`fs`模块提供的API基本上可以分为以下三类：
- 文件属性读
  其中常见的有`fs.state` `fs.chmod` `fs.chown`
- 文件属性写
  其中常见的有`fs.readFile` `fs.readdir` `fs.writeFile` `fs.mkdir`
- 底层文件操作
  其中常见的操作有`fs.open` `fs.read` `fs.write` `fs.close`
所有的`fs`模块API的回调参数都有两个，第一个参数在有错误发生时等于异常对象，第二个参数始终用于返回API方法执行结果
  ``` javascript
  fs.readFile(pathname, function (err, data) {
      if (err) {}
      else {}
  })
  ```

## [Path](http://nodejs.cn/api/path.html)
操作文件时难免不与文件路径打交道。NodeJS提供了`path`内置模块来简化路径相关操作，并提升代码可读性。
- `path.normalize`
    将传入的参数转为标准路径，具体讲的话，除了解析路径中的`.`与`..`外，还能去掉多余的斜杠。如果有程序需要使用路径作为某些数据的索引，但又允许用户随意输入路径时，就需要使用该方法保证路径的唯一性。
    ``` javascript
    const cache = {}
    function store () {
        cache[path.normalize(key)] = value
    }

    store('foo/bar', 1);
    store('foo//baz//../bar', 2);
    console.log(cache);  // => { "foo/bar": 2 }
    ```
    标准化之后的路径里的斜杠在Windows系统下是\，而在Linux系统下是/。如果想保证任何系统下都使用/作为路径分隔符的话，需要用`.replace(/\\/g, '/')`再替换一下标准路径。
- `path.join`
    将传入的多个路径拼接为标准路径。该方法可避免手工拼接路径字符串的繁琐，并且能在不同系统下正确使用相应的路径分隔符。
    ``` javascript
    path.join('foo/', 'baz/', '../bar'); // => "foo/bar"
    ```
- `path.extname`
    当我们需要根据不同文件扩展名做不同操作时，该方法就显得很好用。
    ``` javascript
    path.extname('foo/bar.js'); // => ".js"
    ```
## 遍历目录
``` javascript
const fs = require('fs');
const path = require('path');

function travel (dir, callback) {
    fs.readdirSync(dir).forEach(file => {
        let pathname = path.join(dir, file)
        if (fs.statSync(pathname).isDirectory())
            travel(pathname, callback)
        else callback(pathname)
    })
}

travel(__dirname, file => {
    console.log(file)
})

// c:\Users\mrrs8\Desktop\desktop.ini
// c:\Users\mrrs8\Desktop\Postman.lnk
// c:\Users\mrrs8\Desktop\test.js
// c:\Users\mrrs8\Desktop\微信开发者工具.lnk
```
# 文本编码
使用NodeJS编写前端工具时，操作得最多的是文本文件，因此也就涉及到了文件编码的处理问题。我们常用的文本编码有`UTF8`和`GBK`两种，并且`UTF8`文件还可能带有`BOM`。在读取不同编码的文本文件时，需要将文件内容转换为JS使用的`UTF8`编码字符串后才能正常处理。
## BOM的移除
BOM用于标记一个文本文件使用`Unicode`编码，其本身是一个`Unicode`字符`（"\uFEFF"）`，位于文本文件头部。在不同的`Unicode`编码下，`BOM`字符对应的二进制字节如下：
``` javascript
    Bytes      Encoding
----------------------------
    FE FF       UTF16BE
    FF FE       UTF16LE
    EF BB BF    UTF8
```
因此，我们可以根据文本文件头几个字节等于啥来判断文件是否包含`BOM`，以及使用哪种`Unicode`编码。但是，`BOM`字符虽然起到了标记文件编码的作用，其本身却不属于文件内容的一部分，如果读取文本文件时不去掉`BOM`，在某些使用场景下就会有问题。
``` js
// 识别和去除UTF8 BOM
const fs = require('fs');

function readText (pathname) {
    const bin = fs.readFileSync(pathname);

    if (bin[0] === 0xEF && bin[1] === 0xBB && bin[2] === 0xBF) {
        bin = bin.slice(3);
    }

    return bin.toString('utf-8');
}
```
## GBK转UTF8
NodeJS支持在读取文本文件时，或者在`Buffer`转换为字符串时指定文本编码，但遗憾的是，`GBK`编码不在NodeJS自身支持范围内。因此，一般我们借助`iconv-lite`这个三方包来转换编码
``` javascript
// GBK转UTF8
const iconv = require('iconv-lite');

function readGBKText(pathname) {
    const bin = fs.readFileSync(pathname);

    return iconv.decode(bin, 'gbk');
}
```
## 单字节编码
``` javascript
function replace(pathname) {
    const str = fs.readFileSync(pathname, 'binary');
    str = str.replace('foo', 'bar');
    fs.writeFileSync(pathname, str, 'binary');
}
```