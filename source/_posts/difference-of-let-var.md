---
title: JavaScript中变量的作用域小结
layout: post
cdn: header-on
date: 2017-09-26 18:23:56
tags:
    - JavaScript
---

有这么一个问题：
```JavaScript
function count() {
    var arr = [];
    for (var i = 1; i <= 3; i++) {
        arr.push(function () {
            return i * i;
        });
    }
    return arr;
}

var results = count();
for (let j of results) {
    console.log(j());
}
```
运行结果是3个16。但是我们若将代码改为这样子：
```JavaScript
function bb_test() {
    let arr = [];
    for (let i = 0; i < 4; i++) {
        arr.push(function () {
            return i * i;

        });
    }
    return arr;
}

let arr = bb_test();
for (let i of arr) {
    console.log(i());
}
```
运行结果是0、1、4、9，是我们所期望的结果。
我尝试着就这个问题解释一下为什么用`var`和`let`得到不同的结果。
1. 用`var`，`var`声明的变量的作用域是整个封闭函数，没有块级作用域。
```
function count() {
    var arr = [];
    for (var i = 1; i <= 3; i++) {
        arr.push(function () {
            return i * i;
        });
    }
    return arr;
}
```
这个函数`count`只有一个作用域，就是`count`函数本身，因此在这个函数作用域内，`i`都不会被销毁，`i`的值均存在。如你所说你试了在外界是不能调用`i`的，那是因为在外界这个函数执行完毕后，该函数作用域就会结束，`i`的值就会被销毁，所以无法调用`i`的值。
为什么返回结果是3个16？因为，在这个函数作用域内，我们每一次`push`进去的是一个函数，在`count`函数中的`for`循环结束后，此时`arr`是这样的：
```
arr = [
  function () {return i * i;},
  function () {return i * i;},
  function () {return i * i;}
];
```
此时由于函数还未执行结束，作用域还在，`i`的作用域还在，并不会立即算出结果，在`for`的下一次循环，先`i++`得到`i=4`，判定`i<=3`为`false`，所以结束此这个循环（这时在这个函数作用域内`i`的值为`4`），接下来`return arr;`，在`return`之后，`count`函数就执行完毕了，`count`函数的函数作用域也会消失，即`i`也会立即消失，但我们将`arr`返回了，所以`count`函数执行后的`arr`相当于：
```
arr = [
  function () {return 4 * 4;},
  function () {return 4 * 4;},
  function () {return 4 * 4;}
];
```
所以我们在
```
var results = count();
for (let j of results) {
    console.log(j());
}
```
时会直接返回`16 16 16`。
2. 用`let`，`let`声明的变量只在其声明的块或子块中可用，块级作用域可以理解为`{}`，一对大括号中是一个作用域。
```
function let_test() {
    let arr = [];
    for (let i = 0; i < 4; i++) {
        arr.push(function () {
            return i * i;
        });
    }
    return arr;
}
```
观察函数` let_test`，会发现这里有二个块级作用域，且`i`只存在于*块级作用域2*下。如下：
```
function let_test() {
    // 块级作用域1
    for (let i = 0; i < 4; i++) {
        // 块级作用域2
        arr.push(function () {
            return i * i;
        });
    }
    return arr;
}
```
而`let`声明的i的作用域为自己所在的块和其子块中可用，且每一次结束其块级作用域均会清除该值，因此在执行函数内部的循环时，会先进入*块级作用域2*，然后`push`到数组。注意，就下来就会离开*块级作用域2*进入下一次循环了，因此在离开*块级作用域2*的时候，`i`的值会被立即结算并释放。此时`arr`的值就相当于：
```
arr = [
  function () {return 0 * 0;},
  function () {return 1 * 1;},
  function () {return 2 * 2;},
  function () {return 3 * 3;}
];
```
然后就会将该数组返回。
所以在执行：
```
let arr = let_test();
for (let i of arr) {
    console.log(i());
}
```
的时候，就是输出`0 1 4 9`。
💯
