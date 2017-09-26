---
title: ES6常用语法总结
layout: post
cdn: header-on
date: 2017-08-28 10:59:30
tags:
    - JavaScript
---

ES6现在在项目中使用的频率越来越高了，总结下在ES6中常用的语法。

![ES6](images/0806-ES6.png)

## TL;DR版本
1. 模块：ES6引入了模块系统，可以使用`export`和`import`对各个模块中的对象进行导入导出。
2. 类 & 继承：
3. 解构赋值
4. 箭头函数
5. 新增库函数
6. Let & Const
7. 模板字符串
8. 函数默认值


## ES6模块系统
### 为什么要有模块系统
1. 如果没有模块系统，程序中所有代码冗杂在一起，不能将程序拆分成互相依赖的小文件，再用简单的方法拼装起来，这对开发大型的项目十分不利。
2. 全局命名冲突，没有模块，那么全局就在一个命名空间里，可能会造成命名冲突的问题。
3. 若没有模块，程序中所有代码可能会纠缠在一起，可能会有很高的耦合性，通过模块化的方式，可以有效的解耦合，形成高内聚的模块。

### 关于`export`,`import`与`module.exports`，`require`的区别：
 1.  `module.exports`，`require`是CommonJS模块规范下的模块导入导出方法，而`export`,`import`是ES6模块规范下的模块导入导出方法，两者是不同的概念。
 2. CommonJS模块规范：
	 1. 在CommonJS模块规范下，模块（`module`）是一个对象，`exports`是`module`对象的一个属性，`module.exports`对外提供一个接口。同时还存在一个`exports`变量，被设置为`module.exports`，相当与在每个模块中都有一行`var exports = module.exports;`。我们也可以方便的通过设置`exports.newKey = newValue`增加一个对外接口，但是切记不要为`exports`赋值，这样会切断`exports`和`module.exports`的关系。
	 2. 在一个模块文件的末尾，会返回`module.exports`的结果给`require`函数，因此我们可以在另一个文件中通过`require(a.js)`获取到`a.js`的对外接口。
	 3.  一个模块大概就是这样子。
```
var module = { exports: {} };
var exports = module.exports;

// your code

return module.exports;
```

3. ES6模块规范：
	1. ES6 模块不是对象，通过export命令显式指定输出对外的接口，再通过import命令输入其他模块中的功能。
	2. CommonJS是运行时加载，只有在该文件运行时才能得到得到这个对象。比如： `let { stat, exists, readFile } = require('fs');`，虽然看起来是引入了`fs`模块中的三个方法，但其实内部会先将`require('fs')`赋值给一个`_fs`变量，在程序运行时在分别获取它的`stat, exists, readFile`三个方法，如下：
```
let { stat, exists, readFile } = require('fs');
// 也就是如下方式获取
let _fs = require('fs');
let stat = _fs.stat;
let exists = _fs.exists;
let readfile = _fs.readfile;
```
	3. ES6是编译时加载，比如`import { stat, exists, readFile } from 'fs';` ，这样就会直接在引入这三个变量，不需要向CommonJS那样先将`fs`赋值给一个变量，在分别获取各个变量，更何况ES6中的模块也不是对象。ES6可以直接在编译阶段就完成模块的加载。
	4. `export`命令规定的是对外的接口，必须与模块内部的变量建立一一对应关系。也就是说在使用`export`命令的时候，必须要向外暴露一个别名，且这个名字和内部的某一个变量是有对应关系的，不能直接将一个值暴露出去，同样不能将一个被赋了值的变量（就相当于是一个值）暴露出去。
```
// 报错 → 不能直接export一个值
export 2;

// 报错 → 它本身是一个被赋了值的变量，此时m就是1，不能直接export
var m = 1;
export m;
```
	应该这样写才对：
```
// 写法一
// 这里是暴露了接口m，和1之间建立了联系，即相当于通过m把1暴露出去；
export var m = 1;

// 写法二
// 这里是对象的形式，m相当于别名（接口），相当于这样： {m : m}（对象解构），即将对外接口m和内部变量m建立联系；
var m = 1;
export {m};

// 写法三
var n = 1;
// 这里是显式的为变量n指定别名，虽然这里n是1（被赋了值变量），但是显式的设置它的别名（as）为m，即对外暴露接口m；
export {n as m};
```
	5. export default 命令：假如在写一个库文件，用户肯定是希望上手就能用，肯定未必会先去看一大堆文档，那么怎么让将来库的使用者快速的了解到该库又哪些属性和方法呢？这时就可以使用`export default`暴露出默认的方法和属性，且其他模块加载该模块时，import命令可以为默认的方法指定任意名字。比如：
