---
title: 在Node.js中查找JavaScript内存泄漏简略指南
---

## 目录

- [介绍](https://www.alexkras.com/simple-guide-to-finding-a-javascript-memory-leak-in-node-js/#intro)
- [最小理论](https://www.alexkras.com/simple-guide-to-finding-a-javascript-memory-leak-in-node-js/#min-theory)
- [步骤1.重现并确认问题](https://www.alexkras.com/simple-guide-to-finding-a-javascript-memory-leak-in-node-js/#reproduce)
- [第2步。至少取3堆堆](https://www.alexkras.com/simple-guide-to-finding-a-javascript-memory-leak-in-node-js/#heapdump)
- [步骤3.查找问题](https://www.alexkras.com/simple-guide-to-finding-a-javascript-memory-leak-in-node-js/#find-problem)
- [步骤4.确认问题已解决](https://www.alexkras.com/simple-guide-to-finding-a-javascript-memory-leak-in-node-js/#confirm)
- [链接到一些其他资源](https://www.alexkras.com/simple-guide-to-finding-a-javascript-memory-leak-in-node-js/#links)
- [概要](https://www.alexkras.com/simple-guide-to-finding-a-javascript-memory-leak-in-node-js/#summary)

> 你可能想要的书签：简单指南查找JavaScript内存泄漏在Node.js由[@ akras14 ](https://twitter.com/akras14)[https://t.co/oRyQboa8Uw](https://t.co/oRyQboa8Uw)
>
> \- Node.js（@nodejs）[2016年1月6日](https://twitter.com/nodejs/status/684678799896625152)

请考虑[在亚马逊上查看本指南](https://www.alexkras.com/recommends/kindle-memory-leak)，如果你会发现它有所帮助。

## 介绍

几个月前，我不得不调试Node.js中的内存泄漏。我发现了很多文章专门的主题，但即使仔细阅读其中一些，我仍然很困惑，我究竟应该做什么来调试我们的问题。

我的目的是这个职位是一个简单的指南，在节点中查找内存泄漏。我将概述一个易于遵循的方法，应该（在我看来）成为任何内存泄漏调试在节点的起点。在某些情况下，这种方法可能不够。我将链接到您可能想要考虑的一些其他资源。

## 最小理论

JavaScript是一种垃圾收集语言。因此，Node进程使用的所有内存都由V8 JavaScript引擎自动分配和取消分配。

V8如何知道何时解除分配内存？V8保留程序中所有变量的图形，从根节点开始。JavaScript中有4种类型的数据类型：Boolean，String，Number和Object。前3个是简单类型，它们只能保留分配给它们的数据（即文本字符串）。对象和JavaScript中的一切都是一个对象（即数组是对象），可以保持引用（指针）到其他对象。

[![内存图](https://www.alexkras.com/wp-content/uploads/memory-graph.png)](https://www.alexkras.com/wp-content/uploads/memory-graph.png)

周期性地，V8将遍历存储器图，尝试识别从根节点不再能够到达的数据组。如果从根节点无法访问，V8假定数据不再使用并释放内存。这个过程称为**垃圾收集**。

### 什么时候发生内存泄漏？

当一些不再需要的数据仍然可以从根节点到达时，在JavaScript中发生内存泄漏。V8将假设数据仍在使用，并且不会释放内存。**为了调试内存泄漏，我们需要找到错误保存的数据，并确保V8能够清理它。**

还有一点很重要，要注意的是，垃圾回收不会一直运行。通常V8可以在认为合适时触发垃圾收集。例如，它可以定期运行垃圾收集，或者它可以触发垃圾收集，如果它感测到可用内存量越来越低。节点对每个进程可用的内存数量有限，因此V8必须明智地使用它。

[![节点错误](https://www.alexkras.com/wp-content/uploads/node-error.png)](https://www.alexkras.com/wp-content/uploads/node-error.png)

后来的情况下，**垃圾收集**可能是**性能明显下降的来源**。

想象一下，你有一个应用程序有很多内存泄漏。很快，Node进程会开始耗尽内存，这将导致V8触发一个无法回收的垃圾收集。但是由于大多数数据仍然可以从根节点到达，非常少的内存将被清理，保持大部分的位置。

比以后更快，Node进程会再次运行内存，触发另一个垃圾收集。在你知道它之前，你的应用程序进入一个不断的垃圾收集周期，只是为了保持过程的功能。由于V8花费大部分时间来处理垃圾收集，因此只剩下很少的资源来运行实际程序。

## 步骤1.重现并确认问题

正如我前面指出的，V8 JavaScript引擎有一个复杂的逻辑，它用于确定何时运行垃圾收集。记住这一点，即使我们可以看到Node进程的内存继续上升，**我们不能确定我们目睹了内存泄漏，直到我们知道Garbage Collection已经运行**，允许未使用的内存被清除。

幸运的是，Node允许我们手动触发垃圾收集，这是我们在尝试确认内存泄漏时应该做的第一件事。这可以通过运行带有`--expose-gc`标志（ie `node --expose-gc index.js`）的Node来实现。一旦节点在该模式下运行，您可以随时通过`global.gc()`从您的程序调用来以编程方式触发垃圾收集。

您还可以通过调用来检查进程使用的内存量`process.memoryUsage().heapUsed`。

**通过手动触发垃圾收集和检查使用的堆，你可以确定你是否实际上观察你的程序中的内存泄漏。**

### 示例程序

我创建了一个简单的内存泄漏程序，你可以在这里看到：[https](https://github.com/akras14/memory-leak-example) : [//github.com/akras14/memory-leak-example](https://github.com/akras14/memory-leak-example)

您可以克隆它，运行`npm install`，然后运行`node --expose-gc index.js`以查看它的操作。

| 12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849 | "use strict";require('heapdump'); var leakyData = [];var nonLeakyData = []; class SimpleClass {  constructor(text){    this.text = text;  }} function cleanUpData(dataStore, randomObject){  var objectIndex = dataStore.indexOf(randomObject);  dataStore.splice(objectIndex, 1);} function getAndStoreRandomData(){  var randomData = Math.random().toString();  var randomObject = new SimpleClass(randomData);   leakyData.push(randomObject);  nonLeakyData.push(randomObject);   // cleanUpData(leakyData, randomObject); //<-- Forgot to clean up  cleanUpData(nonLeakyData, randomObject);} function generateHeapDumpAndStats(){  //1. Force garbage collection every time this function is called  try {    global.gc();  } catch (e) {    console.log("You must run program with 'node --expose-gc index.js' or 'npm start'");    process.exit();  }   //2. Output Heap stats  var heapUsed = process.memoryUsage().heapUsed;  console.log("Program is using " + heapUsed + " bytes of Heap.")   //3. Get Heap dump  process.kill(process.pid, 'SIGUSR2');} //Kick off the programsetInterval(getAndStoreRandomData, 5); //Add random data every 5 millisecondssetInterval(generateHeapDumpAndStats, 2000); //Do garbage collection and heap dump every 2 seconds |
| ---------------------------------------- | ---------------------------------------- |
|                                          |                                          |

程序将：

1. 每5毫秒生成一个随机对象并将其存储在2个数组中，一个名为*leakyData*和另一个*nonLeakyData*。我们将每5毫秒清除nonLeakyData数组，但是我们会**“忘记”**清理leakyData数组。
2. 每2秒，程序将输出所使用的内存量（并生成堆转储，但我们将在下一节中讨论更多）。

如果用`node --expose-gc index.js`（或`npm start`）运行程序，它将开始输出内存统计信息。让它运行一两分钟，并杀死它`Ctr + c`。

你会看到内存快速增长，即使我们每2秒触发一次垃圾收集，在我们得到统计数据之前：

| 123456789101112 | //1. Force garbage collection every time this function is calledtry {  global.gc();} catch (e) {  console.log("You must run program with 'node --expose-gc index.js' or 'npm start'");  process.exit();} //2. Output Heap statsvar heapUsed = process.memoryUsage().heapUsed;console.log("Program is using " + heapUsed + " bytes of Heap.") |
| --------------- | ---------------------------------------- |
|                 |                                          |

使用stats输出看起来像下面：

| 12345678910111213141516 | Program is using 3783656 bytes of Heap.Program is using 3919520 bytes of Heap.Program is using 3849976 bytes of Heap.Program is using 3881480 bytes of Heap.Program is using 3907608 bytes of Heap.Program is using 3941752 bytes of Heap.Program is using 3968136 bytes of Heap.Program is using 3994504 bytes of Heap.Program is using 4032400 bytes of Heap.Program is using 4058464 bytes of Heap.Program is using 4084656 bytes of Heap.Program is using 4111128 bytes of Heap.Program is using 4137336 bytes of Heap.Program is using 4181240 bytes of Heap.Program is using 4207304 bytes of Heap. |
| ----------------------- | ---------------------------------------- |
|                         |                                          |

如果你绘制数据，内存增长变得更加明显。

[![带内存泄漏](https://www.alexkras.com/wp-content/uploads/with-memory-leak.png)](https://www.alexkras.com/wp-content/uploads/with-memory-leak.png)

*注意：如果你好奇我如何绘制数据，请继续阅读。如果没有，请跳到下一节。*

我将输出的统计信息保存到一个JSON文件中，然后读入它并用几行Python绘制它。我把它保持在单独的早午餐以避免混乱，但你可以在这里查看：[https](https://github.com/akras14/memory-leak-example/tree/plot)：[//github.com/akras14/memory-leak-example/tree/plot](https://github.com/akras14/memory-leak-example/tree/plot)

相关部分为：

| 1234567891011121314151617181920212223 | var fs = require('fs');var stats = []; //--- skip --- var heapUsed = process.memoryUsage().heapUsed;stats.push(heapUsed); //--- skip --- //On ctrl+c save the stats and exitprocess.on('SIGINT', function(){  var data = JSON.stringify(stats);  fs.writeFile("stats.json", data, function(err) {    if(err) {      console.log(err);    } else {      console.log("\nSaved stats to stats.json");    }    process.exit();  });}); |
| ------------------------------------- | ---------------------------------------- |
|                                       |                                          |

和

| 1234567891011121314 | #!/usr/bin/env python import matplotlib.pyplot as pltimport json statsFile = open('stats.json', 'r')heapSizes = json.load(statsFile) print('Plotting %s' % ', '.join(map(str, heapSizes))) plt.plot(heapSizes)plt.ylabel('Heap Size')plt.show() |
| ------------------- | ---------------------------------------- |
|                     |                                          |

你可以检查出[plot](https://github.com/akras14/memory-leak-example/tree/plot)分支，并像往常一样运行程序。一旦你完成运行`python plot.py`生成情节。您需要在您的机器上安装[Matplotlib](http://matplotlib.org/)库才能正常工作。

或者可以在Excel中绘制数据。

## 第2步。至少取3堆堆

好了，所以我们重现了问题，现在是什么？现在我们需要弄清楚问题在哪里，并解决它

您可能已经注意到我的示例程序中的以下行：

| 12345678 | require('heapdump');// ---skip--- //3. Get Heap dumpprocess.kill(process.pid, 'SIGUSR2'); // ---skip--- |
| -------- | ---------------------------------------- |
|          |                                          |

我使用一个node-heapdump模块，你可以在这里找到：[https](https://github.com/bnoordhuis/node-heapdump) : [//github.com/bnoordhuis/node-heapdump](https://github.com/bnoordhuis/node-heapdump)

为了使用node-heapdump，你只需要：

1. 安装它。
2. 要求它在你的程序的顶部
3. `kill -USR2 {{pid}}`在Unix上调用像平台

如果你从来没有看到该`kill`部分，它是Unix中的一个命令，它允许你（除了别的以外）发送一个自定义信号（aka User Signal）给任何正在运行的进程。Node-heapdump配置为进行进程的堆转储，任何时候它接收**用户信号两个**因此`-USR2`，后跟进程id。

在我的示例程序中，我`kill -USR2 {{pid}}`通过运行自动化命令`process.kill(process.pid, 'SIGUSR2');`，其中`process.kill`是一个`kill`命令的节点包装器，`SIGUSR2`是Node的说法`-USR2`，并`process.pid`获取当前Node进程的ID。我在每个垃圾收集之后运行此命令以获得干净的堆转储。

我不认为`process.kill(process.pid, 'SIGUSR2');`会在Windows上工作，但你可以运行`heapdump.writeSnapshot()`。

这个例子可能会更容易一些`heapdump.writeSnapshot()`，但是我想提一提的是，你可以`kill -USR2 {{pid}}`在Unix上像平台一样触发堆 信号，这样可以派上用场。

下一节将讨论如何使用生成的堆转储来隔离内存泄漏。

## 步骤3.查找问题

在第2步中，我们生成了一堆堆转储，但**我们至少需要3个**，你很快就会明白为什么。

一旦你有你的堆转储。转到Google Chrome浏览器，打开Chrome开发工具（Windows上为F12或Mac上为Command + Options + i）。

一旦进入开发工具导航到“配置文件”选项卡，选择屏幕底部的“加载”按钮，导航到您采取的第一个堆转储，并选择它。堆转储将加载到Chrome视图中，如下所示：

[![第一堆](https://www.alexkras.com/wp-content/uploads/1st-Heap-Dump.png)](https://www.alexkras.com/wp-content/uploads/1st-Heap-Dump.png)

继续加载2个堆转储到视图中。例如，您可以使用您所采取的最后2个堆转储。最重要的是，堆转储必须按照它们被采用的顺序加载。您的“配置文件”选项卡应类似于以下内容。

[![3堆堆](https://www.alexkras.com/wp-content/uploads/3-Heap-Dumps.png)](https://www.alexkras.com/wp-content/uploads/3-Heap-Dumps.png)

从上面的图像可以看出，堆随着时间的推移继续增长。

### 3堆倾销法

一旦堆转储被加载，您将在“个人档案”选项卡中看到很多子视图，并且很容易丢失它们。然而，有一种观点，我发现特别有帮助。

点击你已经采取的最后一个堆转储，它会立即将你进入“摘要”视图。在“摘要”下拉列表的左侧，您应该会看到另一个显示“全部”的下拉菜单。点击它并选择“在heapdump-YOUR-FIRST-HEAP-DUMP和heapdump-YOUR-SECOND-TO-LAST-HEAP-DUMP之间分配的对象”，如下图所示。

[![3堆堆视图](https://www.alexkras.com/wp-content/uploads/3-Heap-Dump-View.png)](https://www.alexkras.com/wp-content/uploads/3-Heap-Dump-View.png)

它将显示有时在您的第一个堆转储和第二个到最后一个堆转储之间分配的所有对象。这些事情，这些对象仍然挂在你的最后堆转储是引起关注，应该调查，因为他们应该被拾起由垃圾收集。

相当惊人的东西实际上，但不是很直观，发现和容易忽视。

### 忽略括号中的任何内容，例如（字符串），至少在开头

完成示例应用程序的概述步骤后，我结束了以下视图。

注意，**浅尺寸**表示对象本身的大小，而**保留尺寸**表示对象的尺寸和它的所有子。

[![内存泄漏](https://www.alexkras.com/wp-content/uploads/memory-leak.png)](https://www.alexkras.com/wp-content/uploads/memory-leak.png)

似乎有5个条目保留在我上次的快照，应该不存在：（数组），（编译代码），（字符串），（系统）和SimpleClass。

其中只有**SimpleClass**看起来很熟悉，因为它来自示例应用程序中的以下代码。

| 12   | var randomObject = new SimpleClass(randomData); |
| ---- | ---------------------------------------- |
|      |                                          |

可能很有可能先通过（数组）或（字符串）条目开始查找。摘要视图中的所有对象按其构造函数名称分组。在数组或字符串的情况下，这些是JavaScript引擎内部的构造函数。虽然你的程序肯定坚持通过这些构造函数创建的一些数据，你也会在那里得到很多噪音，使得更难找到内存泄漏的来源。

这就是为什么最好跳过这些，而是看看你是否可以发现任何更明显的嫌疑犯，如示例应用程序中的**SimpleClass**构造函数。

单击SimpleClass构造函数中的下拉箭头，并从结果列表中选择任何创建的对象，将填充窗口下部的保留路径（参见上图）。从那里，很容易跟踪leakyData数组持有我们的数据。

如果你在你的应用程序没有幸运，像我在我的示例应用程序，你可能需要看看内部构造函数（如字符串），并试图找出是什么导致内存泄漏。在这种情况下，诀窍是尝试识别在一些内部构造器组中经常出现的值组，并尝试使用它作为指向可疑内存泄漏的提示。

例如，在示例应用程序案例中，您可能会观察到很多字符串看起来像转换为字符串的随机数。如果您检查其保留路径，Chrome开发工具将指向leakyData数组。

## 步骤4.确认问题已解决

在您确定并修复了可疑的内存泄漏后，您应该会发现堆使用情况有很大的不同。

如果我们在示例应用中取消注释以下行：

| 12   | cleanUpData(leakyData, randomObject); //<-- Forgot to clean up |
| ---- | ---------------------------------------- |
|      |                                          |

并按照步骤1中所述重新运行应用程序，请注意以下输出：

| 12345678910111213141516 | Program is using 3756664 bytes of Heap.Program is using 3862504 bytes of Heap.Program is using 3763208 bytes of Heap.Program is using 3763400 bytes of Heap.Program is using 3763424 bytes of Heap.Program is using 3763448 bytes of Heap.Program is using 3763472 bytes of Heap.Program is using 3763496 bytes of Heap.Program is using 3763784 bytes of Heap.Program is using 3763808 bytes of Heap.Program is using 3763832 bytes of Heap.Program is using 3758368 bytes of Heap.Program is using 3758368 bytes of Heap.Program is using 3758368 bytes of Heap.Program is using 3758368 bytes of Heap. |
| ----------------------- | ---------------------------------------- |
|                         |                                          |

如果我们绘制数据，它将看起来如下：

[![无内存泄漏](https://www.alexkras.com/wp-content/uploads/without-memory-leak.png)](https://www.alexkras.com/wp-content/uploads/without-memory-leak.png)

Hooray，内存泄漏了。

*注意，内存使用的初始峰值仍然存在，这是正常的，而你等待程序稳定。注意你的分析中的尖峰，以确保你不会将其解释为内存泄漏。*

## 链接到一些其他资源

### 使用Chrome DevTools进行内存分析

<iframe width="560" height="315" src="https://www.youtube.com/embed/L3ugr9BJqIs" frameborder="0" allowfullscreen></iframe>

您在本文中阅读的大部分内容都来自上面的视频。本文存在的唯一原因是，我必须在两个星期内观看这个视频3次，以发现（我相信是）的关键点，我想让发现过程更容易为其他人。

我强烈建议观看这个视频补充这篇文章。

### 另一个有用的工具 - memwatch-next

这是另一个很酷的工具，我认为值得一提。你可以[在这里](https://hacks.mozilla.org/2012/11/tracking-down-memory-leaks-in-node-js-a-node-js-holiday-season/)阅读更多的一些推理（短读，值得你的时间）。

或者直接去回购：[https](https://github.com/marcominetti/node-memwatch)：[//github.com/marcominetti/node-memwatch](https://github.com/marcominetti/node-memwatch)

为了节省您的点击，您可以安装它 `npm install memwatch-next`

然后使用它与两个事件：

| 12345678910 | var memwatch = require('memwatch-next');memwatch.on('leak', function(info) { /*Log memory leak info, runs when memory leak is detected */ });memwatch.on('stats', function(stats) { /*Log memory stats, runs when V8 does Garbage Collection*/ }); //It can also do this...var hd = new memwatch.HeapDiff();// Do something that might leak memoryvar diff = hd.end();console.log(diff); |
| ----------- | ---------------------------------------- |
|             |                                          |

最后一个控制台日志将输出如下内容，显示内存中已经生成了什么类型的对象。

| 12345678910111213141516171819 | {  "before": { "nodes": 11625, "size_bytes": 1869904, "size": "1.78 mb" },  "after":  { "nodes": 21435, "size_bytes": 2119136, "size": "2.02 mb" },  "change": { "size_bytes": 249232, "size": "243.39 kb", "freed_nodes": 197,    "allocated_nodes": 10007,    "details": [      { "what": "String",        "size_bytes": -2120,  "size": "-2.07 kb",  "+": 3,    "-": 62      },      { "what": "Array",        "size_bytes": 66687,  "size": "65.13 kb",  "+": 4,    "-": 78      },      { "what": "LeakingClass",        "size_bytes": 239952, "size": "234.33 kb", "+": 9998, "-": 0      }    ]  }} |
| ----------------------------- | ---------------------------------------- |
|                               |                                          |

很酷。

### 从developer.chrome.com的JavaScript内存分析

[https://developer.chrome.com/devtools/docs/javascript-memory-profiling](https://developer.chrome.com/devtools/docs/javascript-memory-profiling)

绝对是必读。它涵盖了我所涉及的所有主题和更多，更多的细节，更准确的🙂

不要忽略底部的Addy Osmani的演讲，他提到了一堆调试提示和资源。

你可以幻灯片[在这里](https://speakerdeck.com/addyosmani/javascript-memory-management-masterclass)：和示例代码[在这里](https://github.com/addyosmani/memory-mysteries)：

## 概要

请考虑[在Amazon上查看本指南](https://www.alexkras.com/recommends/kindle-memory-leak)，如果您发现它有帮助。

1. 尝试重现和识别内存泄漏时手动触发垃圾收集。您可以从程序中运行带有`--expose-gc`标志和调用的Node `global.gc()`。
2. 使用[https://github.com/bnoordhuis/node-heapdump](https://github.com/bnoordhuis/node-heapdump)采取至少3堆堆转储
3. 使用3堆转储方法隔离内存泄漏
4. 确认内存泄漏已消失
5. 利润



* *原文：[Simple Guide to Finding a JavaScript Memory Leak in Node.js](https://www.alexkras.com/simple-guide-to-finding-a-javascript-memory-leak-in-node-js/)*