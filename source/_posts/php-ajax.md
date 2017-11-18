---
title: php-ajax
date: 2017-04-08 11:42:02
tags:
    - ajax
    - php
categories: web
copyright: true
---
**前言**
日常我们最常见的就是百度搜索，输入的时候，还没输入完，搜索结果已经出现了，并且会随着你的输入而改变.其实这就是ajax
<!--more-->

第一次需要Ajax的时候还不知道它叫ajax，那时候只想实现系统的一个功能，选中主选项后，剩余的子选项会随之而变，从而对选择结果有个过滤，然后就查到了AJAX。


#### 什么是AJAX
> 简单来说就是异步传输
专业点：Asynchronous JavaScript and XML（异步的 JavaScript 和 XML）。
这里又引出了个新概念，同步和异步

##### 同步和异步
> 专业的知识也就不说了，毕竟也没深究大概解释下同步和异步的概念，
我们打电话是同步，而发消息是异步。
同步交互：发出一个请求后，要等待服务器响应结束，才能发第二个。
异步交互：发出一个请求后，无需等待服务器响应结束，就可发出第二个。

#### Ajax原理
> Ajax的原理简单来说通过XmlHttpRequest对象来向服务器发异步请求，从服务器获得数据，然后用javascript来操作DOM而更新页面。
这其中最关键的一步就是从服务器获得请求数据。要清楚这个过程和原理，我们必须对 XMLHttpRequest有所了解。
XMLHttpRequest是ajax的核心机制，它是在IE5中首先引入的，是一种支持异步请求的技术。简单的说，也就是javascript可以及时向服务器提出请求和处理响应，而不阻塞用户。达到无刷新的效果。
所以我们先了解下XMLHttpRequest

##### XMLHttpRequest

| 属性| 解释| 
| ------------- |:-------------| 
| onreadystatechange     | 每次状态改变所触发事件的事件处理程序| 
| responseText      | 服务器进程返回数据的字符串形式     | 
| responseXML  | 服务器进程返回的DOM兼容的文档数据对象|  
|status | 服务器返回的数字代码，比如常见的404（未找到）200（已就绪）|
|status Text   |    伴随状态码的字符串信息|
|readyState   |    对象状态值|

###### readyState
|数值|含义|
| :-------------: |:-------------| 
|0 |(未初始化) 对象已建立，但是尚未初始化（尚未调用open方法）|
|1| (初始化) 对象已建立，尚未调用send方法|
|2| (发送数据) send方法已调用，但是当前的状态及http头未知|
|3| (数据传送中) 已接收部分数据，因为响应及http头不全，这时通过responseBody和responseText获取部分数据会出现错误|
|4 |(完成) 数据接收完毕,此时可以通过通过responseXml和responseText获取完整的回应数据|
> 很明显，当readyState返回值为4的时候，证明是程序是正常走完的。

#### Ajax的实现

**由于各浏览器之间存在差异，所以创建一个XMLHttpRequest对象可能需要不同的方法。下面是一个当初学习的笔记，算是个比较标准的创建XMLHttpRequest对象的方法。**
```
ajax.js 后面还有两个页面程序

    function getXmlHttpObject(){
        var xmlhttp;
        if(window.XMLHttpRequest){
            //code for  IE7+, Firefox, Chrome, Opera, Safari
            xmlhttp=new XMLHttpRequest();
        }else{
            //code for IE6, IE5
             xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
        }
        return xmlhttp;
    }

 //验证
    var myXmlHttpRequest="";
    function checkName(){
        myXmlHttpRequest=getXmlHttpObject();
        //通过ajax对象发送请求
        if (myXmlHttpRequest){
            //get 方式
            var url="tb.php?username="+document.getElementById('username').value;
            myXmlHttpRequest.open("get",url,true);
            //第一个参数：表示提交数据方式, 即post还是get
            //第二个参数：请求的url地址和传递的参数
            //第三个参数：传输方式，false为同步，true为异步。默认为true。

           //指定回调函数也就是chuli  由于会实验两次使用，所以封装了函数调用
           myXmlHttpRequest.onreadystatechange=chuli;
           真的发送请求,如果是get请求则填入空，如果是post填写实际数据
           myXmlHttpRequest.send(null);    


            //psot方式

            var url="tb.php";
            var data="username="+document.getElementById('username').value;
            myXmlHttpRequest.open("POST",url,true);
            //必须要的一句话,定义传输的文件HTTP头信息
            myXmlHttpRequest.setRequestHeader("Content-Type","application/x-www-form-urlencoded");
            //回调函数
            myXmlHttpRequest.onreadystatechange=chuli;
            // 发送
            myXmlHttpRequest.send(data);
        }
    }

    function chuli(){
        //window.alert("chuli 被调用"+myXmlHttpRequest.readyState)
        if(myXmlHttpRequest.readyState==4){
            //window.alert('服务器返回的是：'+myXmlHttpRequest.responseText);
            document.getElementById('tishi').value=myXmlHttpRequest.responseText;
        }    
    }
```

```
ta.php（注册界面）

<html>
<head>
<title>regist</title>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<script  src='ajax.js' charset='gbk' ></script>
</head>
<body>
<h4>账户注册</h4>

    <form action="#" method="post">  
        账户：<input type="text" name="username" onkeyup="checkName()" id="username"/>
        <input type="button" value="验证" onclick="checkName()" />
        <input type="text" style="border:0" id="tishi" /> 
        <br/> <br/>  
        密码：<input type="password" name="password"/>  
        <br/>  <br/> 
        邮箱：<input type="email" name="email"/>  
        <br/>  <br/> 
        <input type="Submit" name="regist" value="注册"/>  
    </form>  
</body>
</html>
```

```
tb.php（输出页面）
<?php 
//$username=$_GET['username'];
$username=$_POST['username'];

if($username==someone){
    echo $username."用户已存在";
}else{
    echo $username."用户可用";
}
```

一个简单的案例就完成了，帮助自己理解Ajax
___
