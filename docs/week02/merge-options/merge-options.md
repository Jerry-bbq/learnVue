# 合并配置

1. vue为什么要进行**合并配置**？

    目的是将相关的属性和方法都放到`vm.$options`中

2. vue是如何将这么多的配置进行合并的？

    在`src/core/instance/init.js`中可以看到如下一段代码：

    ``` javascript
    // 合并配置项
    if (options && options._isComponent) {
      // 如果 options 存在并且有子组件时，options._isComponent为true
      // 内部component 的options初始化
      initInternalComponent(vm, options)
    } else {
      // 非内部Component的 options 初始化
      // 将Vue.options与实例化Vue时传入的options进行合并
      vm.$options = mergeOptions(
        // 解析构造函数的options，Vue.options
        resolveConstructorOptions(vm.constructor),
        options || {}, //实例化Vue时传入的options： {  el: '#app',components: { App },template: '<App/>',}
        vm // 当前实例
      )
    }
    ```

    可以看到两个重要的方法：`initInternalComponent`,`mergeOptions`

    ``` javascript
    /* 初始化 内部组件
    * @param {*} vm
    * @param {*} options
    */
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

    其中`mergeOptions`中包含了`resolveConstructorOptions`方法，`resolveConstructorOptions`的作用是返回`vm.constructor`的`options`

    ``` javascript
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

3. `mergeOptions`方法内部做了什么

    ``` javascript
    export function mergeOptions (
        parent: Object,
        child: Object, // 实例化Vue时，传入的参数，child = {  el: '#app',components: { App:{beforeCreate：...,data:...,name:'App'} },template: '<App/>',}
        vm?: Component // Vue实例（非必填）
        ): Object {
        if (process.env.NODE_ENV !== 'production') {
            // 检测组件命名是否符合规范
            checkComponents(child)
        }
        // child是function类型的话，我们取其options属性作为child
        if (typeof child === 'function') {
            child = child.options
        }

        // 规范化，将props,inject,directives都转化为对象的形式
        normalizeProps(child, vm)
        normalizeInject(child, vm)
        normalizeDirectives(child)
        if (!child._base) {
            // 如果child.extends存在，则递归合并parent, child.extends
            if (child.extends) {
            parent = mergeOptions(parent, child.extends, vm)
            }
            // 如果child.mixins存在，则递归合并parent, child.mixins
            if (child.mixins) {
            for (let i = 0, l = child.mixins.length; i < l; i++) {
                parent = mergeOptions(parent, child.mixins[i], vm)
            }
            }
        }

        const options = {}
        let key
        for (key in parent) {
            mergeField(key)
        }
        for (key in child) {
            if (!hasOwn(parent, key)) {
            mergeField(key)
            }
        }
        function mergeField (key) {
            const strat = strats[key] || defaultStrat
            options[key] = strat(parent[key], child[key], vm, key)
        }
        return options
        }
    ```

    分析：

    1. parent: `Vue.options ，parent = { components:{KeepAlive: {},Transition:{},TransitionGroup:{}},directives:{},filters:{},_base:Vue}`
    2. child: 实例化Vue时，传入的参数，` {  el: '#app',components: { App:{beforeCreate：...,data:...,name:'App'} },template: '<App/>',}`
    3. vm: Vue实例（非必填）
    4. 分别遍历`parent`，`child`，通过**合并策略**合并相关的配置

    那么**合并策略**又是如何将众多配置进行合并的呢，请看[下一节](.//merge-strats.md)
