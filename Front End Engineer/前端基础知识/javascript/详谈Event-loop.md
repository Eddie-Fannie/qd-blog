### 引子

JS是单线程的，JS是通过事件队列（Event Loop）的方式来实现异步回调的。单线程的JS为什么拥有异步的能力，接下是从进程，线程的角度来解释这个问题。

### 进程

计算机的核心CPU就好像一个工厂时刻运行中，工厂的电力有限一次只能供给一个车间使用，也就是说单个CPU一次只能运行一个任务。

进程就好比工厂的车间，进程之间相互独立，任一时刻CPU总是运行一个进程，其他进程处于非运行状态。CPU使用时间片轮转进度算法来运行多个进程。

### 线程

一个车间里，可以有很多工人，共享车间所有的资源，共同协同完成一个任务。线程就好比车间里的工人，一个进程可以包括多个线程，多个线程共享进程资源。

### CPU，进程，线程三者关系

- 进程是CPU资源分配的最小单位（是能拥有资源和独立运行的最小单位）
- 线程是CPU调度的最小单位（线程是建立在进程的基础上的一次程序运行单位，一个进程中可以有多个线程）
- 不同进程之间也可以通信，不过代价比较大
- 单线程与多线程都是指在一个进程内的单和多

### 浏览器是多进程的

对于计算机来说，每一个应用程序都是一个进程，而每一个应用程序都会分别有很多的功能模块，这些功能模块实际上是通过子进程来实现的。对于这种子进程的扩展方式，我们可以称这个应用程序是多进程的。

对于浏览器来说，浏览器是多进程的，每一个tab页就是一个独立的进程。

### 浏览器包含了哪些进程

- 主进程
  - 协调控制其他子进程（创建，销毁）
  - 浏览器界面显示，用户交互，前进，后退，收藏
  - 将渲染进程得到的内存中的Bitmap，绘制到用户界面上
  - 处理不可见操作，网络请求，文件访问等
- 第三方插件进程
  - 每一种类型的插件对应一个进程，仅当使用该插件时才会创建
- GPU进程
  - 用于3D绘制等
- 渲染进程，就是我们说的浏览器内核（前端操作最重要的进程）
  - 负责页面渲染，脚本执行，事件处理等
  - 每个tab页就是一个渲染进程

### 浏览器内核（Render进程）

该进程也同样是多线程的，包含了以下线程

- GUI渲染线程
  - 负责渲染页面，布局和控制
  - 页面需要重绘和回流时，该线程就会执行
  - 与js引擎线程互斥，防止渲染结果不可预期
- JS引擎线程
  - 负责处理解析和执行js脚本程序
  - 只有一个JS引擎线程（单线程）
  - 与GUI渲染线程互斥，防止渲染结果不可预期
- 事件触发线程
  - 用来控制事件循环（鼠标点击，setTimeout，Ajax等）
  - 当事件满足触发条件时，将事件放入到JS引擎所在的执行队列中
- 定时触发线程
  - setInterval和setTimeout所在的线程
  - 定时任务并不是由JS引擎计时的，是由定时触发线程计时的
  - 计时完毕，通知事件触发线程
- 异步http请求线程
  - 浏览器有一个单独的线程用于处理AJAX请求
  - 当请求完成时，若有回调函数，通知事件触发线程

### 为什么GUI渲染线程和JS引擎线程互斥

这是由于JS是可以操作DOM的，如果同时修改元素属性并同时渲染界面，那么渲染线程前后获得的元素就可能不一致了。

当JS引擎线程执行时GUI渲染线程就会被挂起，GUI更新则会被保存在一个队列中等待JS引擎线程空闲时立即被执行。

### 从Event Loop看JS的运行机制

- JS分为同步和异步任务
- 同步任务都在JS引擎线程上执行，形成一个执行栈
- 事件触发线程管理一个任务队列，异步任务触发条件达成，将回调事件放到任务队列中
- 执行栈中所有同步任务执行完毕，此时JS引擎线程空闲，系统会读取任务队列，将可运行的异步任务回调事件添加到执行栈中，开始执行

我们知道不管是定时器还是网络请求代码，在这些代码执行时，本身是同步任务，而其中的回调函数才是异步任务。

当代码执行到setTimeout/setInterval时，实际上是JS引擎线程通知定时触发线程，间隔一个时间后，会触发一个回调事件，而定时触发线程在接收到这个消息后，会在等待的时间后，将回调事件放入到由事件触发线程所管理的事件队列中。

而当代码执行XHR/fetch网络请求时候，则是JS线程通知异步http请求线程

用代码来说话：

```javascript
let timerCallback = function(){
console.log('wait one second')
}
let httpCallback = function(){
console.log('get server data success')
}
//同步任务
console.log('hello')
//同步任务
//通知定时器线程JS后将timerCallback交由事件触发线程处理
//1s后事件触发线程将该事件加入到事件队列中
setTimeout(timerCallback,1000);
//同理。。。
$.get('www.xxx.com',httpCallback);
//同步任务
console.log('world')
```

总结：

- JS引擎线程只执行执行栈中的事件
- 执行栈中的代码执行完毕，就会读取事件队列中的事件
- 事件队列中的回调事件，是由各自线程插入到事件队列中的
- 如此循环

