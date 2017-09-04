---
layout: post
title: Markdown学习笔记
date: 2016-12-22 20:34:37
cdn: 'header-on'
tags:
    - markdown
---

Markdown是一款轻量级的并且语法简单的标记语言，适用于各种文章的样式写作。

## 什么是Markdown？

Markdown是一种在web上编写文章样式的方法，是一种语法简单的标记语言。可以控制文档的展示方式；将文本格式化显示为加粗或者斜体，添加图片，也可以创建一个列表。这些也仅仅是Markdown的一部分。大多数情况下，Markdown是一种用非字母标记（eg：‘#’，‘*’）的语言规则。

## Examples

### 文本

```
It's very easy to make some words **bold** and other words *italic* with Markdown. You can even [link to Google!](http://google.com)
```
It's very easy to make some words **bold** and other words *italic* with Markdown. You can even [link to Google!](http://google.com)

### 列表

```
1. One
2. Two
3. Three
```
1. One
2. Two
3. Three

```
* Start a line with a star
* Profit!
```
* Start a line with a star
* Profit!

> 列表也可以嵌套！
```
- Dashes work just as well
- And if you have sub points, put two spaces before the dash or star:
  - Like this
  - And this
```
- Dashes work just as well
- And if you have sub points, put two spaces before the dash or star:
  - Like this
  - And this

### 图像
```
![Image of Yaktocat](https://octodex.github.com/images/yaktocat.png)
```
![Image of Yaktocat](https://octodex.github.com/images/yaktocat.png)

### 标题和引用
> 在组织文章结构的时候，拥有不同级别的标题是非常有用的。

> 在一行的前面加上`#`就可以标记为标题，从`#`至`######`分别代表一级到六级标题，`#`越多，标题文字越小，但最多只能有六级标题。
```
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
```
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题

> 有时候我们会在文章中引用一些内容，这时候用`>`开始一行就可以将这一行作为引用。
```
> 我是引用
```
> 我是引用

### 代码

> 比如我们在写技术博客的时候，以及其他的一些场景，会用到文章中显示代码，Markdown有两种类型的代码方式，行内代码和块级代码。
>* **行内代码块**：用单引号`（*键盘左上角Esc下那个键*）将代码块包裹。
>* **块级代码**：1.代码块上下分别用三个单引号```包裹,也可在单引号后面加上代码语言。 2.将代码块的每一行分别缩进四个字符。

```
    if (isAwesome){
      return true
    }
```
    if (isAwesome){
      return true
    }

<pre>
```
if (isAwesome){
 return true
}
```
</pre>
```
if (isAwesome){
  return true
}
```

> 加上编程语言后，后使用语法高亮。
<pre>
```javascript
if (isAwesome){
  return true
}
```
</pre>
```javascript
if (isAwesome){
  return true
}
```

### 其他 
* 任务清单
```
- [x] This is a complete item
- [ ] This is an incomplete item
```
![tasklist](http://7xss68.com1.z0.glb.clouddn.com/blog/post/2016/12/Markdown_taskList.png)
## 语法规则
### 标题
```
# This is an <h1> tag
## This is an <h2> tag
###### This is an <h6> tag
```
### 字体加粗和斜体

```
*会显示斜体字*
_会显示斜体字_
**字体加粗**
__字体加粗__
_你可以 **结合** 他们_
```
*会显示斜体字*

_会显示斜体字_

**字体加粗**

__字体加粗__

_你可以 **结合** 他们_

### 列表
* 无序列表
```
* Item 1
* Item 2
  * Item 2a
  * Item 2b
```
* Item 1
* Item 2
  * Item 2a
  * Item 2b

* 有序列表
```
1. Item 1
2. Item 2
3. Item 3
   * Item 3a
   * Item 3b
```
1. Item 1
2. Item 2
3. Item 3
   * Item 3a
   * Item 3b

* 图像

```
![图像标题](图像链接)
```

* 链接

```
[GitHub](http://github.com)
```
[GitHub](http://github.com)

* 引用
```
> 我是一个引用
```
> 我是一个引用

* 行内代码
```
`<addr>` element here instead.
```
`<addr>` element here instead.

* 块级语法高亮
```
```javascript
function fancyAlert(arg) {
  if(arg) {
    $.facebox({div:'#foo'})
  }
}
```
```javascript
function fancyAlert(arg) {
  if(arg) {
    $.facebox({div:'#foo'})
  }
}
```

* 表格
> 创建表格只需要将所有的标题和内容，按照表格的格式写在一起。然后将第一行的标题和下面的内容用连字符`-`分隔，把每一列用竖线`|`分隔即可。
```
标题一|标题二 |标题三
-----|------|----
内容一| 内容二| 内容三
内容一| 内容二| 内容三
内容一| 内容二| 内容三
```
标题一|标题二 |标题三
-----|------|----
内容一| 内容二| 内容三
内容一| 内容二| 内容三
内容一| 内容二| 内容三

* 删除线
```
~~我被删除了~~
```
~~我被删除了~~
## 注意

* 标签的输入状态，几乎所有标签都需要在英文状态下输入
* 在标签后面都需要空一格再输入内容，否则会被当做文本处理

## 参考

* [DaringFireball](https://daringfireball.net/projects/markdown/syntax)
* [Markdown 语法说明](http://wowubuntu.com/markdown/)