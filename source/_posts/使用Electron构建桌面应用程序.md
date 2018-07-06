---
title: 使用Electron构建桌面应用程序
---

> 使用JavaScript，Node.js和Electron 构建您自己的声音机器的详细指南

### JavaScript桌面应用程序的用途和用途

桌面应用程序总是在我心中有一个特别的地方。自从浏览器和移动设备功能强大以来，桌面应用程序的稳步下降，这些应用程序正在被移动和Web应用程序所取代。尽管如此，编写桌面应用程序仍然有很多优势 - 一旦他们在开始菜单或停靠栏中，它们总是存在，它们是*alt（cmd）-tabbable*（我希望这是一个字），并且大多数与底层操作系统（其快捷方式，通知等）比Web应用程序。

在本文中，我将尝试引导您完成构建简单桌面应用程序的过程，并了解如何使用JavaScript构建桌面应用程序的重要概念。

![img](https://cdn-images-1.medium.com/max/400/1*GS-t3eNz9Jy7YWKIxxmJPg.png)

​										GitHub Electron

使用JavaScript开发桌面应用程序的主要思想是您构建一个代码库，并分别为每个操作系统打包。这样可以消除构建本机桌面应用程序所需的知识，并使维护变得更简单。如今，发展与JavaScript的一个桌面应用程序依赖于任何[电子](http://electron.atom.io/)或[NW.js](http://nwjs.io/)。虽然这两种工具提供的功能或多或少相同，但我已经和Electron进行[了交流](https://github.com/atom/electron/blob/master/docs/development/atom-shell-vs-node-webkit.md)，因为它具有[一些重要的优势](https://github.com/atom/electron/blob/master/docs/development/atom-shell-vs-node-webkit.md)。在一天结束的时候，你也不会出错。

#### 基本假设

我假设你已经安装了基本的文本编辑器（或IDE）和[Node.js / npm](https://nodejs.org/download/)。我也假设你有HTML / CSS / JavaScript的知识（Node.js的CommonJS模块的知识将是巨大的，但并不重要），所以我们可以专注于学习电子概念，而不用担心构建用户界面（其中，事实证明，只是普通的网页）。如果没有，你可能会感到有些失落，我建议您访问[我以前的博客帖子](https://medium.com/@bojzi/overview-of-the-javascript-ecosystem-8ec4a0b7a7be)来刷新您的基础知识。

#### 电子的10,000英尺视图

简而言之，Electron提供了使用纯JavaScript构建桌面应用程序的运行时。它的工作原理是 - Electron采用您的*package.json*文件中定义的*主*文件并执行它。这个主文件（通常命名为*main.js*）然后创建包含渲染网页的应用程序窗口，其中增加了与操作系统的本地GUI（图形用户界面）进行交互的功能。****

详细来说，一旦您使用Electron启动应用*程序，*就会创建一个*主要的过程*。这个*主要过程*负责与您的操作系统的本机GUI进行交互，并创建应用程序的GUI（您的应用程序窗口）。

![img](https://cdn-images-1.medium.com/max/600/1*EJETq7XOPz5RVY5IfF6NIg@2x.png)

纯粹启动的*主要过程*不会给您的应用*程序*的用户任何应用程序窗口。这些由主文件中的*主进程*使用称为*BrowserWindow*模块的*东西创建*。然后，每个浏览器窗口运行其自己的*渲染器进程*。这个*渲染器进程*需要一个网页（引用通常的CSS文件，JavaScript文件，图像等的HTML文件）并将其呈现在窗口中。您的网页使用[Chromium](https://www.chromium.org/)呈现，所以与标准保持非常高的兼容性。

例如，如果您只有一个计算器应用程序，您的*主要过程*将使用实际网页（计算器）的网页实例化一个窗口。

虽然说只有*主进程*与操作系统的本机GUI进行交互，但是有一些技术可以将一些工作卸载到*渲染器进程（*我们将研究如何利用这种技术构建一个特性*）。*

的*主要过程*可以通过一系列模块的访问本地GUI [直接在电子提供](https://github.com/atom/electron/tree/master/docs/api)。您的桌面应用程序可以访问所有节点模块，如优秀的[*节点通知程序*](https://github.com/mikaelbr/node-notifier)，以显示系统通知，[*请求*](https://www.npmjs.com/package/request)进行HTTP呼叫等。

------

### 你好，世界！

让我们开始一个传统的问候语，并安装所有必要的先决条件。

#### 随附存储库

本指南附有[声音机教程资料库](https://github.com/bojzi/sound-machine-electron-guide)。
使用存储库在某些点跟随或继续。克隆存储库以开始：

```
git clone https://github.com/bojzi/sound-machine-electron-guide.git
```

然后您可以跳转到*sound-machine-tutorial*文件夹中的git标签：

```
git checkout <tag-name>
```

当代码块的代码可用时，我会通知您：

```
跟随：
 git checkout 00-blank-repository
```

克隆/检出您想要的标签后，运行：

```
npm安装
```

以便您没有丢失任何Node模块。

如果您不能切换到另一个标签，最好只需重置存储库状态，然后执行结帐：

```
git add  - 
git reset --hard
```

#### 开设店铺

```
跟随标签00-blank-repository：
 git checkout 00-blank-repository
```

在项目文件夹中创建一个新的*package.json*文件，其中包含以下内容：

<script src="https://gist.github.com/bojzi/8d6d0273a4a196f5e9d7.js"></script>

这个准系统*package.json：*

- 设置应用程序的名称和版本，
- 让Electron知道*主进程*将要运行哪个脚本（*main.js*）和
- 设置一个有用的快捷方式 - 通过在CLI（终端或命令提示符）中运行“ *npm start* ”，可轻松运行应用程序的*npm*脚本。**

现在是获得*电子*的时候了。实现这一目标的最简单方法是通过*npm*为您的操作系统安装一个预先构建的二进制文件，并将其保存为package.json中的*开发依赖关系（*由--save *-dev自动进行）*。在CLI中运行以下命令（在项目文件夹中）：

```
npm安装--save-dev电子预制
```

预先构建的二进制码是针对正在安装的操作系统而定制的，并允许运行“ *npm启动* ”。我们正在将其安装为开发依赖关系，因为我们只需要在开发过程中。

也就是说，或多或少地，您开始使用*Electron开发*所需的一切。

#### 问候世界

在该文件夹中创建一个**应用程序**文件夹和一个*index.html*文件，其内容如下：

```html
<h1>你好， 世界！</h1>
```

在项目的根目录中创建一个*main.js*文件。这就是Electron的主要进程将会启动并允许创建我们的“Hello，world！”网页的文件。创建具有以下内容的文件：

<script src="https://gist.github.com/bojzi/6c63fb06d0b8b306481c.js"></script>

```js
// Please update it, I had to do:

'use strict';

const {app, BrowserWindow} = require('electron')
let mainWindow = null

app.on('ready', function() {
    mainWindow = new BrowserWindow({
        height: 600,
        width: 800
    });

    mainWindow.loadURL('file://' + __dirname + '/app/index.html');
});
// The require line is wrong.
// It's mainWindow.loadURL, not mainWindow.loadUrl.
// I'm using Electron 1.3.4.
```

没什么可怕的吧？*应用程序*模块控制您
的*应用程序*生命周期（例如 - 响应*应用程序*的就绪状态）*。*该*BrowserWindow*模块允许窗口创建。
在*主窗口*对象将是你的主应用程序窗口，并宣布为*无效*，因为窗口否则将一次JavaScript的垃圾收集踢关闭。

一旦*应用程序*获得*就绪*事件，我们使用*BrowserWindow*创建一个新的800像素宽和600像素高的窗口。
该窗口的*渲染器进程*将渲染我们的*index.html*文件。

运行我们的“Hello，World！”应用程序，在您的CLI中运行以下命令：

```
npm start
```

并沐浴在您的应用程序的荣耀中。

### 开发一个真正的应用程序

#### 一台光荣的音响机器

第一件事 - 首先  *是什么声音机器*？
声音机器是一种小型设备，当您按各种按钮，主要是卡通或反应声音时，会发出声音。这是一个有趣的小工具，以减轻办公室的情绪和一个很好的用例来开发一个桌面应用程序，因为我们将在开发过程中探索很多概念（并获得一个漂亮的声音引导）。

![img](https://cdn-images-1.medium.com/max/800/1*El4nvvh3h3vjRgwh_wVO7A.png)



我们将要构建的功能和我们将要探索的概念是：

- 基本声音机（基本浏览器窗口实例化），
- 关闭所述声机（远程消息之间*主要*和*渲染过程），*
- 播放声音而不使应用程序焦点（全局键盘快捷键），
- 创建快捷键修改键（Shift，Ctrl和Alt）的设置屏幕（将用户设置存储在主文件夹中）
- 添加托盘图标（远程创建本机GUI元素并了解菜单和托盘图标）和
- 打包您的应用程序（打包您的应用程序为Mac，Windows和Linux）。

------

### 构建声音机的基本功能

#### 起点和应用组织

在您的腰带下工作的“Hello，world！”应用程序，现在是开始构建音响机器的时候了。

典型的声音机器具有几行按钮，通过制作声音来响应印刷机。声音主要是卡通和/或反应（笑声，鼓掌，玻璃破碎等）。

这也是我们构建的第一个功能 - 响应点击的基本声音机器。

![img](https://cdn-images-1.medium.com/max/400/1*CsKlT-mLLFJlRoYZnAk9Tg@2x.png)

基本文件和文件夹结构

我们的应用程序结构将非常简单。

在应用程序的**根目录**中，我们将保留*package.json*文件，*main.js*文件和我们需要的任何其他应用程序范围的文件。

该**应用程序**文件夹会将我们的各种类型的HTML文件放在像**css**，**js**，**wav**和**img**这样的文件夹中。

为了使事情更容易，网页设计所需的所有文件已经被包含在存储库的初始状态中。请检查标签01-start-project。如果您遵循并创建了“Hello，world！”应用程序，则必须重新设置存储库，然后执行checkout：

```sh
If you followed along with the "Hello, world!" example:
git add -A
git reset --hard
Follow along with the tag 01-start-project:
git checkout 01-start-project
```

为了保持简单，我们将只有两个声音，但扩展到完整的16个声音只是一个额外的声音，额外的图标和修改*index.html的问题*。

#### 定义主要过程的其余部分

我们再来看看*main.js*来定义声音机的外观。用以下替换文件的内容：

<script src="https://gist.github.com/bojzi/44b5de344405860026ea.js"></script>

我们通过给它一个维度来定制我们创建的窗口，使其不可调整大小并且无框架。它将看起来像一个真正的声音机悬停在桌面上。

现在的问题是 - 如何移动无框窗口（没有标题栏）并关闭它？
我将很快谈论定制窗口（和应用程序）关闭（并介绍一种在*主进程*和*渲染器进程*之间进行通信的方式），但拖动部分很容易。如果您查看*index.css*文件（在**app / css中**），您将看到以下内容：

```css
html,
body {
    ...
    -webkit-app-region: drag;
    ...
}
```

*-webkit-app-region：drag; *允许整个*html*是一个可拖动的对象。现在有一个问题，但是您不能单击可拖动对象上的按钮。谜题的另一部分是*-webkit-app-region：no-drag; *这允许您定义不可破坏（因此可点击的元素）。请考虑以下摘录*index.css*：

```css
.button-sound {
    ...
    -webkit-app-region: no-drag;
}
```

#### 在自己的窗口中显示声音机

在*main.js*现在文件可以使一个新的窗口，并显示机器声音。而且真的，如果您以*npm开始*启动您的应用*程序，*您将看到声音机器活着。现在没有什么发生，这并不奇怪，因为我们只有一个静态网页。

将以下内容放在*index.js*文件中（位于**app / js中**）以获得交互性：

<script src="https://gist.github.com/bojzi/c8d21791c2ab34834aea.js"></script>

这段代码很简单。我们：

- 查询声音按钮，
- 通过读取*数据声音*属性的按钮，
- 为每个按钮添加一个背景图像
- 并为播放音频的每个按钮添加一个点击事件（使用[HTMLAudioElement界面](https://developer.mozilla.org/en/docs/Web/API/HTMLAudioElement)）

通过在CLI中运行以下命令来测试应用程序：

```
npm开始
```

![img](https://cdn-images-1.medium.com/max/800/1*Umzbl_QY7tatF_9Khm400Q@2x.png)			 

​									一个工作的声音机！

### 通过远程事件从浏览器窗口关闭应用程序

```sh
# Follow along with the tag 02-basic-sound-machine:
git checkout 02-basic-sound-machine
```

要重述 - 应用程序窗口（更准确地说，它们的*渲染器进程*）不应该与GUI进行交互（这就是关闭窗口）。在[官方电子快速入门指南](https://github.com/atom/electron/blob/master/docs/tutorial/quick-start.md)说：

> 在网页中，不允许调用本机GUI相关的API，因为在网页中管理本机GUI资源非常危险，容易泄漏资源。如果要在网页中执行GUI操作，则网页的渲染器进程必须与主进程通信，以请求主进程执行这些操作。

Electron提供用于该类型通信的[*ipc*（进程间通信）模块](https://github.com/atom/electron/blob/master/docs/api/ipc-renderer.md)。*ipc*允许订阅*频道*上的消息并向*通道的*订户发送消息。信道用于区分消息的接收者，并用字符串（例如“channel-1”，“channel-2”...）表示。消息还可以包含数据。收到消息后，用户可以通过做一些工作做出反应，甚至可以回答。消息传递的最大好处是分离问题 - *主进程*不必知道哪些*渲染器进程*有哪些或哪个发送消息。

![img](https://cdn-images-1.medium.com/max/800/1*VkIt0RY92_fSiZDa9RC6Ng@2x.png)

你有邮件。

这正是我们在这里做的 - 将*主进程*（*main.js*）订阅到“ *关闭主窗口* ”通道，并在有人点击关闭按钮时从*渲染器进程*（*index.js*）在该*通道*上发送消息。****

将以下内容添加到*main.js*以订阅频道：

```js
var ipc = require('ipc');

ipc.on('close-main-window', function () {
    app.quit();
});
```

在要求模块之后，在通道上订阅消息非常简单，并且涉及使用带有通道名和回调函数的*on（）*方法。

要在该频道上发送消息，请将以下内容添加到*index.js中*：

```js
var ipC =  require（' ipc '）;

var closeEl =  document。querySelector（' .close '）;
closeEl。addEventListener（' click '，function（）{
    ipc。send（' close-main-window '）;
}）;
```

再次，我们需要*ipc*模块，并使用关闭按钮将*点击*事件绑定到元素。点击关闭按钮，我们通过*send（）*方法通过“close-main-window”通道*发送消息*。

还有一个可能咬你更多的细节和我们已经谈过它- *可点击*拖动领域。*index.css*必须将关闭按钮定义为不可拖动。

```css
.settings {
    ...
    -webkit-app-region：no-drag;
}
```

就是这样，我们的应用程序现在可以通过关闭按钮关闭。通过检查事件或传递参数，通过*ipc进行通信*可能会变得复杂，我们稍后会看到一个传递参数的例子。

### 通过全局键盘快捷键播放声音

```sh
# Follow along with the tag 03-closable-sound-machine:
git checkout 03-closable-sound-machine
```

我们的基本音响机器工作很棒。但是我们确实有一个可用性的问题 - 一个声音机器有什么用途，它必须一直坐在所有窗口的前面，并重复点击？

这是全球键盘[快捷键的](https://github.com/atom/electron/blob/master/docs/api/global-shortcut.md)地方。Electron提供了一个[全球快捷键](https://github.com/atom/electron/blob/master/docs/api/global-shortcut.md)模块，可让您聆听自定义的键盘组合并作出反应。键盘组合被称为[*加速器*](https://github.com/atom/electron/blob/master/docs/api/accelerator.md)，并且是按键组合的字符串表示（例如“Ctrl + Shift + 1”）。

![img](https://cdn-images-1.medium.com/max/800/1*xx7pP0czBfFTskrN5tmyZQ@2x.png)

插上呃，按播放！

由于我们想要捕获一个本机GUI事件（全局键盘快捷键）并执行一个应用程序窗口事件（播放声音），我们将使用我们的可信*ipc*模块将消息从*主进程*发送到*渲染器进程*。

在潜入代码之前，需要考虑两件事情：

1. 在*应用程序“ready”*事件（代码应该在该块中）之后，必须注册全局快捷方式
2. 当通过*ipc*从*主进程*发送消息到*渲染器进程*时，必须使用该窗口的*引用（*像*“createdWindow.webContents.send（'channel'）*）

考虑到这一点，让我们改变我们的*main.js*并* *添加以下代码：

<script src="https://gist.github.com/bojzi/e805132f2c737d1a39c4.js"></script>

首先，我们需要*全局快捷方式*。然后，一旦我们的应用程序准备就绪，我们会注册两个快捷方式 - 一个将按Ctrl，Shift和1一起响应，另一个会一起响应Ctrl，Shift和2。其中的每一个都将在“ *全局快捷方式* ”通道上发送一条消息，其中包含一个参数。我们将使用该参数来播放正确的声音。将以下内容添加到*index.js中*：

```js
//index.js
ipc.on('global-shortcut', function (arg) {
    var event = new MouseEvent('click');
    soundButtons[arg].dispatchEvent(event);
});
```

为了简单起见，我们将模拟一个按钮单击，并使用我们在绑定按钮播放声音时创建的soundButtons选择器。一旦一个消息带有参数1，我们将使用*soundButtons [1]*元素并触发鼠标点击（*注意*：在生产应用程序中，您将要封装声音播放代码并执行）。

### 通过新窗口中的用户设置配置修改键

```sh
 # Follow along with the tag 04-global-shortcuts-bound:
 git checkout 04-global-shortcuts-bound
```

有许多应用程序在同一时间运行，可能非常好，我们设想的快捷方式已经被采用。这就是为什么我们要介绍一个设置屏幕，并存储我们要使用哪些修饰符（Ctrl，Alt和/或Shift）。

要完成所有这些，我们将需要以下内容：

- 我们主窗口中的设置按钮，
- 设置窗口（附带HTML，CSS和JavaScript文件），
- *ipc*消息打开和关闭设置窗口并更新我们的全局快捷方式和
- 从用户系统存储/读取设置JSON文件。

呃，这是一个很好的名单。

#### 设置按钮和设置窗口

类似关闭主窗口，我们要一个上发送消息*通道*从*index.js*的设置按钮被点击时。将以下内容添加到*index.js中*：

```js
var settingsEl = document.querySelector('.settings');
settingsEl.addEventListener('click', function () {
    ipc.send('open-settings-window');
});
```

点击设置按钮后，会在频道“ *open-settings-window* ”上发送一条消息。*main.js*现在可以对该事件做出反应并打开新窗口。将以下内容添加到*main.js中*：

<script src="https://gist.github.com/bojzi/9bda0bbc4e6bd9ba043b.js"></script>

没有什么新鲜事可以看到，我们正在打开一个新窗口，就像我们在主窗口中一样。唯一的区别是，我们正在检查设置窗口是否已经打开，以便我们不打开两个实例。

一旦工作，我们需要一种关闭设置窗口的方法。再次，我们将在频道上发送消息，但这次是从*settings.js*（这是设置关闭按钮所在的位置）。使用以下内容创建（或替换）*settings.js*的内容：

<script src="https://gist.github.com/bojzi/eee51448178ebceafc24.js"></script>

并在*main.js中*监听该*频道*。添加以下内容：**

<script src="https://gist.github.com/bojzi/c18a4af41b82a4d51c77.js"></script>

我们的设置窗口现在可以实现自己的逻辑了。

#### 存储和阅读用户设置

```
跟随标签05-settings-window-working：
 git checkout 05-settings-window-working
```

与设置窗口进行交互，存储设置并将其推广到我们的应用程序的过程将如下所示：

- 创建一种在JSON文件中存储和读取用户设置的方法，
- 使用这些设置显示设置窗口的初始状态，
- 更新用户交互时的设置
- 让*主流程*知道变化。

我们可以在*main.js*文件中实现对设置的存储和读取，但它似乎是一个很好的用例来编写一个可以包含在各个地方的小模块。

**使用JSON配置**

这就是为什么我们要创建*configure.js*文件* *，并在需要时要求它。Node.js使用[CommonJS模块模式](https://nodejs.org/docs/latest/api/modules.html)，这意味着您仅导出API，而其他文件需要/使用该API上可用的功能。

![img](https://cdn-images-1.medium.com/max/800/1*YnvmoGoSwPb-EMDK5GJwzA@2x.png)

main.js和settings.js都使用了configure.js。

为了使存储和阅读更容易，我们将使用*nconf*模块，为我们提供JSON文件的读取和写入。这是一个很好的合适。但首先，我们必须将它包含在项目中，并在CLI中执行以下命令：

```
npm install --save nconf
```

这告诉npm将*nconf*模块安装为*应用*程序依赖关系，当我们为最终用户打包应用程序时（与使用仅包含用于开发目的的模块的*save-dev*参数进行安装相反），它将被包含和使用。

该*configuration.js*文件是非常简单的，所以让我们充分研究它。在项目根目录中创建一个*配置*文件，具有以下内容：

<script src="https://gist.github.com/bojzi/54598153b65bf4b832c2.js"></script>

*nconf*只想知道在哪里存储你的设置，我们给它的位置的用户主文件夹和文件名。获取用户主文件夹仅仅是要求Node.js（*process.env*）和区分各种平台（如在*getUserHome（）*函数中所观察到的）。

然后，使用*nconf*（*set（）*存储用于使用*save（）*读取的*get **（）*和用于文件操作的*load（））*的内置方法并使用标准的CommonJS *模块*导出API来实现存储或读取设置*。导出*语法。

**初始化默认快捷键修饰符**

在进行设置交互之前，让我们初始化设置，以防我们第一次启动应用程序。我们将把修饰符键存储为一个带有“ *shortcutKeys* ” 键的数组，并在*main.js中进行*初始化。对于所有这些工作，我们必须首先要求我们的*配置*模块：

<script src="https://gist.github.com/bojzi/69dca93b69eec3c6b25a.js"></script>

如果设置键“ *shortcutKeys* ” 下有任何东西存储，我们尝试阅读。如果没有，我们设置一个初始值。

作为*main.js*中的另外一件事，我们将重写全局快捷键的注册，作为我们稍后在更新设置时可以调用的功能。从*main.js*中删除注册快捷键，并以这种方式更改文件：

<script src="https://gist.github.com/bojzi/2e12c414223085dd1387.js"></script>

该功能重置全局快捷方式，以便我们可以设置新的，从设置读取修饰符键数组，将其转换为[Accelerator兼容的](https://github.com/atom/electron/blob/master/docs/api/accelerator.md)字符串，并执行通常的全局快捷键注册。

**在设置窗口中进行交互**

回到*settings.js*文件，我们需要绑定将要更改我们的全局快捷方式的点击事件。首先，我们将遍历复选框并标记活动的（从配置模块读取值）：

<script src="https://gist.github.com/bojzi/4f69b66856d1011d3a13.js"></script>

现在我们将绑定复选框行为。考虑到设置窗口（及其*渲染器进程）*不允许更改GUI绑定。这意味着我们需要从*settings.js*发送一条*ipc*消息（稍后再处理该消息）**：**

<script src="https://gist.github.com/bojzi/05e26e3da937e5c1c3f5.js"></script>

这是一个更大的代码，但仍然很简单。
我们遍历所有复选框，绑定一个点击事件，并且每次点击检查设置数组是否包含修饰符键 - 并根据该结果修改阵列，将结果保存到设置并向*主进程*发送消息这应该更新我们的全球快捷方式。

所有剩下要做的就是订阅*IPC*通道“ *设置全局快捷键* ”，在*main.js*和更新我们的全球快捷方式：

```js
ipc.on('set-global-shortcuts', function () {
    setGlobalShortcuts();
});
```

就是这样，而且我们的全球快捷键是可配置的！

### 菜单上有什么？

```
跟随标签06-shortcuts可配置：
 git checkout 06-shortcuts-configured
```

桌面应用程序中的另一个重要概念是菜单。有一个永远有用的上下文菜单（AKA右键单击菜单），托盘菜单（绑定到托盘图标），应用程序菜单（在OS X上）等。

![img](https://cdn-images-1.medium.com/max/400/1*HinA-91xPYoCKel_obvuug@2x.png)

Megamenu。

在本指南中，我们将添加一个带有菜单的托盘图标。我们也将利用这个机会去探索进程间通信的另一种方式- [ 在*远程*模块](https://github.com/atom/electron/blob/master/docs/api/remote.md)。

该*远程*模块，使从RPC风格调用*渲染过程*的*主要过程*。您需要模块并在*渲染器进程中*使用它们，但是它们正在*主流程中*实例化，您调用它们的方法正在*主进程中*执行。实际上，这意味着您可以在*index.js中*远程请求本机GUI模块，并在其上调用方法，但是它们在*main.js中*执行。这样，您可以要求来自*index.js*的BrowserWindow模块并实例化一个新的浏览器窗口。幕后，**

![img](https://cdn-images-1.medium.com/max/800/1*vLP63PA79c_s3ZdbJCe-yg@2x.png)

打电话给我，可能的话？

让我们看看如何创建一个菜单，并将其绑定到托盘图标，同时在*渲染器进程中执行*。将以下内容添加到*index.js中*：

<script src="https://gist.github.com/bojzi/19c859e8cedd407bfdac.js"></script>

本地GUI模块（*菜单*和*托盘*）是远程需要的，这样可以安全地使用它们。

托盘图标通过其图标定义。OS X支持图像模板（根据约定，如果文件的文件名以“Template”结尾），则图像被认为是模板图像，这样可以轻松处理黑暗和轻薄的主题。其他操作系统获得常规图标。

电子有多种方式来建立菜单。这样可以创建一个菜单模板（一个带菜单项的简单数组），并从该模板中构建一个菜单。最后，新菜单附加到托盘图标。

### 包装你的应用程序

```
跟随标签07-ready-for-packaging：
 git checkout 07-ready-for-packaging
```

你不能让人下载和使用的应用程序的用途是什么？

![img](https://cdn-images-1.medium.com/max/400/1*O-ZI4nwQON9kNosiyfEfMw@2x.png)

包起来。

[使用*电子包装机*](https://github.com/maxogden/electron-packager)轻松[*包装*](https://github.com/maxogden/electron-packager)所有平台的[应用](https://github.com/maxogden/electron-packager)[*程序*](https://github.com/maxogden/electron-packager)。简而言之，电子包装机将所有的工作全部抽出来，将电子包裹在您的应用程序中，并生成您要发布的所有平台。

它可以用作CLI应用程序或构建过程的一部分。构建更复杂的构建方案不在本文的范围之内，但是我们将利用*npm*脚本的强大功能使打包更容易。使用电子包装机是微不足道的，包装应用的一般形式是：

```
电子包装机<项目位置> <项目名称> <platform> <architecture> <电子版> <可选选项>
```

哪里：

- 项目的位置指向您项目所在的文件夹，
- 项目名称定义您的项目的名称，
- 平台决定构建哪些平台（*全部*为Windows，Mac和Linux构建），
- 架构决定了要构建的架构（x86或x64，*全部*为两者）和
- 电子版可让您选择要使用的电子版本。

第一个包将需要一段时间，因为所有平台的所有二进制文件都必须下载。随后的包快得多。

我通常像这样（在Mac上）打包声音机：

```
电子包装机〜/工程/声音机SoundMachine --all --version = 0.30.2 --out =〜/ Desktop --overwrite --icon =〜/ Projects / sound-machine / app / img / app-icon .icns
```

命令中包含的新选项是不言自明的。要获得一个漂亮的图标，您首先必须将其转换为.icns（适用于Mac）和/或.ico（Windows）。只要搜索到您的PNG文件转换为这些格式就像一个工具[这一块](https://iconverticons.com/online/)（一定要下载的文件与  *.icns*扩展，而不是  *.HQX*）。如果从非Windows操作系统打包Windows，则需要葡萄酒（Mac用户可以使用brew，而Linux用户可以使用apt-get）。

每次运行这个大命令是没有意义的。我们可以在*package.json*中添加另一个脚本。首先，安装电子包装机作为开发依赖：

```
npm安装--save-dev电子包装机
```

现在我们可以在我们的*package.json*文件中添加一个新脚本：

```json
"scripts": {
  "start": "electron .",
  "package": "electron-packager ./ SoundMachine --all --out ~/Desktop/SoundMachine --version 0.30.2 --overwrite --icon=./app/img/app-icon.icns"
}
```

*包装变得容易得多*

然后在CLI中运行以下命令：

```
npm start
```

软件包命令启动电子打包器，查看当前目录并构建到桌面。如果您使用Windows，应该更改脚本，但这是微不足道的。

目前状态下的声音机器最终重量高达100 MB。不要担心，一旦存档（zip或您选择的存档类型），它将失去一半以上的大小。

如果你真的想去镇上，看看[*电子制造商*](https://github.com/loopline-systems/electron-builder)，它采用*电子包装机*生产的*包装*，并创建自动安装。

------

### 附加功能添加

随着应用程序打包并准备好，您现在可以开始开发自己的功能。

这里有一些想法：

- 一个帮助屏幕，包括应用程序的信息，其快捷方式和作者，
- 添加图标和菜单项以打开信息屏幕，
- 构建一个漂亮的包装脚本，以加快构建和分配，
- 使用[节点通知器](https://github.com/mikaelbr/node-notifier)添加[通知](https://github.com/mikaelbr/node-notifier)，让用户知道他们正在播放哪些声音，
- 在更大程度上使用*lodash*来实现更清晰的代码库（如通过数组迭代）
- 在打包之前使用构建工具缩小所有CSS和JavaScript，
- 将上述节点通知器与服务器调用相结合，以检查应用程序的新版本并通知用户...

对于一个很好的挑战 - 尝试提取您的声音机浏览器窗口逻辑，并使用像browserify这样的一个网页创建一个与刚才创建的相同的声音机器。一个代码库 - 两个产品（桌面应用程序和Web应用程序）。漂亮！

------

### 深入浅出Electron

我们只是刮了电子带给桌子的表面。很容易做到像在主机上观看电源事件或在屏幕上获取各种信息（如光标位置）等操作。

对于所有这些内置实用程序（通常在使用Electron开发应用程序时），[请查看Electron API文档](https://github.com/atom/electron/tree/master/docs/api)。

这些Electron API文档是Electron GitHub存储库中docs文件夹的一部分，该文件夹非常值得一试。

Sindre Sorhus指出[一系列电子资源](https://github.com/sindresorhus/awesome-electron)，您可以在其中找到非常酷的项目和信息，如一个典型的电子应用程序架构的优秀概述，可以作为我们迄今开发的代码的复习。

最终，Electron基于io.js（将被并入Node.js），大部分Node.js模块是兼容的，可用于扩展应用程序。只需浏览[npmjs.com](https://www.npmjs.com/)并抓住所需的内容。

------

### 这就是全部？

不，这才是开始

现在是时候建立更大的应用程序了。我主要跳过在本指南中使用额外的库或构建工具来专注于重要问题，但您可以轻松地在ES6或Typescript中编写应用程序，使用Angular或React并简化您的构建与gulp或grunt。

使用您最喜欢的语言，框架和构建工具，为什么不使用Flickr API和node-flickrapi或使用Google官方Node.js客户端库的GMail客户端构建Flickr同步桌面应用程序？

选择一个想法点子，它将促进激励你进一步学习，初始化一个存储库，马上开始行动吧！

[Electron已经经历了一些API流失。特别：```const {app} = require（'electron'）const {BrowserWindow} = require（'electron'）```此外，`loadUrl`函数也被重命名为`loadURL`



> [Building a desktop application with Electron – Developers Writing – Medium](https://medium.com/developers-writing/building-a-desktop-application-with-electron-204203eeb658)