---
title: 【译】机器学习与JavaScript（一）
layout: post
cdn: header-on
date: 2017-07-05 23:50:10
tags:
    - Javascript
    - 翻译
---

> 本文首发于**前端之巅**公众号：【[点此查看](https://mp.weixin.qq.com/s?__biz=MzIwNjQwMzUwMQ==&mid=2247485287&idx=1&sn=2896e4b549ebc51da46e48e4f6e27786&chksm=972365a5a054ecb383a7fc1e4f93b833b7ba4c1b1028384f9da6a2d6ea38acd108f6a3c4836e&scene=38#wechat_redirect)】

你应该觉得基于JavaScript的机器学习不简单吧。

![采用plot.ly生成的现行回归图](https://cdn-images-1.medium.com/max/1600/1*Fhc-ZFRp1ZRZ2SkSdkYvJA.png)

JAVASCRIPT？！我难道不应该用Python么？我难道要用JavaScript去做如此复杂的运算？难道我不应该使用Python或者R语言么？scikit-learn算法库会不会不能在JavaScript中使用？简单来说：基于JavaScript的机器学习完全没有问题。

详细来讲，基于JavaScript的机器学习是有可能的，并且我总是很吃惊为什么开发者们没有给予它应有的关注。就scikit-learn算法库而言，JavaScript开发者已经开发了一系列实现该算法的库，一会儿就会用到一个库。接下来会先讲一点机器学习的知识，然后就放松心情一起来看代码吧。

![unsplash.com](https://cdn-images-1.medium.com/max/1600/1*mwczhqPN-RbSEXPv-ChhWg.jpeg)

据Arthur Samuel所讲，**机器学习**就是在不对其进行具体编程的情况下，使计算机拥有学习的能力。换句话说，它在我们不操作计算机的情况下，却能拥有自我学习的能力，并能执行正确的指令。并且谷歌公司已经将策略从移动优先转变为AI优先很长一段时间了。

## 为什么在机器学习领域没有提到JavaScript呢？
1. JavaScript很慢。（完全错误的观念 !?! ）
2. JavaScript很难进行矩阵操作。（但是有很多库的，比如`math.js` ）
3. JavaScript仅仅被认为是用来做web开发的。（**`Node.js`**默默的笑了）
4. 机器学习中很多库都是基于Python开发的。（那是因为JavaScript开发者并没有在场）

现在已经有很多的JavaScript库了，它们已经预定义了机器学习算法，比如：线性回归、支持向量机、朴素贝叶斯算法等，以下列出了几个库：

1. [brain.js](https://github.com/harthur-org/brain.js)（神经网络）
2. [Synaptic](https://github.com/cazala/synaptic)（神经网络）
3. [Natural](https://github.com/NaturalNode/natural)（自然语言处理）
4. [ConvNetJS](http://cs.stanford.edu/people/karpathy/convnetjs/)（卷积神经网络）
5. [mljs](https://github.com/mljs)（一种具有多个函数方法的子库）

我将使用`mljs`的回归库来执行线性回归模型的分析。全部代码都在Github上：[machine-learning-with-js](https://github.com/abhisheksoni27/machine-learning-with-js)。

**第一步.** 安装依赖的库

`$ yarn add ml-regression csvtojson`

或者你更喜欢npm：

`$ npm install ml-regression csvtojson`

`ml-regression`所做的事正如它的名字那样，机器学习线性回归库。

`csvtojson`是在`node.js`环境中的一个`cvs`数据解析器，它可以在你加载完`cvs数据`后将其快速的转换为`JSON`。

**第二步.** 初始化依赖库并加载数据

首先从[这里](http://www-bcf.usc.edu/~gareth/ISL/Advertising.csv)下载数据文件，并将数据文件放在你的工程目录中。
假设你已经初始化了一个空的npm工程，打开`index.js`文件，并输入以下代码：（你可以直接复制/粘贴，但为了能够更好的理解它，建议你能亲自输入这段代码）
```JavaScript
const ml = require('ml-regression');
const csv = require('csvtojson');
const SLR = ml.SLR; // 简单线性回归

const csvFilePath = 'advertising.csv'; // 数据文件
let csvData = [], // 已解析的数据
    X = [], // 输入
    y = []; // 输出

let regressionModel;
```

我把这个文件放在了项目的根目录下，因此如果你放在了别的目录下，请同时更改上述代码中的`csvFilePath `变量。

这样的代码看起来相当整洁，不是么？

接下来使用`csvtojson`库的`fromFile`方法加载数据文件。
```JavaScript
csv()
    .fromFile(csvFilePath)
    .on('json', (jsonObj) => {
        csvData.push(jsonObj);
    })
    .on('done', () => {
        dressData(); // 从JSON对象中获取数据点
        performRegression();
    });
```

**第三步.** 将数据加以装饰，以准备开始执行

保存在`csvData`变量中的JSON对象已经准备好了，同时还分别需要一个数组，用来存储输入点数据和输出点数据。然后将通过`dressData`函数来运行数据，且`dressData`函数将会计算出`X`和`Y`变量。

```JavaScript
function dressData() {
    /**
     * 一个数据对象应该这样:
     * {
     *   TV: "10",
     *   Radio: "100",
     *   Newspaper: "20",
     *   "Sales": "1000"
     * }
     *
     * 因此，在添加数据点的同时，
     * 我们需要将String类型的值解析为Float类型。
     */
    csvData.forEach((row) => {
        X.push(f(row.Radio));
        y.push(f(row.Sales));
    });
}

function f(s) {
    return parseFloat(s);
}
```

**第四步.** 训练模型，并开始进行预测

现在数据已经装饰好了，是时候来训练模型了。

为了实现这一目标，我们需要一个`performRegression`函数：
```JavaScript
function performRegression() {
    regressionModel = new SLR(X, y); // 基于训练数据来训练模型
    console.log(regressionModel.toString(3));
    predictOutput();
}
```
`regressionModel`有一个`toString`方法，它所接收的参数代表输出值浮点数的精度。

`predictOutput `方法能够接收所输入的值，并且向终端输出所预测的值。

以下就是这个函数的代码：（这里使用了`node.js`的`readline`模块）
```JavaScript
function predictOutput() {
    rl.question('Enter input X for prediction (Press CTRL+C to exit) : ', (answer) => {
        console.log(`At X = ${answer}, y =  ${regressionModel.predict(parseFloat(answer))}`);
        predictOutput();
    });
}
```
以下代码读取了用户的输入值：
```JavaScript
const readline = require('readline'); // 同时预测用户的输入值

const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
});
```

**第五步.** 恭喜你！做到了。

如果你跟着我一步一步的做，现在你的`index.js`文件应该是这样子的：
```JavaScript
const ml = require('ml-regression');
const csv = require('csvtojson');
const SLR = ml.SLR; // 简单线性回归

const csvFilePath = 'advertising.csv'; // 数据
let csvData = [], // 已解析的数据
    X = [], // 输入
    y = []; // 输出

let regressionModel;

const readline = require('readline'); // 同时预测用户的输入值
const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
});

csv()
    .fromFile(csvFilePath)
    .on('json', (jsonObj) => {
        csvData.push(jsonObj);
    })
    .on('done', () => {
        dressData(); // 从JSON对象中获取数据点
        performRegression();
    });

function performRegression() {
    regressionModel = new SLR(X, y); // 基于训练数据来训练模型
    console.log(regressionModel.toString(3));
    predictOutput();
}

function dressData() {
    /**
     * 一个数据对象应该这样：
     * {
     *   TV: "10",
     *   Radio: "100",
     *   Newspaper: "20",
     *   "Sales": "1000"
     * }
     *
     * 因此，在添加数据点的同时，
     * 我们需要将String类型的值解析为Float类型。
     */
    csvData.forEach((row) => {
        X.push(f(row.Radio));
        y.push(f(row.Sales));
    });
}

function f(s) {
    return parseFloat(s);
}

function predictOutput() {
    rl.question('Enter input X for prediction (Press CTRL+C to exit) : ', (answer) => {
        console.log(`At X = ${answer}, y =  ${regressionModel.predict(parseFloat(answer))}`);
        predictOutput();
    });
}
```
打开终端，输入并运行`node index.js`，它将会输出如下所示内容：
```JavaScript
$ node index.js
f(x) = 0.202 * x + 9.31
Enter input X for prediction (Press CTRL+C to exit) : 151.5
At X = 151.5, y =  39.98974927911285
Enter input X for prediction (Press CTRL+C to exit) :
```
恭喜你！刚刚用JavaScript训练了你的第一个线性回归模型。（你有注意到它的速度么？）

> PS: 我将使用ml和其他的库（上面所列出的那些）在各种数据集上执行目前比较流行的机器学习算法。请时刻关注我的动态，获取最新的机器学习教程。

**感谢你的阅读**!如果你喜欢这篇文章的话，请为我点赞，以让别人知道JavaScript是多么的强大，以及为什么在机器学习领域中JavaScript不应该落后。


查看英文原文：[Machine Learning with JavaScript : Part 1](https://hackernoon.com/machine-learning-with-javascript-part-1-9b97f3ed4fe5)
