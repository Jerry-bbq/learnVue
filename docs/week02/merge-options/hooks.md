# hooks合并策略

`strats`对象上添加如下`11`个钩子函数，并进行对应的hooks合并

- beforeCreate
- created
- beforeMount
- mounted
- beforeUpdate
- updated
- beforeDestroy
- destroyed
- activated
- deactivated
- errorCaptured

``` javascript
// Hooks and props are merged as arrays.
function mergeHook (
  parentVal: ?Array<Function>,
  childVal: ?Function | ?Array<Function>
): ?Array<Function> {
  // 根据三木运算符的右结合性，做如下区分
 return childVal ? ( parentVal ? parentVal.concat(childVal) : ( Array.isArray(childVal) ? childVal : [childVal] )) : parentVal
  // return childVal
  //   ? parentVal
  //     ? parentVal.concat(childVal)
  //     : Array.isArray(childVal)
  //       ? childVal
  //       : [childVal]
  //   : parentVal
}
```

分析：

- 如果`childVal`不存在，返回`parentVal`（数组）;
- 否则，进入下一个判断，判断`parentVal`（数组）是否存在，如果存在，`parentVal`拼接`childVal`;
- 否则，进入下一个判断， 判断`childVal`是否是数组，如果是，直接返回`childVal`，否则，把`childVal`变成数组。

遍历定义好的钩子常量，并对每个钩子常量添加`mergeHook`方法进行hooks合并：

```javascript
LIFECYCLE_HOOKS.forEach(hook => {
  strats[hook] = mergeHook
})
```
