### 关于this对象

在《JavaScript高级程序设计》一书中说明了`this`对象是运行时基于函数的执行环境绑定的：**在全局函数中，`this`等于`window`，而当函数被作为某个对象的方法调用时，`this`等于那个对象**。

不过，匿名函数的执行环境具有全局性，因此其`this`对象通常指向`window`。

```javascript
    var name = 'lin jia heng'
    var object = {
        name: 'My Object',
        getNameFunc: function () {
            return function () {
                return this.name; //匿名函数的执行环境具有全局性，所以this指向window
            }
        }
    }
    alert(object.getNameFunc()()) // lin jia heng(非严格模式）
```



而下面的例子就可以成功返回“My Object”

```javascript
    var name = 'lin jia heng'
    var object = {
        name: 'My Object',
        getNameFunc: function () {
            return this.name; //当函数被作为某个对象的方法调用时，this等于那个对象
        }
    }
    alert(object.getNameFunc()) // My Object
    var name = 'lin jia heng'
    var object = {
        name: 'My Object',
        getNameFunc: function () {
            let that = this // 改变this指向
            return function () {
                return that.name; //匿名函数的执行环境具有全局性，所以this指向window
            }
        }
    }
    alert(object.getNameFunc()()) // My Object
```



### 在函数中执行

1. 在非严格模式（默认绑定）

```javascript
function func(){
    console.log(this)
}
func() //Window
```

2. 在严格模式（默认绑定）

```javascript
function func(){
    console.log(this)
}
func() //undefined
```

这就验证了第一个例子中为什么特意表明在非严格模式下才成立了。



### 作为一个构造函数使用

在JS中，为了实现类我们需要定义一些构造函数，在调用一个构造函数的时候加上`new`这个关键字：

```javascript
function Person(name) {
    this.name = name;
    console.log(this)
}
var p1 = new Person('kk');
// Person
```

此时`this`指向这个构造函数调用的时候实例化出来的对象

当然，构造函数其实也是一个函数，如果将构造函数当成普通函数来调用，`this`指向`Window`

```javascript
function Person(name) {
    this.name = name;
    console.log(this)
}
var p1 = Person('kk');
// Window
```



### 在定时器中使用

```javascript
setTimeout(function() {
    console.log(this);
},0)// Window，setInterval也一样
```

如果没有特殊指向，定时器的回调函数中`this`的指向都是`Window`。这是因为JS的定时器方法是定义在`Window`下的。



### 在箭头函数中使用

1. 在全局环境中使用：

```javascript
var func = () => {
    console.log(this)
}
func() //Window
```

2. 作为一个对象的一个函数使用：（隐式绑定）

```javascript
var obj = {
    name: 'hh',
    func: function() {
        console.log(this);
    }
}
obj.func();//obj
var obj = {
    name: 'hh',
    func: () => {
        console.log(this)
    }
}
obj.func(); //Window
```

3. 作为对象的特殊情况，结合定时器来使用：

```javascript
var obj = {
    name: 'hh',
    func: function() {
        setTimeout(function() {
            console.log(this);
        },0)
    }
}
obj.func();// Window
var obj = {
    name: 'hh',
    func: function() {
        setTimeout(() => {
            console.log(this);
        },0)
    }
}
obj.func();// obj
```

> 箭头函数中的this的值取决于该函数外部非箭头函数的this的值，否则this的值会被设置为全局对象Window,且不能通过call(),apply()和bind()方法来改变this的值。--《深入理解ES6》



### 赋值给另外一个变量进行调用

```javascript
var name = "windowsName";
var a = {
    name: null,
    fn: function() {
        console.log(this.name); //windowsName
    }
}
var f = a.fn;
f();
```

> fn()最后仍然是被window调用的，所以this指向的也是window。this的指向并不是在创建的时候就确定并一成不变的，在es5中this永远指向最后调用它的那个对象。

```javascript
'use strict' //严格模式下
let user = {
    name: 'John',
    hi(){alert(this.name);},
    bye() {alert('Bye');}
};
user.hi();//John
(user.name == 'John' ? user.hi : user.bye)();// this指向undefined,所以此处报错，此处的函数user.hi没有加上括号进行立即执行，有点类似于上个例子的赋值变量进行调用
```



### 改变this的指向

- 像上述部分例子一样使用箭头函数
- 在函数内部定义一个变量`_this = this`
- 使用`apply`，`call`，`bind`（显示绑定）
- `new`实例化一个对象（new绑定）

#### 使用_this = this

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

> 在 func2 中，首先设置` var _this = this`;，这里的 this 是调用 func2 的对象 a，为了防止在 func2 中的 `setTimeout `被 `window` 调用而导致的在 `setTimeout` 中的 this 为 `window`。我们将 `this`(指向变量 a) 赋值给一个变量` _this`，这样，在 func2 中我们使用 `_this` 就是指向对象 `a `了。