```
// export.js
export default function () {
  console.log('hello world');
}

// import.js
import saySomething from './export';
```
	在这个export default的例子中，我们可以随便给其命名，（文件名后面的.js后缀也可省略）并且要注意，`import`后不需要加`{}`。
	比较以下两组函数：
```
// 第一组： default.js
export default function a(){  // 导出
	console.log('hello');
}
import a from './default'; // 导入
// 虽然导出时有函数名，但是在导出时会忽略函数的变量名，仍然导出匿名函数。
// 一个模块只能有一个默认输出，因此export default命令只能使用一次。所以，import命令后面才不用加大括号，因为只可能对应一个方法。

// 第二组：normal.js
export function a(){ // 导出
	console.log('hello');
}
import {a} from './normal'; // 导入
// 导出接口a，指向一个函数，在导入时{a}相当于{a:a}，a变量执行导出的a函数。
```

	6. export default其实就是输出一个叫做default的变量或方法，但我们可以任意命名。 

```
// export.js
function a() {
	console.log('hello');
}
export {a as default};
// 等同于
// export default a;

// import.js
import { default as ** } from 'export';
// 等同于
// import ** from 'export';
```

## 类 & 继承

类也是 ES6 一个不可忽视的新特性，虽然很多人都说ES6的类只是一个语法糖，但是相对于 ES5，学习 ES6 的类之后对原型链会有更加清晰的认识。

### 特性

* 本质为对原型链的二次包装
* 类没有提升
* 不能使用字面量定义属性
* 动态继承类的构造方法中 super 优先 this

### 类的定义
```javascript
/* 类不会被提升 */
let puppy = new Animal('puppy'); // => ReferenceError
class Animal {
    constructor(name) {
        this.name = name;
    }
    sleep() {
        console.log(`The ${this.name} is sleeping...`);
    }
    static type() {
        console.log('This is an Animal class.');
    }
}
let puppy = new Animal('puppy');
puppy.sleep();    // => The puppy is sleeping...
/* 实例化后无法访问静态方法 */
puppy.type();     // => TypeError
Animal.type();    // => This is an Animal class.
/* 实例化前无法访问动态方法 */
Animal.sleep();   // => TypeError
/* 类不能重复定义 */
class Animal() {} // => SyntaxError
```
以上我们使用 class 关键字声明了一个名为 Animal 的类。

虽然类的定义中并未要求类名的大小写，但鉴于代码规范，推荐类名的首字母大写。

> 两点注意事项：

* 在类的定义中有一个特殊方法 `constructor()`，该方法名固定，表示该类的构造函数（方法），在类的实例化过程中会被调用（`new Animal('puppy')`）；
* 类中无法像对象一样使用 `prop: value` 或者 `prop = value` 的形式定义一个类的属性，我们只能在类的构造方法或其他方法中使用 `this.prop = value` 的形式为类添加属性。

最后对比一下我们之前是怎样写类的：
```javascript
function Animal(name) {
    this.name = name;
}
Animal.prototype = {
    sleep: function(){
        console.log('The ' + this.name + 'is sleeping...');
    }
};
Animal.type = function() {
    console.log('This is an Animal class.');
}
```
> `class`关键字真真让这一切变得清晰易懂了~

### 类的继承
```javascript
class Programmer extends Animal {
    constructor(name) {
        /* 在 super 方法之前 this 不可用 */
        console.log(this); // => ReferenceError
        super(name);
        console.log(this); // Right!
    }
    
    program() {
        console.log("I'm coding...");
    }
    sleep() {
        console.log('Save all files.');
        console.log('Get into bed.');
        super.sleep();
    }
}
let coder = new Programmer('coder');
coder.program(); // => I'm coding...
coder.sleep();   // => Save all files. => Get into bed. => The coder is sleeping.
```

这里我们使用 `class` 定义了一个类 `Programmer`，使用 `extends` 关键字让该类继承于另一个类 `Animal`。

