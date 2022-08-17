---
title: 微信小程序[object,object]这样的数据如何打印出来 方便调试~
date: 2020-04-15 09:41:27
tags: javascript 微信游戏 小程序
categories: wechat 小程序
---

<!--more-->

> 你肯定会遇到过打印json数据或者object类型的数据的时候，看不到数据内容的情况，那么你可以往下看。

先上接口获取数据的相关代码

```
// 获取社保缴费年份列表. 参数为被查询人的id
  insurance_YearInfo(userId) {
    var that = this
    api.post({
      url: 'wxapp/chaxun/get_detail_year',
      data: {
        userid: userId,
        account:that.data.idCard,
      },
      success: data => {
        if (data.code == 1) { 
         console.log(data);
         console.log('get_detail_year接口中data数据如下'+data);
         console.log('get_detail_year接口中data数据如下',data);
          
        for(var item in data.data){
          data.data[item].isHidden = true;
          data.data[item].yearisClick = false;
        }
          console.log(data);

          this.setData({ // 设置数据到UI上
            insurance_yearList: data.data
          });


        } else {
          // 隐藏加载框
          // wx.hideLoading();
          api.show_toast(data.msg, 2000);
        }

      },
      fail: err => {

      }
    });
  },
```

打印结果如下

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8yMzY0OTQwLTZhNzUxMzVkY2IxZDAzOTAucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvMTIwMC9mb3JtYXQvd2VicA?x-oss-process=image/format,png)

 

第1个log的弊端:虽然能看到数据的内容，但是前面没有文字描述，当有很多log要打印时候，哪个log是自己需要的log需要找很长时间。

第2个log的弊端:虽然前面有文字描述具体是哪个接口的log，但是数据的内容展示不出来。`（+ 的方式）`

第3个log：既可以看到是哪个接口的log，而且也看到数据的内容。`（,的方式）`

## 总结：小程序开发过程中，完全可以使用第3个log的方式 打印你的log

  
  
作者：CoderZb  
链接：https://www.jianshu.com/p/342478e4bf54  
来源：简书