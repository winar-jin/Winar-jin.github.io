---
title: 【译】机器学习与JavaScript（二）
layout: post
cdn: header-on
date: 2017-07-06 00:10:10
tags:
    - JavaScript
    - 翻译
---

> 本文首发于**前端之巅**公众号：【[点此查看](https://mp.weixin.qq.com/s?__biz=MzIwNjQwMzUwMQ==&mid=2247485292&idx=1&sn=779276cef1ab540f807fab521b94f730&chksm=972365aea054ecb825a88afcc49e249227cd119a077d86ed1c2a292a137f7adabe2404e394f8&scene=38#wechat_redirect)】

![Petal Length vs Sepal Length by plot.ly](/images/0706-iras.png)

> 这是机器学习与JavaScript系列文章的第二篇。第一篇请[点击这里](http://xujin.pro/2017-07-05-machine-learning-with-javascript-part-1.html)查看。

这篇文章我们主要学习KNN算法。

KNN代表K-Nearest-Neighbours，是一种监督学习的算法。它可以用来处理分类问题和回归问题。接下来我们会先讲解下KNN算法的工作原理，当然你也可以直接看代码，Github仓库在这里：[基于JavaScript的机器学习](https://github.com/abhisheksoni27/machine-learning-with-js)。

## KNN算法的工作原理
KNN算法根据同一类别数据点的最大邻居数，决定新数据点的类别。

如果一个新数据点的邻居数如下所示：NY：7，NJ：0，IN：4，那么这个新的数据点将属于NY这一类。

假设你在邮局工作，工作的内容就是为不同的邮递员规划和分发信件，当然要尽量减少邮递员在不同社区之间往返的次数。既然现在是在假设，那么我们就说这里共有7个不同的社区。这其实就是一种分类问题，需要将不同的信件进行分类，哪些要送到上东区，哪些到曼哈顿区，等等。

如果你喜欢浪费时间和资源，你可以让每个邮递员一次只向每个社区送一封信，然后邮递员们就会在相同的社区中碰面，然后发现你的规划有多么糟糕。当然这是你可以实现的最糟糕的分配方式。

此外，你也可以依据社区所在的位置进行规划，将位置相近的信件归为一类。

你可以这样想：如果这些信件彼此的距离小于三个街区，就把他们分配给同一个邮递员。在这个例子中，离的最近的街区的数量，就是KNN算法中k的来源。你可以继续增大距离相近的街区的数量，直到达到最高效的分配方式。这个值就是分类问题中的最佳k值。

因此，根据一个或多个参数，比如收件人的位置，你可以将所有的信件进行分类，哪些送到上东区，哪些到曼哈顿区，等等。

## KNN算法的代码实践
和上一篇教程中一样，我们将使用`ml.js`的`KNN`模块来训练k最近邻算法的分类器。每个机器学习领域中的问题都需要数据的支持，因此接下来此教程将会使用到IRIS的数据集。

**Iris**也称鸢尾花卉数据集，该数据集包含三种类型的鸢尾花（Setosa, Versicolour和 Virginica），它们都拥有不同的花瓣和花萼长度，还有一个字段用于表示它们的类型。

### 第一步：安装依赖库

`$ yarn add ml-knn csvtojson prompt`

或者你更喜欢`npm`：

`$ npm install ml-knn csvtojson prompt`

**[ml-knn](https://github.com/mljs/knn)**：ml.js中的KNN模块
**[csvtojson](https://github.com/Keyang/node-csvtojson)**：数据解析
**[prompt](https://github.com/flatiron/prompt)**：可以根据用户的输入值进行预测

### 第二步：初始化库并加载数据

Iris数据集由加利福尼亚大学尔湾分校提供。但是，由于它的组织方式，我们必须在浏览器中复制它的内容（全选并复制），然后将其粘贴到名为`iris.csv`的文件中。你可以随便命名该文件，只要后缀名是`.csv`都行。

接下来，初始化该库，并加载数据。假设你已经提前建好了一个空的npm工程。但是你如果对`npm`还不是很熟悉，可以看看关于`npm`的[简要介绍](https://docs.npmjs.com/cli/init)。

```JavaScript
const KNN = require('ml-knn');
const csv = require('csvtojson');
const prompt = require('prompt');
const knn = new KNN();

const csvFilePath = 'iris.csv'; // 数据
const names = ['sepalLength', 'sepalWidth', 'petalLength', 'petalWidth', 'type']; // 表头

let seperationSize; // 分离训练数据和测试数据

let data = [],
    X = [],
    y = [];

let trainingSetX = [],
    trainingSetY = [],
    testSetX = [],
    testSetY = [];
```

表头`names`变量仅仅是为了可视化和方便理解，最后会删除它的。

`seperationSize `变量可以将数据分离为训练数据和测试数据。

看起来是不是很酷？

我们已经导入了`csvtojson`库，接下来使用它的`fromFile `方法加载数据。（因为拷贝的数据并没有表头，因此我们为它添加了表头。）

```JavaScript
csv({noheader: true, headers: names})
    .fromFile(csvFilePath)
    .on('json', (jsonObj) => {
        data.push(jsonObj); // 将每一个对象都加载到数据数组中
    })
    .on('done', (error) => {
        seperationSize = 0.7 * data.length;
        data = shuffleArray(data);
        dressData();
    });
```

将文件中每一行的数据都添加到`data`变量中，当数据加载完成后，我们将`seperationSize `设置为数据集中样本容量的0.7倍。谨记：如果训练集的样本容量太小，分类器可能不会像训练大数据集那样有那么好的表现。

由于我们的数据集是根据类型进行排序的（可以通过查看`console.log`进行确认），为了能更好的分割数据，使用`shuffleArray`函数将数据集中的数据打乱。（如果不将数据打乱，模型很有可能在前两类中工作的很好，但是第三类就失败了。）

以下就是`shuffleArray`函数的定义，我是从[StackOverFlow](https://stackoverflow.com/a/12646864)上得到的答案。

```JavaScript
/**
 * https://stackoverflow.com/a/12646864
 * 将数组元素的顺序打乱
 * 使用了Durstenfeld洗牌算法
 */
function shuffleArray(array) {
    for (var i = array.length - 1; i > 0; i--) {
        var j = Math.floor(Math.random() * (i + 1));
        var temp = array[i];
        array[i] = array[j];
        array[j] = temp;
    }
    return array;
}
```

### 第三步：装饰数据

我们的数据应该是下面这样的结构：
```Json
{
 sepalLength: ‘5.1’,
 sepalWidth: ‘3.5’,
 petalLength: ‘1.4’,
 petalWidth: ‘0.2’,
 type: ‘Iris-setosa’
}
```

在将数据提供给KNN分类器之前，我们需要对其做两件事情：
1. 将`String`类型的值转为`float`类型（`parseFloat `）
2. 将鸢尾花的实际类型转换为数字类型。（毕竟电脑更喜欢数字，你知道么？）

```JavaScript
function dressData() {
    /**
     * 有三种不同的鸢尾花类型，
     * 也就是数据集的分类器
     *
     * 1. Iris Setosa (Iris-setosa)
     * 2. Iris Versicolor (Iris-versicolor)
     * 3. Iris Virginica (Iris-virginica)
     *
     * 将这些类型从字符串转为数组
     * 比如说，
     * 0 代表 setosa,
     * 1 代表 versicolor,
     * 3 代表 virginica
     */

    let types = new Set(); // 获取不同的类型

    data.forEach((row) => {
        types.add(row.type);
    });

    typesArray = [...types]; // 保存不同的类型

    data.forEach((row) => {
        let rowArray, typeNumber;

        rowArray = Object.keys(row).map(key => parseFloat(row[key])).slice(0, 4);

        typeNumber = typesArray.indexOf(row.type); // 将字符串转为数字

        X.push(rowArray);
        y.push(typeNumber);
    });

    trainingSetX = X.slice(0, seperationSize);
    trainingSetY = y.slice(0, seperationSize);
    testSetX = X.slice(seperationSize);
    testSetY = y.slice(seperationSize);

    train();
}
```

简单介绍下`Set`，它和数学中的概念类似，在JS中可以存储原始类型和对象引用类型的数据，只是Set中不会有重复的元素，并且它的元素没有索引。（这一点和数组相反。）

使用扩展运算符或Set构造函数可以轻松的将它转换为数组。

### 第四步：训练模型并对它进行测试

数据已经装饰好了，接下来就准备训练模型吧：

```JavaScript
function train() {
    knn.train(trainingSetX, trainingSetY, {k: 7});
    test();
}
```

`train`方法至少需要传入两个参数，一个是输入的数据，比如：花瓣长度和花萼宽度；另一个是它所属的种类，比如：`Iris-setosa`等。该方法也可以接收一个可选的选项参数，该参数仅仅是一个对象，用来调整算法的内部参数。在上述代码中，我们传递k值作为选项参数，k的默认值是7。

现在模型已经训练好了，接下来看看它在测试集上的表现如何。我们主要关注预测分类时出现错误的次数。（也就是说，它预测输入的是**一个值**，但实际输入的却是**另一个值**的次数。）

```JavaScript
function test() {
    const result = knn.predict(testSetX);
    const testSetLength = testSetX.length;
    const predictionError = error(result, testSetY);
    console.log(`Test Set Size = ${testSetLength} and number of Misclassifications = ${predictionError}`);
    predict();
}
```

错误计算的方法如下。使用`for`循环来遍历全部数据集，观察模型的预测输出的值是否和实际输出的值相等，若不同就是一个错误的分类。

```JavaScript
function error(predicted, expected) {
    let misclassifications = 0;
    for (var index = 0; index < predicted.length; index++) {
        if (predicted[index] !== expected[index]) {
            misclassifications++;
        }
    }
    return misclassifications;
}
```

### 第五步：开始预测（可选的）

是时候展示预测值了。

```JavaScript
function predict() {
    let temp = [];
    prompt.start();

    prompt.get(['Sepal Length', 'Sepal Width', 'Petal Length', 'Petal Width'], function (err, result) {
        if (!err) {
            for (var key in result) {
                temp.push(parseFloat(result[key]));
            }
            console.log(`With ${temp} -- type =  ${knn.getSinglePrediction(temp)}`);
        }
    });
}
```

如果你不想在新的输入值上测试模型，可以随时跳过此步骤。

### 第六步：大功告成！

如果你跟着我一步一步的做，现在你的`index.js`文件应该是这样子的：

```JavaScript
const KNN = require('ml-knn');
const csv = require('csvtojson');
const prompt = require('prompt');
const knn = new KNN();

const csvFilePath = 'iris.csv'; // 数据
const names = ['sepalLength', 'sepalWidth', 'petalLength', 'petalWidth', 'type']; // 表头

let seperationSize; // 分离训练数据和测试数据

let data = [], X = [], y = [];

let trainingSetX = [], trainingSetY = [], testSetX = [], testSetY = [];

csv({noheader: true, headers: names})
    .fromFile(csvFilePath)
    .on('json', (jsonObj) => {
        data.push(jsonObj); // 将每一个对象都加载到数据数组中
    })
    .on('done', (error) => {
        seperationSize = 0.7 * data.length;
        data = shuffleArray(data);
        dressData();
    });

function dressData() {
    let types = new Set(); // 获取不同的类型
    data.forEach((row) => {
        types.add(row.type);
    });
    typesArray = [...types]; // 保存不同的类型

    data.forEach((row) => {
        let rowArray, typeNumber;
        rowArray = Object.keys(row).map(key => parseFloat(row[key])).slice(0, 4);
        typeNumber = typesArray.indexOf(row.type); // 将字符串转为数字

        X.push(rowArray);
        y.push(typeNumber);
    });

    trainingSetX = X.slice(0, seperationSize);
    trainingSetY = y.slice(0, seperationSize);
    testSetX = X.slice(seperationSize);
    testSetY = y.slice(seperationSize);

    train();
}

function train() {
    knn.train(trainingSetX, trainingSetY);
    test();
}

function test() {
    const result = knn.predict(testSetX);
    const testSetLength = testSetX.length
    const predictionError = error(result, testSetY);
    console.log(`Test Set Size = ${testSetLength} and number of Misclassifications = ${predictionError}`);
    predict();
}

function error(predicted, expected) {
    let misclassifications = 0;
    for (var index = 0; index < predicted.length; index++) {
        if (predicted[index] !== expected[index]) {
            misclassifications++;
        }
    }
    return misclassifications;
}

function predict() {
    let temp = [];
    prompt.start();
    prompt.get(['Sepal Length', 'Sepal Width', 'Petal Length', 'Petal Width'], function (err, result) {
        if (!err) {
            for (var key in result) {
                temp.push(parseFloat(result[key]));
            }
            console.log(`With ${temp} -- type =  ${knn.getSinglePrediction(temp)}`);
        }
    });
}

/**
 * https://stackoverflow.com/a/12646864
 * 将数组元素的顺序打乱
 * 使用了Durstenfeld洗牌算法

 */
function shuffleArray(array) {
    for (var i = array.length - 1; i > 0; i--) {
        var j = Math.floor(Math.random() * (i + 1));
        var temp = array[i];
        array[i] = array[j];
        array[j] = temp;
    }
    return array;
}
```

打开终端，输入并运行`node index.js`，它将会输出如下所示内容：

```JavaScript
$ node index.js
Test Set Size = 45 and number of Misclassifications = 2
prompt: Sepal Length:  1.7
prompt: Sepal Width:  2.5
prompt: Petal Length:  0.5
prompt: Petal Width:  3.4
With 1.7,2.5,0.5,3.4 -- type =  2
```

很好，以上就是一个可运行的KNN算法，一种优雅的分类。

所有的代码都在Github上：[machine-learning-with-js](https://github.com/abhisheksoni27/machine-learning-with-js)。

KNN算法的好坏很大程度上取决于k的值，它被称为超参数。那么超参数是什么呢，我从[Quora](https://www.quora.com/What-are-hyperparameters-in-machine-learning)上得到了这样的答案：“它是一种在常规训练中不能直接获得的参数，这些参数代表模型更高级的属性，比如：模型的复杂性或者模型学习的速度。它们都被称为超参数。”

> 在上述的邮递信件的例子中，k定义了街区的数量。也就是在社区中，多少个街区距离内的信件应该被分为一类。

![](/images/0706-beautiful.jpeg)

我正在研究`ml-knn`模块，希望很快能实现k选择过程的自动化。

如果你对它也比较感兴趣，并且想看看它还能做什么。可以到[加州大学尔湾分校机器学习资料库](http://archive.ics.uci.edu/ml/index.php)中查看相关资料 ，也可以在不同的数据集上使用你的分类器。（这个资料库有数百个数据集。）


查看英文原文：[Machine Learning with JavaScript : Part 2](https://hackernoon.com/machine-learning-with-javascript-part-2-da994c17d483)
