---
title: es6中for循环遍历各种使用场景
date: 2024-04-03 10:24:00
tags: es6 js
categories: 前端
---

<!--more-->

#### for循环的各种使用场景

- **forEach**

  - 性能上小于for，for循环直接操作索引，没有额外的函数调用
  - for可以使用break终止，forEach不支持跳出循环，使用return时类似于for的continue，结束当次循环
  - forEach不支持await异步等待

- **map**

  - 对数组遍历不破坏原数组，将会创建一个新数组，按照原始数组元素顺序依次执行给定的函数，map方法非常适合用于处理数组中的每个元素并生成新的数组

  - `map(callbackFn)` callbackFn包含参数(element 数组中当前正在处理的元素, index 索引, array 数组本身)

  ```
    let arrs = [{name:"华为",price:6999},{name:"苹果",price:9888},{name:"小米",price:4999}]
    let newArrs = arrs.map(item=>{
    	return {
    		...item,
    		price:item.price+"元",
    		number:888
    	}
    });
  ```

- **filter**

  - 过滤方法，会对原数组中的每个元素应用指定的函数，并返回一个新数组，其中包含符合条件的元素。原数组不会受到影响。

  - 在filter回调函数中，满足true即可被处理到新函数中，false不做处理。

  - `filter(callbackFn)` callbackFn包含参数(element 数组中当前正在处理的元素, index 索引, array 数组本身)

  ```
    let arrs = [5,7,8,15,22,1,2];
    let newArrs = arrs.filter(item=>{
    	return item>10
    })
  ```

- **reduce**

  - 对数组中的每个元素按序执行一个指定方法，每一次运行 reducer 会将先前元素的计算结果作为参数传入。

  - `reduce(callbackFn, initialValue)` callbackFn包含参数(prev上一次调用 callbackFn 的结果, current当前值, index索引-可选,array 数组本身-可选)，**initialValue** 第一次调用回调函数时初始值

  ```
    let arrs = [1,2,3,4];
    let result = arrs.reduce((prev,current,index)=>{	
    	console.log(prev,current,index);
    	return prev+current;
    },0)
  ```

- **every**

  - 判断数组中所有元素是否满足函数中给定的条件，全部满足返回true,只要有一项不满足则返回false。

  - `every(callbackFn)` callbackFn包含参数(element 数组中当前正在处理的元素, index 索引, array 数组本身)

  ```
    let arrs = [1, 2, 3, 4, 5];
    // 写成两行时记得要return
    let result = arrs.every(num => num > 0);
  ```

- **some**

  - 判断数组，只要有一个满足条件即返回true，全部不满足条件才会返回false。

  - `some(callbackFn)` callbackFn包含参数(element 数组中当前正在处理的元素, index 索引, array 数组本身)

  ```
    let arrs = [55,26,3,12,39];
    let result = arrs.some(item => item < 10);
    console.log(result);
  ```

- **includes**

  - 判断一个数组是否包含一个指定的值，根据情况，如果包含则返回 true，否则返回 false。

  - `includes(searchElement)` searchElement 需要查找的值。

  ```
    const arrs = [1, 2, 3];
    console.log(arrs.includes(2));
  ```

> [!TIP]
> 参考链接
> https://blog.csdn.net/qq_18798149/article/details/135089225