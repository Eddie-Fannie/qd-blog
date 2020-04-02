参考：https://juejin.im/post/59bfe84351882531b730bac2

​	   https://zh.javascript.info/object-methods

## 一、this概念

原理：**this 永远指向最后调用它的那个对象**

注意：（非严格模式下）前面没有调用对象，那么就是全局对象window；

​            （严格模式下）全局对象就是undefined，当访问全局对象的时候就会报错。

#### 1.初识this（非严格模式下）

例1：

```javascript
var name = "windowsName";
function a() {
    var name = "Cherry";

    console.log(this.name);          // windowsName

    console.log("inner:" + this);    // inner: Window
}
a();
console.log("outer:" + this)         // outer: Window
```
前面没有调用的对象那么就是全局对象 window，这就相当于是 `window.a()`；注意，这里我们没有使用严格模式，如果使用严格模式的话，全局对象就是 `undefined`，那么就会报错 `Uncaught TypeError: Cannot read property 'name' of undefined`。

例2：

```javascript
var name = "windowsName";
var a = {
    name: "Cherry",
    fn : function () {
        console.log(this.name);      // Cherry
    }
}
a.fn();
```
函数 fn 是对象 a 调用的，所以打印的值就是 a 中的 name 的值。

例3：

```javascript
var name = "windowsName";
    var a = {
        name: "Cherry",
        fn : function () {
            console.log(this.name);      // Cherry
        }
    }
window.a.fn();
```

例4：当前对象没有相应属性的时候，不会继续向上级寻找

```javascript
var name = "windowsName";
var a = {
    // name: "Cherry",
    fn : function () {
        console.log(this.name);      // undefined
    }
}
window.a.fn();
```
调用 fn 的是 a 对象，也就是说 fn 的内部的 this 是对象 a，而对象 a 中并没有对 name 进行定义，所以 log 的 `this.name` 的值是 `undefined`。

就算 a 中没有 name 这个属性，也不会继续向上一个对象寻找 `this.name`，而是直接输出 `undefined`。

例5：

```javascript
var name = "windowsName";
var a = {
    name : null,
    // name: "Cherry",
    fn : function () {
        console.log(this.name);      // windowsName
    }
}

var f = a.fn;
f();
```
`fn()` 最后仍然是被 window 调用的。所以 this 指向的也就是 window。

总结：this 的指向并不是在创建的时候就可以确定的，在 es5 中，永远是**this 永远指向最后调用它的那个对象**。

例6：

```javascript
var name = "windowsName";

function fn() {
    var name = 'Cherry';
    innerFunction();
    function innerFunction() {
        console.log(this.name);      // windowsName
    }
}

fn()
```
#### 2."复杂"的方法调用（严格模式下）

```javascript
let user = {
  name: "John",
  hi() { alert(this.name); },
  bye() { alert("Bye"); }
};

user.hi(); // John (the simple call works)

// 现在我们要判断 name 属性，来决定调用 user.hi 或是 user.bye。
(user.name == "John" ? user.hi : user.bye)(); // Error!
```

**深入理解 `obj.method()` 调用的原理:**

仔细看，我们可能注意到 `obj.method()` 语句中有两个操作符。

1. 首先，点 `'.'` 取得这个 `obj.method` 属性。
2. 其后的括号 `()` 调用它。

那么，`this` 是如何从第一部分传递到第二部分的呢？

如果把这些操作分离开，那么 `this` 肯定会丢失：

```javascript
let user = {
  name: "John",
  hi() { alert(this.name); }
}

// 将赋值与方法调用拆分为两行
let hi = user.hi;
hi(); // 错误，因为 this 未定义
```

这里 `hi = user.hi` 把函数赋值给变量，其后的最后一行是完全独立的，所以它没有 `this`。

**为了让 user.hi() 有效，JavaScript 用一个技巧 —— 这个 '.' 点返回的不是一个函数，而是一种特殊的引用类型的值。**(????特殊的引用类型？？？)

`()` 被引用类型调用时，将接收关于该对象及其方法的所有信息，并且设定正确的 `this` 值（这里等于 `user`）。

`hi = user.hi` 赋值等其他的操作，将引用类型作为一个整体丢弃，只获取 `user.hi`（一个函数）的值进行传递。因此，进一步操作『失去』了 `this`（值）。

#### 3.箭头函数没有自己的“this”

它们没有自己的 `this`，`，`this` 值取决于外部『正常的』函数。

```javascript
let user = {
  firstName: "Ilya",
  sayHi() {
    let arrow = () => alert(this.firstName);
    arrow();
  }
};

