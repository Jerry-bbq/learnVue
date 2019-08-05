# initComputed

``` javascript
/**
 * 初始化计算属性
 * @param {*} vm
 * @param {*} computed
 */
function initComputed (vm: Component, computed: Object) {
  // $flow-disable-line
  // 创建两个一样的空对象 watchers，vm._computedWatchers
  const watchers = vm._computedWatchers = Object.create(null)
  // computed properties are just getters during SSR
  const isSSR = isServerRendering()

  // 遍历计算属性 computed 对象
  for (const key in computed) {
    // 获取计算属性key对应的value：userDef
    const userDef = computed[key]
    // 如果对应的计算属性是函数，赋值给getter常量，或则将userDef.get赋值给getter
    const getter = typeof userDef === 'function' ? userDef : userDef.get
    // 非生产环境，如果getter不存在，报警告
    if (process.env.NODE_ENV !== 'production' && getter == null) {
      warn(
        `Getter is missing for computed property "${key}".`,
        vm
      )
    }
    if (!isSSR) {
      // create internal watcher for the computed property.
      // 为每一个 getter 创建 watcher ---- computed watcher 和 watcher不同
      watchers[key] = new Watcher(
        vm,
        getter || noop,
        noop,
        computedWatcherOptions
      )
    }

    // component-defined computed properties are already defined on the
    // component prototype. We only need to define computed properties defined
    // at instantiation here.
    if (!(key in vm)) {// 如果计算属性不在vm实例上
      // 定义计算属性，传入vm实例，计算属性的key，计算属性的value
      defineComputed(vm, key, userDef)
    } else if (process.env.NODE_ENV !== 'production') {
      // 非生产环境，校验计算属性是否已经存在data或者props中，并给出相应的提示
      if (key in vm.$data) {
        warn(`The computed property "${key}" is already defined in data.`, vm)
      } else if (vm.$options.props && key in vm.$s.props) {
        warn(`The computed property "${key}" is already defined as a prop.`, vm)
      }
    }
  }
}
```

defineComputed

``` javascript
/**
 * 定义计算属性
 * @param {*} target   vm对象
 * @param {*} key      计算属性的key
 * @param {*} userDef  key对应的value
 * 作用：利用 Object.defineProperty 给计算属性对应的 key 值添加 getter 和 setter
 */
export function defineComputed (
  target: any,
  key: string,
  userDef: Object | Function
) {
  // 是否是服务器渲染，true: 非服务器渲染，false：服务器渲染
  const shouldCache = !isServerRendering()
  // 如果 计算属性 是函数
  if (typeof userDef === 'function') {
    // 给属性描述符添加getter
    sharedPropertyDefinition.get = shouldCache
      ? createComputedGetter(key)// 返回值是一个函数 computedGetter，即计算属性对应的getter函数
      : createGetterInvoker(userDef)
    sharedPropertyDefinition.set = noop
  } else { // 如果 计算属性 不是函数
    sharedPropertyDefinition.get = userDef.get
      ? shouldCache && userDef.cache !== false
        ? createComputedGetter(key)
        : createGetterInvoker(userDef.get)
      : noop
    sharedPropertyDefinition.set = userDef.set || noop
  }
  if (process.env.NODE_ENV !== 'production' &&
      sharedPropertyDefinition.set === noop) {// 非生产环境下，并且，setter是空函数
    // 添加setter函数，并抛出异常提示
    sharedPropertyDefinition.set = function () {
      warn(
        `Computed property "${key}" was assigned to but it has no setter.`,
        this
      )
    }
  }
  // 利用 Object.defineProperty 给计算属性对应的 key 值添加 getter 和 setter
  Object.defineProperty(target, key, sharedPropertyDefinition)
}
```

createComputedGetter

``` javascript
/**
 * 给计算属性每个key添加getter
 * @param {*} key
 */
function createComputedGetter (key) {
  // 返回计算属性对应的getter函数 computedGetter
  return function computedGetter () {
    // 如果this._computedWatchers对象存在，则获取对象中对应key的value，即Wathcer
    const watcher = this._computedWatchers && this._computedWatchers[key]
    if (watcher) {
      // dirty默认是true
      if (watcher.dirty) {
        // 依赖收集相关的方法,获取新的属性值
        watcher.evaluate()
      }
      if (Dep.target) {
        // 添加依赖
        watcher.depend()
      }
      // 返回value
      return watcher.value
    }
  }
}
```