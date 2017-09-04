---
title: 【译】Angular组件间通信
layout: post
cdn: header-on
date: 2017-05-26 11:11:56
tags:
    - Angular
    - 翻译
---

> 阅读原文：[Getting Components to Communicate in Angular](https://www.infoq.com/articles/angular-component-communication) 

![Angular](/images/0526-05.jpeg)

> 核心要点：
> 1. 由于Angular是基于组件的应用架构，因此可以在组件之间进行进行数据传递。
> 2. 你可以使用`@Input()`将数据传递到子组件中。
> 3. 你可以使用`@Output()` 从子组件中获取数据。
> 4. 随着应用变得越来越复杂，父/子组件之间的数据传递将会变得越来越困难。
> 5. 你也可以采用外部数据存储方案（如ngrx）来构建大型应用程序。

组件是Angular的构建单元。在Angular应用程序中，每一个可视化的元素都由组件构成。基于组件的体系架构最大的优点就是可拆分，就像JavaScript中的函数一样，如果一个函数太复杂或承担了过多的职责，那么就可以将其拆分为多个函数，使一个函数只承担一个职责。

也即是说，当组件被拆分为更小的组件时，必须要保证各组件之间的数据能够来回传递。因此，为了使应用程序中的数据保持同步，正确的使用组件间通信的方法变得至关重要。幸运的是，Angular已经封装了能够实现组件间通信的方法。

![components](/images/0526-01.jpg)

当然也可以在根组件—`AppComponent`下构建整个应用，但如果这么做的话，`AppComponent`将会承担过多的职责。在基于组件的应用架构中，能够将复杂的组件拆分为承担单一职责的组件已然成为了一种最佳实践。

## 传递数据到子组件
在Angular中，使用`@Input`将父组件的数据传递给子组件。假设我们正在构建一个展示所有评论的应用，`AppComponent`将负责加载所有的评论数据，然后再将每一条评论的数据分别发送给子组件—`Comment`组件中。

通过`@Input() comment`将`comment`参数传递到子组件中。以下是整个组件的代码：
```JavaScript
@Component({
  selector: 'comment',
  templateUrl: './comment.component.html',
  styleUrls: ['./comment.component.css']
})
export class CommentComponent {
  @Input() comment;
}
```

现在就可以在代码的其他部分调用`Comment`组件，并传递它所期望接收的评论数据了。调用组件的代码如下：
```JavaScript
<comment [comment]="comment"></comment>
```

### 理解语法

首先，可以看到一个组件的自定义标签：`<comment></comment>`。如果你之前用过Angular的话，这些看起来就会很熟悉了。

接下来是使用`[comment]`进行数据绑定。刚开始看到属性外面的中括号，可能有会些困惑；但事实上，在数据传递中，中括号也并不是必须加的，如果不加中括号数据传递的话，则只会传递一个字符串到子组件的`@Input()`中。中括号的目的是为了告诉Angular这是一个属性绑定，并且值是动态传输的，因此Angular会动态的插入并传递数据到子组件中，而不仅仅只传递字符串。

最后，通过绑定`comment`属性，就获取了评论的值。Angular就会在当前组件中寻找`this.comment `属性的值，并将其传入到子组件-`Comment`组件中。

### 概念的组合

为了展示如下图所示的评论列表，可以将一些Angular中的概念组合起来使用，包括`*ngFor`。假设该实例已经加载了评论数据，它将作为组件类中的一部分，并且可以通过`this.comments`属性进行调用。
```JavaScript
<comment
  *ngFor="let comment of comments"
  [comment]="comment"></comment>
```
最终的渲染结果如下图所示：
![components](/images/0526-02.jpg)
使用了`Comment`组件的评论列表

## 捕获子组件事件
现在我们已经知道如何将评论传递到`Comment`组件中，但如果一个评论被删除了，该怎样将它从整个评论列表中移除呢？这会是非常棘手的事情，因为评论数据是保存在`Comment`组件的父组件—`AppComponent `中的。

使用`@Output()`是解决上述问题的一种方法，它可以利用`EventEmitter`帮助子组件触发能够被父组件捕获的事件。

要使子组件和父组件通信，首先要使用`@Output()`装饰器为子组件添加一个新的类属性。
```JavaScript
@Component({
  selector: 'comment',
  templateUrl: './comment.component.html'
})
export class CommentComponent {

  @Input() private comment;
  @Output() private onDelete = new EventEmitter();

  deleteComment() {
    this.onDelete.emit(this.comment);
  }
}
```

如上述`Comment`组件的代码片段所示，当点击删除按钮的时候，就会调用`deleteComment()`方法，并且也做好了捕获来自父组件响应事件的准备。
```JavaScript
<button (click)="deleteComment()">Delete</button>
```

## 从父组件中捕获事件
在根组件—`AppComponent`组件中，需要一个新的方法来处理删除评论的行为。
```JavaScript
@Component({
  selector: 'app-root',
  templateUrl: './app.component.html'
})
export class AppComponent {

  /**[code omitted]**/

  onCommentDelete(comment) {
    // logic to remove comment from comments array
  }
}
```
在视图层，仅仅需要告诉Angular当`onDelete`事件被触发时，立即调用`onCommentDelete()`方法即可。
```JavaScript
<comment
      *ngFor="let comment of comments"
      [comment]="comment"
      (onDelete)="onCommentDelete($event)"></comment>
```

如下图所示就是该应用完成删除功能之后的样子：
![components](/images/0526-03.jpg)
现在有了Angular的`@Output()`装饰器，就能够删除评论了。

## 另外一种获取数据的方式
到目前为止，我们仅知道了如何在父/子组件之间进行通信。尽管这种方法已经能够满足大多数的需求，但随着应用规模的增长，维护父/子组件间的通信也变得越来越复杂。对于大型应用，使用数据存储（`data store`）来减少单个组件的工作量是有效的方案。数据存储是一个中心存储的方案，它可以在每一个组件需要数据时就调用一部分数据给该组件使用。每一个组件可以在数据存储中订阅他们所需要的数据，而不用手动的将数据传递到子组件，以此来减少父/子组件中来回传递数据的压力。

如果你熟悉React的话，那么这正是Redux解决的问题。在Angular中，可以使用ngrx库来解决，这个库也是受到了Redux的启发。但这两个库还有很多关键的区别：类型和可观察对象（`observables `）。ngrx库严重依赖`TypeScript`的生态系统，因此会比Redux更加繁重一点，但是却很容易调试和进行问题的追踪。

当使用ngrx时，数据的状态是单个不可变的数据结构。为了改变数据的状态，可以使用行为的方式来调用函数。这些行为同时又会反过来告诉`reducers`，哪些是纯函数，哪些数据状态可改变。通过这种方式，应用就可以拥有新的数据状态了，而订阅这些数据的组件也可以立即收到新的数据。当组件收到新的数据时，它们会立即重新渲染，以时刻保持视图层的数据和数据存储中的数据相同。

这样做对于开发又意味着什么呢？

虽然在概念上能够直接将外部数据传递到组件中，而不用通过父子关系来传递，但这样做对于项目的开发到底意为着什么呢？

![components](/images/0526-04.jpg)

假设想在两个不同的地方展示评论数怎么办？如果仅仅依赖组件层级间的数据传递会很复杂，因此这也就是中心存储`ngrx`的意义所在。

在许多大型应用中，直接向组件传递数据在许多方面都是很有用的。由于组件需要处理越来越多的任务，且再添加额外的任务（像‘持续追踪传递到子组件的数据’这样的任务）也变得很复杂。如果不能把传输数据的负担交给外部的数据存储，那么当你维护大型Angular应用程序时，可就得时刻小心了。

在上面的例子里，一个外部数据存储（未在图中画出）可以用来把信息传给`SidebarComponent`，表示有多少条评论可用——“有两条评论”，同时还把具体的评论内容发给`CommentList`组件。这些数据完全绕过了父组件`AppComponent`。

## 快速回顾一下Angular的组件通信机制

通过上边这些内容，已经对Angular的组件之间如何通信有了一些了解。知道了怎样把数据从一个父组件传入一个子组件，也知道了怎么使用Angular中的`@Input()`装饰器。

也知道了怎样使用`@Output()`装饰器，以及如何使用`EventEmitter`把数据从一个子组件发回给一个父组件。

除此之外，当应用程序规模不断增长的时候，就可以采用`ngrx`来减轻父子组件之间的压力。通过使用一个单独的数据存储，就可以让各个组件自己获取数据，而不必通过多层子组件把数据传到最终的目标。这就使大规模应用程序的维护变得容易多了。这些原理同样适用于React之类的库，他们也用非常类似的办法解决了相同的问题。

你可以在这个[链接](https://sergiocruz.github.io/ng-sample-comments/)上看到该系统的演示程序，相应的[代码](https://github.com/sergiocruz/ng-sample-comments)也可以在这里下载到。

## 如何进一步学习
建议大家可以再看看 [Angular关于组件交互的文档](https://angular.io/docs/ts/latest/cookbook/component-communication.html)，进一步了解底层机制，以及随着框架的快速演进时，最新的语法和最佳实践。

最后，你也可以再读读ngrx官方网站上的ngrx文档，或者也推荐看看我用ngrx构建的[示例程序](https://github.com/sergiocruz/ng-sample-comments/tree/ngrx)。
