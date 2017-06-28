---
layout: post
title: JavaScript 常见面试题分析
category: Web 开发
tags: [ 'javascript', 'programming' ]
---

# 问题列表
1. 说出以下代码的行为，分析原因，如何改正：

    ```JavaScript
    for (var i = 5; i >= 0; i--) {
        setTimeout(function () {
            console.log(i);
        }, i*1000);
    }
    ```

2. 说出以下代码的输出结果，解释原因：

    ```JavaScript
    a = 1
    if (function f() {}) {
        a += typeof f;
    }
    console.log(f);
    ```

3. 如何使用 prototype 实现继承？

4. 实现 foo()，使得每次点击 `button` 时，控制台输出 `msg` 的值:

    ```JavaScript
    var button = document.getElementById('button');
    msg = 'hello';
    button.onclick = foo(msg);
    ```

5. 解释一下 `bind` 的作用，说出一个常见的是 `bind` 使用场景。

# 答案
1. 代码会每一秒输出一个 `-1`，一共6秒。原因是在执行代码时，循环是瞬间执行完的，`i` 的值会递减到 `-1`，而 `setTimeout` 中闭包执行的时间是在循环执行完之后，而闭包里使用外部变量的值是在闭包执行是才查看这个变量的值的，所以当闭包使用 `i` 时其值已经为 `-1`。改正方法有两个：

    ```JavaScript
    var foo = function (t) {
        return function() {
            console.log(t);
        }
    }
    for (var i = 5; i >= 0; i--) {
        setTimeout(foo(i), i*1000);
    }
    ```
    或者
    ```JavaScript
    for (var i = 5; i >= 0; i--) {
        setTimeout(function (t) {
            console.log(t);
        }, i*1000, i);
    }
    ```
第二种方法明显更简单，但部分浏览器可能不支持。

2. 结果是 `1undefined`。原因是 `if(function f() {})` 这部分代码在编译时就已经执行了，编译的结果类似于 `if(true)`，在运行时 `f` 已经不可见了。

3. 代码如下：

    ```JavaScript
    var ClassA = function () {

    }
    ClassA.prototype.a = 'a';
    var ClassB = function () {}
    ClassB.prototype = new ClassA();
    var bObj = new ClassB();
    console.log(bObj.a);
    ```

4. 代码如下：

    ```JavaScript
    function foo (msg) {
        return function () {
            console.log(msg);
        }
    }
    ```

5. `bind` 有两个参数，第一个是一个 function，第二个是一个 object，`bind` 的作用是创建一个新的 function，将第一个参数中所有的 `this` 指向第二个参数，然后返回这个 function。常见的作用就是在 JavaScript 中让 `this` 的行为能够类似其他面向对象语言中 `this` 的行为。具体来说，一个常见的使用场景就是用 JavaScript 新建列表中的项目时（比如 TODO 列表），我们通常会有一个项目的模板作为每一个列表项目的 class。在生成一个新项目后，我们需要调用 `bind` 来让 class 中的 `this` 指向这个具体的列表项目。