### 宏任务，微任务（异步任务,宏任务可以有多个，微任务队列只有一个）

#### 什么是宏任务

我们可以将每次执行栈执行的代码当作一个宏任务（包括每次从事件队列中获取一个事件回调并放到执行栈中执行),每个宏任务会从头到尾执行完毕，不会执行其他。 浏览器为了能够使宏任务和DOM任务有序进行，会在一个宏任务执行结果后，在下一个宏任务执行前，GUI渲染线程开始工作，对页面进行渲染。

> 主代码块，setTimeout,setInterval等，都属于宏任务

**第一个例子**

```
document.body.style = "background:black";
document.body.style = "background:red";
document.body.style = "background:blue";
document.body.style = "background:grey"
```

我们可以将这段代码放到浏览器的控制台执行一下，可以看到效果: 

![alt Event-loop](https://mmbiz.qpic.cn/mmbiz_gif/09OicMgzsTbGFfyEPsNELqnXTCW86lhWcfuYlmHFaWoeF3d8ibYJUNQsJjGDTwv5T1QDf3rIlVARh5DvNHQnicrsQ/0?wx_fmt=gif)

我们会看到页面背景在瞬间变成灰色，以上代码属于一次宏任务，所以全部执行完才会触发页面渲染，渲染时GUI线程会将所有的UI改动优化合并，所以视觉效果上，只会看到页面变成灰色。

**第二个例子**

```javascript
document.body.style="background:blue"
setTimeout(function(){
document.body.style ="background:black"
},0)
```

[![img](https://mmbiz.qpic.cn/mmbiz_gif/09OicMgzsTbGFfyEPsNELqnXTCW86lhWcHU5EWQwue5nkoib0oBNSGTCdg8yibV4Z0wsOib9libnnNopYCEUkbypgRQ/0?wx_fmt=gif)](https://github.com/Eddie-Fannie/Eddie-Fannie.github.io/blob/hexo/images/3.gif) 

我们可以看到页面先变成蓝色，再瞬间变成黑色，这是因为上面代码为两次宏任务，分别执行一次然后再触发渲染，所以两种颜色都会被渲染出来。

#### 什么是微任务

我们知道宏任务结束后会执行渲染，然后执行下一个宏任务，而微任务可以理解为在当前宏任务执行后立即执行的任务。

> Promise,process,nextTick，then()等属于微任务,在微任务中process.nextTick优先级高于Promise

**第一个例子**

```javascript
document.body.style="background:blue"
console.log(1)
Promise.resolve().then(()=>{
    console.log(2)
    document.body.style = "background:black"
});
console.log(3)
```

[![img](https://mmbiz.qpic.cn/mmbiz_gif/09OicMgzsTbGFfyEPsNELqnXTCW86lhWcm8EvUKqQX3ria6IaBqiaARgiaND5Oibp5vTE0r4tPjomfZVTfCwoKQnvjA/0?wx_fmt=gif)](https://github.com/Eddie-Fannie/Eddie-Fannie.github.io/blob/hexo/images/4.gif) 

页面的背景直接变成黑色，没有经过蓝色的阶段，是因为，我们在宏任务中将背景设置为蓝色，但在进行渲染前执行了微任务，在微任务中将背景变成黑色，然后才执行的渲染

**第二个例子**

```javascript
setTimeout(()=>{
    console.log(1)
    Promise.resolve(3).then(data => console.log(data))
},0)
setTimeout(() => {
    console.log(2)
},0)
```

上面代码共有两个setTimeout，也就是说除主代码外，共有两个宏任务，其中第一个宏任务执行中，输出1，并且创建微任务队列，所以在下一个宏任务队列执行前，先执行微任务，在微任务执行中输出3，微任务执行后，执行下次宏任务，执行中输出2。

**当异步任务进入栈执行时，微任务和宏任务并排进入执行队列时，先执行微任务**

```javascript
setTimeout(function(){
       console.log(1)
       Promise.resolve().then(function () {
           console.log(2)
       })
   },0)
    setTimeout(function () {
        console.log(3)
    },0)
    Promise.resolve().then(function () {
        console.log(4)
    })
    console.log(5)//5，4，1，2，3
```

- 第一轮循环

  - 同样从全局任务入口，遇到宏任务setTimeout，交给异步处理模块，我们暂且记为setTimeout 1，由于等待时间为0，直接加入宏任务队列。
  - 再次遇到宏任务setTimeout，交给异步处理模块，我们暂且记为setTimeout2,同样直接加入宏任务队列
  - 遇到微任务then()，加入微任务队列。
  - 直接打印日志5，所以先输出5

- 第二轮循环

  - 栈空后，先执行微任务队列，输出4
  - 读取宏任务队列最靠前的任务setTimeout1
  - 先直接执行打印语句，打印日志1，又遇到微任务then(),加入微任务队列，第二轮循环结束

- 第三轮循环

  - 先执行微任务队列中的then(),输出2

  - 执行setTimeout2，输出3，执行完毕

    ```bash
    git clone https://github.com/mrdoob/three.js.git
    cd ./three.js
    npm install
    npm run build
    该文件就在three.js/build/three.min.js下
    ```

    

