# initState

`initState`包括如下`5`个方法的初始化（有顺序）：

1. props
2. methods
3. data
4. computed
5. watch

源码分析：

```javascript
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

总结：

* 从上代码可以看出，为什么`props`是在`data`之前执行的
* `props、methods、data、computed、watch`都是挂在`vm`实例的`$options`上的

example:

``` javascript
<template>
  <div>home</div>
</template>

<script>
export default {
  props: {
    msg: {
      type: String,
      default: ''
    },
    boolFalse: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      name: 'Jack',
      age: 10,
      className: '小班'
    }
  },
  mounted() {
    // console.log(this)
  },
  methods: {
    handleClick() {}
  },
  computed: {
    time() {
      return new Date()
    }
  },
  watch: {
    name: {
      handler(n){
        console.log(n)
      }
    }
  },
}
</script>

```

结果如下图：

<img src="./images/initState.jpg">