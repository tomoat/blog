---
title: JS数据类型判断的方式
---

# 一、typeof

```js
console.log(typeof 123456); // number
console.log(typeof null); // object
console.log(typeof undefined); // undefined
console.log(typeof '123456'); // string
console.log(typeof ''); // string
console.log(type new String); // object
console.log(typeof new Object); // object
console.log(typeof {}); // object
console.log(typeof function(){}); // function
console.log(typeof []); // object
console.log(typeof true); // boolean
console.log(typeof NaN); // number
console.log(typeof /^[-+]?\d+$/); // object
```

使用typeof检测数据类型，首先返回的都是一个字符串，其次字符串中包含了对应的数据类型，例如："number"、"string"、"boolean"、"undefined"、"function"、"object"

- typeof (引用类型) 除了函数, 都是 'object',比如 typeof /123/

- typeof null 为'object'

- typeof undefined 为 'undefined',通常, 如果使用两等号, null == undefined 为真.

- 转换为数字的常见用法 "10"-0或+"10", 如果没有转换成功,返回NaN,由于NaN 的一个特性: NaN != NaN,故判断转换成功与否的常见做法: (这也是我参见 jQuery的源码发现的,jQuery源码读100遍都不为过)

  ```js
    ("10x" - 0) == ("10x" - 0);
    // 结果为假!   
  ```



面试题：

```js
console.log(typeof typeof typeof function () {}); // string
```

typeof的局限性：不能具体的细分是数组还是正则，还是对象中其他的值，因为使用typeof检测数据类型，对于对象数据类型中的所有的值，最后返回的结果都是"object"。

**应用一**：添加默认值

```js
function fn(num1, num2) {
    // 方式一
    // if (typeof num2 === 'undefined') {
    //   num2 = 0;
    // }

    // 方式二：不适用fn(10,false)这种情况
    num2 = num2 || 0;
}
fn(10);
```

**应用二**：回调函数调用

```js
function fn(callback) {
    //typeof callback === 'function' ? callback() : null;
    callback && callback();
}
fn(function () {

});
```

# 二、instanceof

用于判断一个变量是否某个对象的实例,或用于判断一个变量是否某个对象的实例；

```js
var ary = [];
console.log(ary instanceof Array); // true
console.log(ary instanceof Object); // true

function fn() {}
console.log(fn instanceof Function); // true
console.log(fn instanceof Object); // true
```

只要在当前实例的原型链上，用instanceof检测出来的结果都是true，所以在类的原型继承中，最后检测出来的结果未必是正确的，例如：

```js
function Fn() {
}
Fn.prototype = new Array; // 原型继承：让子类的原型等于父类的一个实例
var f = new Fn;
console.log(f instanceof Array); // true
```

# 三、constructor

用于判断一个变量的原型，constructor 属性返回对创建此对象的数组函数的引用

Javascript中对象的prototype属性的解释是:返回对象类型原型的引用

constructor即构造函数，作用和instanceof非常的相似。

```js
var obj = [];
console.log(obj.constructor === Array); // true
console.log(obj.constructor === RegExp); // false
```

constructor可以处理基本数据类型的检测

```js
var num = 1;
console.log(num.constructor === Number); // true
```

constructor检测Object和instanceof不一样，一般情况下是检测不了的

```js
var reg = /^$/;
console.log(reg.constructor === RegExp); // true
console.log(reg.constructor === Object); // false
```

constructor的局限性：我们可以把类的原型进行重写，在重写的过程中，很有可能把之前的constructor给覆盖了，这样检测出来的结果就是不准确的。

```js
function Fn() {
}
Fn.prototype = new Array;
var f = new Fn;
console.log(f.constructor); // Array
```

对于特殊的数据类型null和undefined，它们的所属类型是Null和Undefined，但是浏览器把这两个类保护起来了，不允许在外面访问使用。

