---
title: JavaScript中的内存释放
---

> *内容来自网络 http://www.jianshu.com/p/3b7946c4b118*

## 一、如何查找当前作用域的上级作用域

```js
let num = 20;
function fn() {
    let num = 200;
    return function () {
        console.log(num);
    };
}
const f = fn();
f(); // 输出 200
```

以上代码fn中返回了一个函数，用f去接收这个返回的函数，然后再执行f()，最后输出的是200，刚接触的同学可能会有疑问，为什么在全局作用域下执行的f()输出的num不是全局作用域中的20，而是fn函数的私有作用域中的200！

**上级作用域查找规则：看当前函数是在哪个作用域下定义的，那么它的上级作用域就是谁，和函数哪里执行没有任何的关系。**

以上的代码在全局的变量和函数进行预解析之后，执行fn函数，fn函数又进行预解析形成自己的私有作用域，然后执行fn函数中的代码，最后返回一个函数，该函数被f接收。当f执行的时候，有一行代码输出num：`console.log(num)`，根据作用域搜索规则，首先在自己的作用域中找，没有找到，然后再到上级的作用域中查找，根据作用域查找的规则，**只看当前函数在哪个作用域下定义的**，所以f函数的上级作用域是fn。



![img](http://upload-images.jianshu.io/upload_images/2555024-a0df1d45fa0feb95.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

​									如何查找上级作用域.png

有了以上的基础之后，再看：

```js
let num = 20;
function fn() {
    let num = 200;
    return function () {
        console.log(num);
    };
}
const f = fn();
f(); // 输出 200

~function () {
    var num = 2000;
    f(); // 输出什么呢？
}();
// 等价于
~(function () {
  let num = 2000
  f()
}())
```

加上了一个自执行函数，我们知道自执行函数是有自己的作用域的，但是此时f函数执行，依然输出200。 要时刻记住上级作用域的查找规则：**只看当前函数在哪个作用域下定义的**。

这时候很多同学可能会疑惑了，我们不是讲解JavaScript的内存释放的嘛？怎么还讲起了作用域的内容，稍安勿躁...在JavaScript的内存释放中要用到这些知识呢！

## 二、堆内存的释放

对象数据类型或者函数类型在定义的时候，首先都会开辟一个堆内存，堆内存有一个引用地址，如果外面有引用这个地址，我们就说这个内存被占用了，就不能销毁了。

```js
var obj1 = {name:"iceamn"};
var obj2 = obj1;
```

​	![img](http://upload-images.jianshu.io/upload_images/2555024-5de991df187d2654.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)		

​										堆内存.png

如果想要让堆内存释放（销毁），只需要把所有引用它的变量赋值为null即可，如果当前的堆内存没有任何东西被占用了，那么浏览器会在空闲的时候把它销毁。也就是说，在上面的那种感觉情况下，只有把obj1和同obj2都置为null之后，0xff11这块对堆内存才会被释放，只要还有变量引用0xff11这块内存，它就不会释放。

## 三、栈内存的释放

#### 3.1、全局作用域

在全局作用域下，只有当页面关闭的时候，全局作用域才会被销毁。

#### 3.2、私有作用域

一般情况下，函数执行会形成一个新的私有作用域（在ES6之前只有函数执行才会产生私有作用域），当私有作用域中的代码执行完成后，当前作用域都会主动的进行释放和销毁。

不过依然有特殊的情况存在：当前私有作用域中的部分内容被作用域以外的东西占用了，那么当前作用域就不能销毁了。

##### 3.2.1、 函数返回了一个**引用数据类型的值（数组、函数...）**，并且**该引用类型的值在函数的外面被一个其他变量接收了**，这种情况下形成的私有作用域都不会销毁。

注意两个条件：
（1）函数返回引用数据类型的值；
（2）该引用类型的值在函数外面被一个其他变量接收了；

```js
function fn() {
    var num = 100;
    return function () {
        num ++;
        console.log(num);
    }
}
var f = fn(); // fn执行形成的作用域就不能再销毁了
```

**注意**：即使fn返回的函数中什么代码都没有，没有使用到fn私有作用域中的任何变量和函数，在以上情况下，fn的私有作用域也不会被销毁，即：

```js
function fn() {
    var num = 100;
    return function () {
    }
}
var f = fn();
```

##### 3.2.2、 在一个私有作用域中，给DOM元素绑定方法，私有作用域不能被销毁

```js
var btn = document.getElementById('btn1');
~function () {
    btn.onclick = function () {
    }
}();
```

在自执行函数中形成了一个私有的作用域，在这个私有作用域中为页面上的一个button元素绑定了点击事件，所以这个私有作用域也不能被销毁。

![img](http://upload-images.jianshu.io/upload_images/2555024-20fce3c4fbf5aac0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

​									自执行函数的情况.png

##### 3.2.3、 “不立即销毁”

```js
function fn() {
    var num = 100;
    return function () {
    }
}
fn()(); // 首先执行fn，返回一个小函数对应的内存地址，然后紧接着让返回的小函数再执行
```

以上代码就是“不立即销毁”的情况，fn返回的函数没有被其他的任何变量占用，但是还需要执行一次，所以暂时不能销毁，但返回的值执行完成后，浏览器会在空闲的时候把它销毁了。

还记得一开始介绍的上级作用域吗，我们再对那张图进行分析：

![img](http://upload-images.jianshu.io/upload_images/2555024-040545d454116ba1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

​									上级作用域的情况.png

只要某作用域还有被引用，那么该作用域就不能被销毁，一旦没有任何变量引用了，该私有作用域就会被销毁了。

## 四、练习题

------

- 第一题

  ```js
  function fn() {
    var i = 10;
    return function (n) {
        console.log(n + (++i));
    };
  }
  var f = fn();
  f(10); // 21
  f(20); // 32
  fn()(10); // 21
  fn()(20); // 31
  ```

- 第二题

  ```js
  function fn(i) {
        return function (n) {
            console.log(n + i++);
        }
    }
    var f = fn(13);
    f(12);//->25
    f(14);//->28
    fn(15)(12);//->27
    fn(16)(13);//->29
  ```