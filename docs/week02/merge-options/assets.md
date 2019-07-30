# 资源合并

`strats`对象添加如下3个方法：

- components
- directives
- filters

```javascript
// Assets
function mergeAssets (
  parentVal: ?Object,
  childVal: ?Object,
  vm?: Component,
  key: string
): Object {
  const res = Object.create(parentVal || null)
  if (childVal) {
    process.env.NODE_ENV !== 'production' && assertObjectType(key, childVal, vm)
    return extend(res, childVal)
  } else {
    return res
  }
}
```

分析：

- 以`parentVal`为原型创建一个`res`对象
- 如果`childVal`存在,将`childVal`对象合并到`res`对象上，同名key覆盖，不同命赋值，并返回合并后的`res`对象
- 如果`childVal`不存在，直接返回`res`对象

遍历`ASSET_TYPES`,为每个资源添加`mergeAssets`方法

``` javascript
// ASSET_TYPES = ['component','directive','filter']
ASSET_TYPES.forEach(function (type) {
  strats[type + 's'] = mergeAssets
})
```

`assertObjectType`函数的作用是检测一直变量是否是纯对象

```javascript
function assertObjectType (name: string, value: any, vm: ?Component) {
  if (!isPlainObject(value)) {
    warn(
      `Invalid value for option "${name}": expected an Object, ` +
      `but got ${toRawType(value)}.`,
      vm
    )
  }
}
```