```js
console.log("----------------Number---------------");
var A = 123;
console.log(A instanceof Number); //false
console.log(A.constructor == Number); //true
console.log(A.constructor);
console.log("----------------String---------------");
var B = "javascript";
console.log(B instanceof String); //false
console.log(B.constructor == String); //true
console.log(B.constructor);
console.log("----------------Boolean---------------");
var C = true;
console.log(C instanceof Boolean); //false
console.log(C.constructor == Boolean); //true
console.log(C.constructor);
console.log("----------------null---------------");
var D = null;
console.log(D instanceof Object); //false
//console.log(D.constructor == null); //报错
//console.log(D.constructor); //报错
console.log("----------------undefined---------------");
var E = undefined;
//console.log(E instanceof undefined); // //报错
//console.log(E.constructor == undefined); //报错
//console.log(E.constructor); //报错
console.log("----------------function---------------");
var F = function() {};
console.log(F instanceof Function);
console.log(F.constructor == Function);
console.log(F.constructor);
console.log("----------------new function---------------");
function SB() {};
var G = new SB();
console.log(G instanceof SB);
console.log(G.constructor == SB);
console.log(G.constructor);
console.log("----------------new Object---------------");
var H = new Object;
console.log(H instanceof Object);
console.log(H.constructor == Object);
console.log(H.constructor);
console.log("-----------------Array--------------");
var I = [];
console.log(I instanceof Array);
console.log(I.constructor == Array);
console.log(I.constructor);
console.log("-----------------JSON--------------");
var J = {
  "good": "js",
  "node": "very good"
};
console.log(J instanceof Object);
console.log(J.constructor == Object);
console.log(J.constructor);
```

# 四、Object.prototype.toStrong.call()

toString的理解：

- 咋一看应该是转换为字符串，但是某些toString方法不仅仅是转换为字符串。对于Number、String、Boolean、Array、RegExp、Date、Function原型上的toString方法都是把当前的数据类转换为字符串的类型（它们的作用仅仅是用来转换为字符串的）

- Object.prototype.toString并不是用来转换为字符串的。

  ```js
  ({name:"iceman"}).toString() // --> "[object Object]"
  Math.toString() // --> "[object Math]"
  ```

Number.prototype.toString是转换字符串的

```js
console.log((1).toString()); // --> Number.prototype.toString，--> "1"  转换为字符串
console.log((1).__proto__.__proto__.toString()); // Object.prototype.toString --> "[object Object]"
console.log((128).toString(2/8/10)); // 把数字转换为二进制/八进制/十进制
```

Object.prototype.toStrong.call()是检测数据类型最准确最常用的方式，起原理为：

- 先获取Object原型上的toString方法，让方法执行，并且改变方法中的this关键字的指向；
- Object.prototype.toString 它的作用是返回当前方法的执行主体（方法中this）所属类的详细信息；

```js
var obj = {name:'iceman'};
// toString中的this是obj，返回的是obj所属类的信息 --> "[object Object]"
// 第一个object代表当前实例是对象数据类型的（这个是固定死的）
// 第二个Object，代表的是obj所属的类是Object
console.log(obj.toString());

// toString中的this是Math，那么返回的是Math所属类的信息 --> "[object Math]"
console.log(Math.toString());
```

检测其他类型：

```js
var ary = [];
console.log(Object.prototype.toString.call(ary)); // --> "[object Array]"
console.log(Object.prototype.toString.call(/^$/)); // --> "[object RegExp]"
console.log(({}).toString.call(1)); // --> "[object Number]"

console.log(({}).toString.call('珠峰')); // --> "[object String]"
console.log(({}).toString.call(true)); // --> [object Boolean]
console.log(({}).toString.call(undefined)); // -->[object Undefined]
console.log(({}).toString.call(null)); // -->[object Null]
console.log(({}).toString.call(function () {})); // -->[object Function]
```

实际使用：

```js
var ary = [];
console.log(Object.prototype.toString.call(ary) === '[object Array]'); // true

var reg = /^\[object Array\]$/;
console.log(reg.test(Object.prototype.toString.call(ary))); // true
```

