# el和propsData的合并策略

`strats`对象上添加两个相同的方法`el`和`propsData`,接收两个参数`parent`和`child`，返回一个其中的一个参数，有`childVal`返回`childVal`,否则就返回`parentVal`

``` javascript
/**
 * Options with restrictions
 */
if (process.env.NODE_ENV !== 'production') {
  strats.el = strats.propsData = function (parent, child, vm, key) {
    if (!vm) {
      warn(
        `option "${key}" can only be used during instance ` +
        'creation with the `new` keyword.'
      )
    }
    // 返回默认合并策略函数
    return defaultStrat(parent, child)
  }
}
```

通过上面的源码代码可以看出，合并策略对象`strats`添加两个相同的方法`el`和`propsData`，分别都接收两个参数`parent`和`child`，并且返回同一个默认合并策略函数`defaultStrat`

``` javascript
// Default strategy.
const defaultStrat = function (parentVal: any, childVal: any): any {
  return childVal === undefined
    ? parentVal
    : childVal
}
```

从上面代码可以看出，默认合并策略`defaultStrat`接收`parentVal`和`childVal`两个参数，,返回两个参数中的某一个参数

规则是：有`childVal`返回`childVal`,否则就返回`parentVal`
