---
layout: post
cdn: 'header-on'
title: 【译】被人遗忘了的打印样式表
date: 2016-12-29 23:59:52
tags:
	- 翻译
---

> [原文地址](https://uxdesign.cc/i-totally-forgot-about-print-style-sheets-f1e6604cfd6#.p99ybq17x)
> 已发表至[众成翻译](http://www.zcfy.cc/article/i-totally-forgot-about-print-style-sheets-uxdesign-cc-user-experience-design-2130.html)

[Aaron Gustafson](https://twitter.com/AaronGustafson)指出当他在打印某一个页面的时候，原本有序的界面也变得杂乱不堪，也因此向[Indiegogo](http://indiegogo.com)发送了一篇[微博](https://twitter.com/AaronGustafson/status/788073583528538112)。

![](http://p0.qhimg.com/t01595dc33b6136272c.jpg)

当我看到这个消息的时候真的感到震惊，我忽然意识到已经有很长的时间没有优化一个页面的打印样式了，甚至都没有在页面的检查上花过心思。

这样真的很不好，也许是因为我们一直关注在浏览器上，我们希望我们的网站能够在各种类型各种分辨率的浏览器中都能完美的呈现给用户，也或许是因为我平常很少打印页面的缘故吧，不管怎样，print style sheets真的不应该被遗忘的。

为了能够在各种环境下，我们的网站都能够无障碍的使用，对页面的打印效果进行优化就显得很有必要。我们不能够[无故猜想](https://adactio.com/journal/11409)用户的行为。总有用户会打印页面的。我们应该考虑的是我们所编写的文章、博客推送、食谱、联系人信息、说明书或者是房地产网站，总会有某些人在需要的时候进行页面的打印。

> *很早以前我就放弃了在家里使用打印机的人，因为他们几乎在使用打印机十分钟后就放弃了。但是，并不是每一个人都和我一样... *
_-Heydon Pickering (Inclusive Design Patterns)_

自我反省下，如果你也和我处在相同的境地，那么这篇文章将会对你非常有用。

如果你还没有尝试过用CSS美化你的打印页面样式，那么我们就一起开始吧。


### 1\. 嵌入打印样式表

嵌入打印样式最好的方式就是在你的CSS样式中声明 `@media` 规则。
```
body {
    font-size: 18px;
} 
```

```
@media print {
    /* 添加打印规则 */
    body {
        font-size: 28px;
    }
} 
```
同样你也可以在HTML文件中嵌入打印样式文件，但是这样会额外多出一个的请求。
```
`<link media="print" href="print.css" />` 
```

### 2\. 测试

你不需要每调一次代码就打印一页。用浏览器就可以将你的页面导出为PDF文件，该文件会展示打印预览，甚至你可以直接在浏览器中调试。

* 在Firefox中调试打印样式的方法如下：打开_开发者工具_（`Shift + F2` 或 `工具 > Web Developer > Developer Toolbar`）；在浏览器窗口底部的输入框输入`media emulate print`然后单击`enter`；当前激活的标签页将会以`print`的_媒体类型_来显示，直到你关闭或者刷新该标签页。
![](http://p0.qhimg.com/t014c03a1e15a2166a7.png)

Firefox的打印样式模拟

* 在Chrome中：打开_开发者工具_(`CMD + Opt + I` (macOS) 或 `Ctrl + Shift + I` (Windows)或`视图>开发者>开发者工具`)；打开console标签(`Esc`)；打开渲染面板(rendering pane)，开启_Emulate CSS Media_ and select _Print_选项。
![](http://p0.qhimg.com/t015102189584c78674.png)
Chrome的打印样式模拟

### 3\. 绝对单位

绝对单位对屏幕来说是很不友好的，但是对于印刷来说则相反。在打印样式中，使用像`cm`, `mm`, `in`, `pt` or `pc`这样的绝对[单位](https://developer.mozilla.org/de/docs/Web/CSS/length)是很安全的，并且也推荐这么使用。
```
section {
    margin-bottom: 2cm;
} 
```

### 4\. 页面具体规则

可以使用`@page`规则具体的定义页面的尺寸，方向以及页边距属性，如果想要所有的页面都有一个边距，使用这个规则会非常方便。
```
@media print {
     @page {
        margin: 1cm;
    }
} 
```
`@page`规则是[页面媒体模块](https://drafts.csswg.org/css-page-3/)的一部分。包括了很多有用的东西，比如说：选择打印的第一页，或空白页，在页面的一角定位元素，还有其他一些[功能](https://www.smashingmagazine.com/2015/01/designing-for-print-with-css/)。。甚至可以用该属性[出版书籍](https://alistapart.com/article/building-books-with-css3)。

### 5\. 控制页面断页

因为打印的页面是不像网站那样可以无穷尽的向下面拉的。内容在一个页面上肯定会断开，到第二个页面。如果断页这种情乱出现了，我们可以用5个属性控制它。

#### 在一个元素前使页面断页

如果我们想要一个元素总是在页面的开头，我们可以强制页面设置`page-break-before`属性来进行断页。

```
section {
    page-break-before: always;
}
```
[page-break-before on MDN](https://developer.mozilla.org/en/docs/Web/CSS/page-break-before)

#### **在一个元素之后断页**

`page-break-after`属性可以让我们强制或者避免页面在一个元素之后断页。

```
h2 {
    page-break-after: always;
}
```
[page-break-after on MDN](https://developer.mozilla.org/en/docs/Web/CSS/page-break-after)

#### **在一个元素内部进行断页**

如果你想避免页面的一个元素被分在了两张纸上，就采用`page-break-inside`属性吧。
```
ul {
    page-break-inside: avoid;
} 
```

[page-break-inside on MDN](https://developer.mozilla.org/en/docs/Web/CSS/page-break-inside)

#### **Widows and Orphans**

有时候你不想强制页面断开，至少你可以控制在压面上显示多少行或者在下一页显示多少行。比如说：如果一个段落的最后一行在当前页显示不下，那么该段落的倒数两行都会显示在下一页。因为`widows`属性控制这个特性，默认值是`2`，我们也可以改变这个值。
```
p {
    widows: 4;
}
```
如果页面只能容纳下一行，还有另一种解决办法，整段都打印在下一页,`orphans`属性代表这种方法，该属性的默认值也是`2`。
```
p {
    orphans: 3;
}
```
上面的代码表示，该页面至少能够容纳下三行，否则的话就将该段打印在下一页。

这里有一些[例子](http://codepen.io/matuzo/pen/oYvBjN)。(也可以查看[调试版本](http://s.codepen.io/matuzo/debug/oYvBjN)进行测试)。

[并不是所有的属性都能适配所有的浏览器](http://caniuse.com/#feat=css-page-break)，你应该在不同的浏览器下测试你的打印样式。

### 6\. 重新设置你的页面样式

为了打印效果，改变页面的样式（像`background-color`, `box-shadow` 或 `color`）也是可以的。

可以参考一下[HTML5样板打印样式表](https://github.com/h5bp/html5-boilerplate/blob/master/dist/css/main.css):

```
*,
*:before,
*:after,
*:first-letter,
p:first-line,
div:first-line,
blockquote:first-line,
li:first-line {
    background: transparent !important;
    color: #000 !important;
    box-shadow: none !important;
    text-shadow: none !important;
} 
```

打印样式表是可以使用`!important`属性的少数之一。

### 7\. 删掉不必要的内容

为避免你不必要的内容浪费墨水，你可以删除掉展示性内容，广告，导航，等，将他们设置为`display: none`。
你也可以只显示主要的内容，而将其他的内容隐藏掉。
```
body > *:not(main) {
    display: none;
} 
```

### 8\. 将链接中的网址显示出来

在不知道网址的情况下将一个链接打印出来是一点用处也没有的。

我们可以很容易的把一个网址显示在他的文字旁边。

```
a[href]:after {
    content: " (" attr(href) ")";
}
```
以下属性设置会显示出网站的相对链接，绝对链接，锚点等，这样可以大大增加打印出来的实用性。
```
a[href^="http"]:not([href*="mywebsite.com"]):after {
    content: " (" attr(href) ")";
}
```
我知道这看起来很疯狂。这一行表示：在每一个具有`href`属性的链接旁展示`href`属性的值。他以_http_开头，但可能他的值不像_mywebsite.com_这样。

### 9\. 将缩写的内容展开

缩写应该被包含在元素里面，他的缩写包含在`title`属性中，将元素的缩写展示在打印页中也是很有用的。
```
abbr[title]:after {
    content: " (" attr(title) ")";
} 
```

### 10\. 强制打印页面背景
通常，如果默认设置的话，浏览器不会打印背景颜色和背景图片，但有时你可能想要强制打印出来背景。不标准的[属性](https://developer.mozilla.org/de/docs/Web/CSS/-webkit-print-color-adjust)`print-color-adjust`会允许你重写浏览器的默认设置。

```
header {
    -webkit-print-color-adjust: exact;
    print-color-adjust: exact;
} 
```

### 11\. 媒体查询
如果你像下面这样写了媒体查询的设置，请注意，这些设置并不会应用在打印样式中。

```
@media screen and (min-width: 48em) {
    /* screen only */ 
} 
```
为什么呢？因为这些规则仅仅使用在以下这些情况：`min-width` 是 `48em`，并且`media-type` 是`screen`。删除掉`screen`关键字之后，媒体查询仅仅被`min-width`所限制。
```
@media (min-width: 48em) {
    /* all media types */ 
} 
```

### 12\. 打印地图
当前版本的火狐浏览器和Chrome浏览器支持打印地图，但是还有一些浏览器不支持，像Safari。一些服务商提供[静态地图](http://staticmapmaker.com/)服务，也可以用这样的服务来进行地图的打印。
```
.map {
    width: 400px;
    height: 300px;
    background-image: url('http://maps.googleapis.com/maps/api/staticmap?center=Wien+Floridsdorf&zoom=13&scale=false&size=400x300&maptype=roadmap&format=png&visual_refresh=true');
    -webkit-print-color-adjust: exact;
    print-color-adjust: exact;
} 
```

### 13\. 二维码
一篇[杂志上的文章](https://www.smashingmagazine.com/2013/03/tips-and-tricks-for-print-style-sheets/#print-qr-codes-for-easy-url-references)写了一些很不错的建议。其中的一条就是在你的打印页放一个二维码，这样你的用户就可以不用输入整个网址就可以获得最新的版本了。

### Bonus: 打印非优化页面
在我调查的过程中，我发现了对于打印非优化页面非常有用的工具，有了[Printliminator](https://css-tricks.github.io/The-Printliminator/)，你可以很简单的通过点击元素来删除他们。
这是他的一个[样例](https://www.youtube.com/watch?v=Dt8hpqEIL1c)和在[Github](https://github.com/CSS-Tricks/The-Printliminator)的库。

### Bonus II: Gutenberg
如果你愿意使用框架，你也许会喜欢[Gutenberg](https://github.com/BafS/Gutenberg)，他会帮助你优化你的打印页面。

### Bonus III: Hartija
还有一个由[Vladimir Carrer](https://medium.com/u/4c25f07f8cbe)出的打印样式框架，叫做[Hartija](https://github.com/vladocar/Hartija---CSS-Print-Framework)。

* * *

就是这样了，如果你想要看这些属性在实际中应用的例子，可以看我在[CodePen](http://codepen.io/matuzo/pen/oYvBjN?editors=1100)上做的一个[调试版本](http://s.codepen.io/matuzo/debug/oYvBjN))。

希望你会喜欢这篇文章。

PS:感谢[Eva](https://twitter.com/eva_trostlos)帮助我校对了文稿以及Mario的建议。

#### 参考资源

*   [https://www.youtube.com/watch?v=jF-OQ-BrIAM](https://www.youtube.com/watch?v=jF-OQ-BrIAM)

*   [https://github.com/h5bp/html5-boilerplate/blob/master/dist/css/main.css#L217](https://github.com/h5bp/html5-boilerplate/blob/master/dist/css/main.css#L217)

*   [https://www.smashingmagazine.com/2013/03/tips-and-tricks-for-print-style-sheets/](https://www.smashingmagazine.com/2013/03/tips-and-tricks-for-print-style-sheets/)

*   [http://stackoverflow.com/questions/9519556/faster-way-to-develop-and-test-print-stylesheets-avoid-print-preview-every-time](http://stackoverflow.com/questions/9519556/faster-way-to-develop-and-test-print-stylesheets-avoid-print-preview-every-time)

*   [https://www.smashingmagazine.com/2015/01/designing-for-print-with-css/](https://www.smashingmagazine.com/2015/01/designing-for-print-with-css/)

*   [https://github.com/BafS/Gutenberg](https://github.com/BafS/Gutenberg)

*   [https://helloanselm.com/2014/unified-media-queries/](https://helloanselm.com/2014/unified-media-queries/)
