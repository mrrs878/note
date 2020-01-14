---
title: Java-IO
date: 2018-12-03 21:03:45
tags: Java IO
---

# Java-流

## **概念**

流：程序与文件|数组|网络连接|数据库等进行交互的通道，以程序为中心

### **IO 流分类**

#### **流向**

- 输入流
- 输出流

#### **数据**

- 字节流----二进制，可以包含一切文件
- 字符流----文本文件，只能处理纯文本

#### **功能**

- 节点流----包裹源头
- 处理流----增强功能，提高性能

### **字符流与字节流**

#### **字节流**

- 输入流：InputStream/FileInputStream

```java
	read(byte[] b);
  	read(byte[] b, int off, int len);
  	close();
```

- 输出流：OutputStream/FileOutputStream

```java
	write(byte[] b);
  	write(byte[] b, int off, int len);
  	flush();
  	close();
```

#### **字符流**

- 输入流：Reader/FileReader

```java
	read(char[] cbuf);
	read(char[] cbuf, int off, int len);
	close();
```

- 输出流：Writer/FileWriter

```java
	write(char[] cbuf);
	write(char[] cbuf, int off, int len);
	append(char c);
	flush();
	close();
```

### **经典步骤**

- 建立联系
- 选择流
- 操作
- 释放资源

# **字节流--Input/OutputStream、FileInputOutputStream**

字节流可以处理伊西起文件，包括二进制文件/音频/视频等

## **读取文件**

- 建立联系-->File 对象--源头
- 选择流-->文件输入流--InputStream/FileInputStream
- 操作-->read + 输出
- 释放资源-->close

## **写入文件**

- 建立联系-->File 对象--源头、目的地
- 选择流-->文件输出流--OutputStream/FileOutputStream
- 操作-->write + flush
- 释放资源-->close

## **文件复制**

- 建立联系-->File 对象--源头、目的地
- 选择流-->Input/OutputStream、FileInputOutputStream
- 操作

```java
	byte[] flush = new byte[1024];
	int len = 0;
	while((len = 输入流.read(byte)) != -1)
		输出流.write(flush, 0, len);
	输出流.flush;
```

- 释放资源

## **文件夹复制**

- 递归查找子孙文件/文件夹
- 文件复制(IO 流复制)、文件夹创建

# **字符流--Reader/Writer、FileReader/FileWriter**

只能操作纯文本数据

## **纯文本读取**

- 建立联系-->File 对象--源头
- 选择流-->字符输出流--Reader/FileReader
- 操作-->读取
- 释放资源-->close

## **纯文本写入**

- 建立联系-->File 对象--目的地
- 选择流-->字符输入流--Writer/FileReader
- 操作-->write + flush
- 关闭-->close

# **处理流**

用来增强功能、提高性能，处于节点流之上

## **缓冲流**

- 字节缓冲流--BufferedInputStream/BufferedOutputStream
- 字符缓冲流--BufferedReader/BufferedWriter

## **转换流**

字节流转换为字符流，处理乱码

### **编码与解码**

- 编码：字符--编码字符集-->二进制
- 解码：二进制--解码字符集-->字符

### **乱码**

- 编码与解码的字符集不一致
- 字节缺少，长度丢失
- 解决办法：

```java
	InputStreamReader(字节输入流, "解码集");
	OutputStreamWriter(字符输出流, "编码集");
```

![](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-12-3/44267583.jpg)
![](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-12-3/88652044.jpg)