如果子类有构造方法，那么在子类构造方法中使用 `this` 对象之前必须使用 `super()` 方法运行父类的构造方法以对父类进行初始化。

在子类方法中我们也可以使用 `super` 对象来调用父类上的方法。如示例代码中子类的 `sleep()` 方法：在这里我们重写了父类中的 `sleep()` 方法，添加了两条语句，并在方法末尾使用 `super` 对象调用了父类上的 `sleep()` 方法。

最后我们最后来看一下以前我们是怎么来写继承的：
```javascript
function Programmer(name) {
    Animal.call(this, name);
}
Programmer.prototype = Object.create(Animal.prototype, {
    program: {
        value: function() {
            console.log("I'm coding...");
        }
    },
    sleep: {
        value: function() {
            console.log('Save all files.');
            console.log('Get into bed.');
            Animal.prototype.sleep.apply(this, arguments);
        }
    }
});
Programmer.prototype.constructor = Programmer;
```
 (*゜ー゜*), 相信看到这段代码，应该是一脸懵逼.jpg吧。

## 解构赋值

### 数组解构
```javascript
var [a, ,b] = [1, 2, 3, 4, 5];
console.log(a); // => 1
console.log(b); // => 3
```
数组解构，使用变量声明关键字声明一个形参数组（`[a, , b]`），等号后跟一个待解构目标数组（`[1, 2, 3]`），解构时可以通过留空的方式跳过数组中间的个别元素，但是在形参数组中必须留有相应空位才可以继续解构之后的元素，如果要跳过的元素处于数组末端，则在形参数组中可以不予留空。

### 对象解构
```javascript
var {b, c} = {a: 1, b: 2, c: 3};
console.log(b); // => 2
console.log(c); // => 3
```
对象解构与数组解构大体相同，不过需要注意一点

* 形参对象（{b, c}）的属性或方法名必须与待解构的目标对象中的属性或方法名完全相同才能解构到对应的属性或方法

### 对象匹配解构
```javascript
var example = function() {
    return {a: 1, b: 2, c: 3};
}
var {a: d, b: e, c: f} = example();
console.log(d, e, f); // => 1, 2, 3
```
对象匹配解构是对象解构的一种延伸用法，我们可以在形参对象中使用:来更改解构后的变量名。

### 函数入参解构
```javascript
function example({param: value}) {
    return value;
}
console.log(example({param: 5})); // => 5
```
函数的入参解构也是对象解构的一种延伸用法，我们可以通过改写入参对象目标值为变量名的方式，在函数内部直接获取到入参对象中某个属性或方法的值。

### 函数入参默认值解构
```javascript
function example({x, y, z = 0}) {
    return x + y + z;
}
console.log(example({x: 1, y: 2}));       // => 3
console.log(example({x: 1, y: 2, z: 3})); // => 6
```
这是入参解构的另一种用法，我们可以在入参对象的形参属性或方法中使用等号的方式给入参对象的某些属性或方法设定默认值。

## 箭头函数

箭头函数无疑是 ES6 中一个相当重要的新特性。

### 特性

* 共享父级 this 对象
* 共享父级 arguments
* 不能当做构造函数

### 语法

#### 最简表达式
```javascript
var arr = [1, 2, 3, 4, 5, 6];
// Before
arr.filter(function(v) {
    return v > 3;
});
// After
arr.filter(v => v > 3); // => [4, 5, 6]
```
前后对比很容易理解，可以明显看出箭头函数极大地减少了代码量。

#### 完整语法
```javascript
var arr = [1, 2, 3, 4, 5, 6];
arr.map((v, k, thisArr) => {
    return thisArr.reverse()[k] * v;
})  // => [6, 10, 12, 12, 10, 6]
```
一个简单的首尾相乘的算法，对比最简表达式我们可以发现，函数的前边都省略了 `function` 关键字，但是多个入参时需用括号包裹入参，单个入参是时可省略括号，入参写法保持一致。后面使用胖箭头 `=>` 连接函数名与函数体，函数体的写法保持不变。

