## 数组方法

### reduce方法

> - 接受一个函数作为累加器，数组中的每个值开始缩减，最终计算为一个值。
> - 可以作为一个高阶函数，用于函数的compose
> - 对于空数组是不会执行回调函数的
>
> ```javascript
> array.reduce(function(prev,current,currentIndex,arr), initialValue)
> ```
>
> 1. prev: 函数传进来的初始值或上一次回调的返回值
> 2. current：数组中当前处理的元素值
> 3. currentIndex：当前元素索引
> 4. arr: 当前元素所属的数组本身
> 5. initialValue: 传给函数的初始值
>
> - 初始值initalValue为数值时：
>
>   ```javascript
>   const arr = [1,2,3,4,5,6,7,8,9,10]
>   const sum = arr.reduce(function(prev,current){
>       return prev + current
>   },4)
>   console.log(sum) 
>   /*
>    输出59
>    reduce根据函数传进来的初始值，不断回调叠加最终算出数组的和
>    */
>   ```
>
> - 初始值为对象时：
>
>   ```javascript
>   const arr = [1,2,3,4,5,6,7,8,9,10]
>   const sum = arr.reduce(function(prev,current){
>       prev.count = prev.count + current
>       return prev
>   },{count: 0})
>   console.log(sum)
>   //输出{count:55}
>   ```
>
> - 初始值为数组时
>
>   ```javascript
>   const str = 'hello'
>   const newstr = str.split('')
>   const result = newstr.reduce(function(prev,current){
>       const obj = {}
>       obj.current = current;
>       prev.push(obj)
>       return prev
>   },[])
>   console.log(newstr)
>   /* [ { current: 'h' },
>     { current: 'e' },
>     { current: 'l' },
>     { current: 'l' },
>     { current: 'o' } ]
>    */
>   ```
>
>   

