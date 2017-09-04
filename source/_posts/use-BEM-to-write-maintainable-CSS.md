---
layout: post
title: 【译】用BEM写出更具有可维护性的CSS
cdn: 'header-on'
date: 2017-01-04 22:00:13
tags:
    - 翻译
---
> [原文地址](http://www.integralist.co.uk/posts/bem.html)
> 已发表至[众成翻译](http://www.zcfy.cc/article/maintainable-css-with-bem-2169.html)

*   [介绍](http://www.integralist.co.uk/posts/bem.html#1)

*   [BEM: Block, Element, Modifier](http://www.integralist.co.uk/posts/bem.html#2)

*   [举例](http://www.integralist.co.uk/posts/bem.html#3)

*   [为什么最终BEM战胜了其他?](http://www.integralist.co.uk/posts/bem.html#4)

*   [总结](http://www.integralist.co.uk/posts/bem.html#5)

## 介绍

这篇文章教你一种方法，能够写出更具有可维护性的CSS，它就是"[BEM](http://bem.info/)"。

更新: [@necolas](http://twitter.com/necolas) 在推特上写了一个引人注意的评论，说我用的是修改版的 BEM 命名约定（BEM 是一个完整的框架，它不仅仅只是如何命名 class 以及如何写可维护的 CSS）。所以我在这里加以说明，以免引起任何混淆。

## BEM: Block, Element, Modifier

BEM 代表 "Block, Element, Modifier" ，它以一个虽简单却极其高效的方式将不同的组件和插件分类(就像下面所展示的那样)。

在每一个被定义的`块`下都有不同的`元素`，他由对象而组成。对于每一个元素（依据他在块内出现的位置），你还可以去`修饰`该元素的状态。

BEM和其他构建CSS的方法 ([OOCSS](https://github.com/stubbornella/oocss/wiki)/[SMACSS](http://smacss.com/))相比，在原则上有些相似，但是它在不损失构建效果的前提下，大大简化了其规则。

理解BEM最好的方法就是看使用它的例子（见下一章节）。但是如果你想了解它的历史以及一些更细节/可视化的原理剖析，请参考 [BEM](http://bem.info/) 官方网站。

## 举例

下面展示的是一个<span style="font-size: 1rem;">计算</span>金钱的小插件。当你输入总金额(e.g. £2.12p)并点击'calculate'按钮后，它会返回给你组成指定金额所需要的硬币清单。

HTML结构是很简单的...

```
<section>
    <h1>Sterling Calculator</h1>
    <form action="process.php" method="post">
        <p>Please enter an amount: (e.g. 92p, £2.12)</p>
        <p>
            <input name="amount"> 
            <input type="submit" value="Calculate">
        </p>
    </form>
</section>

```

接下来我们加入类名，以便后期方便的为该插件写样式，最后我会分别解释添加了什么，以及为什么这样添加......

```
<section class="widget">
    <h1 class="widget__header">Sterling Calculator</h1>
    <form class="widget__form" action="process.php" method="post">
        <p>Please enter an amount: (e.g. 92p, £2.12)</p>
        <p>
            <input name="amount" class="widget__input widget__input--amount"> 
            <input type="submit" value="Calculate" class="widget__input widget__input--submit">
        </p>
    </form>
</section>

```

我们首先确定了最顶级的`<section>`元素作为我们的'block'。这是包裹元素的顶层。我们为其添加了`widget`类，这也将成为我们这个对象/插件的命名空间（不管你想为他起什么名称）。

所有我们添加在这个'block'里元素的类的命名，都将在顶层`widget`的命名空间下。

当我们要为`<form>`元素添加样式，那么我们就给该元素添加`widget__form`类。双下划线可以使我们更容易的辨认出来该类是`widget`块的一部分。我们也可以看到`<input>`元素命名为`widget__input`类。

以下是元素样式的命名列表...

*   `widget`

*   `widget__header`

*   `widget__form`

*   `widget__input`

注意到在`widget__input`类下有两个类：`widget__input--amount` 和 `widget__input--submit`。他们是修饰符。代表着我们当前元素的状态。

让我们看一下我们在哪里用到了这些规则。我们在两个`<input>`元素上都使用了类`widget__input`（因为它们有同样的基本结构/样式）。但是他们也有些微的不同，因此我们可以用修饰符去应用额外的独特样式。

修饰符用两个连字符（破折号）连接，像这样：`block__element--modifier`。

最终，该插件的CSS代码如下所示...

```
.widget {
    background-color: #FC3;
}

.widget__header {
    color: #930;
    font-size: 3em;
    margin-bottom: 0.3em;
    text-shadow: #FFF 1px 1px 2px;
}

.widget__input {
    -webkit-border-radius: 5px;
       -moz-border-radius: 5px;
         -o-border-radius: 5px;
            border-radius: 5px;

    font-size: 0.9em;
    line-height: 1.3;
    padding: 0.4em 0.7em;
}

.widget__input--amount {
    border: 1px solid #930;
}

.widget__input--submit {
    background-color: #EEE;
    border: 0;
}

```

## 为什么最终BEM胜出?

我在这几年内尝试过不同的方法来写CSS。其经过是这样的.....

*   没有结构，一个页面中的所有东西都会在每一个页面打开的时候加载一遍。

*   为了将文件特定的内容放在一起而将文件分隔开，但是仍然没有实质性的结构而言。

*   标准的 [OOCSS (Object-Oriented CSS)](https://github.com/stubbornella/oocss/wiki)

*   [SMACSS (Scalable and Modular Architecture for CSS)](http://smacss.com/)

而现在采用的是BEM。

**我之所以采用BEM而不是其他技术，是基于这样的原因：它不像其他的方法那样令人困惑（比如 SMACSS），却任然提供了我们所需要的好架构(比如 OOCSS)而且具有高度一个可辨识的命名逻辑。**

对于我来说，OOCSS不够严谨。它让开发者随意命名他们的对象。但我见过在大型或超过一名开发者的项目中，对象命名变得很混乱，而由于缺乏严谨的命名约束，开发者们开始对每个类用来做什么感到困惑。

再谈谈SMACSS：它几乎太严谨了，以至于我认为在某些方面它_结构化过头了_。当我一开始使用它的时候，我认为它正是我一直寻找的解决方案，但最终的结果是我的 CSS 散得到处都是，而我根本不知道该从哪里开始。我搞不定它。

对某些人来说，可能这都不是问题，但对于我来说它们就像那个老的格言“_不要让我思考_”说的那样。如果我必须仔细地去想它是怎么工作的，或者我需要在哪里找代码，那么这种方法就不适合我。

但BEM成功了，因为它提供了一个好的面向对象的结构，对我来说没有陌生的词，并且足够简单，不用改变你原来使用 CSS 的方式。

但就像任何工具一样，它也可能被滥用。如何使用终归还是取决于开发者的整体技能和理解。

### 简洁

正如我之前说的那样，我之所以使用BEM的一个原因就是因为它足够简洁。

甚至连用词都比其他方法简洁。比如，有人跟你谈论结构化 CSS 你可能会听到这些词：

*   对象

*   模块

*   插件

*   组件

…注意这些叫法虽然不同，但是它们都指向相同的概念。毫无疑问，对于有些人来说，这么多不同叫法一定会让他们觉得困惑。

BEM与前者不同，因为它的用词是围绕它工作的环境：HTML和CSS。当我们说到CSS的 '块（block）' 的时候，我们都知道它指的是什么“块”，就是基础的构建块，决定着页面上元素的渲染方式<span style="font-size: 1rem;">（毫无歧义）</span>，但是'块'这个术语在其他语境中有不同的理解：

> 有一天我看到这块代码，它真烂。

…你明白当人们说上面这句话的时候，块是指一段或者一组代码。

单词'Block'是个很简单而非常关键的术语，并且更重要的是，它是一个人们非常熟悉的术语。

我们也知道，当我们在使用 CSS 的时候，最终我们针对的是“元素（elements）”。在这里没有别的词比这个词更适合描述，因为这正是我们所做的事情的最准确描述。

最后，“修饰器（modifier）”也是个简单而开发者能够完全理解和熟悉的词……

> 我想修饰下这个元素，我应该怎样做呢？

### 依然保持结构化

虽然用词和结构都很简单，它依然给了我们写可维护和易于理解的代码所需的全部工具。BEM 能够很方便地随着项目规模而扩展。

## 总结

我知道我之前提到了关于SMACSS，（没错，就是它！），但是即使我在第一次使用SMACSS的时候，我仍然小有怨言，“嗯哼，虽然完全搞定它有点小复杂，但是它似乎还不错”。有了BEM之后，我就不再有这种担心了。一开始我唯一不满的是它的外观。我不喜欢用双下划线或者双破折号。但是现在我也很喜欢使用它们！

如果你想看更多的关于BEM的用法，那么我向你推荐一个小的CSS抽象库，叫做[inuit.css](https://github.com/csswizardry/inuit.css)，它是由[Harry Roberts](http://csswizardry.com/) 编写，我也推荐你看[我网站的源代码](https://github.com/Integralist/integralist.github.com)。
                
