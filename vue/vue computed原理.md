# Vue computed 原理

**computed**和 **data**的原理一样，都是通过Object.defineProperty和观察者模式实现的，同时因为两者都是挂载在vm下，所以两者不能同名

-------------------

### computed

在initState调用时，initComputed，主要代码如下，解析见备注：

```javascript
function initComputed (vm: Component, computed: Object) {
  for (const key in computed) {
    const userDef = computed[key]
    // 获取getter
    const getter = typeof userDef === 'function' ? userDef : userDef.get

    if (!isSSR) {
      // 初始化watcher，并传入getter
      watchers[key] = new Watcher(
        vm,
        getter || noop,
        noop,
        computedWatcherOptions
      )
    }
    // 判断vm中不存在该计算属性时，调用defineComputed
    if (!(key in vm)) {
      defineComputed(vm, key, userDef)
    } else if (process.env.NODE_ENV !== 'production') {
      if (key in vm.$data) {
        warn(`The computed property "${key}" is already defined in data.`, vm)
      } else if (vm.$options.props && key in vm.$options.props) {
        warn(`The computed property "${key}" is already defined as a prop.`, vm)
      }
    }
  }
}

export function defineComputed (
  target: any,
  key: string,
  userDef: Object | Function
) {
  const shouldCache = !isServerRendering()
  // 计算属性中只有getter，为getter添加监听器，并把set制空
  if (typeof userDef === 'function') {
    sharedPropertyDefinition.get = shouldCache
      ? createComputedGetter(key)
      : userDef
    sharedPropertyDefinition.set = noop
  } else {
    // 计算属性中有getter和setter，为getter添加监听器，并把传入set
    sharedPropertyDefinition.get = userDef.get
      ? shouldCache && userDef.cache !== false
        ? createComputedGetter(key)
        : userDef.get
      : noop
    sharedPropertyDefinition.set = userDef.set
      ? userDef.set
      : noop
  }
  if (process.env.NODE_ENV !== 'production' &&
      sharedPropertyDefinition.set === noop) {
    sharedPropertyDefinition.set = function () {
      warn(
        `Computed property "${key}" was assigned to but it has no setter.`,
        this
      )
    }
  }
  // 进行计算属性的数据劫持
  Object.defineProperty(target, key, sharedPropertyDefinition)
}

//创建computed的getter，在调用getter时收集依赖关系
function createComputedGetter (key) {
  return function computedGetter () {
    // 从_computedWatchers中取出初始化时传入的getter
    const watcher = this._computedWatchers && this._computedWatchers[key]
    if (watcher) {
      if (watcher.dirty) {
        watcher.evaluate()
      }
      // 判断是否处于依赖收集状态
      if (Dep.target) {
	    // 建立依赖关系
        watcher.depend()
      }
      // 返回getter
      return watcher.value
    }
  }
}
```