#### 函数上下文 this
```javascript
// Before
var obj = {
    arr: [1, 2, 3, 4, 5, 6],
    getMaxPow2: function() {
        var that = this,
            getMax = function() {
                return Math.max.apply({}, that.arr);
            };
        
        return Math.pow(getMax(), 2);
    }
}
// After
var obj = {
    arr: [1, 2, 3, 4, 5, 6],
    getMaxPow2: function() {
        var getMax = () => {
            return Math.max.apply({}, this.arr);
        }
        return Math.pow(getMax(), 2);
    }
}
```
注意看中第 5 行 `var that = this` 这里声明的一个临时变量 `that`。在对象或者原型链中，我们以前经常会写这样一个临时变量，或 `that` 或 `_this`，诸如此类，以达到在一个函数内部访问到父级或者祖先级 `this` 对象的目的。

如今在箭头函数中，函数体内部没有自己的 `this`，默认在其内部调用 `this` 的时候，会自动查找其父级上下文的 `this` 对象（如果父级同样是箭头函数，则会按照作用域链继续向上查找），这无疑方便了许多，我们无需在多余地声明一个临时变量来做这件事了。

*注意：*

* 某些情况下我们可能需要函数有自己的 `this`，例如 `DOM` 事件绑定时事件回调函数中，我们往往需要使用 `this` 来操作当前的 `DOM`，这时候就需要使用传统匿名函数而非箭头函数。
* 在严格模式下，如果箭头函数的上层函数均为箭头函数，那么 `this` 对象将不可用。

> 由于箭头函数没有自己的 this 对象，所以箭头函数不能当做构造函数。

#### 父级函数 `arguments`
我们知道在函数体中有 `arguments` 这样一个伪数组对象，该对象中包含该函数所有的入参（显示入参 + 隐式入参），当函数体中有另外一个函数，并且该函数为箭头函数时，该箭头函数的函数体中可以直接访问父级函数的 `arguments` 对象。
```javascript
function getSum() {
    var example = () => {
        return Array
            .prototype
            .reduce
            .call(arguments, (pre, cur) => pre + cur);
    }
    return example();
}
getSum(1, 2, 3); // => 6
```
> 由于箭头函数本身没有 `arguments` 对象，所以如果他的上层函数都是箭头函数的话，那么 `arguments` 对象将不可用。

*最后再巩固一下箭头函数的语法：*

* 当箭头函数入参只有一个时可以省略入参括号；
* 当入参多余一个或没有入参时必须写括号；
* 当函数体只有一个 return 语句时可以省略函数体的花括号与 return 关键字。

## 新增库函数

ES6 新增了许多（相当多）的库函数，这里只介绍一些比较常用的。

> 多了解一下内建函数与方法有时候可以很方便高效地解决问题。有时候绞尽脑汁写好的一个算法，没准已经有内建函数实现了！而且内建函数经过四海八荒众神的考验，性能一定不错。

### Number
```javascript
Number.EPSILON
Number.isInteger(Infinity); // => false
Number.isNaN('NaN');        // => false
```
首先是 ᶓ 这个常量属性，表示小数的极小值，主要用来判断浮点数计算是否精确，如果计算误差小于该阈值，则可以认为计算结果是正确的。

然后是 `isInteger()` 这个方法用来判断一个数是否为整数，返回布尔值。

最后是 `isNaN()` 用来判断入参是否为 `NaN`。是的，我们再也不用通过 `NaN` 不等于 `NaN` 才能确定一个 `NaN` 就是 `NaN` 这种反人类的逻辑来判断一个 `NaN` 值了！
```javascript
if(NaN !== NaN) {
    console.log("Yes! This is actually the NaN!");
}
```
另外还有两个小改动：两个全局函数 `parseInt()` 与 `parseFloat()` 被移植到 `Number` 中，入参反参保持不变。这样所有数字处理相关的都在 `Number` 对象上嘞！规范多了。

### String
```javascript
'abcde'.includes('cd'); // => true
'abc'.repeat(3);        // => 'abcabcabc'
'abc'.startsWith('a');  // => true
'abc'.endsWith('c');    // => true
```
* `inclueds()` 方法用来判断一个字符串中是否存在指定字符串
* `repeat()` 方法用来重复一个字符串生成一个新的字符串
* `startsWith()` 方法用来判断一个字符串是否以指定字符串开头，可以传入一个整数作为第二个参数，用来设置查找的起点，默认为 0，即从字符串第一位开始查找
* `endsWith()` 与 `startsWith()` 方法相反

