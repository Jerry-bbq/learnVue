# initWatch

分析:

* 遍历`watch`对象
    * 如果当前`handler`是数组，遍历该数组，调用`createWatcher`
    * 否则，调用`createWatcher`

``` javascript
/**
 * 初始化watch
 * @param {*} vm
 * @param {*} watch
 */
function initWatch (vm: Component, watch: Object) {
  // 遍历watch
  for (const key in watch) {
    // 获取到每一个key的 handler
    const handler = watch[key]
    if (Array.isArray(handler)) {// 如果 handler 是一个数组
      // 遍历 handler 执行 createWatcher
      for (let i = 0; i < handler.length; i++) {
        createWatcher(vm, key, handler[i])
      }
    } else {// 如果 handler 不是一个数组直接执行 createWatcher
      createWatcher(vm, key, handler)
    }
  }
}
```

createWatcher

``` javascript
/**
 * 创建watcher
 * @param {*} vm
 * @param {*} expOrFn
 * @param {*} handler
 * @param {*} options
 */
function createWatcher (
  vm: Component,
  expOrFn: string | Function,
  handler: any,
  options?: Object
) {
  // 如果handler是一个纯对象
  if (isPlainObject(handler)) {
    options = handler
    handler = handler.handler
  }
  // 如果handler是一个字符串
  if (typeof handler === 'string') {
    handler = vm[handler]
  }
  // 调用回调函数,方法在 stateMixin() 中定义
  return vm.$watch(expOrFn, handler, options)
}
```
