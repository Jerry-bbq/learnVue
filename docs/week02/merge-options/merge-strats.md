# 合并策略

合并策略`strats`实际上是一个对象，通过给这个对象添加不同的方法来进行配置合并

``` javascript
// 合并策略，默认是是一个空对象，optionMergeStrategies: Object.create(null)
const strats = config.optionMergeStrategies
```

作用：主要是对`el、propsData，data，assets，hooks，props、methods、inject、computed，provid、wathers`的合并

在了解合并策略之前，先分析一下`src/shared/util.js`的几个简单的方法，因为接下来，会频繁的使用到这些方法：

1. extend

  ``` javascript
  export function extend (to: Object, _from: ?Object): Object {
    for (const key in _from) {
      to[key] = _from[key]
    }
    return to
  }
  ```

  作用：
    - 将源对象from，添加到目标对象to中，并返回to
    - key不同名会赋值，key同名会覆盖
2. hasOwn

  ``` javascript
  const hasOwnProperty = Object.prototype.hasOwnProperty
  export function hasOwn (obj: Object | Array<*>, key: string): boolean {
    return hasOwnProperty.call(obj, key)
  }
  ```

  作用：判断一个对象是否有该属性
3. isPlainObject

  ``` javascript
  export function isPlainObject (obj: any): boolean {
    return _toString.call(obj) === '[object Object]'
  }
  ```
  
  作用：判断一个对象是否是纯对象