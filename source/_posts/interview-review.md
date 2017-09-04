---
title: 面试的一点总结
layout: post
cdn: header-on
date: 2017-03-16 18:48:11
tags:
    - Javascript
    - Interview
---

昨天去参加了一次面试，在做面试题的过程中，真的是有够丢人的，js方面的题错了许多，还犯下了一次比较严重的错误，就是关于js中的引用和复制问题，确实是很不应该的，虽然事后自己想了起来，但这足以说明自己的基础掌握的还不够扎实，不知最后结果如何，都要继续加油，抓住语言本质，沉得下去，简单记录下这次自己所吸取的经验。

## js中的引用和复制
```javascript
let a = {
    name: 'Mike',
    age: 24
};
let b = a;
b.name = 'Trans';
console.log(a.name); //Trans
```
这里a是一个对象，在向b赋值的时候，实际上赋的值是a的引用，我认为可以理解为C语言中的地址，既然将a的地址赋值给b，那么当我们对b进行修改的时候，b理所当然的会修改地址所指向的值，同时这个地址也是a的地址，即a也会被改变，因此当我们输入`a.name`的时候，输出的是已经改变过的值。

那么js中几种数据类型分别传的什么呢，如下：
* 引用：对象、数组、函数
* 复制：数字、布尔


[这篇文章](http://blog.csdn.net/zzzaquarius/article/details/4902235)给了很大的启发。

## `Alert(1&&2);`
结果是2，这道题考查的是关于逻辑运算符的正确使用和其原则。

`||`和`&&`都遵循短路原则，即:

`||`：只要左面值为true，那么无论右面的值是什么，最终的结果都为true，因此只要左边的值为true，则最终的结果就是*左边*的值。

若左边的值为false，那么最终的值就由右边的值决定，右边为true则为true，右边为false则为false，因此只要左边的值为false，则最终的结果就是*右边*的值。

`&&`：只要左边的值为true，则最终的结果为右边的值，因为当左边值为true时，右边的值决定了最后的结果。

若左边的值为false，则最终的结果为左边的值，因为左边的值已经是false了，则最后结果就是false，即为左边的值。

## 字符串逆序
```javascript
function reverseString(str){
    return Array.prototype.slice.call(str).reverse().join('');
}
let a = '123456';
reverseString(a); // 654321
```

## 字符串去空格
```javascript
function trimString(str){
    return Array.prototype.slice.call(str).filter((char) => char != ' ').join('');
}
let a = ' 1 2 3 4 5 6 ';
trimString(a); // 123456
```
同样类似思想的方法就可以使用来去掉字符串中的'a'或者'aa'等，如下：

## 去掉字符串中的指定字符
```javascript
function delSpecOfString(str,delchar){
    return Array.prototype.slice.call(str).filter((char) => char != delchar).join('');
}
let a = 'helalao woaralad!';
delSpecOfString(a,'a'); // hello world!
```

## 去掉字符串中的指定字符串
```javascript
function delSubstrOfString(str,delstr){
    return str.split(delstr).join('');
}
let a = 'heaallaao woaarlaad!';
delSubstrOfString(a,'aa'); // hello world!
```

OK，就这样了，正如昨日面试官老师所说，先码他个三五七行的，撸起袖子，就是干。📘
