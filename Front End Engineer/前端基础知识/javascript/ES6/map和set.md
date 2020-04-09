## Map概念

ES6提供了Map数据结构，类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的🈯️（包含对象）都可以当作键。是一种更完善的Hash结构实现。**Object数据结构的键仅局限于字符串类型**。



## Map的方法

- **Map结构的set方法，将对象o当作m的一个键，然后又使用get方法读取这个值** 

```javascript
const m = new Map();
const o = {p: 'Hello world'};
m.set(o,'content')
m.get(o) //'content'
m.has(o) //true
m.delete(o)//true
m.has(o)//false
```

- **Map作为构造函数，Map可以接受一个数组作为参数，该数组的成员是一个个表示键值对的数组。**  

  ```javascript
  const map = new Map([                         
    ['name','张三'],
    ['title','Author']
  ])       
  map.size//2
  map.has('name')//true
  map.get('name')//'张三'
  // 上面代码新建Map实例时，就指定了两个键name和title
  ```

  

  **上述构造函数实例中执行的是以下算法** 

  ```javascript
  const items = [
    ['name', '张三'],
    ['title', 'Author']
  ];
  const map = new Map();
  items.forEach(
  	([key,value]) => map.set(key, value)
  )
  ```

  

  

  