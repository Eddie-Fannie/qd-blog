## RegExp类型

### 创建正则表达式（以字面量形式）

```javascript
var regexp = / pattern / flags 
```

**flags**

- g: 表示全局模式，即模式将被应用于所有字符串，而非在发现第一个匹配项就立即停止。
- i: 不区分大小写
- m: 表示多行模式

**不同组合产生不同效果** 

```javascript
/*
 * 匹配字符串中所有"at"实例
 */
var pattern1 = /at/g
/*
 * 匹配字符串第一个'bat'或者'cat'实例,不区分大小写
 */
var pattern2 = /[bc]at/i
/*
 * 匹配字符串中所有以"at"结尾的实例
 */
var pattern3 = /\.at/gi
 
// 测试
console.log(pattern1.test('catbat')) // true
console.log(pattern2.test('cadtbdat')) // false
console.log(pattern3.test('catbat')) // false
```

**元字符需要转义** 

> 正如pattern3例子，如果想匹配字符串中包含的元字符，就必须对它们进行转义。
>
> **元字符** 
>
> ```bash
> ( { [ \ ^ $ | ? * + . ] } )
> ```
>



### 使用构造函数创建正则表达式

**接受两个参数：一个是要匹配的字符串，另一个是可选的标志字符串** 

```javascript
var pattern4 = new RegExp('/\.at/','gi')
```

**由于参数为字符串形式的，所以构造函数创建正则表达式时要记得双重转义**

### 两种方式的差别

因为在ES3中正则表达式字面量始终会共享同一个RegExp实例，而构造函数则每次都创建一个新的实例。ES5规定，使用正则表达式字面量必须像直接调用构造函数一样，每次都创建新的实例。