检测原型继承的情况：

```js
function Fn() {
}
Fn.prototype = new Array;
var f = new Fn;
console.log(f instanceof Array); // true
console.log(Object.prototype.toString.call(f) === '[object Array]'); // false
```
```js
console.log({}.toString.call(1)); // [object Number]
console.log({}.toString.call("11")); // [object String]
console.log({}.toString.call(/123/)); // [object RegExp]
console.log({}.toString.call({})); // [object Object]
console.log({}.toString.call(function() {})); // [object Function]
console.log({}.toString.call([])); // [object Array]
console.log({}.toString.call(true)); // [object Boolean]
console.log({}.toString.call(new Date())); // [object Date]
console.log({}.toString.call(new Error())); // [object Error]
console.log({}.toString.call(null)); // [obejct Null]
console.log({}.toString.call(undefined)); // [object Undefined]
console.log(String(null)); // null
console.log(String(undefined)); // undefind
```

# **使用jQuery中的方法$.type()**

现在看看jQuery是怎么做的

```js
// 先申明一个对象,目的是用来做映射
var class2type = {};
// 申明一个core_toString() 的方法,得到最原始的toString() 方法,因为在很多对象中,toStrintg() 已经被重写 
var core_toString() = class2type.toString;
// 这里为 toStrintg() 后的结果和类型名做一个映射,申明一个core_toString() 后的结果,而值就是类型名
jQuery.each("Boolean Number String Function Array Date RegExp Object Error".split(" "), function(i, name) {
    class2type[ "[object " + name + "]" ] = name.toLowerCase();
});
```

```js
console.log($.type(1)); // number
console.log($.type("11")); // string
console.log($.type(/123/)); // regexp
console.log($.type({})); // object
console.log($.type(function() {})); // function
console.log($.type([])); // array
console.log($.type(true)); // boolean
console.log($.type(new Date())); // date
console.log($.type(new Error())); // error
console.log($.type(null)); // null
console.log($.type(undefined)); // undefined
console.log(String(null)); // null
console.log(String(undefined)); // undefined
```

上面的打印结果与

```js
class2type[ "[object " + name + "]" ] = name.toLowerCase();
```

不谋而合!

这是jQuery.type 的核心方法

```js
type: function( obj ) {
    if ( obj == null ) {
        return String( obj );
    }
    // Support: Safari <= 5.1 (functionish RegExp)
    return typeof obj === "object" || typeof obj === "function" ?
        class2type[ core_toString.call(obj) ] || "object" :
        typeof obj;
},
```

> 注意,为什么把 null 或者 undefined 单独讨论呢,因为 在一些版本浏览器中
>
> ```js
> console.log(core_toString.call(null));
> console.log(core_toString.call(undefined));
> ```
>
> 这是会报错的!

​     如果是对象类型,另:由于 在一些低版本的浏览器中,typeof /123/ 会返回的是 "function" 而不是 "object",所以这里要判断是否是函数,要明白 这里的 `typeof obj === function` 不是为了函数讨论的,因为函数本身就可以通过typeof 来得到类型.

```js
 typeof obj === "object" || typeof obj === "function" ?
        class2type[ core_toString.call(obj) ]
```

就直接返回class2type 中键值对的结果,,如果不是,那么一定就是基本类型, 通过 typeof 就可以啦.

```js
class2type[ core_toString.call(obj) ] || "object" :
// 这是防止一些未知情况的,如果未取到,就返回object
```

# **但是 jQuery.type 有一个很大的缺陷**

这是一个自定义类型

```js
function Person() {
    this.name = 'pawn';
}
var p = new Person();
console.log($.type(p));
console.log({}.toString.call(p));

```

> // 注意,这里会打印 [object Object],通过上面的方法,无法得到精确的自定义类型
> 这也是 它的一个大缺陷了!

