# wathers合并策略

`strats`对象上添加`watch`方法

``` javascript
strats.watch = function (
  parentVal: ?Object,
  childVal: ?Object,
  vm?: Component,
  key: string
): ?Object {
  /**
   * // Firefox has a "watch" function on Object.prototype...
    const nativeWatch = ({}).watch
   */
  // 兼容火狐原生的watch
  // work around Firefox's Object.prototype.watch...
  if (parentVal === nativeWatch) parentVal = undefined
  if (childVal === nativeWatch) childVal = undefined
  // 如果childVal不存在，返回 以 parentVal 为原型创建一个对象
  /* istanbul ignore if */
  if (!childVal) return Object.create(parentVal || null)
  if (process.env.NODE_ENV !== 'production') {
    // 非生产环境，检测子对象是不是纯对象
    assertObjectType(key, childVal, vm)
  }
  // 如果 parentVal不存在，返回 childVal
  if (!parentVal) return childVal
  const ret = {} // 创建空对象
  extend(ret, parentVal)  // 将parentVal合并到空对象ret上(key同名会覆盖)
  // 遍历childVal对象
  for (const key in childVal) {
    // 获取合并后的ret对象的key对应的value，赋值给parent（parent可能是undefined）
    let parent = ret[key]
    // 获取childVal对应的key的value
    const child = childVal[key]
    // 如果parent存在，且parent不是一个数组，将parent处理为一个数组
    if (parent && !Array.isArray(parent)) {
      parent = [parent]
    }
    // 如果parent存在（此时数数组），拼接上child
    // 如果parent不存在，并且child是一个数组，返回child
    // 否则吧child处理为一个数组返回
    ret[key] = parent
      ? parent.concat(child)
      : Array.isArray(child) ? child : [child]
  }
  // 返回ret对象
  return ret
}
```

分析：

- 如果`childVal`不存在，返回 以`parentVal`为原型创建一个对象
- 如果`parentVal`不存在，返回`childVal`
- 如果`parentVal`和`childVal`存在，合并`childVal`和`parentVal`，返回合并后的`ret`对象