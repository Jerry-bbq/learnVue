# Vue是如何进行合并配置的

在入口文件中`src/core/instance/init.js`中，`Vue.prototype._init`方法：

``` javascript
  // 合并配置项
    if (options && options._isComponent) {// 如果 options 存在并且有子组件时，options._isComponent为true
      // optimize internal component instantiation
      // since dynamic options merging is pretty slow, and none of the
      // internal component options needs special treatmen
      // 内部component 的options初始化
      initInternalComponent(vm, options)
    } else {// 非内部Component的 options 初始化
      // 将Vue.options与实例化Vue时传入的options进行合并
      vm.$options = mergeOptions(
        // 解析构造函数的options，Vue.options
        resolveConstructorOptions(vm.constructor),
        options || {}, //实例化Vue时传入的options： {  el: '#app',components: { App },template: '<App/>',}
        vm // 当前实例
      )
    }
```

从源码中可以看到两个关键方法`initInternalComponent(vm, options)`和`mergeOptions`

**initInternalComponent**的实现：

``` javascript
export function initInternalComponent (vm: Component, options: InternalComponentOptions) {
  const opts = vm.$options = Object.create(vm.constructor.options)
  // doing this because it's faster than dynamic enumeration.
  const parentVnode = options._parentVnode
  opts.parent = options.parent
  opts._parentVnode = parentVnode

  const vnodeComponentOptions = parentVnode.componentOptions
  opts.propsData = vnodeComponentOptions.propsData
  opts._parentListeners = vnodeComponentOptions.listeners
  opts._renderChildren = vnodeComponentOptions.children
  opts._componentTag = vnodeComponentOptions.tag

  if (options.render) {
    opts.render = options.render
    opts.staticRenderFns = options.staticRenderFns
  }
}
```

resolveConstructorOptions的实现：

```javascript
/**
 * 解析构造函数的options（Vue.options）
 * @param {*} Ctor
 * @return 返回构造函数的options，比如 options:{ components:{},directives:{},filters:{},_base:Vue}
 */
export function resolveConstructorOptions (Ctor: Class<Component>) {
  let options = Ctor.options
  if (Ctor.super) {
    const superOptions = resolveConstructorOptions(Ctor.super)
    const cachedSuperOptions = Ctor.superOptions
    if (superOptions !== cachedSuperOptions) {
      // super option changed,
      // need to resolve new options.
      Ctor.superOptions = superOptions
      // check if there are any late-modified/attached options (#4976)
      const modifiedOptions = resolveModifiedOptions(Ctor)
      // update base extend options
      if (modifiedOptions) {
        extend(Ctor.extendOptions, modifiedOptions)
      }
      options = Ctor.options = mergeOptions(superOptions, Ctor.extendOptions)
      if (options.name) {
        options.components[options.name] = Ctor
      }
    }
  }
  return options
}
```

**mergeOptions**的`实现在`src/core/util/options.js`，
该方法接受三个参数：

- vm.constructor的options,
- new Vue时传入的options,
- vue对象

```javascript


```