user.sayHi(); // Ilya
```

## 二、怎样改变this的指向

改变 this 的指向我总结有以下几种方法：

- 使用 ES6 的箭头函数
- 在函数内部使用 `_this = this`
- 使用 `apply`、`call`、`bind`
- new 实例化一个对象

```javascript
var name = "windowsName";

var a = {
    name : "Cherry",

    func1: function () {
        console.log(this.name)     
    },

    func2: function () {
        setTimeout(  function () {
            this.func1()
        },100);
    }

};

a.func2()     // this.func1 is not a function
```
在不使用箭头函数的情况下，是会报错的，因为最后调用 `setTimeout` 的对象是 window，但是在 window 中并没有 func1 函数。

#### 1.箭头函数

**箭头函数的 this 始终指向函数定义时的 this，而非执行时。**

箭头函数需要记着这句话：“箭头函数中没有 this 绑定，必须通过查找作用域链来决定其值，如果箭头函数被非箭头函数包含，则 this 绑定的是最近一层非箭头函数的 this，否则，this 为 undefined”。

```javascript
var name = "windowsName";

var a = {
    name : "Cherry",

    func1: function () {
        console.log(this.name)     
    },

    func2: function () {
        setTimeout( () => {
            this.func1()
        },100);
    }

};

a.func2()     // Cherry
```
#### 2.在函数内部使用 `_this = this`

如果不使用 ES6，那么这种方式应该是最简单的不会出错的方式了，我们是先将调用这个函数的对象保存在变量 `_this` 中，然后在函数中都使用这个 `_this`，这样 `_this` 就不会改变了。

```javascript
var name = "windowsName";

var a = {

    name : "Cherry",

    func1: function () {
        console.log(this.name)     
    },

    func2: function () {
        var _this = this;
        setTimeout( function() {
            _this.func1()
        },100);
    }

};

a.func2()       // Cherry
```
在 func2 中，首先设置 `var _this = this;`，这里的 `this` 是调用 `func2` 的对象 a，为了防止在 `func2` 中的 setTimeout 被 window 调用而导致的在 setTimeout 中的 this 为 window。我们将 `this(指向变量 a)` 赋值给一个变量 `_this`，这样，在 `func2` 中我们使用 `_this` 就是指向对象 a 了。

#### 3.使用 apply、call、bind

1. 使用 apply

```javascript
var a = {
    name : "Cherry",

    func1: function () {
        console.log(this.name)
    },

    func2: function () {
        setTimeout(  function () {
            this.func1()
        }.apply(a),100);
    }

};

a.func2()            // Cherry
```
  2.使用 call

```javascript
var a = {
    name : "Cherry",

    func1: function () {
        console.log(this.name)
    },

    func2: function () {
        setTimeout(  function () {
            this.func1()
        }.call(a),100);
    }

};

a.func2()            // Cherry
```
   3.使用 bind

```javascript
var a = {
    name : "Cherry",

    func1: function () {
        console.log(this.name)
    },

    func2: function () {
        setTimeout(  function () {
            this.func1()
        }.bind(a)(),100);
    }

};

a.func2()            // Cherry
```
##### 三者概念及其区别

  **1.apply概念**:call 方法接受的是若干个参数列表

> 概念：apply() 方法调用一个函数, 其具有一个指定的this值，以及作为一个数组（或类似数组的对象）提供的参数

> 语法：fun.apply(thisArg, [argsArray])

- thisArg：在 fun 函数运行时指定的 this 值。需要注意的是，指定的 this 值并不一定是该函数执行时真正的 this 值，如果这个函数处于非严格模式下，则指定为 null 或 undefined 时会自动指向全局对象（浏览器中就是window对象），同时值为原始值（数字，字符串，布尔值）的 this 会指向该原始值的自动包装对象。
- argsArray：一个数组或者类数组对象，其中的数组元素将作为单独的参数传给 fun 函数。如果该参数的值为null 或 undefined，则表示不需要传入任何参数。从ECMAScript 5 开始可以使用类数组对象。浏览器兼容性请参阅本文底部内容。

```javascript
var a ={
    name : "Cherry",
    fn : function (a,b) {
        console.log( a + b)
    }
}

var b = a.fn;
b.apply(a,[1,2])     // 3
```
 **2.call概念**:apply 接收的是一个包含多个参数的数组

> 语法：fun.call(thisArg[, arg1[, arg2[, ...]]])

```javascript
var a ={
    name : "Cherry",
    fn : function (a,b) {
        console.log( a + b)
    }
}

