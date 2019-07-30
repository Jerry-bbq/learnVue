# props，methods，inject，computed的合并策略

`strats`对象上添加`props，methods，inject，computed`四个相同的方法

``` javascript
// Other object hashes.
strats.props =
strats.methods =
strats.inject =
strats.computed = function (
  parentVal: ?Object,
  childVal: ?Object,
  vm?: Component,
  key: string
): ?Object {
  // 如果childVal存在，且是非生成环境，检测childVal是不是一个纯对象
  if (childVal && process.env.NODE_ENV !== 'production') {
    assertObjectType(key, childVal, vm)
  }
  // 如果parentVal不存在，直接返回childVal
  if (!parentVal) return childVal
  // 创建一个空对象ret
  const ret = Object.create(null)
  // 将parentVal合并到ret对象上(key同名会覆盖)
  extend(ret, parentVal)
  // 如果childVal存在，再将childVal合并到ret上(key同名会覆盖)
  if (childVal) extend(ret, childVal)
  // 返回所有合并后的结果 ret对象
  return ret
}
```

分析：

- 如果`parentVal`不存在，直接返回`childVal`
- 如果`parentVal`存在，合并`childVal`和`parentVal`到`ret`对象并返回