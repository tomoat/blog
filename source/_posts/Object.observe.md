---
title: 关于Object.observe的知识
---



## Object.observe

当前MVC模式在Angular.JS等冲击下已经变成了View视图和Model模型两个， 控制器由事件驱动机制替代，见 ：[MVI是一种Reactive MVC](http://www.jdon.com/46833)。

那么为什么JS不能自然支持事件通知机制呢？ES7引入了Object.observe：

```js
var person = {};j
var changesHandler = changes => console.log(changes);
Object.observe(person, changesHandler);
```

这样，当person的内容有更改，将自动触发changes函数，如下：

```js
person = {
　　name: " jdon"
}
　　如果person的名称从jdon改为jdon.com:
person = {
　　name: " jdon.com"
}
```

这种name内容变化，将触发函数changes输出：

****

```js
[
    {
        "type":"update",
        "name":"name",
        "oldValue: "jdon"
    }
]
```

如果person增加内容：

```js
person = {
　　name: " jdon.com"
　　employer: "banq"
}
```

激活函数输出：

```js
[
    {
        "type":"new",
        "name":"employer",
    }
]
```

如果person删除内容employer，那么输出是：

```js
[
    {
        "type":"delete",
        "name":"employer",
        "oldValue: "banq"
    }
]
```

解除观察是：

```js
Object.unobserve(person, changesHandler);
```

为了防止事件风暴，ES7使用了 Event Loop Queue + Microtasks，当事件处理器EventHandler发出改变事件给ChangeHandler以后，ChangeHandler不会排队在队列后面，而是替换当前EventHandler位置，这样在浏览器渲染页面之前完成所有事件通知：

![js7.png](http://karat.cc/img/93256ccb-d34f-4419-bd82-5cbb8c1cde88.png)