var b = a.fn;
b.call(a,1,2)       // 3
```
**3.bind概念**:

> 概念：bind()方法创建一个新的函数, 当被调用时，将其this关键字设置为提供的值，在调用新函数时，在任何提供之前提供一个给定的参数序列。

```javascript
var a ={
    name : "Cherry",
    fn : function (a,b) {
        console.log( a + b)
    }
}

var b = a.fn;
b.bind(a,1,2)()           // 3
```
bind 是创建一个新的函数，我们必须要手动去调用。

## 补充：JS中的函数调用

函数调用的方法一共有 4 种

1. 作为一个函数调用
2. 函数作为方法调用
3. 使用构造函数调用函数
4. 作为函数方法调用函数（call、apply）

#### 1.作为一个函数调用

```javascript
var name = "windowsName";
function a() {
    var name = "Cherry";

    console.log(this.name);          // windowsName

    console.log("inner:" + this);    // inner: Window
}
a();
console.log("outer:" + this)         // outer: Window
```
这样一个最简单的函数，不属于任何一个对象，就是一个函数，这样的情况在 JavaScript 的在浏览器中的非严格模式默认是属于全局对象 window 的，在严格模式，就是 undefined。 

但这是一个全局的函数，很容易产生命名冲突，所以不建议这样使用。

#### 2.函数作为方法调用

更多的情况是将函数作为对象的方法使用。

```javascript
var name = "windowsName";
var a = {
    name: "Cherry",
    fn : function () {
        console.log(this.name);      // Cherry
    }
}
a.fn();
```
这里定义一个对象 `a`，对象 `a` 有一个属性（`name`）和一个方法（`fn`）。

然后对象 `a` 通过 `.` 方法调用了其中的 fn 方法。

然后我们一直记住的那句话“**this 永远指向最后调用它的那个对象**”，所以在 fn 中的 this 就是指向 a 的。

#### 3.使用构造函数调用函数

>如果函数调用前使用了 new 关键字, 则是调用了构造函数。
>这看起来就像创建了新的函数，但实际上 JavaScript 函数是重新创建的对象：

```javascript
// 构造函数:
function myFunction(arg1, arg2) {
    this.firstName = arg1;
    this.lastName  = arg2;
}

// This    creates a new object
var a = new myFunction("Li","Cherry");
a.lastName;                             // 返回 "Cherry"
```

#### 4.作为函数方法调用函数

>在 JavaScript 中, 函数是对象。
>
>JavaScript 函数有它的属性和方法。
>call() 和 apply() 是预定义的函数方法。 两个方法可用于调用函数，两个方法的第一个参数必须是对象本身
>
>在 JavaScript 严格模式(strict mode)下, 在调用函数时第一个参数会成为 this 的值， 即使该参数不是一个对象。
>在 JavaScript 非严格模式(non-strict mode)下, 如果第一个参数的值是 null 或 undefined, 它将使用全局对象替代。

```javascript
var name = "windowsName";

function fn() {
    var name = 'Cherry';
    innerFunction();
    function innerFunction() {
        console.log(this.name);      // windowsName
    }
}

fn()
```
这里的 innerFunction() 的调用是不是属于第一种调用方式：作为一个函数调用（它就是作为一个函数调用的，没有挂载在任何对象上，所以对于没有挂载在任何对象上的函数，在非严格模式下 this 就是指向 window 的）

```javascript
var name = "windowsName";

var a = {
    name : "Cherry",

    func1: function () {
        console.log(this.name)     
    },

    func2: function () {
        setTimeout(  function () {
            this.func1()
        },100 );
    }

};

a.func2()     // this.func1 is not a function
```
这个简单一点的理解可以理解为“**匿名函数的 this 永远指向 window**”，你可以这样想，还是那句话**this 永远指向最后调用它的那个对象**，那么我们就来找最后调用匿名函数的对象，这就很尴尬了，因为匿名函数名字啊，笑哭，所以我们是没有办法被其他对象调用匿名函数的。所以说 匿名函数的 this 永远指向 window。

如果这个时候你要问，那匿名函数都是怎么定义的，首先，我们通常写的匿名函数都是自执行的，就是在匿名函数后面加 `()` 让其自执行。其次就是虽然匿名函数不能被其他对象调用，但是可以被其他函数调用啊，比如例 7 中的 setTimeout。