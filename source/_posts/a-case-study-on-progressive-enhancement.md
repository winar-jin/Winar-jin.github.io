---
layout: post
title: 【译】关于渐进增强方法的实例研究：“Building Social”
cdn: 'header-on'
date: 2017-01-12 13:20:15
tags:
	- 翻译
---

> [原文地址](https://www.smashingmagazine.com/2016/09/building-social-a-case-study-on-progressive-enhancement/#top)
> 已发表至[众成翻译](http://www.zcfy.cc/article/2227)


我们曾激烈讨论过渐进增强的方法以及它是如何提升网站的向下兼容性的。但是怎么样才能将渐进增强的概念应用在一个实际的项目中呢？当我们为网站设计一个复杂的交互效果时，我们很难决定哪些效果仅仅采用HTML和CSS就可以实现，而哪些效果则必须使用JavaScript。

本文我们将以[Building Social](http://buildingsocial.com/)网站的重新设计为例，为大家分享一些能够尽量延迟使用JavaScript的前端技术，它们看似简单却常常被忽略；同时也会分享一些提高JavaScript代码质量的方法。

### Building Social 简介

_Building Social_是一个APP，它提供一个和建筑相关的社交媒体平台，能够为在该平台上分享写字楼等建筑的人们建立联系，尽管人们互不相识。

人们能够获取到建筑的活动，留言，点赞，回复，比赛，市场等信息，只需要点击一下就可以从建筑管理者那里获得，同时也可以获得建筑的紧急或一般性的通知。

_Building Social_网站最初单页设计的理念是艺术，由 Patrick Riley和Paul Stanton在纽约的公司所指导。我们有满足设计理念的静态版面设计稿和一个展示了各种元素运动的视频。我们的工作就是进一步的它验证设计，更重要的是将它在浏览器中还原出来。

[![网站最初的静态版面设计图](http://p0.qhimg.com/t01a5df4a48a6577f31.jpg)](https://www.smashingmagazine.com/wp-content/uploads/2016/09/full-site-opt.jpg)



网站最初的静态版面设计图 ([查看大图](https://www.smashingmagazine.com/wp-content/uploads/2016/09/full-site-opt.jpg))

考虑到该项目在规模上来讲是相对较小的，不比[SGS](https://www.smashingmagazine.com/2016/09/redesigning-sgs-seven-level-navigation-system-a-case-study/)这个项目，因此我们不需要建立具体的设计或开发策略。这也就意味着我们应当主要集中在页面的实现以及一些具体的关于渐进增强方法的挑战，主要包含以下几个方面：

*   字符串的交换是自动的还是同步的；

*   混合部分的背景是动态的还是静态的；

*   基于位置的动画切换是手动的还是滚动的；

*   使用JavaScript增强纯CSS实现的模态窗口。

** 除了HTML之外一切都是非必须的 **

首先我们要设置一个起点，通过采用合适的[结构化和语义化](https://www.smashingmagazine.com/2009/04/progressive-enhancement-what-it-is-and-how-to-use-it/)的HTML标签来设计所有的元素。考虑到Pahtrick和Paul想要利用视频内容，我们又必须确保关于视频内容有合适的[回退策略](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Using_HTML5_audio_and_video#Fallback_options)。

### 挑战 1: 自动的 Vs. 同步的字符串交换

第一眼看来，网站的头部是纯装饰性的内容。然而，它实际上包含了可用的内容，促进了_Building Social_网站服务的几个特色。Patrick和Paul也想要在网站头部包含视频，每一个视频的场景都和一个具体的特色服务有关联。

渐进增强总结：

*   在HTML的无序列表中标记出关键的特色功能，以便在视频，Javascript或CSS无法加载的时候提供基本的用户体验。

*   通过CSS关键帧动画实现动画服务功能。

*   当视频加载的时候，每一个后续场景更改后都要交换每个服务功能，同时和播放头保持同步。

在建立了HTML的基础体验之后，我们尝试简单的使用CSS关键帧动画技术，为独立于视频的服务功能列表添加动画，每一个场景持续3秒（总共21秒）。下面是我们的CSS原始代码片段：

```
.list-options li { animation: animate_options 21s linear infinite; }
.list-options li:nth-child(1) { animation-delay: 0s; }
.list-options li:nth-child(2) { animation-delay: 3s; }

/* … and so on. We actually used a Sass 
snippet that specifies the animation delay 
value for each item:

@for $i from 1 through 7 {
  .list-options li:nth-child(#{$i}) { 
    animation-delay: #{($i*3)-3}s;
  }
}
*/ 
```


```
@keyframes animate_options {
    /* 0s   */  0%      { opacity: 0; bottom: -10vw; }
    /* 0.3s */  1.4%    { opacity: 1; bottom: 0; }
    /* 1.5s */  7.1%    { opacity: 1; bottom: 0; }
    /* 2.7s */  12.6%   { opacity: 1; bottom: 0; }
    /* 3s   */  14%     { opacity: 0; bottom: -10vw; }
    /* 21s  */  100%    { opacity: 0; bottom: -10vw; }
} 
```

也许不是你看到的最优雅的CSS代码片段，但它确实可以工作。然而，我们完全能意识到有时候视频会加载失败或者Javascript没有正常工作。在这种情况下，网站会根据上述CSS代码片段的关键帧时序，显示出每个服务功能的一张静态背景图片。

每一个关键服务功能都只通过CSS做出改变。

当视频完全加载完成后，我们还必须确保每一个服务功能和每一个场景的变换保持同步：

在视频中的每一个场景改变之后，每一个关键服务功能都能保持同步。

虽然我们可以简单的依赖CSS关键帧动画，但风险是：如果视频在其他资源之后完成加载，动画列表很快便会失去同步。解决方案如下：同步字符串交换和JavaScript片段，以时刻跟踪播放头的位置。

当Javascript加载时，它就开始跟踪播放头的位置，同时根据检索的值服务功能字符串产生交换。这是通过在JavaScript (with jQuery)中监听`timeupdate`事件，并且当视频的`currentTime`属性满足以下条件的时候，切换相应的元素的类名.

```
$('video').on('timeupdate', function() {
  var ct = $('video')[0].currentTime;
    var j = Math.floor(ct + 0.9);
    var k = Math.floor(ct / 3);
    if (j > 0 && j % 3 == 0) {
        $('.list-options li').removeClass('current');
    } else {
        $('.list-options li').eq(k).addClass('current');
    }
}); 
```

你也可能注意到我们使用的因子是0.9，用在实际场景变化之前切换类。这能给动画的正确执行提供足够的时间，而动画本身也是由CSS控制的。

```
.list-options li { bottom: -10vw; opacity: 0; transition: all .3s ease; }
.list-options li.current { bottom: 0; opacity: 1; } 
```

### 挑战 2: 静态的 Vs. 动态的混合部分背景

默认情况下，每个页面都有一个指定的背景颜色和一张或多张背景图片，没有什么其他太奢侈的东西在。除了这个纽约的团队，当用户向下滚动页面的时候，他们想要每一部分的背景能够巧妙的融合在一起。

[![默认情况下是静态背景，将融合背景添加作为渐进增强的方法。](http://p0.qhimg.com/t01804df267618628f3.jpg)](https://www.smashingmagazine.com/wp-content/uploads/2016/09/background-static-opt.jpg)



默认情况下是静态背景，将融合背景添加作为渐进增强的方法。 ([查看大图](https://www.smashingmagazine.com/wp-content/uploads/2016/09/background-static-opt.jpg))

渐进增强方法总结：

*   为每一个页面设置默认的背景颜色

*   通过JavaScript应用增强的背景混合

混合背景不是你每天看到或者工作中使用的。然而，一些有趣的例子已经出现了；比如说：一家设计机构的网站[ustwo](http://ustwo.com)。想要当用户向下滚动的时候始终出现这种效果，我们必须使用JavaScript，但是这仅仅是一种渐进增强的方法。在研究和测试了一系列的优秀的JavaScript库之后，我们决定使用[Scroll Magic](http://scrollmagic.io/)，主要由于以下两个原因：

*   它有很全面的文档，同时配有大量的例子，我们可以找到我们所需要的。

*   它是轻量级的，总大小不过8 KB.

起初我们的想法是监测每一个连续页面的位置，以便应用相关的CSS背景渐变。然而，很快就出现了问题，每当我们停止滚动时，都没有发生渐变！因为只有一个实体的背景颜色应用在了`body`元素上，所以渐变确实是由滚动所造成的错觉。

混合部分的背景创造了无限梯度的错觉。

一旦意识到我们是那个阶段的幻觉的受害者，我们放弃了渐变的想法。相反，我们决定在用户向下滚动页面时，在相邻的页面的纯背景色之间创建补间动画。

```
var bodyBackground = function() {

    // TRANSITION FROM GRAY TO WHITE
    var overviewHeight = parseInt($('#overview').height());
    new ScrollMagic.Scene({
    triggerElement: '#overview',
        offset: overviewHeight,
        duration: overviewHeight / 2
    })
    .setTween('body', { backgroundColor: ‘white' })
    .addTo(controller);

    // TRANSITION FROM WHITE TO BLUE
    var theAppHeight = parseInt($('#the-app').height());
    new ScrollMagic.Scene({
        triggerElement: '#the-app',
        triggerHook: 1,
        offset: theAppHeight,
        duration: theAppHeight / 2
    })
    .setTween('body', { backgroundColor: ‘blue' })
    .addTo(controller);

    // TRANSITION FROM BLUE TO RED
    var audiencesHeight = parseInt($('#providers').height());
    new ScrollMagic.Scene({
        triggerElement: '#audiences',
        offset: audiencesHeight,
        duration: audiencesHeight / 2
    })
    .setTween('body', { backgroundColor: ‘red' })
    .addTo(controller);

} 
```

已经确定了这种工作方法，然后我们采用了相同的配置，再次使用Scroll Magic，用来触发于滚动位置所绑定的场景动画。但在我们尝试这样做之前，我们必须确保这些动画能够被触发，而不必完全依赖JavaScript。

### 挑战 3: 手动 Vs. 基于滚动位置的动画切换

我们真的想要在用户向下滚动页面的时候融入移动的想法，但同时还要确保它不是难以忍受的（正如许多网站都采用了视差滚动效果一样）。

渐进增强方法总结：

*   默认使用CSS的`:hover`伪类来切换动画

*   用JavaScript，依据滚动的位置来切换动画

对于那些不是位于内容中心的简单的动画，CSS中可用的动画选项是足够的。比如说：动画长度和定时函数，还有一些CSS动画属性，比如说：定位，边距和透明度。此外，如果单个场景元素不需要任何特定的独立的交互，则仅仅需要一个事件来触发所有的动画元素。

[![使用:hover伪类触发简单的动画](http://p0.qhimg.com/t01cbdaf444b67fcd1b.gif)](https://www.smashingmagazine.com/wp-content/uploads/2016/09/vendors-hover.gif)



使用:hover伪类触发简单的动画 ([查看大图](https://www.smashingmagazine.com/wp-content/uploads/2016/09/vendors-hover.gif))

在我们所举得例子中，因为这一部分的宽度是100%，我们能够在每一部分使用:hover伪类触发动画效果。这样做是一种快速的并且很不显眼的方法来触发动画，而不必使用JavaScript。然而，这种方法不适用在可触摸设备上，大多数情况下，考虑到动画仅仅起到视觉增强的效果，我们决定弃用对于小视口设备的支持。

[![当滚动的时候触发更复杂的动画](http://p0.qhimg.com/t01109f6b72348bfc6a.gif)](https://www.smashingmagazine.com/wp-content/uploads/2016/09/vendors-scrollmagic.gif)



当滚动的时候触发更复杂的动画 ([查看大图](https://www.smashingmagazine.com/wp-content/uploads/2016/09/vendors-scrollmagic.gif))

当检测到JavaScript已经完全加载的时候，我们基于滚动位置触发动画。通过在动画中包括背景元素，并且对于每独调整每个动画的时长，我们能够更精确的每个单独的场景，从而增强整体的体验。

### 挑战 4: 使用Javascript增强的纯CSS模态框

考虑到最初是一个单页式的设计，有两个不同的交互目标（即对于业主和“提供商”），我们真的想要使用户在此页面上留存。此外，由于页面右侧的浮动文本，在每一次点击后逐步打开任何后续的联系表单的方法是不理想的。因此，模态框是目前最好的解决方案。

渐进增强方法总结：

*   默认使用语义化的HTML和CSS切换模态框。

*   通过引入不需要使用Javascript对页面重载的行内验证方法和表单提交来增强用户体验。

当我们在书写语义化的HTML时，我们可以很容易的切换元素的开和关的状态，这意味着元素可以实现逐渐增强而不会污染实际的代码。比如：通过保证每个需切换的元素都有一个语义化的`id`属性和锚点链接（指向相同的`id`），我们不仅能够提供钩子来支持有趣的交互，而且能够显著的改变无样式页面的可访问性和可用性。

[![纯CSS的模态框web表单](http://p0.qhimg.com/t01d766175ce7ca46c0.jpg)](https://www.smashingmagazine.com/wp-content/uploads/2016/09/modal-form-opt.jpg)



纯CSS的模态框web表单 ([查看大图](https://www.smashingmagazine.com/wp-content/uploads/2016/09/modal-form-opt.jpg))

但是锚点怎样才能准确的切换模态框呢？CSS`:target`伪类可以做到！元素的`id`属性和浏览器地址栏的哈希字符串相匹配时，`:target`伪类就会被触发，这也意味着该元素也可以被设置不同的样式。此外，CSS就很简单了：
```
/* E.g. URL https://yoursite.com/ */
#modal-contact { display: none; }

/* E.g. URL https://yoursite.com/#modal-contact */
#modal-contact:target { display: block; } 
```

如果你知道`:target`伪类技术（毕竟，该技术已经写在[参考文档](https://developer.mozilla.org/en-US/docs/Web/CSS/:target#Creating_a_pure_CSS_lightbox)里一段时间了）,那么就可以使用该伪类切换元素状态而不必依赖与Javascript了。

当同时包含了指向初始化锚点元素（或者是它的父元素）的第二个锚点时，我们就可以使用它来作为“close”链接。这是因为和模态框的`id`相匹配的`:target`伪类，当我们点击“close”链接后，就不再处于激活状态了。以下是一个简单的例子：

```
<div id="section-01">
  <a href="#modal-contact">Contact us</a>
</div>
<div id="modal-contact">
  <!-- contact form -->
  <a href="#section-01">Close</a>
</div> 

```

此外，当你指向最初被切换的模态框元素时，它将在用户激活模态框之前将页面精确的设置回它本来的位置。

我们也可以进一步使用`:target`伪类。假设模态窗包含一个web表单，我们想在同一个模态框中显示表单提交后的结果（例如：作为一个确认消息）。

[![在模态框中显示确认消息以产生加速的错觉，并让用户专注于任务](http://p0.qhimg.com/t016f26ea03dfac1f99.jpg)](https://www.smashingmagazine.com/wp-content/uploads/2016/09/modal-form-confirmation-opt.jpg)



在模态框中显示确认消息以产生加速的错觉，并让用户专注于任务 ([查看大图](https://www.smashingmagazine.com/wp-content/uploads/2016/09/modal-form-confirmation-opt.jpg))

为此，我们只需要向结果的URL中添加一个散列`id`（一个与模态框的“id”相同的“id”），即在用户提交表单后发送给用户的URL。 这里是一个简单的例子,在PHP中使用`header`重定向：

```
`<? header('Location: https://yoursite.com/#modal-contact'); ?>` 
```

在一个真正快速的互联网连接中，它将看起来好像浏览器从来都没有重载过页面一样。在网速慢的情况和功能较差的设备上，浏览器将自动跳转到确认信息，而不是重载页面并将它定位在页面的头部，因此能够大大的提高用户体验。

有了这个基本功能，我们可以使用Javascript进一步增强它的效果，通过引入[行内验证和无需整页重载的表单提交](https://www.smashingmagazine.com/2009/07/web-form-validation-best-practices-and-tutorials/)（使用良好的XML HTTP请求（XHR），也称为AJAX）。这样做不仅能节省时间，而且在发生任何输入错误的情况下，将使体验更方便。

最后，为什么不支持Escape键关闭模态框呢？在切换不同的类和锚定到不同的元素时遵循同样的原则，所需的代码是相对简单的：

```
$(document).on('keyup', function(e) {
    // If there is a visible modal, i.e. .md-show…
    if (e.keyCode == 27 && $('.md-show').length) {

        // Store the target from the modal's close link…
        var newTarget = $('.md-show .md-close').attr('href');

        // Hide the modal   
        $('.md-show').removeClass('md-show');

        // Position the page to the stored target
        document.location.hash = newTarget;
    };
}); 
```

考虑到我们所讨论的所有内容，下面是关于模态框联系表单的渐进增强方法列表：

1.  使用纯CSS打开和关闭模态框 - 不必依赖Javascript

2.  启用基本的HTML5验证，使用`required`属性和正确的输入类型（例如`type =“email”`，它将根据正确的电子邮件地址验证条目） - 不必依赖Javascript

3.  完全重载表单提交，在模态框中自动自动打开表单，并返回确认消息或服务器端验证的错误消息 - 不必依赖Javascript

4.  验证行内输入字段 - 在客户端和服务器端都需要JavaScript和重复的验证逻辑，这可能会难以维护。

5.  通过XHR提交表单，并返回确认消息或服务器端验证的错误消息，而无需重新加载页面或关闭模式框。

6.  其他的增强功能也是可能的  - 例如，用Escape键关闭模态框。

如果表单相对简单，并且已经用XHR发送到服务器了，则可以跳过步骤4；否则，根据项目，这将会创建不必要的维护开销。然而，你可能知道，尽管已经实现了客户端验证，表单也总是应该在[服务端被验证](https://www.smashingmagazine.com/2009/07/web-form-validation-best-practices-and-tutorials/#validation-methods)。

### 小奖励：随机的梦幻感

当你需要在客户端随机化某些东西的时候，你会想到什么？你是否会想到Javascript的`Math.random()`函数呢？

对于这个项目，我们想要在“提供商”（红色部分）部分使上面的云朵随机运动，以模拟真实的云朵的运动。云朵一般都处在不同的高度，但是它们或多或少会在相同的方向上移动。同时，在观察者的角度来看，云朵之间也会以不同与彼此的速度运动。

[![云朵在相同的方向，但却以不同的速度行进.](http://p0.qhimg.com/t015e5d8f9029787899.gif)](https://www.smashingmagazine.com/wp-content/uploads/2016/09/clouds.gif)



云朵在相同的方向，但却以不同的速度行进. ([查看大图](https://www.smashingmagazine.com/wp-content/uploads/2016/09/clouds.gif))

结果就是在对动画的设置上，使用相同的行进距离，但设置不同的持续时间：

```
.cloud-01 { animation: clouds 7s linear infinite; }
.cloud-02 { animation: clouds 5s linear infinite; }
.cloud-03 { animation: clouds 10s linear infinite; }

@keyframes clouds {
    0%  { opacity: 0; margin-left: 0; }
    50%     { opacity: 1; margin-left: -100px; }
    /* By hiding the cloud at 80%, we provide a break during which the cloud is hidden. */
    80%     { opacity: 0; margin-left: -200px; }
    100%    { opacity: 0; margin-left: -200px; }
} 
```

以上的技术证明了，有时我们可以没有必要使用Javascript。如果我们回退一步，重新审视问题和目标，我们可以通过使用现成和被广泛支持的技术，但以更聪明的方式实现相同的结果。

### 结论：这是可以做到的

上述提到的一些技术，也可以很容易的运用在其他的UI元素中，比如：标签，表格和下拉列表。通过使用本地HTML的功能，再加上伪类选择器和CSS动画，我们只需花费很少的代价，就可以提供相对简单但是却有很丰富的交互体验。

虽然使用JavaScript来增强用户体验是没有问题的，但考虑到基于JavaScript的动画和纯CSS动画几乎[一样快](https://css-tricks.com/myth-busting-css-animations-vs-javascript/)，在考虑任何解决方案以前，要先考虑它的[语义](https://www.smashingmagazine.com/2013/01/the-importance-of-sections/),可访问性和性能。特别是HTML的语义，已经被广泛的[传播](https://robertnyman.com/2007/10/29/explaining-semantic-mark-up/)了很多年,通过标记来增强其内容的含义，而不是它的表现。因此，在大多数情况下，通过调查多个解决方案，你就能找到最不复杂的那一个。

最后，通过创造性的使用你会用的基础的工具，你就可以大大提高性能和可访问性，也可以大大简化代码的维护。我们要谨记，没有任何两个项目是完全相同的，并且每一个项目在性能和可访问性上都可能会有具体的用例。比如：度量和数据就会很不同。例如：英国政府的数字服务就要确保仅有[1.1%](https://gds.blog.gov.uk/2013/10/21/how-many-people-are-missing-out-on-javascript-enhancement/)(或者1/93)的用户错过JavaScript增强。为了表明[渐进增强方法是更快速](https://www.smashingmagazine.com/2013/09/progressive-enhancement-is-faster/)的,Jake Archibald进行了一系列的令人信服的测试，尽管，他指出“现实生活中的不确定性”可以影响任何测试的公平性。但是，有一点始终是一样的，如果能够使用户尽快的获取到屏幕上的内容，将会大大提高用户体验。在这样做的过程中，你也将会获得额外的经验值。加油，每个人都能成功！
_(vf, il, al)_
                