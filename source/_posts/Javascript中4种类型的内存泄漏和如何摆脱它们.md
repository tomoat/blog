---
title: Javascript中4种类型的内存泄漏和如何摆脱它们
---

> [4 Types of Memory Leaks in JavaScript and How to Get Rid Of Them](https://auth0.com/blog/four-types-of-leaks-in-your-javascript-code-and-how-to-get-rid-of-them/)
>
> 了解JavaScript中的内存泄漏，以及可以做什么来解决它！

在本文中，我们将探讨客户端JavaScript代码中的常见类型的内存泄漏。我们还将学习如何使用Chrome开发工具找到它们！



## 介绍

内存泄漏是每个开发人员最终面临的问题。即使使用内存管理的语言，也有可能泄漏内存的情况。泄漏是整类问题的原因：减速，崩溃，高延迟，甚至与其他应用程序的问题。

>诸如 C 语言这般的低级语言一般都有低级的内存管理接口，比如 `malloc() 和` `free()`。而另外一些高级语言，比如 JavaScript， 其在变量（对象，字符串等等）创建时分配内存，然后在它们不再使用时“自动”释放。后者被称为**垃圾回收**。“自动”是容易让人混淆，迷惑的，并给 JavaScript（和其他高级语言）开发者一个印象：他们可以不用关心内存管理。然而这是错误的。

### 什么是内存泄露？

实质上，内存泄漏可以定义为应用程序不再需要的内存，因为某种原因，该内存不会返回到操作系统或可用内存池。编程语言有利于不同的管理内存的方式。这些方式可以减少泄漏内存的机会。然而，某一块内存是否未被使用实际上是一个[不可判定的问题](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Memory_Management#Release_when_the_memory_is_not_needed_anymore)。换句话说，只有开发人员才能明确是否可以将一块内存返回到操作系统。某些编程语言提供了帮助开发人员做这些的功能。其他人期望开发人员完全明确一段内存未使用。维基百科有关于[手动](https://en.wikipedia.org/wiki/Manual_memory_management)和[自动](https://en.wikipedia.org/wiki/Manual_memory_management)内存管理的好文章。

### JavaScript中的内存管理

JavaScript是所谓的*垃圾收集语言*之一。垃圾收集语言通过定期检查哪些先前分配的内存仍然可以从应用程序的其他部分“达到”来帮助开发人员管理内存。换句话说，垃圾收集语言将管理内存的问题从“还需要什么内存？降低到“应用程序的其他部分仍然可以重新分配内存？”。差别是微妙的，但重要的是：虽然只有开发人员知道将来是否需要一块分配的内存，取不到的内存可以通过算法确定并标记为返回到操作系统。

> 非垃圾收集语言通常使用其他技术来管理内存：显式管理，开发人员明确告诉编译器何时不需要一块内存; 和引用计数，其中使用计数与存储器的每个块相关联（当计数达到零时，其被返回到OS）。这些技术有自己的权衡（和潜在的泄漏原因）。

## JavaScript中的内存溢出

垃圾收集语言内存泄漏的主要原因是*不需要的引用*。要理解什么是不需要的引用，首先我们需要了解垃圾回收器如何确定是否可以到达一块内存。

> [“垃圾收集语言泄漏的主要原因是不需要的引用。TWEET这个 ![img](https://cdn.auth0.com/blog/resources/twitter.svg)](https://twitter.com/intent/tweet?text=%22The%20main%20cause%20for%20leaks%20in%20garbage%20collected%20languages%20are%20unwanted%20references.%22%20via%20@auth0%20http://auth0.com/blog/four-types-of-leaks-in-your-javascript-code-and-how-to-get-rid-of-them/)

### 标记和扫描

大多数垃圾收集器使用称为*标记和扫描*的算法。该算法由以下步骤组成：

1. 垃圾回收器构建“根”的列表。根通常是在代码中保存引用的全局变量。在JavaScript中，“window”对象是可以充当根的全局变量的示例。窗口对象总是存在，所以垃圾收集器可以考虑它和它的所有子对象总是存在（即不是垃圾）。
2. 所有根被检查并标记为活动（即不是垃圾）。所有孩子也被递归检查。从根可以到达的一切都不被认为是垃圾。
3. 所有未标记为活动的内存块现在可以被认为是垃圾。收集器现在可以释放该内存并将其返回到操作系统。

现代垃圾收集器以不同的方式改进了该算法，但本质是相同的：可访问的内存段被标记为这样，其余被认为是垃圾。

不需要的引用是对开发者知道他或她将不再需要，但由于某种原因保存在活动根的树内部的存储器的引用。在JavaScript的上下文中，不需要的引用是保存在代码中某处的变量，它不再被使用，并指向可以被释放的一块内存。有些人会认为这些都是开发者的错误。

所以要了解哪些是JavaScript中最常见的内存泄漏，我们需要知道在哪些方式引用通常被遗忘。

## JavaScript内存泄漏的三种类型

### 1：意外全局变量

JavaScript背后的目标之一是开发一种看起来像Java的语言，但是它允许足以被初学者使用。JavaScript允许的方式之一是处理未声明的变量：对未声明的变量的引用在*全局*对象内创建一个新的变量。在浏览器的情况下，全局对象是`window`。换一种说法：

```js
function foo(arg) {
    bar = "this is a hidden global variable";
}
```

其实是：

```js
function foo(arg) {
    window.bar = "this is an explicit global variable";
}
```

如果`bar`应该在`foo`函数的范围内保存对变量的引用，并且您忘记使用`var`它来声明它，那么会创建一个意外的全局变量。在这个例子中，泄漏一个简单的字符串不会做很多伤害，但它肯定可能更糟。

可以创建偶然的全局变量的另一种方式是`this`：

```js
function foo() {
    this.variable = "potential accidental global";
}

// Foo called on its own, this points to the global object (window)
// rather than being undefined.
foo();
```

> 为了防止发生这些错误，请`'use strict';`在JavaScript文件的开头添加。这使得可以更严格地解析JavaScript以防止意外全局变量。

#### 关于全局变量的注释

即使我们谈论不可预测的全局变量，仍然是这样的情况，许多代码是与显式的全局变量。这些是根据定义不可收集的（除非被取消或重新分配）。特别地，用于临时存储和处理大量信息的全局变量是令人关注的。如果必须使用全局变量来存储大量数据，请确保将其置空或在完成后重新分配它。与全局变量有关的增加的内存消耗的一个常见原因是[高速缓存](https://en.wikipedia.org/wiki/Cache_(computing)）。缓存存储重复使用的数据。为了有效率，高速缓存必须具有其大小的上限。无限增长的缓存可能导致高内存消耗，因为无法收集其内容。

### 2：被遗忘的计时器或回调

`setInterval`在JavaScript中使用是相当常见的。其他图书馆提供观察员和其他设施来接受回调。大多数这些库在自己的实例变得不可访问之后，负责使任何对回调的引用不可达。在setInterval的情况下，然而，像这样的代码是很常见的：

```js
var someResource = getData();
setInterval(function() {
    var node = document.getElementById('Node');
    if(node) {
        // Do stuff with node and someResource.
        node.innerHTML = JSON.stringify(someResource));
    }
}, 1000);
```

此示例说明了可能发生的悬挂计时器：计时器，引用不再需要的节点或数据。由`node`未来表示的对象可能会被删除，使得区间处理程序内部的整个块不必要。但是，处理程序（因为时间间隔仍处于活动状态）无法收集（需要停止时间间隔才能发生这种情况）。如果无法收集间隔处理程序，则也无法收集其依赖项。这意味着，`someResource`不可能收集大概存储大小的数据。

对于观察者的情况，重要的是进行显式调用，以便在不再需要它们时删除它们（或者相关对象即将无法访问）。在过去，以前特别重要，因为某些浏览器（Internet Explorer 6）无法管理循环引用（参见下面的更多信息）。现在，大多数浏览器可以并将收集观察者处理程序，一旦观察到的对象变得不可达，即使没有明确删除侦听器。但是，在处理对象之前显式删除这些观察者仍然是良好的做法。例如：

```js
var element = document.getElementById('button');

function onClick(event) {
    element.innerHtml = 'text';
}

element.addEventListener('click', onClick);
// Do stuff
element.removeEventListener('click', onClick);
element.parentNode.removeChild(element);
// Now when element goes out of scope,
// both element and onClick will be collected even in old browsers that don't
// handle cycles well.
```

#### 关于对象观察者和循环引用的注释

观察者和循环引用曾经是JavaScript开发者的祸根。这是由于Internet Explorer的垃圾回收器中的错误（或设计决策）。旧版本的Internet Explorer无法检测DOM节点和JavaScript代码之间的循环引用。这是典型的观察者，通常保持对可观察者的引用（如上例所示）。换句话说，每当观察者被添加到Internet Explorer中的一个节点时，它就会导致泄漏。这是开发人员在节点或在观察者内引用null引用之前显式删除处理程序的原因。现在，现代浏览器（包括Internet Explorer和Microsoft Edge）使用现代垃圾收集算法，可以检测这些周期并正确处理它们。换一种说法，`removeEventListener`

框架和库（例如*jQuery）*在处理节点之前删除侦听器（当为其使用特定的API时）。这是由库内部处理，并确保不产生泄漏，即使在有问题的浏览器（如旧的Internet Explorer）下运行。

### 3：超出DOM引用

有时，将DOM节点存储在数据结构中可能很有用。假设要快速更新表中多个行的内容。在字典或数组中存储对每个DOM行的引用可能是有意义的。当发生这种情况时，将保留对同一DOM元素的两个引用：一个在DOM树中，另一个在字典中。如果在将来的某个时候决定删除这些行，则需要使这两个引用不可访问。

```js
var elements = {
    button: document.getElementById('button'),
    image: document.getElementById('image'),
    text: document.getElementById('text')
};

function doStuff() {
    image.src = 'http://some.url/image';
    button.click();
    console.log(text.innerHTML);
    // Much more logic
}

function removeButton() {
    // The button is a direct child of body.
    document.body.removeChild(document.getElementById('button'));

    // At this point, we still have a reference to #button in the global
    // elements dictionary. In other words, the button element is still in
    // memory and cannot be collected by the GC.
}
```

对此的另外考虑与对DOM树内的内部或叶节点的引用有关。假设您`<td>`在JavaScript代码中保留对表（标签）的特定单元格的引用。在将来的某个时候，您决定从DOM中删除表，但保留对该单元格的引用。直观地，可以假定GC将收集除了该单元之外的所有东西。在实践中，这不会发生：单元格是该表的子节点，并且子级保持对其父级的引用。换句话说，从JavaScript代码对表单元格的引用导致整个表保留在内存中。在保持对DOM元素的引用时仔细考虑这一点。

### 4：关闭

JavaScript开发的一个关键方面是闭包：从父作用域捕获变量的匿名函数。Meteor开发人员[发现了一种特殊情况](http://info.meteor.com/blog/an-interesting-kind-of-javascript-memory-leak)，由于JavaScript运行时的实现细节，可能以微妙的方式泄漏内存：

```js
var theThing = null;
var replaceThing = function () {
  var originalThing = theThing;
  var unused = function () {
    if (originalThing)
      console.log("hi");
  };
  theThing = {
    longStr: new Array(1000000).join('*'),
    someMethod: function () {
      console.log(someMessage);
    }
  };
};
setInterval(replaceThing, 1000);
```

这个片段做了一件事：每次`replaceThing`调用，`theThing`获取一个包含大数组和一个新的闭包（`someMethod`）的新对象。同时，变量`unused`保存有一个引用`originalThing`（`theThing`从上一次调用`replaceThing`）的闭包。已经有点混乱了，是吗？重要的是，一旦为同一父作用域中的闭包创建了作用域，则该作用域是共享的。在这种情况下，为闭包创建的作用域由`someMethod`共享`unused`。`unused`有引用`originalThing`。即使`unused`从未使用，`someMethod`可以通过使用`theThing`。并且`someMethod`与封闭范围共享`unused`，即使`unused`从未使用，`originalThing`其引用强制其保持活动（防止其收集）。当此代码段重复运行时，可以观察到内存使用量的稳定增加。这在GC运行时不会变小。实质上，创建了闭包的链接列表（其根以`theThing`变量的形式），并且这些闭包的范围中的每一个都对大数组进行间接引用，导致相当大的泄漏。

> 这是一个实现工件。可以处理这种情况的闭包的不同实现是可能的，如[Meteor博客文章中所解释的](http://info.meteor.com/blog/an-interesting-kind-of-javascript-memory-leak)。

## 垃圾收集者的不直观行为

虽然垃圾收集器很方便，他们有自己的一套权衡。这些权衡之一*是非确定性*。换句话说，GC是不可预测的。通常不可能确定何时执行收集。这意味着在某些情况下，正在使用比程序实际需要的更多的内存。在其他情况下，短暂停顿在特别敏感的应用中可能是明显的。虽然非确定性意味着无法确定何时执行集合，但大多数GC实现都在分配期间共享执行集合传递的常见模式。如果没有执行分配，则大多数GC保持静止。考虑以下情况：

1. 执行相当大的一组分配。
2. 大多数这些元素（或所有这些元素）被标记为不可达（假设我们将指向我们不再需要的缓存的引用置空）。
3. 不执行进一步的分配。

在这种情况下，大多数GC将不会运行任何进一步的集合过程。换句话说，即使有不可达的引用可用于收集，收集器也不要求这些引用。这些不是严格的泄漏，但仍然导致高于通常的内存使用。

Google在他们的[JavaScript内存分析文档中](https://developer.chrome.com/devtools/docs/demos/memory/example2)提供了这种行为的一个很好的例子[，示例＃2](https://developer.chrome.com/devtools/docs/demos/memory/example2)。

## Chrome内存分析工具概述

Chrome提供了一组很好的工具来分析JavaScript代码的内存使用情况。有两个与内存相关的基本视图：*时间轴*视图和*配置文件*视图。

### 时间轴视图

![Google开发工具时间表行动](https://cdn.auth0.com/blog/jsleaks/timeline.png)时间轴视图对于在代码中发现异常内存模式至关重要。如果我们正在寻找大的泄漏，周期性的跳跃，不收缩，因为收集后他们长大了是一个红旗。在这个截图中，我们可以看到泄漏对象的稳定增长可能是什么样子。即使在大收集结束后，使用的内存总量高于开始时。节点计数也较高。这些都是代码中某处泄漏的DOM节点的迹象。

### 配置文件视图

![Google开发工具配置文件](https://cdn.auth0.com/blog/jsleaks/profiles.png)这是你将花费大部分时间看的视图。配置文件视图允许您获取快照并比较JavaScript代码的内存使用快照。它还允许您记录分配的时间。在每个结果视图中，可以使用不同类型的列表，但对于我们的任务最相关的是摘要列表和比较列表。

摘要视图为我们概述了分配的不同类型的对象及其聚合大小：浅大小（特定类型的所有对象的总和）和保留大小（浅大小加上由于此对象保留的其他对象的大小）。它也给了我们一个对象相对于它的GC根（距离）有多远的概念。

比较列表给了我们相同的信息，但允许我们比较不同的快照。这是特别有用的找到泄漏。

## 示例：使用Chrome查找泄漏

基本上有两种类型的泄漏：泄漏导致内存使用的周期性增加，以及一次发生的泄漏，并且不会进一步增加内存。由于显而易见的原因，当它们是周期性的时更容易发现泄漏。这些也是最麻烦的：如果内存在时间上增加，这种类型的泄漏将最终导致浏览器变慢或停止脚本的执行。非周期性泄漏可以很容易地发现，当它们足够大，在所有其他分配中显着。这通常不是这样，所以他们通常保持不被注意。在某种程度上，发生一次的小泄漏可以被认为是优化问题。然而，周期性的泄漏是错误并且必须是固定的。

对于我们的示例，我们将使用[Chrome的文档中的](https://developer.chrome.com/devtools/docs/demos/memory/example1)一个[示例](https://developer.chrome.com/devtools/docs/demos/memory/example1)。完整代码粘贴如下：

```js
var x = [];

function createSomeNodes() {
    var div,
        i = 100,
        frag = document.createDocumentFragment();
    for (;i > 0; i--) {
        div = document.createElement("div");
        div.appendChild(document.createTextNode(i + " - "+ new Date().toTimeString()));
        frag.appendChild(div);
    }
    document.getElementById("nodes").appendChild(frag);
}
function grow() {
    x.push(new Array(1000000).join('x'));
    createSomeNodes();
    setTimeout(grow,1000);
}

```

当`grow`被调用时，它将开始创建div节点并将它们附加到DOM。它还将分配一个大数组，并将其附加到全局变量引用的数组。这将导致使用上述工具可以找到的内存的稳定增加。

> 垃圾收集的语言通常显示振荡存储器使用的模式。如果代码在执行分配的循环中运行，这是预期的，这是通常的情况。我们将寻找在收集后不会回退到之前级别的内存的定期增加。

### 了解内存是否周期性增加

时间表视图是伟大的。在Chrome中打开[示例](https://developer.chrome.com/devtools/docs/demos/memory/example1)，打开开发工具，转到*时间轴*，选择*内存*并单击记录按钮。然后转到该页面并单击`The Button`以开始泄漏内存。过一会儿停止录音，看看结果：

![时间轴视图中的内存泄漏](https://cdn.auth0.com/blog/jsleaks/example-timeline.png)

> 此示例将继续每秒泄漏内存。停止录制后，请在`grow`函数中设置断点，以停止脚本强制Chrome关闭页面。

在这个图像有两个大的迹象，显示我们正在泄漏的记忆。*节点*（绿线）和*JS堆*（蓝线）的图形。节点正在稳步增加，从不减少。这是一个大的警告标志。

JS堆还显示内存使用的稳定增长。这是很难看到由于垃圾回收器的影响。您可以看到初始内存增长的模式，随后是大幅下降，随后是增加，然后是峰值，继续记忆的下降。在这种情况下的关键在于事实，在每次内存使用后，堆的大小保持大于在上一次下降。换句话说，尽管垃圾收集器正在成功地收集大量的存储器，但是它的一些被周期性地泄漏。

我们现在确定我们有一个泄漏。让我们找到它。

### 获取两个快照

要查找泄漏，我们现在将转到Chrome的开发工具的*profiles*部分。要将内存使用限制在可管理的级别，请在执行此步骤之前重新加载页面。我们将使用*Take Heap Snapshot*函数。

重新加载页面，并在完成加载后立即获取堆快照。我们将使用此快照作为我们的基线。之后，`The Button`再次点击，等待几秒钟，并拍摄第二个快照。捕获快照后，建议在脚本中设置断点，以防止泄漏使用更多内存。

![堆快照](https://cdn.auth0.com/blog/jsleaks/example-snapshots-1.png)

有两种方法可以查看两个快照之间的分配。选择*摘要*，然后选择右侧选择*在快照1和快照2之间分配的对象*，或选择*比较*而不是*摘要*。在这两种情况下，我们将看到在两个快照之间分配的对象的列表。

在这种情况下，很容易找到泄漏：他们很大。看看的`Size Delta`的的`(string)`构造函数。8MBs有58个新对象。这看起来很可疑：新对象被分配但是不被释放，并且8MB被消耗。

如果我们打开构造函数的`(string)`分配列表，我们会注意到在许多小的分配中有一些大的分配。大者立即引起我们的注意。如果我们选择其中的任何一个，我们在下面的*retainers*部分得到一些有趣的*东西*。

![所选对象的保留位置](https://cdn.auth0.com/blog/jsleaks/example-snapshots-2.png)

我们看到我们选择的分配是数组的一部分。反过来，数组由`x`全局`window`对象内的变量引用。这给了我们从我们的大对象到其不可收回的根（`window`）的完整路径。我们发现我们的潜在泄漏和被引用的地方。

到现在为止还挺好。但我们的例子很容易：大分配，例如在这个例子中的分配不是常态。幸运的是，我们的例子也泄漏了DOM节点，它们更小。使用上面的快照很容易找到这些节点，但是在更大的网站中，事情变得更麻烦。最新版本的Chrome提供了一个最适合我们工作的附加工具：*记录堆分配*功能。

### 记录堆分配以查找泄漏

禁用之前设置的断点，让脚本继续运行，然后返回Chrome的开发工具的“ *个人档案”*部分。现在点击*Record Heap Allocations*。当工具运行时，您会注意到在顶部的图中的蓝色尖峰。这些表示分配。每秒大的分配由我们的代码执行。让它运行几秒钟，然后停止它（不要忘记再次设置断点，以防止Chrome吃更多的内存）。

![记录的堆分配](https://cdn.auth0.com/blog/jsleaks/example-recordedallocs-overview.png)

在此图像中，您可以看到此工具的杀手级功能：选择一段时间线以查看在该时间段内执行的分配。我们将选择设置为尽可能接近一个大峰值。列表中只显示了三个构造函数：其中一个是与我们的大泄漏（`(string)`）相关的，另一个是与DOM分配相关的，最后一个是`Text`构造函数（包含文本的叶DOM节点的构造函数）。

`HTMLDivElement`从列表中选择一个构造函数，然后选择`Allocation stack`。

![堆分配结果中选择的元素](https://cdn.auth0.com/blog/jsleaks/example-recordedallocs-selected.png)

BAM！我们现在知道该元素的分配位置（`grow`- > `createSomeNodes`）。如果我们密切关注图中每个秒杀，我们会发现，`HTMLDivElement`构造函数被调用了很多。如果我们回到我们快照比较认为我们会发现，这个构造显示有多少拨款，但没有删除。换句话说，它是稳定，而不允许在GC收回一些它分配内存。这有泄漏的种种迹象加上我们确切地知道被分配这些对象（`createSomeNodes`函数）。现在它的时间回到代码，研究它，并修复内存泄漏。

### 另一个有用的功能

在堆分配结果视图中，我们可以选择*Allocation*视图而不是*Summary*。

![结果是堆分配中的分配](https://cdn.auth0.com/blog/jsleaks/example-recordedallocs-list.png)

这个视图给了我们一个与它们相关的函数和内存分配的列表。我们可以立即看到`grow`和`createSomeNodes`站出来。当选择时，`grow`我们看看它所调用的关联对象构造函数。我们注意到`(string)`，`HTMLDivElement`和`Text`它现在我们已经知道是被泄露的对象的构造函数。

这些工具的组合可以大大有助于发现泄漏。玩他们。在生产站点中进行不同的分析运行（理想情况下使用非最小化或混淆代码）。看看你能找到的泄漏或对象被保留超过他们应该（提示：这些更难找到）。

> 要使用此功能，请转到Dev Tools - >设置并启用“记录堆分配堆栈跟踪”。在拍摄之前必须这样做。

## 进一步阅读

- [内存管理 - Mozilla开发人员网络](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Memory_Management)
- [JScript内存泄漏 - Douglas Crockford（旧的，关于Internet Explorer 6泄漏）](http://javascript.crockford.com/memory/leak.html)
- [JavaScript内存分析 - Chrome开发者文档](https://developer.chrome.com/devtools/docs/javascript-memory-profiling)
- [内存诊断 - Google Developers](https://developers.google.com/web/tools/chrome-devtools/profile/memory-problems/memory-diagnosis)
- [有趣的JavaScript内存泄漏 - 流星博客](http://info.meteor.com/blog/an-interesting-kind-of-javascript-memory-leak)
- [Grokking V8关闭](http://mrale.ph/blog/2012/09/23/grokking-v8-closures-for-fun.html)

## 结论

内存泄漏可以并且确实发生在垃圾收集语言中，如JavaScript。这些可以被忽视一段时间，最终他们将肆虐。因此，内存分析工具对于查找内存泄漏至关重要。分析运行应该是开发周期的一部分，特别是对于中型或大型应用程序。开始这样做，为您的用户提供最好的体验。Hack on！