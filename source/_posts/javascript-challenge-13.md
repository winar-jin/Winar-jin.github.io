---
title: Javascript30-day13 - Slide in on Scroll
layout: post
cdn: header-on
date: 2017-02-24 23:46:44
tags:
    - Javascript
---

> 在Github上看到了[wesbos](https://twitter.com/wesbos)的一个Javascript30天挑战的[repo](https://github.com/wesbos/JavaScript30)，旨在使用纯Js来进行练习，不允许使用任何其他的库和框架，该挑战共30天，我会在这里记录下自己练习的过程和遇到的问题。

## Day13 - Slide in on Scroll

第十三天的小练习是实现页面内伴随着鼠标滚动，到每个图片时图片出现，并伴随着动画出现。

[效果如下](http://htmlpreview.github.io/?https://github.com/winar-jin/JavaScript30-Challenge/blob/master/13%20-%20Slide%20in%20on%20Scroll/index.html)

## 实现思路
1. 首先要先获取需要加载动画的元素
2. 监听window的滚动事件`scroll`，绑定图片动画的函数
3. 在`checkSlide()`函数中，实现滚动到每一个图片的一半位置时，图片从两边飞入的动画效果

## 整体代码
```Javascript
const sliderImages = document.querySelectorAll('.slide-in');
function checkSlide(e) {
  sliderImages.forEach(sliderimage => {
    // 滑动到图片显示的一半
    const slideAt = window.innerHeight + window.scrollY - sliderimage.height/2;
    // 图片底部距文档顶部的距离
    const imageBottom = sliderimage.offsetTop + sliderimage.height;
    // 图片是否已经显示了一半
    const isHalfShown = slideAt > sliderimage.offsetTop;
    // 图片是否已经被完全滚动出去
    const isNotScrolledPast = window.scrollY < imageBottom;
    if(isHalfShown && isNotScrolledPast){
      sliderimage.classList.add('active');
    } else {
      sliderimage.classList.remove('active');
    }
  });
}
window.addEventListener('scroll', debounce(checkSlide));
```

## 难点
这个练习整体不难，我认为其中的距离的计算算是这个小练习中最为难以理解的部分，更像是数学问题。

* 首先获取触发动画的位置，在滚动到图片一半的位置时触发。
`const slideAt = window.innerHeight + window.scrollY - sliderimage.height/2;`
	* `window.innerHeight`表示浏览器的内部视图窗口的高度值
	* `window.scrollY`表示浏览器当前的在Y轴上滚动的距离（未滚动时值为0），也可通过采用`window.scroll(X,Y)`方法，设置页面在X轴和Y轴上面的滚动值
* 再获取图片底部到页面文档顶端的距离，采用`const imageBottom = sliderimage.offsetTop + sliderimage.height;`
	* `sliderimage.offsetTop`表示该图片最上面的值，到页面文档顶端的距离，再加上该图片的高度，就是图片底部到页面文档顶端的距离
* 设置两个flag，分别表示图片是否显示了一半和图片是否已经被完全滚动出去了，分别为`const isHalfShown = slideAt > sliderimage.offsetTop;`，`const isNotScrolledPast = window.scrollY < imageBottom;`
* 只有当图片已经显示了一半并且没有被图片没有被滚动出窗口是，图片才会显示出来，此处的动画处理方式如下：默认时将图片向左或向右移动30%，当图片出现在窗口中时，取消该图片的移动，显示在原位置；再加上`transition: all .5s;`，在图片出现的时候，就会显示出约0.5秒的过渡动画。

OK，到这里就实现了，当当💯