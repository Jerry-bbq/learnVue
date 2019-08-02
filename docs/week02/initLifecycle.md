# initLifecycle

学习完了上面的**合并配置**，回到`_init`方法，可以看到还执行了如下的方法：

- initLifecycle(vm)// 初始化生命周期
- initEvents(vm)// 初始化事件
- initRender(vm)// 初始化渲染
- callHook(vm, 'beforeCreate') // 回调 beforeCreate 钩子函数
- initInjections(vm) // resolve injections before data/props，初始化injections
- initState(vm)// 初始化state，响应式
- initProvide(vm) // resolve provide after data/props，初始化provide
- callHook(vm, 'created')// created
- vm.$mount(vm.$options.el) // 挂在实例

`initLifecycle(vm)`源码：

``` javascript
export function initLifecycle (vm: Component) {
  // 获取 当前vm实例的配置options
  const options = vm.$options

  // locate first non-abstract parent
  // 定位第一个 非抽象 的父组件，vue的抽象组件有：keep-alive,transiiton,transition-group

  // 获取 当前vm实例的配置的父组件parent
  let parent = options.parent

  // 如果parent组件存在，并且是非抽象组件
  if (parent && !options.abstract) {
    // 如果parent组件是抽象组件 并且 parent组件存在parent组件
    while (parent.$options.abstract && parent.$parent) {
      // 把 parent组件的parent组件 赋值给 parent
      parent = parent.$parent
    }
    // 将当前vm实例压入parent的$children数组
    parent.$children.push(vm)
  }

  vm.$parent = parent // 指定已创建的实例之父实例，在两者之间建立父子关系。子实例可以用 this.$parent 访问父实例，子实例被推入父实例的 $children 数组中。
  vm.$root = parent ? parent.$root : vm // 当前组件树的根 Vue 实例。如果当前实例没有父实例，此实例将会是其自己。

  vm.$children = [] // 当前实例的直接子组件。需要注意 $children 并不保证顺序，也不是响应式的。
  vm.$refs = {}// 一个对象，持有已注册过 ref 的所有子组件

  vm._watcher = null // 组件实例相应的 watcher 实例对象。
  vm._inactive = null // 表示keep-alive中组件状态，如被激活，该值为false,反之为true。
  vm._directInactive = false// 也是表示keep-alive中组件状态的属性。
  vm._isMounted = false// 当前实例是否完成挂载(对应生命周期图示中的mounted)。
  vm._isDestroyed = false//当前实例是否已经被销毁(对应生命周期图示中的destroyed)。
  vm._isBeingDestroyed = false// 当前实例是否正在被销毁,还没有销毁完成(介于生命周期图示中deforeDestroy和destroyed之间)。
}
```

分析上面代码，可以看出这个`初始化生命周期`方法主要是给当前实例添加如下属性，并初始化：

- $parent
- $root
- $children
- $refs
- _watcher
- _inactive
- _directInactive
- _isMounted
- _isDestroyed
- _isBeingDestroyed
