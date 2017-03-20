---
title: Immutable.js和函数式编程概念介绍
---

​		了解功能数据结构及其在Facebook的流行图书馆JavaScript概述中的用途：Immutable.js

功能规划在过去几年一直在上升。诸如Clojure，Scala和Haskell之类的语言使命令式程序员的眼睛带来了一些有趣的技术，可以在某些用例中提供显着的好处。Immutable.js旨在通过一个简单直观的API为JavaScript带来一些好处。跟随我们通过这个概述，了解一些这些好处，以及如何让他们在你的项目计数！

------

## 介绍：不变性和Immutable.js的情况

虽然函数式编程不仅仅是不变性，许多函数式语言非常强调不可变性。一些，如Clean和Haskell，对数据可以被突变的方式和时间设置硬编译时限制。许多开发商都被这个推迟了。对于那些忍受最初冲击的人，**开始出现解决问题的新模式和方法**。特别地，数据结构是新人对功能范例的主要冲突点。

最后，**不可变对可变数据结构的问题归结为冷，硬的数学**。算法分析告诉我们哪些数据结构最适合不同类型的问题。然而，语言支持可以在很大程度上帮助使用和实现这些数据结构。JavaScript，由于是一种多范式语言，为可变和不可变的数据结构提供了肥沃的基础。其他语言，如C，可以实现不可变的数据结构。然而，语言的限制可以使其使用麻烦。

那么什么是一个突变呢？变异是对数据或包含它的数据结构的原位更改。不变性，另一方面，每当需要改变时，这样的数据和数据结构的副本。

