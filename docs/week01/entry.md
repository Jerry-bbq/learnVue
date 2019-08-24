# vue的入口文件

入口文件的位置：`src/core/instance/index.js`

``` javascript
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

/**
 * 定义构造函数 Vue
 * @param {*} options,{el: '#app',router,components: { App },template: '<App/>',}
 */
function Vue (options) {
  // 非生产环境下，如果this的实例不是Vue，警告提示
  if (process.env.NODE_ENV !== 'production' && !(this instanceof Vue)) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  // 初始化
  this._init(options)
}

initMixin(Vue) // Vue.prototye上添加了_init方法
stateMixin(Vue) // Vue.prototye上定义了属性: $data、$props，方法：$set、$delete、$watch
eventsMixin(Vue)// Vue.prototye上添加了四个方法: $on、 $once 、$off 、$emit
lifecycleMixin(Vue)// lifecycleMixin(Vue)，在原型Vue.prototye上添加了三个方法：_update 、$forceUpdate 、$destory
renderMixin(Vue)// Vue.prototye上添加了方法：$nextTick 、_render、 _o、 _n、 _s、 _l、 _t、 _q、 _i、 _m、 _f、 _k、 _b、 _v、 _e、 _u、 _g、 _d、 _p

export default Vue

```

作用：

1. 定义`Vue`构造函数，并暴露`Vue`方法

2. 调用五个`mixin`混入方法，在`Vue`的原型`prototype`上添加不同的属性和方法

这个时候，我们知道了，我们在new Vue()的时候，实际上就是调用`this._init(options)`方法。`options`包括：`data,props,computed,methods,watch,el,template`等
[选项列表](https://cn.vuejs.org/v2/api/#%E9%80%89%E9%A1%B9-%E6%95%B0%E6%8D%AE)

五个mixin方法，作用是在Vue的原型上添加相关的属性和方法：

- initMixin(Vue)

    作用：Vue.prototye上添加`_init`方法

- stateMixin(Vue)

    作用：Vue.prototye上添加属性: `$data、$props`，方法：`$set、$delete、$watch`

- eventsMixin(Vue)

    作用：Vue.prototye上添加四个方法: `$on、 $once 、$off 、$emit`

- lifecycleMixin(Vue)

    作用：Vue.prototye上添加三个方法：`_update 、$forceUpdate 、$destory`
- renderMixin(Vue)

    作用：Vue.prototye上添加方法：`$nextTick 、_render、 _o、 _n、 _s、 _l、 _t、 _q、 _i、 _m、 _f、 _k、 _b、 _v、 _e、 _u、 _g、 _d、 _p`

一、`initMixin(Vue)`
作用：在Vue的原型上添加一个`_init`方法

```javascript
export function initMixin (Vue: Class<Component>) {
    Vue.prototype._init = function (options?: Object) {}
}
```

二、`stateMixin(Vue)`
作用：在Vue的原型上添加了如下两个属性：`$data、$props；三个方法：$set、$delete、$watch`

```javascript
export function stateMixin (Vue: Class<Component>) {
    Object.defineProperty(Vue.prototype, '$data', dataDef)
    Object.defineProperty(Vue.prototype, '$props', propsDef)
    Vue.prototype.$set = set
    Vue.prototype.$delete = del
    Vue.prototype.$watch
}
```

三、`eventsMixin(Vue)`
作用：在原型上添加了四个方法: `$on、 $once 、$off 、$emit`

```javascript
 export function eventsMixin (Vue: Class<Component>) {
     Vue.prototype.$on = function (event: string | Array<string>, fn: Function): Component {}
     Vue.prototype.$once = function (event: string, fn: Function): Component {}
     Vue.prototype.$off = function (event?: string | Array<string>, fn?: Function): Component {}
     Vue.prototype.$emit = function (event: string): Component {}
 }
```

四、`lifecycleMixin(Vue)`
作用：在Vue.prototye上添加了三个方法：`_update 、$forceUpdate 、$destory`

```javascript
export function lifecycleMixin (Vue: Class<Component>) {
    Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {}
    Vue.prototype.$forceUpdate = function () {} // 迫使Vue实例重新渲染，包括其下的子组件
    Vue.prototype.$destroy = function () {} // 完全销毁一个实例, 触发生命周期beforeDestroy和destroyed
}
```

五、`renderMixin(Vue)`
作用：在原型上添加了方法：`$nextTick 、_render、 _o、 _n、 _s、 _l、 _t、 _q、 _i、 _m、 _f、 _k、 _b、 _v、 _e、 _u、 _g、 _d、 _p`

```javascript
export function renderMixin (Vue: Class<Component>) {
    installRenderHelpers(Vue.prototype)
    Vue.prototype.$nextTick = function (fn: Function) {}
    Vue.prototype._render = function (): VNode {}
}
```

```javascript
export function installRenderHelpers (target: any) {
  target._o = markOnce
  target._n = toNumber
  target._s = toString
  target._l = renderList
  target._t = renderSlot
  target._q = looseEqual
  target._i = looseIndexOf
  target._m = renderStatic
  target._f = resolveFilter
  target._k = checkKeyCodes
  target._b = bindObjectProps
  target._v = createTextVNode
  target._e = createEmptyVNode
  target._u = resolveScopedSlots
  target._g = bindObjectListeners
  target._d = bindDynamicKeys
  target._p = prependModifier
}
```
接下来，我们看看`import Vue from 'vue'`时做了什么[下一章](./import.md)
