---
title: Ajax的使用
date: 2020-06-19 17:05:05
tags: ajax js
categories: 前端
---

<!--more-->

## ajax

ajax一个前后台配合的技术，它可以让javascript发送http请求，与后台通信，获取数据和信息。ajax技术的原理是实例化xmlhttp对象，使用此对象与后台通信。jquery将它封装成了一个函数\$.ajax\(\)，我们可以直接用这个函数来执行ajax请求。

ajax需要在服务器环境下运行。

\$.ajax使用方法

常用参数：  
1、url 请求地址  
2、type 请求方式，默认是'GET'，常用的还有'POST'  
3、dataType 设置返回的数据格式，常用的是'json'格式，也可以设置为'text'或者'html'  
4、data 设置发送给服务器的数据  
5、success 设置请求成功后的回调函数  
6、error 设置请求失败后的回调函数  
7、async 设置是否异步，默认值是'true'，表示异步

以前的写法：

```javascript
$.ajax({
    url: '/change_data',
    type: 'GET',
    dataType: 'json',
    data:{'code':10086}
    success:function(dat){
        alert(dat.name);
    },
    error:function(){
        alert('服务器超时，请重试！');
    }
});
```

新的写法\(推荐\)：

```javascript
$.ajax({
    url: '/change_data',
    type: 'GET',
    dataType: 'json',
    data:{'code':10086}
})
.done(function(dat) {
    alert(dat.name);
})
.fail(function() {
    alert('服务器超时，请重试！');
});
```

\$.ajax的简写方式

\$.ajax按照请求方式可以简写成\$.get或者\$.post方式

```javascript
$.get("/change_data", {'code':10086},
  function(dat){
    alert(dat.name);
});

$.post("/change_data", {'code':10086},
  function(dat){
    alert(dat.name);
});
```

 

## **数据接口**

数据接口是后台程序提供的，它是一个url地址，访问这个地址，会对数据进行增、删、改、查的操作，最终会返回json格式的数据或者操作信息，格式也可以是text、xml等。

## **同步和异步**

现实生活中，同步指的是同时做几件事情，异步指的是做完一件事后再做另外一件事，程序中的同步和异步是把现实生活中的概念对调，也就是程序中的异步指的是现实生活中的同步，程序中的同步指的是现实生活中的异步。

## **局部刷新和无刷新**

ajax可以实现局部刷新，也叫做无刷新，无刷新指的是整个页面不刷新，只是局部刷新，ajax可以自己发送http请求，不用通过浏览器的地址栏，所以页面整体不会刷新，ajax获取到后台数据，更新页面显示数据的部分，就做到了页面局部刷新。