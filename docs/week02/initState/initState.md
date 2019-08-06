# initState

`initState`包括如下`5`个方法的初始化（有顺序）：

1. props
2. methods
3. data
4. computed
5. watch

源码分析：

```javascript
// 可以看出，为什么props是先在data之前执行的
export function initState (vm: Component) {
  vm._watchers = []
  // 获取当前实例的配置项
  const opts = vm.$options
  // 当前实例的配置项存在props(如果有自定义的props)，则初始化props,比如：props:{type: String,default: ''}
  if (opts.props) initProps(vm, opts.props)
  // 当前实例的配置项存在methods，则初始化methods
  if (opts.methods) initMethods(vm, opts.methods)
  // 当前实例的配置项存在data，初始化data
  if (opts.data) {
    initData(vm)
  } else {
    observe(vm._data = {}, true /* asRootData */)
  }
  // 当前实例的配置项存在computed，则初始化computed
  if (opts.computed) initComputed(vm, opts.computed)
  // 当前实例的配置项存在watch，并且不是原生的watch，则初始化watch
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch)
  }
}
```