### Array
```javascript
Array.from(document.querySelectorAll('*')); // => returns a real array.
[0, 0, 0].fill(7, 1); // => [0, 7, 7]
[1, 2, 3].findIndex(function(x) {
    return x === 2;
}); // => 1
['a', 'b', 'c'].entries(); // => Iterator [0: 'a'], [1: 'b'], [2: 'c']
['a', 'b', 'c'].keys();    // => Iterator 0, 1, 2
['a', 'b', 'c'].values();  // => Iterator 'a', 'b', 'c'
// Before
new Array();        // => []
new Array(4);       // => [,,,]
new Array(4, 5, 6); // => [4, 5, 6]
// After
Array.of();         // => []
Array.of(4);        // => [4]
Array.of(4, 5, 6);  // => [4, 5, 6]
```
首先是 `from()` 方法，该方法可以将一个类数组对象转换成一个真正的数组。还记得我们之前常写的 `Array.prototype.slice.call(arguments)` 吗？现在可以跟他说拜拜了~

之后的 `fill()` 方法，用来填充一个数组，第一个参数为将要被填充到数组中的值，可选第二个参数为填充起始索引（默认为 0），可选第三参数为填充终止索引（默认填充到数组末端）。

`findIndex()` 用来查找指定元素的索引值，入参为函数，函数形参跟 `map()` 方法一致，不多说。最终输出符合该条件的元素的索引值。

`entries()`、`keys()`、`values()` 三个方法各自返回对应键值对、键、值的遍历器，可供循环结构使用。

最后一个新增的 `of()` 方法主要是为了弥补 `Array` 当做构造函数使用时产生的怪异结果。

### Object
```javascript
let target = {
    a: 1,
    b: 3
};
let source = {
    b: 2,
    c: 3
};
Object.assign(target, source); // => { a: 1, b: 2, c: 3}
```
`assign()` 方法用于合并两个对象，不过需要注意的是这种合并是浅拷贝。可能看到这个方法我们还比较陌生，不过了解过 `jQuery` 源码的应该知道 `$.extend()` 这个方法，例如在下面这个粗糙的 `$.ajax()` 模型中的应用：
```javascript
$.ajax = function(opts) {
    var defaultOpts = {
        method: 'GET',
        async: true,
        //...
    };
    opts = $.extend(defaultOpts, opts);
}
```
从这我们可以看到 TC39 也是在慢慢吸收百家所长，努力让 JavaScript 变得更好，更方便开发者的使用。

> Object 新增的特性当然不止这一个 `assign()` 方法，一共增加了十多个新特性，特别是对属性或方法名字面量定义的增强方面，很值得一看，感兴趣的自行查找资料进行了解哈，印象会更深刻！

### Math

Math 对象上同样增加了许多新特性，大部分都是数学计算方法，这里只介绍两个常用的
```javascript
Math.sign(5);     // => +1
Math.sign(0);     // => 0
Math.sign(-5);    // => -1
Math.trunc(4.1);  // => 4
Math.trunc(-4.1); // => -4
```
`sign()` 方法用来判断一个函数的正负，使用与对应返回值如上。

`trunc()` 用来取数值的整数部分，我们之前可能经常使用 `floor()` 方法进行取整操作，不过这个方法有一个问题就是：它本身是向下取整，当被取整值为正数的时候计算结果完全 OK，但是当被取整值为负数的时候： `Math.floor(-4.1); // => -5`。

> 插播一个小 Tip：使用位操作符也可以很方便的进行取整操作，例如：~~3.14 or 3.14 | 0，也许这更加方便。

## Let & Const

### Let

* 无变量提升
```javascript
// Before
console.log(num); // => undefined
var num = 1;
// After
console.log(num); // => ReferenceError
let num = 1;
```
使用 `var` 声明的变量会自动提升到当前作用域的顶部，如果声明位置与作用域顶部之间有另一个同名变量，很容易引起难以预知的错误。使用 `let` 声明的变量则不会进行变成提升，规避了这个隐患。

> 注意：`var` 声明的变量提升后虽然在声明语句之前输出为 `undefined`，但这并不代表 `num` 变量还没有被声明，此时 `num` 变量已经完成声明并分配了相应内存，只不过该变量目前的值为 `undefined`，并不是我们声明语句中赋的初始值 `1`。

