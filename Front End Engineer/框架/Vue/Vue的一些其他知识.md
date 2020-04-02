### 全局监听sessionStorage变化

> 在做项目的时候，可能需要在其他模块获取推送的信息或者变量，但是数据量或者数目少，而且项目中没有使用VUEX,那么可以利用sessionStorage类的浏览器存储

```javascript
// 全局获取缓存数据
Vue.prototype.resetSetItem = function (key, newVal) {
  if (key === 'text') { // 需要缓存的key

      // 创建一个StorageEvent事件
      var newStorageEvent = document.createEvent('StorageEvent');
      const storage = {
          setItem: function (k, val) {
              sessionStorage.setItem(k, val);

              // 初始化创建的事件
              newStorageEvent.initStorageEvent('setItem', false, false, k, null, val, null, null);

              // 派发对象
              window.dispatchEvent(newStorageEvent)
          }
      }
      return storage.setItem(key, newVal);
  }
}
```

