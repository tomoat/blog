---
title: 三种Ajax方式
---



## 一、原生 JS 实现 AJAX

JS 实现 AJAX 主要基于浏览器提供的 XMLHttpRequest（XHR）类，所有现代浏览器（IE7+、Firefox、Chrome、Safari 以及 Opera）均内建 XMLHttpRequest 对象。

#### 1. 获取XMLHttpRequest对象

```js
// 获取XMLHttpRequest对象
var xhr = new XMLHttpRequest();

```

#### 2. 发送一个 HTTP 请求

接下来，我们需要打开一个URL，然后发送这个请求。分别要用到 XMLHttpRequest 的 open() 方法和 send() 方法。

## 二、 jQuery 实现 AJAX

jQuery 作为一个使用人数最多的库，其 AJAX 很好的封装了原生 AJAX 的代码，在兼容性和易用性方面都做了很大的提高，让 AJAX 的调用变得非常简单。下面便是一段简单的 jQuery 的 AJAX 代码：

```js
$.ajax({
  method: 'POST',
  url: '/api',
  data: { username: 'admin', password: 'root' }
})
  .done(function(msg) {
    alert( 'Data Saved: ' + msg );
  });

```

对比原生 AJAX 的实现，使用 jQuery 就异常简单了。当然我们平时用的最多的，是下面两种更简单的方式：

```js
// GET
$.get('/api', function(res) {
  // do something
});

// POST
var data = {
  username: 'admin',
  password: 'root'
};
$.post('/api', data, function(res) {
  // do something
});

```

## 三、Fetch API

使用 jQuery 虽然可以大大简化 XMLHttpRequest 的使用，但 XMLHttpRequest 本质上但并不是一个设计优良的 API： + 不符合关注分离（Separation of Concerns）的原则 + 配置和调用方式非常混乱 + 使用事件机制来跟踪状态变化 + 基于事件的异步模型没有现代的 Promise，generator/yield，async/await 友好

Fetch API 旨在修正上述缺陷，它提供了与 HTTP 语义相同的 JS 语法，简单来说，它引入了 fetch() 这个实用的方法来获取网络资源。

原生支持率并不高，幸运的是，引入下面这些 polyfill 后可以完美支持 IE8+：

- 由于 IE8 是 ES3，需要引入 ES5 的 polyfill: [es5-shim, es5-sham**](https://link.zhihu.com/?target=https%3A//github.com/es-shims/es5-shim)
- 引入 Promise 的 polyfill: [es6-promise**](https://link.zhihu.com/?target=https%3A//github.com/stefanpenner/es6-promise)
- 引入 fetch 探测库：[fetch-detector**](https://link.zhihu.com/?target=https%3A//github.com/camsong/fetch-detector)
- 引入 fetch 的 polyfill: [fetch-ie8**](https://link.zhihu.com/?target=https%3A//github.com/camsong/fetch-ie8)
- 可选：如果你还使用了 jsonp，引入 [fetch-jsonp**](https://link.zhihu.com/?target=https%3A//github.com/camsong/fetch-jsonp)
- 可选：开启 Babel 的 runtime 模式，现在就使用 async/await

#### 1. 一个使用 Fetch 的例子

先看一个简单的 Fetch API 的例子 🌰 ：

```javascript
fetch('/api').then(function(response) {
  return response.json();
}).then(function(data) {
  console.log(data);
}).catch(function(error) {
  console.log('Oops, error: ', error);
});
```

使用 ES6 的箭头函数后：

```js
fetch('/api').then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.log('Oops, error: ', error))
```

可以看出使用Fetch后我们的代码更加简洁和语义化，链式调用的方式也使其更加流畅和清晰。但这种基于 Promise 的写法还是有 Callback 的影子，我们还可以用 async/await 来做最终优化：

```js
async function() {
  try {
    let response = await fetch(url);
    let data = response.json();
    console.log(data);
  } catch (error) {
    console.log('Oops, error: ', error);
  }
}

```

使用 await 后，写代码就更跟同步代码一样。await 后面可以跟 Promise 对象，表示等待 Promise resolve() 才会继续向下执行，如果 Promise 被 reject() 或抛出异常则会被外面的 try...catch 捕获。

Promise，generator/yield，await/async 都是现在和未来 JS 解决异步的标准做法，可以完美搭配使用。这也是使用标准 Promise 一大好处。

#### 2. 使用 Fetch 的注意事项

- Fetch 请求默认是不带 cookie，需要设置 fetch(url, {credentials: 'include'})`
- 服务器返回 400，500 错误码时并不会 reject，只有网络错误这些导致请求不能完成时，fetch 才会被 reject

接下来将上面基于 XMLHttpRequest 的 AJAX 用 Fetch 改写：

```js
var options = {
    method: 'POST',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ username: 'admin', password: 'root' }),
    credentials: 'include'
  };

fetch('/api', options).then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.log('Oops, error: ', error))
```

Github Issue: [分别使用 XHR、jQuery 和 Fetch 实现 AJAX · Issue #15 · nodejh/nodejh.github.io](https://link.zhihu.com/?target=https%3A//github.com/nodejh/nodejh.github.io/issues/15)



