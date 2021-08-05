---
priority: 0.6
title: RxSwift入门
excerpt: 响应式编程初体验
categories: [Summarise]
background-image: climb.jpeg
tags:
  - ReactiveX
  - RxSwift
  - 函数式编程
---

### 背景

响应式编程改变了开发者看待世界的方式，之前无论是面向过程还是面向对象编程，采用的都是拉式获取响应，也就是说如果想获得一个函数响应，你得去调用这个函数来获得其结果，无论是同步还是异步。

但响应式编程改变了这一过程，订阅发布是一种推式响应，只需要订阅一个响应，那么响应发生的时候会自动交回到订阅者的手上。这很自然的导致了获取相应是一个异步过程，因为订阅后订阅者不需要等待结果的发生。

响应式编程虽然是对观察者模式的扩大化应用，但衍生出来一个新的开发模式是一开始没想到的，从ReactiveCocoa到ReactiveX，再到Kotlin和SwiftUI只有短短几年时间，这一开发模式的发展速度超乎所有人的预料。也正是这一模式的成熟，MVVM架构等单向数据流架构也在慢慢成为主流。

###Observables & Subjects

每当我学习一些新的概念时，我总喜欢将它与类似的概念相比较。Observable和Subject之间的区别就像是太阳和月亮，Observable拥有要发送的数据，它本身就是数据源，而Subject只是转发它得到的数据，本身并不产生数据，虽然它们都能被订阅。

决定是Observable还是Subject看似很容易，但实际上在某些情形下却很容易弄混，因为很多时候需要Observable还是Subject取决于开发者怎样认识事物的抽象。比如一个网络请求，你可以把整个请求从发起到响应看做一个整体，那么这就是一个Observable，也可以把请求和响应分开，把请求看做Subject，当响应到来时将其转发给观察者。哪种做法更好呢，这取决于实际情况，也是体现开发者价值，对其进行取舍的时候。

Observable 和 Subjects有着相同的生命周期，一旦创建后，直到产生Error或者Complete事件才会触发Dispose事件并释放。当然手动强制dispose也是可行的，不过最长看到的还是利用DisposeBag进行自动释放，这种方式只是简单的利用的DisposeBag对象生命周期去释放disposable对象。因为DisposeBag对象往往是作为类成员变量，所以是要到类对象释放时才会释放disposable对象，如果你的本意不是这样，就需要手动管理disposable对象，并主动调用其dispose方法。

####Observable

Observable有一些创建方法，just、of、from、empty、never和range，基本一看就知道创建的是怎样的Observable，这里就不详细介绍了，除了of和from在数组参数的区别需要注意下外，没有什么需要详细讲的。

更通用的创建方式是Observable.create { observer in ... }，可以通过调用observer.onNext(value), observer.onComplete()和observer.onError()来发出事件，决定Observable发出的事件序列。

Observable有几个Traits，Single、Maybe和Completable，也很好理解，其实就是对Observable发出的元素个数和类型有一点限制，也是对几种典型的情况提供更方便的表示。

- Single 只会有两种时间 success 和 error，这里success 相当于Observable的next + complete
- Completable 只有complete 和 error
- Maybe 有可能是success 也有可能是compete和error，顾名思义，它有可能发出一个元素也可能没有，虽然这时都没发生错误

####Subject

就像只有一个太阳，却有大把卫星一样，绝大多数的常见对象都是传递事件而非产生事件，Subject或许是我们跟RxSwift最常打交道的对象。

Subject和Observable有一个很大的区别，Observable无论在什么时候订阅，都会把自己能够产生的的所有事件推给你，因为它们本身就是事件源，总是可以做到这一点。Subject呢，它们是转交收到的事件，所有订阅可能是在Subject收到事件之后，那么这个时候，根据如何处理订阅前的事件把Subject分为三类：PublishSubject，BehaviorSubject 和 ReplaySubject。

* PublishSubject 对订阅之前收到的事件很简单——忽略掉，所以订阅PublishSubject无法收到订阅之前的事件

* BehaviorSubject 不但能收到订阅后的事件，还能在订阅时收到上一次事件，并且BehaviorSubject 可以设置一个初值，这样即使订阅前还没发生任何事件，订阅时也可以收到这个初值。
* ReplaySubject 更进一步，可以设置事件个数，可以在订阅时收到已发出的指定个数的事件。

Observable和Subject是一种信息流的高度抽象，在程序开发中，足以表示任何状态变化，用户事件及各种响应。正确而恰当的在项目中使用Observable和Subject以及它们的各种Operator是件不容易的事情，但随着熟练度的提高，你会发现编码变得如此有趣。



