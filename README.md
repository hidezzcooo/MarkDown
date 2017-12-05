title:  7个重要的Javascript函数
author: 吴思泉
date:   2015-09-23
categories:
- 前端技术
- JavaScript
tag:
- Javascript
---

> 原文链接：http://davidwalsh.name/essential-javascript-functions

我记得早些时候你需要的仅仅是个javascipt函数，可以运行于任何浏览器中，因为浏览器商实现的特性不尽相同，不仅是最新的特性，而且包括像`addEventListener`和`attachEvent`这样的基础特性。
今非昔比，但是仍有一些函数，每个开发者都应该放入他们的工具库中，出于性能或易用性的目的。

<!-- more -->
## `debouce`

`debounce`函数可以在提升事件驱动的性能方面大显身手。如果你从没在`scroll`，`resize`，`key*`事件中使用过`deboucing`函数，那就大错特错。下面的`debounce`函数使你的代码高效:

```javascript
// 返回一个函数，只要一直被调用就不会执行。
// 只有在N秒后才会呗调用
// 如果传递了`immediate`参数，则在上升沿触发，否则在下降沿触发
function debounce(func, wait, immediate) {
  var timeout;
  return function() {
    var context = this, args = arguments;
    var later = function() {
      timeout = null;
      if (!immediate) func.apply(context, args);
    };
    var callNow = immediate && !timeout;
    clearTimeout(timeout);
    timeout = setTimeout(later, wait);
    if (callNow) func.apply(context, args);
  };
};

// Usage
var myEfficientFn = debounce(function() {
  // All the taxing stuff you do
}, 250);
window.addEventListener('resize', myEfficientFn);
```

在给定的时间帧内，`debounce`函数只允许调用一次`callback`。这对于事件频繁触发回调函数的情况非常重要。

## `poll`

正如我提到的`debouce`函数，有时，你不能在事件内表明一个想得到的状态—如果事件根本不存在，你需要每个时间间隔中不断地去检查这个状态。

```javascript
function poll(fn, callback, errback, timeout, interval) {
  var endTime = Number(new Date()) + (timeout || 2000);
  interval = interval || 100;

  (function p() {
    // If the condition is met, we're done!
    if(fn()) {
      callback();
    }
    // 如果条件不满足，但是并未到结束时间，继续定时
    else if (Number(new Date()) < endTime) {
      setTimeout(p, interval);
    }
    // Didn't match and too much time, reject!
    else {
      errback(new Error('timed out for ' + fn + ': ' + arguments));
    }
  })();
}

  // Usage:  保证元素是可见的
  poll(
    function() {
    return document.getElementById('lightbox').offsetWidth > 0;
  },
  function() {
  // Done, success回调
  },
  function() {
  // Error, failure回调
  }
);
```

polling在网页中一直很有用，未来也是！

## `once`

有时，你希望一个函数仅仅触发一次，就像你用过的`onload`事件。下面的代码展示了此功能：

```javascript
function once(fn, context) {
  var result;

  return function() {
    if(fn) {
      result = fn.apply(context || this, arguments);
      fn = null;
    }
    return result;
  };
}

// Usage
var canOnlyFireOnce = once(function() {
  console.log('Fired!');
});

canOnlyFireOnce(); // "Fired!"
canOnlyFireOnce(); //
```

`once`函数保证了一个给定的函数只能被调用一次，这样可以避免重复地初始化!



## `getAbsoluteUrl`

从一个字符串中获取绝对路径并不如你想象得那么简单。这里有个优雅的技巧，从输入的字符串中获取绝对路径：

```javascript
var getAbsoluteUrl = (function() {
  var a;

  return function(url) {
    if(!a) a = document.createElement('a');
    a.href = url;

    return a.href;
  };
})();

// Usage
getAbsoluteUrl('/something'); // http://davidwalsh.name/something
```



## `isNative`

了解一个函数是内置与否，从而知道它是否可以重写。下面的代码给出答案：

```javascript
;(function() {

  // 用于解析内部`[[Class]]`的值
  var toString = Object.prototype.toString;

  // 用于解析function的反编译源码
  var fnToString = Function.prototype.toString;

  // 用于解析host constructors (Safari > 4; really typed array specific)
  var reHostCtor = /^\[object .+?Constructor\]$/;

  // 编译一个使用内部方法作为模板的正则表达式
  // We chose `Object#toString` because there's a good chance it is not being mucked with.
  var reNative = RegExp('^' +
    // 强制将`Object#toString`转为字符串
    String(toString)
    // 转义regexp字符
    .replace(/[.*+?^${}()|[\]\/\\]/g, '\\$&')
    .replace(/toString|(function).*?(?=\\\()| for .+?(?=\\\])/g, '$1.*?') + '$'
  );

  function isNative(value) {
    var type = typeof value;
    return type == 'function'
      // 使用`Function#toString`绕过vlaue自己的`toString`方法以防被其被重写
      ? reNative.test(fnToString.call(value))
      // host object检测的倒退方法，因为一些环境把像typed arrays的对象，作为DOM方法，而这样的方法不同于普通内部方法的匹配模式
      : (value && type == 'object' && reHostCtor.test(toString.call(value))) || false;
  }

  // export however you want
  module.exports = isNative;
}());

// Usage
isNative(alert); // true
isNative(myCustomFunction); // false
```

这个函数并不是很好，但它确实可以解决问题！

## `insertRule`

我们都知道我们可以通过选择器（`document.querySelectorAll`）来获取`NodeList`，并且给它们每一个设置样式，但是还有种更高效地为选择器设置样式（就像你在stylesheet中做的）：

```javascript
var sheet = (function() {
  // Create the <style> tag
  var style = document.createElement('style');

  // Add a media (and/or media query) here if you'd like!
  // style.setAttribute('media', 'screen')
  // style.setAttribute('media', 'only screen and (max-width : 1024px)')

  // WebKit hack :(
  style.appendChild(document.createTextNode(''));

  // Add the <style> element to the page
  document.head.appendChild(style);

  return style.sheet;
})();

// Usage
sheet.insertRule("header { float: left; opacity: 0.8; }", 1);
```

这在动态的，侧重Ajax的网站非常有用。如果你这样为选择器设置样式，你不必考虑样式中每个元素是否和选择器是否匹配（现在或未来）。

## `matchesSelector`

经常我们在继续工作前，需要校验输入；确保其是真值，保证表单数据合法，等等。但是，多长时间才有一次需要确保元素无误的？你可以使用`matchesSelector`函数来验证一个元素是否在给定的选择器中：

```javascript
function matchesSelector(el, selector) {
  var p = Element.prototype;
  var f = p.matches || p.webkitMatchesSelector || p.mozMatchesSelector || p.msMatchesSelector || function(s) {
  return [].indexOf.call(document.querySelectorAll(s), this) !== -1;
  };
  return f.call(el, selector);
}

// Usage
matchesSelector(document.getElementById('myDiv'), 'div.someSelector[some-attribute=true]')
```

就这样：每个开发者都应该放入工具包的7个JavaScript函数。我有漏掉的函数吗？请与我分享。
