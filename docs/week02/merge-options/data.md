# data的合并策略

`strats`对象上添加`data`方法，接收三个参数`parentVal`、`childVal`、`vm`（可不传）,返回一个对象

``` javascript
strats.data = function (
  parentVal: any,
  childVal: any,
  vm?: Component
): ?Function {
  if (!vm) {
    if (childVal && typeof childVal !== 'function') {
      process.env.NODE_ENV !== 'production' && warn(
        'The "data" option should be a function ' +
        'that returns a per-instance value in component ' +
        'definitions.',
        vm
      )
      return parentVal
    }
    return mergeDataOrFn(parentVal, childVal)
  }

  return mergeDataOrFn(parentVal, childVal, vm)
}
```

代码分析：

1. 接收三个参数：`parentVal`、`childVal`、`vm`（可不传）
2. 如果第三个参数`vm`存在，返回`mergeDataOrFn(parentVal, childVal, vm)`；

    否则，进入下一个判断，如果`childVal`存在并且`childVal`不是一个函数，返回 `parentVal`；

    否则，返回`mergeDataOrFn(parentVal, childVal)`

接下来，分析`mergeDataOrFn`方法

```javascript
export function mergeDataOrFn (
  parentVal: any,
  childVal: any,
  vm?: Component
): ?Function {
  if (!vm) {
    // 如果没有传入vm
    // in a Vue.extend merge, both should be functions，在 Vue.extend 的合并中，两个都应该是函数
    // 子对象不存在返回父对象，父对象不存在返回子对象
    if (!childVal) {
      return parentVal
    }
    if (!parentVal) {
      return childVal
    }
    // 如果childVal，parentVal都存在，返回一个函数 mergedDataFn
    return function mergedDataFn () {
      // 返回 合并函数
      return mergeData(
        typeof childVal === 'function' ? childVal.call(this, this) : childVal,
        typeof parentVal === 'function' ? parentVal.call(this, this) : parentVal
      )
    }
  } else {
    // 如果传入vm，返回函数 mergedInstanceDataFn
    return function mergedInstanceDataFn () {
      // 实例的合并
      // 拿到 childVal 的值，如果是函数，执行它获取返回值，如果不是，直接返回 childVal
      const instanceData = typeof childVal === 'function'
        ? childVal.call(vm, vm)
        : childVal
      // 同理拿到 parentVal 的值
      const defaultData = typeof parentVal === 'function'
        ? parentVal.call(vm, vm)
        : parentVal
      // 如果 childVal 存在，则返回合并函数，否则直接返回 parentVal
      if (instanceData) {
        return mergeData(instanceData, defaultData)
      } else {
        return defaultData
      }
    }
  }
}
```

分析：

1. 如果没有传入`vm`，对`parentVal`,`childVal`两个参数的存在与否进行判断：

    如果`childVal`不存在返回`parentVal`；

    如果`parentVal`不存在返回`childVal`；

    如果两个值都存在返回合并函数 `mergeData`

2. 如果传入`vm`，对`parentVal`,`childVal`两个参数的是不是函数类型进行判断：  

    如果`childVal`是函数，直接执行该函数拿到返回值，否则，返回`childVal`并存到变量 `instanceData`；

    `parentVal`同理，存到变量 `defaultData`中；

    如果 `instanceData` 存在，直接返回合并函数 `mergeData`并传入参数`instanceData` 和`defaultData`；

    否则，直接返回`defaultData`

在上面多次提到函数`mergeData`，那么这个函数的又是做什么的呢：

``` javascript
// Helper that recursively merges two data objects together.
function mergeData (to: Object, from: ?Object): Object {
  // 如果源对象不存，直接返回目标对象
  if (!from) return to
  // 定义三个变量
  let key, toVal, fromVal
  // 获取源对象from的所有key组成的数组keys
  const keys = Object.keys(from)
  // 遍历源对象from的所有key组成的数组
  for (let i = 0; i < keys.length; i++) {
    // 将源对象from的key赋值给变量 key
    key = keys[i]
    // 将源对象from的key作为目标对象to的key获取to对应的value，赋值给变量 toVal
    toVal = to[key]
    // 将源对象key对象的value赋值给变量 fromVal
    fromVal = from[key]

    if (!hasOwn(to, key)) {
      // 如果目标对象to中，没有源对象from的key属性，
      // 则，将源对象from上的key和对应的value，添加到目标对象to上
      set(to, key, fromVal)
    } else if (toVal !== fromVal && isPlainObject(toVal) && isPlainObject(fromVal) ) {
      // 如果to[key]，from[key]不相等，并且to[key]，from[key]都是对象，
      // 则，递归，合并to[key]，from[key]对象
      mergeData(toVal, fromVal)
    }
  }
  // 返回合并后的目标对象 to
  return to
}
```

经过分析可以看出，

作用：合并两个对象，将源对象from合并到目标对象to,并返回目标对象to

过程：对比两个对象，将两个对象不同的key及对应的value合并到目标对象to上，有相同的key的情况，使用目标对象to的key及对应的value