#### 使用apply，call，bind（显示绑定）

##### bind

> 如果你想将某个函数绑定新的this指向并且固定传入几个变量可以在绑定的时候就传入，之后调用新函数传入的参数都会排在后面

```javascript
const obj = {}
function test(...args){
    console.log(this === obj)//true, 因为bind方法把this指向了obj对象
    console.log(args)
}
const newFn = test.bind(obj, 'lin', 'jiaheng')//因为bind返回一个新函数，所以要赋值给一个新变量
newFn('jiahenglin')
var a ={
    name : "Cherry",
    fn : function (a,b) {
        console.log( a + b)
    }
}

var b = a.fn;
b.bind(a,1,2)()           // 3
```

**因为bind返回一个新函数，所以要加多一个括号手动调用它**。

##### call

> 需要注意的是，指定的this并不一定是该函数执行时真正的this，如果这个函数处于非严格模式下，则指定为`null`和`undefined`的this值会自动指向全局对象（window），同时值为原始值（数学，字符串，布尔值）的this会指向该原始值的自动包装对象【理解为 如果你传入一个原始值（字符串，布尔类型或者数字类型）来当作this的绑定对象，这个原始值就会被转换成它的对象形式（也就是`new String(...)`,`new Boolean(...)`或者`new Number(...)`,这通常称为装箱】。

```javascript
var a = {
    name : "Cherry",
    func1: function () {
        console.log(this.name)
    },
    func2: function () {
        setTimeout(  function () {
            this.func1() // 没用利用call改变this的指向的话，这里的this就是Window
        }.call(a),100);
    }
};
a.func2()            // Cherry
```

##### apply

> 第一个参数和call的一样，第二个参数一定是传入一个数组格式的,最终调用函数时候这个数组会拆成一个一个参数传入

**因为apply函数传参的特性，所以可以实现一个全是数字的数组找出最大最小值的效果**

```javascript
const arr = [1,2,3,4,45,43,22]
const max = Math.max.apply(null,arr)
console.log(max)//45
// 也可以使用ES6的拓展运算符
console.log(Math.max(...arr))
```



#### bind,call,apply进阶例子

1. 循环中利用闭包来处理回调

```javascript
for(var i=0;i<10;i++){
    (function(j){
        setTimeout(function(){
            console.log(j)
        },600)
    })(i)
}
```

> 每次循环都会产生一个立即执行的函数，函数内部的局部变量j保存不同时期i的值，循环过程中， setTimeout回调按顺序放入事件队列中，等for循环结束后，堆栈中没用同步的代码，就去事件队列中，执行对应的回调，打印出j的值

同理可以利用bind，每次都创建新的函数，并且已经预先设置了参数

```javascript
function func(i){
    console.log(i)
}
for(var i=0;i<10;i++){
    setTimeout(func.bind(null,i),600)
}
```

2. 实现继承

```javascript
var Person = function(name,age){
    this.name = name;
    this.age = age;
}
var P1 = function(name,age) {
    //借用构造函数方式实现继承
    //利用call继承了Person
    Person.call(this,name,age)
}
P1.prototype.getName = function(){
    console.log('name:'+this.name+',age:'+this.age);
}
var newPerson = new P1('popo',20);//name:popo,age:20
newPerson.getName();
```

3. 实现硬绑定

```javascript
function foo() {
   console.log(this.a);
}
var obj = {
    a:2
};
var bar = function() {
    foo.call(obj);
};
bar(); // 2
setTimeout(bar, 100); // 2
// 硬绑定的bar不可能再修改它的this
bar.call(window); // 2
```

> 我们创建了函数bar()，并在它的内部手动调用了foo.call(obj)，因此强制把foo的this绑定到了obj。无论之后如何调用函数bar，它总会手动在obj上调用foo。这种绑定是一种显式的强制绑定，因此我们称之为硬绑定。



### 总结

> 1. 对于没有挂载在任何对象上的函数，在非严格模式下 this 就是指向 window 的。
>
> 2. 匿名函数的this永远指向window 。
>
> 3. 不要根据this的英文语法角度错误理解成指向函数自身。
>
> 4. 当一个函数被调用时，会创建一个活动记录（也可以成为执行上下文），this就是这记录的一个属性。，所以this指向什么完全取决于函数在哪里被调用。



### 补充

> 根据《你不知道的JavaScript上卷》一书中写的：可以根据优先级来判断函数在某个调用位置应用的是哪条this指向规则（在箭头函数下无效）

1. 函数是否在new中调用，如果是的话this绑定的是新创建的对象。
2. 函数是否通过显示绑定或者硬绑定调用，如果是的话，this指向指定对象
3. 函数是否在某个上下文中调用（隐式绑定），如果是的话，this指向那个上下文对象
4. 如果都不是，使用默认绑定，严格模式下就指向undefined,否则绑定到全局对象。