* 有块级作用域
```javascript
// Before
{
    var num = 1;
    console.log(num); // => 1
}
console.log(num);     // => 1
// After
{
    let num = 1;
    
    console.log(num); // => 1
}
console.log(num);     // => ReferenceError
```
`let` 声明的变量只能在当前块级作用域中使用，最常见的应用大概就是 `for(let i = 0, i < 10; i++) {}`，相信许多小伙伴在面试题中见过，哈哈。

* 禁止重复声明
```javascript
// Before
var dev = true;
var dev = false;
console.log(dev); // => false
// After
let dev = true;
let dev = false; // => SyntaxError
```
`var` 声明的变量可以重复声明，而且不会有任何警告或者提示，就这样悄悄的覆盖了一个值，隐患如变量提升一样让人担忧。

而 `let` 声明的变量如果进行重复声明，则会直接抛出一个语法错误（是的，就是直接明确地告诉你：你犯了一个相当低级的语法错误）

### Const

* 无变量提升
* 有块级作用域
* 禁止重复声明
前 3 点跟 `let` 一个套路，就不多说了

* 禁止重复赋值
```javascript
const DEV = true;
DEV = false; // => TypeError
```
基于静态常量的定义我们可以很明显知道，`const` 声明的常量一经声明便不能再更改其值，无需多说。

* 必须附初始值
```javascript
const DEV; // => SyntaxError
```
也是基于定义，`const` 声明的常量既然一经声明便不能再更改其值，那声明的时候没有附初始值显然是不合理的，一个没有任何值的常量是没有意义的，浪费内存。

## 模板字符串
### 特性 & 语法
```javascript
// Before
// Before.1
var type = 'simple';
'This is a ' + type + ' string join.'
// Before.2
var type = 'multiline';
'This \nis \na \n' + type + '\nstring.'
// Before.3
var type = 'pretty singleline';
'This \
is \
a \
' + type + '\
string.'
// OR
// Before.4
'This ' +
'is' +
'a' +
type +
'string.'
```
```javascript
// After
var type = 'singleline';
`This is a ${type} string.`
var type = 'multiline';
`This
is
a
${type}
string.`
var type = 'pretty singleline';
`This \
is \
a \
${type} \
string.`
```
我们之前在对字符串和变量进行拼接的时候，通常都是反复一段一段使用引号包裹的字符串，再反复使用加号进行拼接（Before.1）。多行字符串的时候我们还要写上蹩脚的 `\n` 来换行以得到一个多行的字符串（Before.2）。

在字符串过长的时候可能会使用 `\` 在编辑器中书写多行字符串来表示单行字符串，用来方便较长的字符串在编辑器中的阅读（Before.3），或者简单粗暴的反复引号加号这样多行拼接（Before.4）。

ES6 中我们可以使用反引号（`，位于 TAB 上方）来输入一段简单明了的多行字符串，还可以在字符串中通过 ${变量名} 的形式方便地插入一个变量，是不是方便多了！

## 函数默认值
### 特性 & 语法
```javascript
// Before
function decimal(num, fix) {
    fix = fix === void(0) ? 2 : fix;
    return +num.toFixed(fix);
}
```
```javascript
// After
function decimal(num, fix = 2) {
    return +num.toFixed(fix);
}
```
首先，我们看一下之前我们是怎么写函数默认值的：我们通常会使用三元运算符来判断入参是否有值，然后决定是否使用默认值运行函数（如示例中 `fix = fix === void(0) ? 2 : fix`）

而在 ES6 中，我们可以直接在函数的显示入参中指定函数默认值（`function decimal(num, fix = 2){}`），很明显，这种写法更自然易懂，也更加方便，不过有一点需要注意：

> 设定了默认值的入参，应该放在没有设置默认值的参数之后，也就是我们不应该这样写：`function decimal(fix = 2, num){}`，虽然通过变通手段也可以正常运行，但不符合规范。

ES6的八个特性这里就先总结完了，当然这里并不是ES6的全部特性，是大部分使用频率较高的语法，如果想要继续深入学习ES6，可以在mdn上找到更加详细的内容。

若有问题，欢迎留言交流！~

END! 💯