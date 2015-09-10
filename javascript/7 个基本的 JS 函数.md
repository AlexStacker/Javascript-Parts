# 7个常用的JS函数
文章来自：伯乐在线-刘健超-J.c http://web.jobbole.com/82540/

英文原文：http://davidwalsh.name/essential-javascript-functions

## 1.debounce
对于高耗能事件，debounce 函数是一种不错解决方案。如果你不对 scroll、resize、和 key* 事件使用 debounce  函数，那么你几乎等同于犯了错误。下面的 debounce 函数能让你的代码保持高效：

```
// 返回一个函数，如果它被不间断地调用，它将不会得到执行。
//该函数在停止调用 N 毫秒后，再次调用它才会得到执行。
//如果有传递 ‘immediate’ 参数，会马上将函数安排到执行队列中，而不会延迟。
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

// 用法
var myEfficientFn = debounce(function() {
        // 所有繁重的操作
}, 250);
window.addEventListener('resize', myEfficientFn);
```
debounce 函数不允许回调函数在指定时间内执行多于一次。当为一个会频繁触发的事件分配一个回调函数时，该函数显得尤为重要。

## 2. poll

尽管上面我提及了 debounce 函数，但如果事件不存在时，你就不能插入一个事件以判断所需的状态，那么就需要每隔一段时间去检查状态是否达到你的要求。

```
function poll(fn, callback, errback, timeout, interval) {
    var endTime = Number(new Date()) + (timeout || 2000);
    interval = interval || 100;

    (function p() {
            // 如果条件满足，则执行！
            if(fn()) {
                callback();
            }
            // 如果条件不满足，但并未超时，再来一次
            else if (Number(new Date()) < endTime) {
                setTimeout(p, interval);
            }
            // 不匹配且时间消耗过长，则拒绝！
            else {
                errback(new Error('timed out for ' + fn + ': ' + arguments));
            }
    })();
}

// 用法：确保元素可见
poll(
    function() {
        return document.getElementById('lightbox').offsetWidth > 0;
    },
    function() {
        // 执行，成功的回调函数
    },
    function() {
        // 错误，失败的回调函数
    }
);
```

## 3. once

有时候，你想让一个给定的功能只发生一次，类似于 onload 事件。下面的代码提供了你所说的功能：

```
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

// 用法
var canOnlyFireOnce = once(function() {
    console.log('Fired!');
});

canOnlyFireOnce(); // "Fired!"
canOnlyFireOnce(); // nada  
                   // 没有执行指定函数
```
once 函数确保给定函数只能被调用一次，从而防止重复初始化！

## 4. getAbsoluteUrl

从一个字符串变量得到一个绝对 URL，并不是你想象中这么简单。对于某些 URL 构造器，如果你不提供必要的参数就会出问题（而有时候你真的不知道提供什么参数）。下面有一个优雅的技巧，只需要你传递一个字符串就能得到相应的绝对 URL。
```
var getAbsoluteUrl = (function() {
    var a;

    return function(url) {
        if(!a) a = document.createElement('a');
        a.href = url;

        return a.href;
    };
})();

// 用法
getAbsoluteUrl('/something'); // http://davidwalsh.name/something
```

a 元素的 href 处理和 url 处理看似无意义，而 return 语句返回了一个可靠的绝对 URL。

## 5. isNative

如果你想知道一个指定函数是否是原生的，或者能不能通过声明来覆盖它。下面这段便于使用的代码能给你答案：

```
;(function() {

  // 用于处理传入参数 value 的内部&nbsp;`[[Class]]`&nbsp;
  var toString = Object.prototype.toString;

  // 用于解析函数的反编译代码
  var fnToString = Function.prototype.toString;

  // 用于检测宿主构造器 （Safari > 4 ;真的输出特定的数组）
  var reHostCtor = /^[object .+?Constructor]$/;

  // 用一个标准的原生方法作为模板，编译一个正则表达式。
  // 我们选择 'Object#toString' 因为它一般不会被污染。
  var reNative = RegExp('^' +
    // 将 'Object#toString' 强制转为字符串 
    String(toString)
    // 转义所有指定的正则表达式字符
    .replace(/[.*+?^${}()|[]/]/g, '$&')
    // 用 '.*?' 替换提及的 'toString' ，以保持模板的通用性。
    // 将 'for ...' 之类的字符替换掉，以兼容 Rhino 等环境，因为这些环境添加了额外的信息，如方法参数数量。
    .replace(/toString|(function).*?(?=()| for .+?(?=])/g, '$1.*?') + '$'
  );

  function isNative(value) {
    var type = typeof value;
    return type == 'function'
      // 用 'Function#toString' （fnToString）绕过了值（value）本身的 'toString' 方法，以免被伪造所欺骗。
      ? reNative.test(fnToString.call(value))
      // 回退到宿主对象的检查，因为某些环境（浏览器）将类型数组（typed arrays）之类的东西当作 DOM 方法，此时可能不遵循标准的原生正则表达式。
      : (value && type == 'object' && reHostCtor.test(toString.call(value))) || false;
  }

  // 导出函数
  module.exports = isNative;
}());

// 用法
isNative(alert); // true
isNative(myCustomFunction); // false
```

## 6. insertRule

我们都知道能通过选择器（通过 document.querySelectorAll ）获取一个 NodeList ，并可为每个元素设置样式，但有什么更高效的方法为选择器设置样式呢（例如你可以在样式表里完成）：

```
var sheet = (function() {
        // 创建 <style> 标签
    var style = document.createElement('style');

        // 如果你需要指定媒介类型，则可以在此添加一个 media (和/或 media query) 
    // style.setAttribute('media', 'screen')
    // style.setAttribute('media', 'only screen and (max-width : 1024px)')

    // WebKit hack :(
    style.appendChild(document.createTextNode(''));

        // 将 <style> 元素添加到页面
    document.head.appendChild(style);

    return style.sheet;
})();

// 用法
sheet.insertRule("header { float: left; opacity: 0.8; }", 1);
```

这对于一个动态且重度依赖 AJAX 的网站来说是特别有用的。如果你为一个选择器设置样式，那么你就不需要为每个匹配到的元素都设置样式（现在或将来）。

## 7. matchesSelector

我们经常会在进行下一步操作前进行输入校验，以确保是一个可靠值，或确保表单数据是有效的，等等。但我们平时是怎么确保一个元素是否有资格进行进一步操作呢？如果一个元素有给定匹配的选择器，那么你可以使用 matchesSelector 函数来校验：
```
function matchesSelector(el, selector) {
    var p = Element.prototype;
    var f = p.matches || p.webkitMatchesSelector || p.mozMatchesSelector || p.msMatchesSelector || function(s) {
        return [].indexOf.call(document.querySelectorAll(s), this) !== -1;
    };
    return f.call(el, selector);
}

// 用法
matchesSelector(document.getElementById('myDiv'), 'div.someSelector[some-attribute=true]')
```
