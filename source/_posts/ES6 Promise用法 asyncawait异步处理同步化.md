---
title: ES6 Promise用法 asyncawait异步处理同步化
date: 2024-04-03 14:49:20
tags: es6
categories: 前端
---

<!--more-->

### promise定义

> promise是解决异步的方法，本质上是一个构造函数，可以用它实例化一个对象。对象身上有resolve、reject、all，原型上有then、catch方法。promise对象有三种状态：pending（初识状态/进行中）、resolved或fulfilled（成功）、rejected（失败）

- **pending** 表示"待定的，将发生的"，相当于是一个初始状态。创建Promise对象时，且没有调用resolve或者是reject方法，相当于是初始状态。这个初始状态会随着你调用resolve，或者是reject函数而切换到另一种状态。
- **resolved** 表示解决了，就是说这个承诺实现了。 要实现从pending到resolved的转变，需要在 创建Promise对象时，在函数体中调用了resolve方法。
- **rejected** 表示”拒绝，失败“。表示这个承诺没有做到，失败了。要实现从pending到rejected的转换，只需要在创建Promise对象时，调用reject函数。

### 以uniapp中请求为例

- **uni.request** 多重嵌套的请求

```js
getData(){
  //获取分类列表id
  uni.request({
    url:"https://ku.qingnian8.com/dataApi/news/navlist.php",
    success:res=>{
      let id=res.data[0].id
      // 根据分类id获取该分类下的所有文章
      uni.request({
        url:"https://ku.qingnian8.com/dataApi/news/newslist.php",
        data:{
          cid:id
        },
        success:res2=>{
          //获取到一篇文章的id，根据文章id找到该文章下的评论
          let id=res2.data[0].id;
          uni.request({
            url:"https://ku.qingnian8.com/dataApi/news/comment.php",
            data:{
              aid:id
            },
            success:res3=>{
              //找到该文章下所有的评论
              console.log(res3)
            }
          })
        }
      })
    }
  })
}
```

- 封装成**promise**对象

```js
methods: {
    //先获取导航分类接口，将结果进行返回，到调用函数的地方获取
    getNav(callback){
      return new Promise((resolve,reject)=>{
        uni.request({
          url:"https://ku.qingnian8.com/dataApi/news/navlist.php",
          success:res=>{
            resolve(res)
          },
          fail:err=>{
            reject(err)
          }
        })
      })
    },


    //获取文章数据，将文章列表进行返回
    getArticle(id){
      return new Promise((resolve,reject)=>{
        uni.request({
          url:"https://ku.qingnian8.com/dataApi/news/newslist.php",
          data:{
            cid:id
          },
          success:res=>{
            resolve(res)
          },
          fail:err=>{
            reject(err)
          }
        })
      })
    },

      //获取文章下的所有评论
      getComment(id){
        return new Promise((resolve,reject)=>{
          uni.request({
            url:"https://ku.qingnian8.com/dataApi/news/comment.php",
            data:{
              aid:id
            },
            success:res=>{
              resolve(res)
            },
            fail:err=>{
              reject(err)
            }
          })
        })
} 
```

- **promise** 用 **then** 方式的链式调用

```js
//promise链式调用
this.getNav().then(res=>{
  let id=res.data[0].id;
  return this.getArticle(id);
}).then(res=>{
  let id=res.data[0].id;
  return this.getComment(id)
}).then(res=>{
  console.log(res)
})
```

- **await / async** 异步处理同步化 

> onload是函数，这个函数必须有async命令，在调用函数的部分，前面都加了一个await，这个命令的意思就是等这一行的异步方法执行成功后，将返回的值赋值给res变量，然后才能再走下一行代码，这就是将原来的异步编程改为了同步编程，这就是标题提到的“异步处理，同步化” 

```js
async onLoad() {
  let id,res;
  res=await this.getNav();
  id=res.data[0].id;
  res=await this.getArticle(id);
  id=res.data[0].id;
  res=await this.getComment(id);
  console.log(res)
}
```

> [!TIP]
> 
> [ES6 Promise的用法，async/await异步处理同步化 文章](https://www.bilibili.com/read/cv18799030/)
>
> [ES6 Promise的用法，ES7 async/await异步处理同步化，异步处理进化史 视频](https://www.bilibili.com/video/BV1XW4y1v7Md/)

