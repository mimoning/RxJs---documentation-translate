# `RxJS` 简介
RxJS 自称是使用可观察序列来构建异步和基于事件编程的 JS 库。

其提供了一个核心类型：`Observable`
> Think of RxJS as Lodash for events. (对此我深表疑问)

主要概念：
1. `Observable`：代表了未来的值或事件的可调用集合
2. `Observer`：一个回调集合，知道怎么监听 Observable 分发的值
3. `Subscription`：代表了 Observable 的调用返回，主要用于取消调用
4. `Operators`：是一些提供了处理操作集合的函数式编程风格的纯函数
5. `Subject`：等同于事件发射器（EventEmitter），并且是唯一的方式多次广播值或时间给多个观察者（Observers）
6. `Schedulers`：是控制并发的集中分发器，允许我们协调计算

## `RxJS` 特性

1. 纯度（Purity）
RxJs 的强大一定程度上在于其使用纯函数的方式来产生值，这意味着你的代码能够更少的出错

2. 流程（Flow）
RxJS 有整个范围的操作函数用来帮助你控制事件流穿过你的可观察对象（observables）

3. 值（Values）
你可以轻易的在 observables 之间传递值

# `RxJS` 详解
***
## 可观察对象（Observable）
可观察对象是多种值的延迟推送集合

|    | Single | Multipe |
|----|--------|---------|
| Pull | Function | Iterator |
| Push | Promise  | Observable |

### Pull versus Push（数据拉取和推送的比较）
拉和推是数据的提供者和数据的用户通信的两种不同的协定

1. 拉
在拉数据的体系中，用户决定什么时候从数据提供者处接受数据。数据提供者并不清楚什么时候数据会被分发给用户

其中所有的 js Function 都是一个拉数据的系统。函数是数据的提供者，且在代码中调用这个函数则是通过一个简单的 return 关键字消耗这个数据

ES2015 提供了 generator 函数和迭代器（iterators）（function*），另外一个拉的系统。诸如`iterator.next()`此类的代码是用户，从迭代器中拉出不同类型的值

|      | Producer | Consumer |
|------|----------|----------|
| pull | 被动：当被请求的时候产生数据 | 主动：决定什么时候请求数据 |
| push | 主动：根据自己的节奏产生数据 | 被动：根据接收到的数据做出反应 |

2. 推
在推数据的体系中数据提供者决定什么时候发送数据给用户。用户并不知道什么时候会受到这个数据

`Promises` 是 js 中最普遍的推数据类型的例子。一个 Promise（数据提供者）分发一个解析值给一个已注册的回调函数（用户）。但是和函数不同的是，Promise 恰恰是负责确定什么时候这个值会“推”给回调函数

RxJS 采用了观察者模式，一种新的 js 推数据体系。一个 Observable 是多种类型值的生产者，将它们“推”给 Observers（用户）

* `Function` 是一种调用时同步返回一个值的懒计算
* `generator` 是一种调用时同步返回 0 至无限个（有潜力的）值的懒计算
* `Promise` 是一个可能（也可能不）最终返回一个值的计算
* `Observable` 是一个在一直向前调用时可以同步或异步返回 0 至无限个（有潜力的）值的懒计算

### 可观察对象（Observables）就如函数的概括
与通俗的声称相反，可观察对象既不像事件发射器也不像多类型值的 Promises。可观察对象可能在一些情境下表现的跟时间发射器一样，即当使用 RxJS Subjects 做多次广播的情景下，但是它们通常表现的跟事件发射器不一样

> Observables are like functions with zero arguments, but generalize those to allow multiple values.（可观察对象就像没有参数的函数，但是把函数归纳为允许多种值）