![img](https://o5wwk8baw.qnssl.com/1e5b073b33c45488730d8deb12868272/large)

下面,我们通过构造函数的方式来获取精确类型

# **通过构造函数来获取类型**

在理解这个方法之前,需要理解两个点

prorotype 原型属性

**       我们知道,任何对象或者函数都直接或者间接的继承自Object 或者 Function， （其实最终Function 是继承自 Object 的，这属于原型链的知识了，见下图）。那么，任何一个对象都具有原型对象 __proto__ (这个对象只在chrome 和 firefox 暴露，但是在其他浏览器中也是存在的)，这个原型对象就是这个对象的构造函数的原型属性(这里可能有点绕,直接上图).**

![img](https://o5wwk8baw.qnssl.com/12213d42e388d24fdff3d89d87759869/large)


由于 任何函数都具有 原型属性prototype,并且这个原型属性具有一个默认属性 constructor,它是这个函数的引用,看下面的代码

```js
  function Person(){
      this.name = 'pawn';
  }
  console.log(Person.prototype.constructor === Person);   //true
```

发现,这两个东西其实一个东西

但是,在某些情况下,需要这么写

```js
  function Person(){
      this.name = 'pawn';
  }
  Person.protype = {
      XX: ... ,
      xx: ... ,
      ...
  }
```

这么做,就会覆盖原本的 protype 方法,那么construcor 就不存在了,这是,必须要显示的申明这个对象，

## **construction: Person, 这句话非常重要，作用是修正this指向**

```js
  Person.protype = {
      construction: Person,   //这句话的作用是修正this指向
      XX: ... ,
      xx: ... ,
      ...
  }
```

在jQuery的中,就是这么做的,

```js
  jQuery.fn = jQuery.prototype = {
    constructor: jQuery,
    init: function( selector, context, rootjQuery ) {
        var match, elem;
```

> 关于 jQuery对象封装的方式 也是非常值得研究

![img](https://o5wwk8baw.qnssl.com/d25e22b435da24b9fc0e48a756fa8923/large)

**注意,这里已经不是熟悉 [object Object],而是 已经重写了.**

也就是,如果调用一个函数的toString() 方法.那么就会打印这个函数的函数体.

![img](https://o5wwk8baw.qnssl.com/a6727d2bb9795b6fca8f5d3e147ee23c/large)

如何通过构造函数来获得变量的类型?

判断是否是基本类型

```js
   var getType = function(obj){
       if(obj == null){
          return String(obj);
       }
       if(typeof obj === 'object' || typeof obj === 'fucntion'){
           ...
       }else{
           // 如果不是引用类型,那么就是基本类型
           return typeof obj
       }
   }
```

如果是对象或者函数类型

```js
   function Person(){
       this.name = 'pawn';
   }
   var p = new Person();
   console.log(p.constructor);   //返回function Person(){...}
```

现在要做的事 : 如何将Person 提取出来呢?
毋庸置疑,字符串切割那一套肯定可以办到,但是太 low 啦!
这里,我使用正则将Person提取出来

```js
   var regex = /function\s(.+?)\(/
```

```js
   function Person(){
    this.name = 'pawn';
   }
   var p = new Person();
   var c = p.constructor
   var regex = /function\s(.+?)\(/;
   console.log('|' + regex.exec(c)[1] + '|');
```

![img](https://o5wwk8baw.qnssl.com/a3bb757e1ffd5347149c8b25c7207429/large)

 

其实,除了上面的正则,**每个函数还有一个name属性**************,返回函数名,但是ie8 是不支持的.

因此上面的代码可以写为:

```js
var getType = function(obj){
    if(obj == null){
        return String(obj);
    }
    if(typeof obj === 'object' || typeof obj === 'function'){ 
        var constructor = obj.constructor;
        if(constructor && constructor.name){
            return constructor.name;
        }
        var regex = /function\s(.+?)\(/;
        return regex.exec(c)[1];
    }else{
        // 如果不是引用类型,那么就是基本;类型
        return typeof obj;
    }
};
```

但是上面的代码太丑啦,将其简化

简化

```js
var getType = function(obj){
    if(obj == null){
        return String(obj);
    }
    if(typeof obj === 'object' || typeof obj === 'function'){ 
        return obj.constructor && obj.constructor.name.toLowerCase() || 
          /function\s(.+?)\(/.exec(obj.constructor)[1].toLowerCase();
    }else{
        // 如果不是引用类型,那么就是基本类型
        return typeof obj;
    }
};
```

还是比较麻烦,继续简化

```js
var getType = function(obj){
    if(obj == null){
       return String(obj);
    }
    return typeof obj === 'object' || typeof obj === 'function' ?
      obj.constructor && obj.constructor.name && obj.constructor.name.toLowerCase() ||
          /function\s(.+?)\(/.exec(obj.constructor)[1].toLowerCase():
      typeof obj;
};

```

好了,已经全部弄完了,写个代码测试一下:

```js
function Person(){
    this.name = 'pawn';
}
var p = new Person();

console.log(getType(p));
console.log(getType(1));
console.log(getType("a"));
console.log(getType(false));
console.log(getType(/123/));
console.log(getType({}));
console.log(getType(function(){}));
console.log(getType(new Date()));
console.log(getType(new Error()));
console.log(getType( null));
console.log(getType( undefined));
```

![img](https://o5wwk8baw.qnssl.com/5c65895b8b9a263ff9ffdb1d08b2e1cd/large)

![img](https://o5wwk8baw.qnssl.com/fd491ebf9ae42d5c35354ab09cc5b846/large)

![img](https://o5wwk8baw.qnssl.com/449e37cc16e3d33edf2322a7fa3df0f0/large)

![img](https://o5wwk8baw.qnssl.com/5c0e5637a1c0dbc8a287910b1c93ff16/large)

## **1.有时会看到Object.prototype.toString.call()**

![img](https://o5wwk8baw.qnssl.com/7b2f7d6431e299a69846a98d0e3e8dbe/large)

## **2.toString()是一个怎样的方法,他定义在哪里？**

![img](https://o5wwk8baw.qnssl.com/db7a90daea89620a5e689d180c68730a/large)

****

## **3.call.apply.bind可以吗？**

![img](https://o5wwk8baw.qnssl.com/1a8feab14763519f95add483816e3af5/large)

****

## **4.为神马要去call呢？用 Object.prototype.toString.call(obj) 而不用 obj.toString() 呢？**

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title></title>
        <script type="text/javascript">
            function A(){
                this.say=function(){
                    console.log("我是1");
                }
            }
            function B(){
                this.say=function(){
                    console.log("我是2");
                }
            }
            var a=new A();
            var b=new B();
            a.say.call(b);    //我是1
        </script>
    </head>
    <body>
    </body>
</html>

```

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title></title>
        <script type="text/javascript">
            function A(){
                this.name='SB';
                this.say=function(){
                    console.log("我是1");
                }
            }
            function B(){
                A.call(this);   //B继承A，重写say方法
                this.say=function(){
                    console.log("我是2");
                }
            }
            var a=new A();
            var b=new B();
            console.log(b.name);  //SB
            b.say();         //我是2
            a.say.call(b);    //我是1
        </script>
    </head>
    <body>
    </body>
</html>

```

## **就是怕你重写了toString,所以才要用object 最原始的他toString,所以才去call。**

## **5.Object.prototype.toString方法的原理是什么？**

参考链接：**http://www.jb51.net/article/79941.htm**

在JavaScript中,想要判断某个对象值属于哪种内置类型,最靠谱的做法就是通过Object.prototype.toString方法.

12var arr = [];console.log(Object.prototype.toString.call(arr)) //"[object Array]"

本文要讲的就是,toString方法是如何做到这一点的,原理是什么.

**ECMAScript 3**

在ES3中,Object.prototype.toString方法的规范如下:

115.2.4.2 Object.prototype.toString()

在toString方法被调用时,会执行下面的操作步骤:

\1. 获取this对象的[[Class]]属性的值.

\2. 计算出三个字符串"[object ", 第一步的操作结果Result(1), 以及 "]"连接后的新字符串.

\3. 返回第二步的操作结果Result(2).

[[Class]]是一个内部属性,所有的对象(原生对象和宿主对象)都拥有该属性.在规范中,[[Class]]是这么定义的

**[[Class]]一个字符串值,表明了该对象的类型.**

****

然后给了一段解释:

所有内置对象的[[Class]]属性的值是由本规范定义的.所有宿主对象的[[Class]]属性的值可以是任意值,甚至可以是内置对象使用过的[[Class]]属性的值.[[Class]]属性的值可以用来判断一个原生对象属于哪种内置类型.需要注意的是,除了通过Object.prototype.toString方法之外,本规范没有提供任何其他方式来让程序访问该属性的值(查看 15.2.4.2).

也就是说,把Object.prototype.toString方法返回的字符串,去掉前面固定的"[object "和后面固定的"]",就是内部属性[[class]]的值,也就达到了判断对象类型的目的.jQuery中的工具方法$.type(),就是干这个的.

在ES3中,规范文档并没有总结出[[class]]内部属性一共有几种,不过我们可以自己统计一下,原生对象的[[class]]内部属性的值一共有10种.分别是:"Array", "Boolean", "Date", "Error", "Function", "Math", "Number", "Object", "RegExp", "String".

**ECMAScript 5**

在ES5.1中,除了规范写的更详细一些以外,Object.prototype.toString方法和[[class]]内部属性的定义上也有一些变化,Object.prototype.toString方法的规范如下:

**15.2.4.2 Object.prototype.toString ( )**

在toString方法被调用时,会执行下面的操作步骤:

****

**如果this的值为undefined,则返回"[object Undefined]".**

**如果this的值为null,则返回"[object Null]".**

**让O成为调用ToObject(this)的结果.**

**让class成为O的内部属性[[Class]]的值.**

**返回三个字符串"[object ", class, 以及 "]"连接后的新字符串.**

****

可以看出,比ES3多了1,2,3步.第1,2步属于新规则,比较特殊,因为"Undefined"和"Null"并不属于[[class]]属性的值,需要注意的是,这里和严格模式无关(大部分函数在严格模式下,this的值才会保持undefined或null,非严格模式下会自动成为全局对象).第3步并不算是新规则,因为在ES3的引擎中,也都会在这一步将三种原始值类型转换成对应的包装对象,只是规范中没写出来.ES5中,[[Class]]属性的解释更加详细:

所有内置对象的[[Class]]属性的值是由本规范定义的.所有宿主对象的[[Class]]属性的值可以是除了"Arguments", "Array", "Boolean", "Date", "Error", "Function", "JSON", "Math", "Number", "Object", "RegExp", "String"之外的的任何字符串.[[Class]]内部属性是引擎内部用来判断一个对象属于哪种类型的值的.需要注意的是,除了通过Object.prototype.toString方法之外,本规范没有提供任何其他方式来让程序访问该属性的值(查看 15.2.4.2).

和ES3对比一下,第一个差别就是[[class]]内部属性的值多了两种,成了12种,一种是arguments对象的[[class]]成了"Arguments",而不是以前的"Object",还有就是多个了全局对象JSON,它的[[class]]值为"JSON".第二个差别就是,宿主对象的[[class]]内部属性的值,不能和这12种值冲突,不过在支持ES3的浏览器中,貌似也没有发现哪些宿主对象故意使用那10个值.

**ECMAScript 6**

ES6目前还只是工作草案,但能够肯定的是,[[class]]内部属性没有了,取而代之的是另外一个内部属性[[NativeBrand]].[[NativeBrand]]属性是这么定义的:

 

内部属性属性值描述
[[NativeBrand]]枚举NativeBrand的一个成员.该属性的值对应一个标志值(tag value),可以用来区分原生对象的类型.

 

**[[NativeBrand]]属性的解释:**

[[NativeBrand]]内部属性用来识别某个原生对象是否为符合本规范的某一种特定类型的对象.[[NativeBrand]]内部属性的值为下面这些枚举类型的值中的一个:NativeFunction, NativeArray, StringWrapper, BooleanWrapper, NumberWrapper, NativeMath, NativeDate, NativeRegExp, NativeError, NativeJSON, NativeArguments, NativePrivateName.[[NativeBrand]]内部属性仅用来区分区分特定类型的ECMAScript原生对象.只有在表10中明确指出的对象类型才有[[NativeBrand]]内部属性.

表10 — [[NativeBrand]]内部属性的值

 

属性值对应类型
NativeFunctionFunction objects
NativeArrayArray objects
StringWrapperString objects
BooleanWrapperBoolean objects
NumberWrapperNumber objects
NativeMathThe Math object
NativeDateDate objects
NativeRegExpRegExp objects
NativeErrorError objects
NativeJSONThe JSON object
NativeArgumentsArguments objects
NativePrivateNamePrivate Name objects

 

可见,和[[class]]不同的是,并不是每个对象都拥有[[NativeBrand]].同时,Object.prototype.toString方法的规范也改成了下面这样:

**15.2.4.2 Object.prototype.toString ( )**

在toString方法被调用时,会执行下面的操作步骤:

如果this的值为undefined,则返回"[object Undefined]".

如果this的值为null,则返回"[object Null]".

让O成为调用ToObject(this)的结果.

如果O有[[NativeBrand]]内部属性,让tag成为表29中对应的值.

否则

让hasTag成为调用O的[[HasProperty]]内部方法后的结果,参数为@@toStringTag.

如果hasTag为false,则让tag为"Object".

否则,

让tag成为调用O的[[Get]]内部方法后的结果,参数为@@toStringTag.

如果tag是一个abrupt completion,则让tag成为NormalCompletion("???").

让tag成为tag.[[value]].

如果Type(tag)不是字符串,则让tag成为"???".

如果tag的值为"Arguments", "Array", "Boolean", "Date", "Error", "Function", "JSON", "Math", "Number", "Object", "RegExp",或

者"String"中的任一个,则让tag成为字符串"~"和tag当前的值连接后的结果.

返回三个字符串"[object ", tag, and "]"连接后的新字符串.

表29 — [[NativeBrand]] 标志值

 

[[NativeBrand]]值标志值
NativeFunction"Function"
NativeArray"Array"
StringWrapper"String"
BooleanWrapper"Boolean"
NumberWrapper"Number"
NativeMath"Math"
NativeDate"Date"
NativeRegExp"RegExp"
NativeError"Error"
NativeJSON"JSON"
NativeArguments"Arguments"

 

 

可以看到,在规范上有了很大的变化,不过对于普通用户来说,貌似感觉不到.

也许你发现了,ES6里的新类型Map,Set等,都没有在表29中.它们在执行toString方法的时候返回的是什么?

``

`console.log(Object.prototype.toString.call(Map())) //"[object Map]"`

`console.log(Object.prototype.toString.call(Set())) //"[object Set]"`

其中的字符串"Map"是怎么来的呢:

**15.14.5.13 Map.prototype.@@toStringTag**

@@toStringTag 属性的初始值为字符串"Map".

由于ES6的规范还在制定中,各种相关规定都有可能改变,所以如果想了解更多细节.看看下面这两个链接,现在只需要知道的是:[[class]]没了,使用了更复杂的机制.

以上所述是JavaScript中Object.prototype.toString方法的原理



- [数据类型检测的四种方式 - 简书](http://www.jianshu.com/p/75057391ad51)
- [JS类型判断 - TalkingCoder](https://www.talkingcoder.com/article/6333557442705696719)