![不变树](https://cdn.auth0.com/blog/immutablejs/tree.svg)

> 从[维基百科](https://en.wikipedia.org/wiki/Persistent_data_structure#/media/File:Purely_functional_tree_after.svg)采取的[图象](https://en.wikipedia.org/wiki/Persistent_data_structure#/media/File:Purely_functional_tree_after.svg)。

那么什么是功能数据结构的原则，特别是什么使得不可变性如此重要？此外，什么是他们的正确的用例？这些是我们将在下面探讨的一些问题。

> **注意：**您可能不知道这一点，但您可能已经在您的JavaScript代码中使用某些函数式编程结构。例如，`Array.map`对数组中的每个项应用一个函数并返回一个新数组，而不修改过程中的原始数据。函数式编程作为一个范例，支持第一类函数，可以传递给返回现有数据的新版本的算法。这其实是什么`Array.map`。这种处理数据的方式有利于组合，在功能编程中的另一个核心概念。

## 关键概念

这些是功能编程背后的一些关键概念。希望在本文中，您将了解这些概念如何适用于Immutable.js和其他函数库的设计和使用。

### 不变性

不变性是指数据（以及管理它的数据结构）在实例化之后的行为：**不允许突变**。在实践中，突变可以分为两组：可见突变和不可见突变。**可见突变**是**修改数据**或包含它的数据结构的方式，可以通过API **由外部观察者注意到**。另一方面，**隐形突变****是不能通过API注意到的变化**（缓存数据结构是这方面的一个很好的例子）。在某种意义上，不**可见的突变可以被认为是副作用**（我们探索这个概念和它的意思下面）。********

```js
var list1 = Immutable.List.of(1, 2);
// We need to capture the result through the return value:
// list1 is not modified!
var list2 = list1.push(3, 4, 5);
```

有趣的好处出现在开发人员（和编译器/运行时）可以肯定数据不能改变：

- 多线程**锁定**不再是一个问题：由于数据不能更改，**因此不需要锁**来同步多个线程。
- 持久性（下面探讨的另一个关键概念）变得更容易。
- 复制成为一个**常量操作**：复制只是创建对数据结构的现有实例的新引用。
- 在某些情况下可以优化值比较：当运行时或编译器可以确保在加载或编译期间某个实例仅在指向同一引用时相等时，**深值比较可以成为引用比较**。这被称为*实习*，通常只适用于在编译或加载时可用的数据。这种优化也可以手动执行（与React和Angular一样，在最后的旁边部分解释）。

#### 您已经使用了不可变的数据结构：字符串

**JavaScript中的字符串是不可变的**。String原型中的所有方法都执行读取操作或返回新字符串。

一些JavaScript运行时利用它来执行实现：在加载时或在JIT编译期间，运行时可以简化字符串比较（通常在字符串文字之间）到简单的引用比较。您可以检查浏览器如何使用简单的[JSPerf测试用例](https://jsperf.com/strinterning/4)处理此[问题](https://jsperf.com/strinterning/4)。检查相同测试的其他修订版本以获得更全面的测试用例。

![Firefox 45字符串在Linux上的实习结果](https://cdn.auth0.com/blog/immutablejs/interning.png)

#### 不可变性和OBJECT.FREEZE（）

JavaScript是一种动态的弱类型语言（如果你熟悉编程语言理论，则是无类型的）。因此，有时难以对对象和数据实施某些约束。`Object.freeze()`在这方面有帮助。调用将`Object.freeze`所有属性标记为不可变。分配将静默失败或抛出异常（在严格模式下）。如果你正在写一个不可变的对象，调用`Object.freeze`后的建设可以帮助。

牢记`Object.freeze()`是浅：子对象的属性可以修改。为了解决这个问题，[Mozilla](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)显示了如何`deepFreeze`编写这个函数的版本：

```js
function deepFreeze(obj) {
    // Retrieve the property names defined on obj
    var propNames = Object.getOwnPropertyNames(obj);

    // Freeze properties before freezing self
    propNames.forEach(function(name) {
        var prop = obj[name];

        // Freeze prop if it is an object
        if (typeof prop == 'object' && prop !== null) {        
            deepFreeze(prop);
        }
    });

    // Freeze self (no-op if already frozen)
    return Object.freeze(obj);
}
```

### 副作用

在编程语言理论中，对任何操作（通常是函数或方法调用）的**副作用是可以在调用函数之外看到的可观察效果**。换句话说，可以在执行呼叫之后找到**状态**的**改变**。每个调用都会*改变*一些状态。与通常与数据和数据结构相关的不变性的概念相反，副作用通常与整个程序的状态相关联。保留数据结构的实例的不变性的函数*可能*具有副作用。一个很好的例子是缓存函数或memoization。虽然对外部观察者，它可能看起来没有发生变化，更新全局或本地高速缓存具有更新用作高速缓存的内部数据结构（所得的加速也是副作用）的副作用。**开发者的工作是了解这些副作用并适当地处理它们**。

例如，按照高速缓存的示例，具有高速缓存作为前端的不可变数据结构不能再自由地传递到不同的线程。缓存必须支持多线程，否则可能会发生意外结果。

**功能性编程作为范例有利于使用副作用自由功能**。为了应用，函数必须仅对传递给它们的数据执行操作，并且这些操作的效果只应该被调用者看到。**不变的数据结构与副作用自由功能是相辅相成的**。

> [“不变的数据结构与副作用自由功能是相辅相成的。TWEET这个 ![img](https://cdn.auth0.com/blog/resources/twitter.svg)](https://twitter.com/intent/tweet?text=%22Immutable%20data%20structures%20go%20hand-in-hand%20with%20side-effect%20free%20functions.%22%20via%20@auth0%20http://auth0.com/blog/intro-to-immutable-js/)

```js
var globalCounter = 99;

// This function changes global state.
function add(a, b) {
    ++globalCounter;
    return a + b;
}

// A call to the seemingly innocent add function above will produce potentially
// unexpected changes in what is printed in the console here.
function printCounter() {
    console.log(globalCounter.toString());
}
```

#### 纯度

纯度是可以强加在函数上的附加条件：**纯函数仅依赖于作为参数传递给它们的结果**。换句话说，纯函数不能依赖于通过其他构造可访问的全局状态或状态。

```js
var globalValue = 99;

// This function is impure: its result will change if globalValue is changed,
// even when passed the same values in 'a' and 'b' as in previous calls.
function sum(a, b) {
    return a + b + globalValue;
}
```

#### 参考透明度

将副作用自由功能与纯度组合的结果是参照透明度。通过相同参数集合的透明函数**可以在任何点通过其结果**知道某些这种变化而**被替换，**而不是作为整体的计算。

正如您可能已经注意到的，每个这些条件对数据和代码的行为有更高的限制。**虽然这导致灵活性降低，但是当涉及分析和证明时，实现了深厚的收益**。可轻易地证明，不具有副作用的不变数据结构可以传递到不同的线程，而不用担心锁定。

```js
function add(a, b) {
    return a + b;
}

// The following call can be replaced by its result: 3. This is possible because
// it is referentially transparent. IOW, side-effect free and pure.
var r1 = add(1, 2); // r1 = 3;
```

### 持久性

正如我们在上一节中看到的，不变性使某些事情更容易。使用不可变的数据结构变得更容易的另一个事情是*持久性*。持久性，在数据结构的上下文中，指的是在**构建**新版本之后**保持数据结构的较旧版本的**可能性。

正如我们之前提到的，当对不可变数据结构执行写操作时，不是改变结构本身或其数据，而是返回新版本的结构。然而，大多数时候，关于数据或数据结构的大小的修改很小。**因此，执行整个数据结构的完整副本是次优的**。大多数不可变的数据结构算法利用第一版本数据的不变性，**仅**执行**需要改变的数据（以及数据结构的部分）的副本**。

**部分持久性**数据结构是支持对其最新版本的修改以及对所有先前版本的数据的只读操作的那些。**完全持久的**数据结构允许对所有版本的数据进行读写。注意，在所有情况下，写入或修改数据意味着创建数据结构的新版本。

它可能不是完全明显的，但持久的数据结构有**利于垃圾收集，**而不是引用计数或手动内存管理。由于每个更改都会导致新版本的数据，并且先前版本必须可用，因此每次执行更改时，都会创建对现有数据的新引用。在手动存储器管理方案上，跟踪哪些数据片段具有引用快速地变得麻烦。另一方面，从开发人员的角度来看，引用计数使得事情更容易，但是从算法的角度来看效率低：每次执行改变时，必须更新改变的数据的引用计数。此外，这种*看不见的变化*实际上是副作用。因此，它限制了某些益处的适用性。垃圾收集，另一方面，没有这些问题。**添加对现有数据的引用是免费的**。

在以下示例中，原始列表自创建以来的每个版本都可用（通过每个变量绑定）：

```js
var list1 = Immutable.List.of(1, 2);
var list2 = list1.push(3, 4, 5);
var list3 = list2.unshift(0);
var list4 = list1.concat(list2, list3);
```

### 懒惰评价

另一个不那么明显的不变性的好处是更容易懒惰操作的形式。**惰性操作是那些在执行这些操作的结果之前不执行任何操作的操作**（通常通过严格的求值操作;严格与上下文中的惰性相反）。**不可变性**在惰性操作的上下文中非常有用，因为惰性求值通常需要在将来执行操作。如果与操作相关的数据在构造操作的时间和需要结果的时间之间以任何方式改变，则操作不能安全地执行。**不可变数据有助于建立惰性操作，因为某些数据不会改变**。换一种说法，

Immutable.js支持惰性操作：

```js
var oddSquares = Immutable.Seq.of(1,2,3,4,5,6,7,8)
                              .filter(x => x % 2)
                              .map(x => x * x);
// Only performs as much work as necessary to get the first result
console.log(oddSquares.get(1)); // 9
```

延迟评估有几个好处。最重要的是**不需要计算不必要的值**。例如，考虑由元素1到10形成的列表。现在让我们对列表中的每个元素应用两个独立的操作。第一个操作将被调用`plusOne`，第二个操作被调用`plusTen`。这两个操作都明显：第一个添加一个到它的参数，第二个添加十。

```js
function plusOne(n) {
    return n + 1;
}
function plusTen(n) {
    return n + 10;
}

var list = [1,2,3,4,5,6,7,8,9,10];
var result = list.map(plusOne).map(plusTen);
```

正如你可能已经注意到的，这是低效的：循环内部`map`运行两次，即使没有`result`使用任何元素。假设你只需要第一个元素：with strict evaluation两个循环完全运行。使用延迟评估，每个循环运行，直到返回请求的结果。换句话说，如果`result[0]`被请求，**只**执行每个`plus...`函数的**一次调用**。

延迟评估还可以允许**无限数据结构**。例如，如果支持延迟评估，则可以安全地表达从1到无穷大的序列。延迟评估也可以允许无效值：如果从不请求计算中的无效值，则不执行无效操作（这可能导致异常或其他错误条件）。

某些功能编程语言还可以在懒惰评估可用时执行高级优化，例如[砍伐森林](http://homepages.inf.ed.ac.uk/wadler/papers/deforest/deforest.ps)或[循环融合](https://en.wikipedia.org/wiki/Loop_fusion)。实质上，这些优化**可以将根据多个循环定义的操作转换为单个循环**，或者换句话说，**移除中间数据结构**。在实践中，`map`上面例子中的两个调用变成一个`map`调用`plusOne`并`plusTen`在同一循环中调用的单个调用。Nifty，嗯？

然而，并不是一切都对懒惰评价是好的：**任何表达式被评估和计算执行的确切点停止显而易见**。分析某些复杂的延迟操作可能相当困难。另一个缺点是**空间泄漏**：由于存储必要的数据以在将来执行给定的计算而导致的泄漏。某些惰性构造可以使此数据无限增长，这可能导致问题。

### 组成

在功能编程的上下文中的组合是指将**不同功能组合成新的强大功能的可能性**。第一类函数（可以作为数据处理并传递给其他函数的函数），闭包和currying（`Function.bind`对类固醇考虑）是必要的工具。JavaScript的语法不像某些函数式编程语言的语法一样方便，但它肯定是可能的。适当的API设计可以产生良好的效果。

Immutable的懒惰功能结合组合产生方便，可读的JavaScript代码：

```js
Immutable.Range(1, Infinity)
    .skip(1000)
    .map(n => -n)
    .filter(n => n % 2 === 0)
    .take(2)
    .reduce((r, n) => r * n, 1);
```

### 逃生舱口：突变

对于不变性可以提供的所有优点，**某些操作和算法仅当突变可用时才有效**。尽管不变性是大多数函数式编程语言（与命令式语言相反）的默认值，但是突变通常可能有效地实现这些操作。

再次，Immutable.js已覆盖：

```js
var list1 = Immutable.List.of(1,2,3);
var list2 = list1.withMutations(function (list) {
    list.push(4).push(5).push(6);
});
```

## 算法注意事项

在算法和数据结构领域，没有免费膳食。在一个领域的改进通常导致在另一个更糟的结果。不变性也不例外。我们已经讨论了不变性的一些好处：易持久性，更简单的推理，更少的锁定等; 但有什么缺点？

当谈论算法时，时间复杂性可能是你应该记住的第一件事。**不可变数据结构具有与可变数据结构不同的运行时特性**。特别地，不变的数据结构通常**在考虑持久性需求时**具有**良好的运行时间特性**。

这些差异的一个简单例子是单链表：通过使每个节点指向下一个节点（但不返回）而形成的列表。

![可能的Immutable.js单链表的实现](https://cdn.auth0.com/blog/immutablejs/linkedlist.png)

> 基于[Leslie Sanford的持久数据结构](http://www.codeproject.com/Articles/9680/Persistent-Data-Structures)图的图。

可变单链表具有以下时间复杂度（最坏情况，假定前，后和插入节点是已知的）：

- 前缀：O（1）
- 附加：O（1）
- 插入：O（1）
- 查找：O（n）
- 副本：O（n）

相反，不变的单链表具有以下时间复杂度（最坏情况，假定前，后和插入节点是已知的）：

- 前缀：O（1）
- 附加：O（n）
- 插入：O（n）
- 查找：O（n）
- 复制：O（1）

> 如果你不熟悉时间分析和大O表示法，阅读[这个](https://rob-bell.net/2009/06/a-beginners-guide-to-big-o-notation/)。

这不为不可变的数据结构绘制好的画面。然而，最坏情况时间分析不考虑对持续需求的影响。换句话说，如果可变数据结构必须符合这个要求，运行时复杂性大多看起来像那些来自不可变版本（至少对于这些操作）。写时复制和其它技术可以改进一些操作的*平均*时间，这也不被考虑用于最坏情况分析。

在实践中，**最坏情况分析可能不总是**选择数据结构**的最具代表性的时间分析形式**。*摊销*分析将数据结构视为一组操作。**具有良好的摊销时间的数据结构可以显示偶尔的最差时间行为，而在通常情况下保持得更好**。分摊分析有意义的一个好例子是一个动态数组，当一个元素需要分配超过它的末尾时，它被优化为其大小的两倍。最坏情况分析为附加操作给出O（n）时间。摊销时间可以被认为是O（1），因为N / 2追加操作可以在单个追加产生O（n）时间之前执行。一般来说，如果您的用例需要确定性时间，则不能考虑摊销时间。

**时间复杂度分析也忽略了其他重要的注意事项**：某个数据结构的使用如何影响它周围的代码？例如，对于不可变的数据结构，在多线程场景中可能不需要锁定。

### CPU高速缓存注意事项

另一个要记住的事情，特别是对于高性能计算，是**数据结构与底层CPU缓存的方式**。通常，[对于执行](http://concurrencyfreaks.blogspot.com.ar/2013/10/immutable-data-structures-are-not-as.html)许多写操作的情况，[可变数据结构的局部性更好](http://concurrencyfreaks.blogspot.com.ar/2013/10/immutable-data-structures-are-not-as.html)（除非持久性被深深使用）。

### 内存使用

不可变的数据结构由于**内存使用**的本质上的**尖峰**。每次修改后，执行复制。如果不需要这些副本，垃圾收集器可以在下一次收集期间收集旧的数据。只要未收集旧的，未使用的数据副本，就会导致使用的峰值。在需要持久性的情况下，不存在尖峰。

正如你可能已经注意到的，**当持久性被考虑时**，**不变性变得非常引人注目**。

## 示例：响应DBMon基准

基于我们[以前的一系列基准](https://auth0.com/blog/2016/01/11/updated-and-improved-more-benchmarks-virtual-dom-vs-angular-12-vs-mithril-js-vs-the-rest/)，我们决定更新我们的React DBMon基准以在适当的地方使用Immutable.js。由于DBMon在每次迭代后基本上更新所有数据，因此切换到React + Immutable.js不会获得任何好处：Immutable允许React在状态更改后阻止深度相等性检查; 如果在每次迭代之后所有状态都改变，则不可能获得增益。我们因此修改了我们的示例以随机跳过状态更改：

```js
// Skip some updates to test re-render state checks.
var skip = Math.random() >= 0.25;

Object.keys(newData.databases).forEach(function (dbname) {
    if (skip) {
        return;
    }

    //(...)
});
```

之后，我们将保存样本的数据结构从JavaScript数组更改为不可变列表。此列表作为参数传递给要渲染的组件。当React的PureRenderMixin添加到组件类中时，可以进行更有效的比较。

```js
if (!this.state.databases[dbname]) {
    this.state.databases[dbname] = {
        name: dbname,
        samples: Immutable.List()
    };
}

this.state.databases[dbname].samples =
    this.state.databases[dbname].samples.push({
        time: newData.start_at,
        queries: sampleInfo.queries
    });
if (this.state.databases[dbname].samples.size > 5) {
    this.state.databases[dbname].samples =
        this.state.databases[dbname].samples.skip(
            this.state.databases[dbname].samples.size - 5);
}
```

```js
var Database = React.createClass({
    displayName: "Database",

    mixins: [React.PureRenderMixin],

    render: function render() {
      //(...)
    }
    //(...)
});
```

这是在这种情况下实现增益所需要的。如果数据被认为不变，则不采取进一步的动作来绘制DOM树的该分支。

正如我们以前的一套基准测试一样，我们使用browser-perf来捕获差异。这是JavaScript代码的总运行时间：

![DBMon + Immutable.js JavaScript总运行时间](https://cdn.auth0.com/blog/immutablejs/chart.svg)

获取[完整的结果](https://github.com/auth0/blog-immutable-js-dbmon-react)。

## Aside：Immutable.js at Auth0

在Auth0，我们总是在看新的图书馆。Immutable.js也不例外。Immutable.js已经进入我们的lock-next和[lock-passwordless](https://github.com/auth0/lock-passwordless)项目（lock-next，我们的下一代锁库，仍然在内部开发）。这两个库都是用React开发的。渲染React组件可以[在使用不可变数据时](https://facebook.github.io/react/docs/advanced-performance.html)获得[很好的提升，](https://facebook.github.io/react/docs/advanced-performance.html)因为可以通过优化来检查相等性：当两个对象共享相同的引用并且您确定基础对象是不可变的时，您可以确保其中包含的数据没有改变。由于React根据它们是否已更改重新呈现对象，因此不再需要深度值检查。

> 一个[类似的优化](http://blog.mgechev.com/2015/03/02/immutability-in-angularjs-immutablejs/)可以在Angular.js应用程序来实现。

你喜欢React和Immutable.js吗？[向我们发送简历](https://auth0.com/jobs)，并指出我们使用这些技术开发的酷项目。

## 结论

由于功能编程，不变性和其他相关概念的好处被尝试和测试。使用Clojure，Scala和Haskell开发项目背后的成功故事为这些语言强烈倡导的许多想法带来了更大的思维。不变性是这些概念之一：对分析，持久性，复制和比较具有明显的好处，不可变的数据结构甚至在您的浏览器中找到了进入特定用例的方式。像往常一样，当涉及算法和数据结构时，需要仔细分析每个场景以选择正确的工具。关于性能，内存使用，CPU高速缓存行为和对数据执行的操作类型的注意事项对于确定不可变性是否会对您有利是至关重要的。使用Immutable。

如果这篇文章激发了您对函数式编程和数据结构的兴趣，我不能够强烈推荐Chris [Okaki](http://www.amazon.com/Purely-Functional-Structures-Chris-Okasaki/dp/0521663504/ref=sr_1_1?ie=UTF8&qid=1458699242&sr=8-1)的[纯功能数据结构](http://www.amazon.com/Purely-Functional-Structures-Chris-Okasaki/dp/0521663504/ref=sr_1_1?ie=UTF8&qid=1458699242&sr=8-1)，这是一个介绍功能数据结构如何在幕后工作以及如何有效使用它们的一个很好的介绍。Hack on！