General:

    function foo() {
      console.log('Hello);
      return 42;
    }

    var x = foo.call();
    console.log(x);
    var y = foo.call();
    console.log(y);
    // expect output
    "Hello"
    42
    "Hello"
    42

With Observables:

    var foo = Rx.Observabe.create(function(observer) {
      console.log('Hello');
      observer.next(42);
    });

    foo.subscribe(function(x){
      console.log(x);
    });

    foo.subscribe(function(y){
      console.log(y);
    });
    // output is the same
    "Hello"
    42
    "Hello"
    42

这是由于函数和可观察对象都是懒计算。如果你不调用函数，`console.log('Hello')` 并不会发生。可观察对象也是如此，如果你不“调用（call）”它（通过 subscribe），`console.log('Hello')` 也不会发生

另外，“调用（calling）”或“订阅（subscribing）”是一个独立的操作：两个函数调用触发了两个隔离的副作用，两个可观察对象的订阅也出发了两个隔离的副作用

与事件发射器共享副作用和不管订阅者是否存在的执行欲望相对的是，可观察对象不共享执行且是懒的
> Subscribing to an Observable is analogous to calling a Function.（订阅一个可观察对象相当于调用一个函数）
> Observables are able to deliver values either synchronously or asynchronously.（可观察对象能够同步和异步的传递数据）

那么，可观察对象和普通函数的区别是什么？**Observables can "return" multiple values over time（可观察对象随时间推移可以“返回”多个值）**
如，函数不能做到：

    function foo() {
      console.log('Hello');
      return 42;
      return 100; // dead code.
    }

但，可观察对象可以做到：

    var foo = Rx.Observable.create(function(observer) {
      console.log('Hello');
      observer.next(42);
      observer.next(100); // "return" another value
      observer.next(200); // "return" yet another
    });

    console.log('before');
    foo.subscribe(function(x) {
      console.log(x);
    });
    console.log('after');
    // 同步的输出
    "before"
    "Hello"
    42
    100
    200
    "after"

且，你也能异步的“返回”值

    var foo = Rx.Observable.create(function(observer) {
      console.log('Hello');
      observer.next(42);
      observer.next(100);
      observer.next(200);
      setTimeout(() => {
        observer.next(300); // happens asynchronously
      }, 1000);
    });

    console.log('before');
    foo.subscribe(function(x) {
      console.log(x);
    });
    console.log('after');

    // with output
    "before"
    "Hello"
    42
    100
    200
    "after"
    300

结论： 
* func.call() means "give me one value synchronously"
* observable.subscribe() means "give me any amount of values, either synchronously or asynchronously"

### 可观察对象的剖析
* 创建：`Rx.Observable.create` 或一个创建 operator
* 订阅：被 Observer 订阅
* 执行：发送`next`/`error`/`complete`通知给 Observer
* 处理：接受通知之后它们的执行可以被处理

#### 创建可观察对象
`Rx.Observable.create` 是 `Observable` 构造函数的别名，且接受一个参数：`subscribe` 函数

创建一个名为 `observable` 的可观察对象

    var observable = Rx.Observable.create(function subscribe(observer) {
      var id = setInterval(() => {
        observer.next('Hi');
      }, 1000); // 每间隔 1 秒钟给你一个 ‘Hi’
    })

> Observables can be created with create, but usually we use the so-called creation operators, like of, from, interval, etc.

#### 订阅可观察对象

`observable` 的订阅：`observable.subscribe(x => console.log(x))`

订阅时的 subscribe 方法和创建时的 subscribe 方法具有同样名字并不是一个巧合。虽然在库中它们并不相同，但是你可以把它们看做概念上相等

这体现了 `subscribe` 的调用并不在同一可观察对象的不同观察者之间共享。当一个观察者调用 `observable.subscribe`，`observable.creare` 中的 `subscribe` 函数会被该观察者执行。每次 `observable.subscribe` 的调用都会为它的观察者触发它的独立创建

> Subscribing to an Observable is like calling a function, providing callbacks where the data will be delivered to.

`observable.subscribe` 和事件绑定 API 如：`addEventListener`/`removeEventListener`大不相同

*一个 `subscribe` 调用仅仅是开始一个可观察对象执行过程的方法，并且在执行过程中传递值或事件给一个观察者*

#### 执行可观察对象
在 `Observable.create(function subscribe(observer) {...})` 里面的代码代表了”可观察对象的执行过程“，懒计算仅仅发生在每个观察者订阅时。执行过程随时间的推移同步或异步的产生多个值

三种可观察对象的执行过程可传递的类型

1. "Next" 通知：发送一个值，如：Number、String、Object, etc.
2. "Error" 通知：发送一个 js 错误
3. "Complete" 通知：不发送值

> In an Observable Execution, zero to infinite Next notifications may be delivered. If either an Error or Complete notification is delivered, then nothing else can be delivered afterwards.

#### 处理可观察对象
由于可观察对象的执行可以是无限的，且观察者要在有限的时间内终止执行是很常见的，所以我们需要一个 API 来取消一个执行过程

因为每个执行过程是被一个观察者所独占的，一旦观察者不再接收数据了，它必须要有方法可以停止执行过程，以防计算资源和内存的浪费

当 `observable.subscribe` 被调用时，观察者可以拿到新建的可观察对象执行，并且这个调用返回了一个对象，即：`Subscription`：

    var subscription = observable.subscribe(x => console.log(x))

订阅（Subscription）代表了不断的执行过程，且有一个最小的 API 可以让你取消这个执行过程：`subscription.unsubscribe()`

    var observable = Rx.Observable.from([10, 20, 30]);
    var subscription = observable.subscribe(x => console.log(x));
    // later:
    subscription.unsubscribe();

> When you subscribe, you get back a Subscription, which represents the ongoing execution. Just call unsubscribe() to cancel the execution.

当使用 `create()` 创建可观察对象时，每个可观察对象必须定义如何处理执行过程的资源。你可以通过从里面的 `function subscribe（）` 返回一个普通的 `unsubscribe` 函数来做到这一点

例：

    var observable = Rx.Observable.create(function subscribe(observer) {
      // Keep track of the interval resource
      var intervalID = setInterval(() => {
        observer.next('hi');
      }, 1000);

      // Provide a way of canceling and disposing the interval resource
      return function unsubscribe() {
        clearInterval(intervalID);
      };
    });

## 观察者（Observer）
观察者是可观察对象传递数据的消费者。观察者只是一个回调函数的集合，每个可观察对象发送的通知类型有：`next`，`error`，`complete`

如下为一个典型的观察者 object 例子：

    var observer = {
      next: x => console.log(`Observer got a next value ${x}`),
      error: err => console.error(`Observer got an error ${err}`),
      complete: () => console.log('Observer got a complete notification'),
    };

要使用观察者的话，将他提供给可观察对象的 `subscribe` 方法

    observable.subscribe(observer);

> Observers are just objects with three callbacks, one for each type of notification that an Observable may deliver.

## 订阅（Subscription）
订阅是一个表示一次性资源的对象，常常是指可观察对象的执行。一个订阅对象有一个重要的方法，即 `unsubscribe`。这个方法不接受任何参数，且仅仅只是处理订阅对象所保持的资源。在更早版本的 RxJS 中，订阅对象（Subcription）被称为“一次性对象（Disposable）”

    var observable = Rx.Observable.interval(1000);
    var subscription = observable.subscribe(x => console.log(x));
    // Later
    // This cancels the ongoing Observable execution which
    // was started by calling subscribe with an Observer.
    subscription.unsubscribe();

> A Subcription essentially just has an `unsubcribe()` function to release resources or cancel Observable executions

订阅对象也能放在一起以至于一个订阅对象的 `unsubscribe()` 函数的调用可以结束订阅多个订阅对象。你可以通过“添加（adding）”一个订阅到另一个来做到这一点

    var observable1 = Rx.Observable.interval(400);
    var observable2 = Rx.Observable.interval(300);

    var subscription = observable1.subscribe(x => console.log(`first: ${x}`));
    var childSubscription = observable2.subscribe(x => console.log(`second: ${x}`));

    subscription.add(childSubscription);

    setTimeout(() => {
      // Unsubscribes Both subscription and childSubscription
      subscription.unsubscribe();
    }, 1000);

订阅对象还有一个 `remove(otherSubscription)` 方法，用于撤销一个子订阅对象的添加

## 主题（Subject）
主题对象是一种特殊类型的可观察对象。它允许值可以被多播到很多观察者。简单的可观察对象是单播（每个已订阅的观察者拥有一个独立的可观察对象的执行），主题对象是多播

> A Subject is like an Observable, but can multicast to many Observers. Subjects are like EventEmitters: they maintain a registry of many listeners.

**每个主题对象都是一个可观察对象。** 对于一个主题对象，你可以订阅 `subscribe` 它，可以提供给一个观察者，然后这个观察者可以正常的开始接收值。从观察者视角来看，它并不能得知可观察对象的执行是来自于一个单播可观察对象还是主题对象

在主题对象的内部实现中，`subscribe` 并没有开始一个新传递数据的执行过程。它仅仅是将观察者注册到一个观察者列表中，类似其他库或语言中 `addListener` 做的那样

**每个主题对象都是一个观察者。** 它是一个有 `next(v)`，`error(e)` 和 `complete()` 方法的对象。要反馈一个新的值个主题对象，只需要调用 `next(theValue)`，且它会被多播到被注册给监听这个主题对象的观察者们

作为可观察对象

    var subject = new Rx.Subject();

    subject.subscribe({
      next: v => console.log(`observerA: ${v}`),
    });
    subject.subscribe({
      next: v => console.log(`observerB: ${v}`),
    });

    subject.next(1);
    subject.next(2);

    // output
    observerA: 1
    observerB: 1
    observerA: 2
    observerB: 2

作为观察者

    var subject = new Rx.Subject();

    subject.subscribe({
      next: v => console.log(`observerA: ${v}`)
    });
    subject.subscribe({
      next: v => console.log(`observerB: ${v}`)
    });

    var observable = Rx.Observable.from([1, 2, 3]);

    observable.subscribe(subject); // You can subscribe providing a Subject

通过上面的方法，我们实质上仅仅是通过主题对象将一个单播的可观察对象执行变为了多播。例子展示了将一些可观察对象的执行共享给多个观察者的唯一方法就是主题对象

还有一些特殊类型的主题对象：`BehaviorSubject`，`ReplaySubject` 和 `AsyncSubject`

### 多播可观察对象
一个“多播可观察对象”通过一个有着多个订阅者的主题对象来发送通知，因为一个简单的“单播可观察对象”只发送通知到一个观察者

> A multicasted Observable uses a Subject under the hood to make multiple Observers see the same Observable execution.

在内部封装中，多播操作是这样工作的：观察者订阅一个底层主题对象，且这个主题对象订阅了源可观察对象。接下来的例子与之前使用 `observable.subscribe(subject)` 的例子有些相似：

    var source = Rx.Observable.from([1, 2, 3]);
    var subject = new Rx.Subject();
    var multicasted = source.multicast(subject);

    // These are, under the hood, `subject.subscribe({...})`:
    multicasted.subscribe({
      next: v => console.log(`observerA: ${v}`)
    });
    multicasted.subscribe({
      next: v => console.log(`observerB: ${v}`)
    });

    // This is, under the hood, `source.subscribe(subject)`:
    multicasted.connect();

`multicast` 返回了一个看上去和普通可观察对象一样的可观察对象，但是当它被订阅时工作起来跟主题对象一样。`multicast` 返回了一个 `ConnectableObservable`，这是一个有着 `connect()` 方法的可观察对象

`connect()` 方法对于确定被分享的可观察对象执行从何开始而言非常重要。因为 `connect()` 在底层使用了 `source.subscribe(subject)`，`connect()` 返回了一个你可以由之取消订阅来取消被分享的可观察对象的执行的订阅对象

#### 引用计数
调用 `connect()` 来手动处理订阅对象是非常笨重的。通常我们希望能够当第一个观察者出现时*自动的*连接，且当最后一个观察者取消订阅时*自动的*取消被分享的执行

思考一下列表：

1. 第一个观察者订阅了多播可观察对象
2. 多播可观察对象已连接
3. 下一个值 0 被分发到第一个观察者
4. 第二个观察者订阅了多播可观察对象
5. 下一个值 1 被分发到第一个观察者
6. 下一个值 1 被分发到第二个观察者
7. 第一个观察者取消订阅多播可观察对象
8. 下一个值 2 被分发到第二个观察者
9. 第二个观察者取消订阅多播可观察对象
10. 多播对象的连接被取消订阅

要达到这样需要清晰的调用 `connect()`，代码如下：

    var source = Rx.Observable.interval(500);
    var subject = new Rx.Subject();
    var multicasted = source.multicast(subject);
    var subscription1, subscription2, subscriptionConnect;

    subscription1 = multicasted.subscribe({
      next: v => console.log(`observerA: ${v}`)
    });
    // We should call `connect()` here, because the first
    // subscriber to `multicasted` is intersted in consuming values
    subscriptionConnect = multicasted.connect();

    setTimeout(() => {
      subscription2 = multicasted.subscribe({
        next: v => console.log(`observerB: ${v}`)
      });
    }, 600);

    setTimeout(() => {
      subscription1.unsubscribe();
    }, 1200);

    // We should unsubscribe the shared Observable execution here,
    // because `multicasted` would have no more subscribers after this
    setTimeout(() => {
      subscription2.unsubscribe();
      subscriptionConnect.unsubscribe();
    }, 2000);

如果想要避免直接的调用 `connect()` 方法，我们可以使用可连接可观察对象的 `refCount()` 方法（引用计数）。它可以返回一个保持追踪自身有多少订阅者的可观察对象。当订阅者的数量从 0 增加到 1，它会为我们调用 `connect()` 方法，并开始被分享的执行。只有当订阅者的数量从 1 减少到 0 他才会取消订阅，并停止继续执行

> *refCount* makes the multicasted Observable automatically start executing when the first subscriber arrives, and stop executing when the last subscriber leaves.

示例如下：

    var source = Rx.Observable.interval(500);
    var subject = new Rx.Subject();
    var refCounted = source.multicast(subject).refCount();
    var subscription1, subscription2, subscriptionConnect;

    // This calls `connect()`, because it is the first subscriber to `refCounted`
    console.log('observerA subscribed');
    subscription1 = refCounted.subscribe({
      next: v => console.log(`observerA: ${v}`)
    });

    setTimeout(() => {
      console.log('observerB subscribed');
      subscription2 = refCounted.subscribe({
        next: v => console.log(`observerB: ${v}`)
      });
    }, 600);

    setTimeout(() => {
      console.log('observerA unsubscribed');
      subscription1.unsubscribe();
    }, 1200);

    // This is when the shared Observable execution will stop, because `refCounted` would
    // have no more subscribers after this
    setTimeout(() => {
      console.log('observerB unsubscribed');
      subscription2.unsubscribe();
    }, 2000);

输出如下：

    observerA subscribed
    observerA: 0
    observerB subscribed
    observerA: 1
    observerB: 1
    observerA unsubscribed
    observerB: 2
    observerB: 3 // 原文示例中没有，但是实际操作中打印出来了
    observerB unsubscribed

`refCount()` 方法只存在于可连接可观察对象，且它返回了一个可观察对象而不是另一个可连接可观察对象

### 行为主题（BehaviorSubject）
行为主题对象是主题对象的一个变种，它有个概念叫做“当前值”。它存储了最后发送给其数据使用者的值，并且每当一个新的观察者订阅它，这个观察者会立刻从行为主题对象处收到“当前值”

> BehaviorSubjects are useful for representing "values over time". For instance, an event stream of birthdays is a Subject, but the stream of a person's age would be a BehaviorSubject.

接下来的例子中，行为主题对象被初始化且当第一个观察者订阅时会接收到一个 0 值。第二个观察者会在订阅时收到一个值 2，即使它是在 2 已经发送之后订阅的

    var subject = new Rx.BehaviorSubject(0); // 0 is the initial value

    subject.subscribe({
      next: v => console.log(`observerA: ${v}`)
    });

    subject.next(1);
    subject.next(2);

    subject.subscribe({
      next: v => console.log(`observerB: ${v}`)
    });

    subject.next(3);

输出：

    observerA: 0
    observerA: 1
    observerA: 2
    observerB: 2
    observerA: 3
    observerB: 3

### 重播主题对象（ReplaySubject）
重播主题对象和行为主题对象有一些相似之处，它可以发送老数据给新的订阅者，但是它还能*保持*（record）一些可观察对象的执行过程

> A *ReplaySubject* records multiple values from the Observable execution and replays them to new subscribers.

当你创建一个重播主题对象，你可以指定有多少个值需要重播

    var subject = new Rx.ReplaySubject(3); // 缓存 3 个值给新的订阅者

    subject.subscribe({
      next: v => console.log(`observerA: ${v}`)
    });

    subject.next(1);
    subject.next(2);
    subject.next(3);
    subject.next(4);

    subject.subscribe({
      next: v => console.log(`observerV: ${v}`)
    });

    subject.next(5);

输出如下：

    observerA: 1
    observerA: 2
    observerA: 3
    observerA: 4
    observerB: 2
    observerB: 3
    observerB: 4
    observerA: 5
    observerB: 5

你也可以指定一个毫秒级的窗口时间或缓冲大小，来决定记录的值有多旧。如下示例，我们使用了一个大小为 100 的缓冲，但是窗口时间只有 500 毫秒

    var subject = new Rx.ReplaySubject(100, 500 /* windowTime */);

    suject.subscribe({
      next: v => console.log(`observerA: ${v}`)
    });

    var i = 1;
    setInterval(() => subject.next(i++), 200);

    setTimeout(() => {
      subject.subscribe({
        next: v => console.log(`observerB: ${v}`)
      });
    }, 1000);

猜猜看输出结果？

### 异步主题对象（AsyncSubject）
异步主题对象只有在可观察对象最后一个值被发送给观察者以及执行完毕时才体现的变种

    var subject = new Rx.AsyncSubject();

    subject.subscribe({
      next: v => console.log(`observerA: ${v}`)
    });

    subject.next(1);
    subject.next(2);
    subject.next(3);
    subject.next(4);

    subject.subscribe({
      next: v => console.log(`observerB: ${v}`)
    });

    subject.next(5);
    subject.complete();

输出如下：

    observerA: 5
    observerB: 5

异步主题对象和 `last()` 操作函数有些相似，它等待 `complete` 通知的来了才传递值

## 操作（Operators）
RxJS 最有用的地方在于其操作函数（Operators），虽然它的基础是可观察对象。操作函数是复杂的异步代码能够轻易的编排到声明的方法中必不可少的部分

### 什么是操作函数
操作函数式可观察对象类型的方法，如：`.map(...)`，`.filter(...)`，`.merge(...)`，等。当被调用时，它们并不改变原本的可观察对象的存在。它们其实返回了一个新的可观察对象，这个可观察对象的逻辑是基于第一个可观察对象的

> An Operator is a function which creates a new Observable based on the current Observable. This is a pure operation: the previous Observable stays unmodified.

一个操作函数是一个可以将可观察对象作为输入然后生成另一个可观察对象作为输出的纯函数。订阅这个输出的可观察对象会同时订阅输入的可观察对象。如下例子中，我们创建了一个普通的操作函数可以将输入的可观察对象处接收的值都乘以10：

    function multiplyByTen(input) {
      var output = Rx.Observable.create(function subscribe(observer) {
        input.subscribe({
          next: v => observer.next(10 * v),
          error: err => observer.error(err),
          complete: () => observer.complete(),
        });
      });
      return output;
    }

    var input = Rx.Observable.from([1, 2, 3, 4]);
    var output = multiplyByTen(input);
    output.subscribe(x => console.log(x));

输出如下：

    10
    20
    30
    40

可以看到，一个输出可观察对象的订阅会导致输入可观察对象的订阅。我们将之称为“操作订阅链（operator subscription chain）”

### 实例操作函数对比静态操作函数
**什么是实例操作函数？**通常来说，我们提到操作函数的时候其实就是指实例操作函数，即可观察对象实例的一些方法。举个例子，如果操作函数 `multiplyByTen` 是一个官方的操作函数，他看上去应该是这样的：

    Rx.Observable.prototype.multiplyByTen = function multiplyByTen() {
      var input = this;
      return Rx.Observable.create(function subscribe(observer) {
        input.subscribe({
          next: v => observer.next(10 * v),
          error: err => observer.error(err),
          complete: () => observer.complete(),
        });
      });
    }

> Instance operators are functions that use the *this* keyword to infer what is the input Observable.

注意为什么输入可观察对象不再是函数的参数，它被假定是 `this` 对象。以下是我们使用实例操作函数的例子：

    var observable = Rx.Observable.from([1, 2, 3, 4]).multiplyByTen();

    observable.subscribe(x => console.log(x));

**什么是静态操作函数？**除了实力操作函数之外，静态操作函数更加直接的贴近可观察对象类。一个静态操作函数不使用 `this` 关键字，它依赖于它的参数

> Static operators are pure functions attached to the Observable class, and usually are used to create Observables from scratch.

最普遍的静态操作函数是所谓的*创建操作函数*。不同于将输入可观察对象转换为输出可观察对象，它们仅仅接受一个跟可观察对象无关的参数，如：一个数值，然后创建一个新的可观察对象

一个典型的静态操作函数例子就是 `interval` 函数。它接受一个数值（而非可观察对象）为输入的参数，且产生一个可观察对象作为输出：

    var observable = Rx.Observable.interval(1000 /* number of milliseconds */);

另外一个创建操作函数的例子是 `create`，它在之前的例子中已经被广泛的使用了

然而，静态操作函数和简单的创建函数有着本质的区别。一些合并操作函数可能是静态的，如：`merge`，`combineLatest`，`concat`等。静态操作函数可以接受多个可观察对象作为参数而非仅仅一个，这也解释了其与创建函数之间的区别。如下例：

    var observable1 = Rx.Observable.interval(1000);
    var observable2 = Rx.Observable.interval(500);

    var merged = Rx.Observable.merge(observable1, observable2);

### 宝石图（Marble diagrams）
要解释操作函数式怎么工作的，文字已经不足以表达了。很多操作函数和时间相关，如延时（delay），样本（sample），掐死（throttle）或者是其他不同方式推迟数据的发送。图表时常是一个很好的工具来体现这些。宝石图是操作函数作用的一些虚拟代表，包含输入的可观察对象，操作函数及其参数，以及输出的可观察对象

> In a marble diagram, time flows to the right, and the diagram describes how values (“marbles”) are emitted on the Observable execution.

如下图所示：

![](./宝石图示例.svg)

这个文档广泛的使用了宝石图来解释操作函数是怎么样工作的。它们在其他情境下也非常有用，如白板、甚至是在单元测试中（作为 ASCII 图表）

### 选择一个操作函数

详见官方文档

## 调度（Scheduler）

什么是调度？一个调度控制着什么时候一个订阅开始和什么时候通知被分发。它由三个组件组成：

  * 一个调度是一个数据结构。它知道怎么基于优先级或其他规则来存储和排列任务
  * 一个调度是一个执行过程上下文。它表示任务在哪里以及在何时被执行（如：立即执行，在例如 `setTimeout`、`process.nextTick`或动画等回调机制里执行）
  * 一个调度有一个（虚拟的）时钟。它提供了一个代表时间的方法 `now()`。任务只遵循这个时钟所表示的时间在一个特定的调度上被安排

> A Scheduler lets you define in what execution context will an Observable deliver notifications to its Observer.

如下示例，我们使用一个简单的同步发送值 `1` `2` `3` 的可观察对象来作为例子，使用操作函数 `observeOn` 来指定 `async` 调度来分发这些值

    var observable = Rx.Observable.create(observer => {
      observer.next(1);
      observer.next(2);
      observer.next(3);
      observer.complete();
    })
    .observeOn(Rx.Scheduler.async);

    console.log('just before subscribe');
    observable.subscribe({
      next: x => console.log(`got value ${x}`),
      error: err => console.error(`something wrong occurred: ${err}`),
      complete: () => console.log('done'),
    });
    console.log('just after subscribe');

输出：

    just before subscribe
    just after subscribe
    got value 1
    got value 2
    got value 3
    done

注意通知 `got value ...` 在 `just after subscribe` 之后是怎么被分发的，与前面我们看到的默认行为有怎么样的不同。这是因为 `observeOn(Rx.Scheduler.async)` 在 `Observable.create` 和最终的观察者之间引入了一个代理观察者。让我们把一些标识符重命名来凸显差异：

    var observable = Rx.Observable.create(proxyObserver => {
      proxyObserver.next(1);
      proxyObserver.next(2);
      proxyObserver.next(3);
      proxyObserver.complete();      
    })
    .observeOn(Rx.Scheduler.async);

    var finalObserver = {
      next: x => console.log(`got value ${x}`),
      error: err => console.error(`something wrong occurred: ${err}`),
      complete: () => console.log('done'),
    };

    console.log('just before subscribe');
    observable.subscribe(finalObserver);
    console.log('just after subscribe');
