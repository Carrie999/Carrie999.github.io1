[[toc]]
# Vuex 2.0 源码解读


**准备工作**
--------

 

1.下载安装运行
--------

这里以2.0.0-rc.6为例
官网github下载链接(对应版本):https://github.com/vuejs/vuex/tree/v2.0.0-rc.6
点击下载到本地
  按照readme.md
``` bash
$ npm install
$ npm run dev # serve examples at localhost:8080
```

2.example示例
-----------

查看package.json dev命令,

```js
"dev": "node examples/server.js" 
```
打开http://localhost:8080/
 examples里面的文件都能运行了,
Counter,
Counter with Hot Reload,
Shopping Cart,
TodoMVC,
FluxChat 这些每一个有很有代表性


**源码 src/index.js**
--------

vuex 只暴露出了6个方法

``` js
export default {
  Store,
  install,
  mapState,
  mapMutations,
  mapGetters,
  mapActions
}

```


1.install方法
-----------

install函数用于vue
vue.use() 里会调用自动install方法

``` js
function install (_Vue) {
  if (Vue) {
    console.error(
      '[vuex] already installed. Vue.use(Vuex) should be called only once.'
    )
    return
  }
  Vue = _Vue
  applyMixin(Vue)
}
  applyMixin(Vue){
    // ...
    function vuexInit () {
        const options = this.$options
        // store injection
        if (options.store) {
          this.$store = options.store
        } else if (options.parent && options.parent.$store) {
          this.$store = options.parent.$store
        }
      }
}
```
new Vue({store})传进去一个store对象,在new Vue()的时候对options进行了处理
```javascript
    vm.$options.store =  store
    // 把传入的对象除了$el都挂在了$options下
    // 参考vue源码对options处理部分
```
这样做
    if (options.store) {
         // 这样在 vue 里全局里通过 $store能够访问store对象
         this.$store = options.store
     } 
 
2.Store构造函数
------------

``` javascript
class Store {
  constructor (options = {}) {assert(Vue, `must call Vue.use(Vuex) before creating a store instance.`)
    assert(typeof Promise !== 'undefined', `vuex requires a Promise polyfill in this browser.`)
    // options解构赋值
    const {
      state = {},
      plugins = [],
      strict = false
    } = options
    
    // store internal state
    this._options = options
    this._committing = false
    this._actions = Object.create(null)
    this._mutations = Object.create(null)
    this._wrappedGetters = Object.create(null)
    this._runtimeModules = Object.create(null)
    this._subscribers = []
    this._watcherVM = new Vue()

    // bind commit and dispatch to self
    const store = this
    // dispatch, commit 解构赋值
    const { dispatch, commit } = this
    // dispatch改变this,使this为store
    this.dispatch = function boundDispatch (type, payload) {
      return dispatch.call(store, type, payload)
    }
    // 同上
    this.commit = function boundCommit (type, payload, options) {
      return commit.call(store, type, payload, options)
    }
    
    // 严格模式
    // strict mode 
    this.strict = strict
    
    // 对传入module的处理
    // init root module.
    // this also recursively registers all sub-modules
    // and collects all module getters inside this._wrappedGetters
    installModule(this, state, [], options)

    // initialize the store vm, which is responsible for the reactivity
    // (also registers _wrappedGetters as computed properties)
    resetStoreVM(this, state)

    // apply plugins
    plugins.concat(devtoolPlugin).forEach(plugin => plugin(this))
    }
}

```

```javascript
 get state () {
     //返回state,state挂在在stare.vm下
    return this._vm.state
  }

  set state (v) {
    //不能被外部改变,私有变量
    assert(false, `Use store.replaceState() to explicit replace store state.`)
  }

```

3.installModule函数
-------------
installModule(this, state, [], options)

``` js
function installModule (store, rootState, path, module, hot) {
 // 如果path为[]那么isRoot为tree
  const isRoot = !path.length
 // 把传入的options请求解构赋值,
  const {
    state,
    actions,
    mutations,
    getters,
    modules
  } = module

  // hot不存在而且path不为[]
  // set state
  if (!isRoot && !hot) {
    const parentState = getNestedState(rootState, path.slice(0, -1))
    const moduleName = path[path.length - 1]
    store._withCommit(() => {
      Vue.set(parentState, moduleName, state || {})
    })
  }

  if (mutations) {
    Object.keys(mutations).forEach(key => {
      registerMutation(store, key, mutations[key], path)
    })
  }

  if (actions) {
    Object.keys(actions).forEach(key => {
      registerAction(store, key, actions[key], path)
    })
  }

  if (getters) {
    wrapGetters(store, getters, path)
  }

  if (modules) {
    //注意 只有传递了modules 才会执行if (!isRoot && !hot)后面的处理,这里对path.concat(key)进行了处理
    Object.keys(modules).forEach(key => {
      installModule(store, rootState, path.concat(key), modules[key], hot)
    })
  }
}
```
这里有个很重要的概念要理解，什么是 path. vuex 的一个 store 实例可以拆分成很多个 module ,不同的 module 可以理解成一个子代的 store 实例（事实上，module 确实和 store 具有一样的结构）,这是一种模块化的概念。因此这里的 path 可以理解成是表示一种层级关系(树状)，当你有了一个 root state 之后，根据这个 root state 和 path 可以找到 path 路径对应的一个 local state， 每一个 module 下的 mutations 和 actions 改变的都是这个local state，而不是 root state.

看一下 getNestedState方法 
 
```js
/*
 * state: Object, path: Array
 * 假设path = ['a', 'b', 'c']
 * 函数返回结果是state[a][b][c]
 */
function getNestedState (state, path) {
  return path.length
    ? path.reduce(function (state, key) { return state[key]; }, state)
    : state
}
```
这里学习一下reduce的妙用  [reduce不清楚的 mdn传送门点我][1]

```js
 /** handler是传入的mutation[key]方法比如
   increment (state) {
       state.count++
   },
   decrement (state) {
       state.count--
   }
   type就是key increment, decrement
  **/

registerMutation函数
----------------

function registerMutation (store, type, handler, path = []) {
  const entry = store._mutations[type] || (store._mutations[type] = [])
  entry.push(function wrappedMutationHandler (payload) {
    handler(getNestedState(store.state, path), payload)
  })
}
```
所有的 mutations 都经过处理后，保存在了 store._mutations 对象里。 _mutations 的结构为

```js
_mutations: {
    increment: [function wrappedMutationHandler(payload){increment(store.state.modules){...}}, ...],
    decrement: [function wrappedMutationHandler(payload){decrement(store.state.modules, payload){...}}, ...],
    ...
}
```
这个对应第一个参数为该模块下的state对象,如果没有modules就是rootState,
第二个参数就是payload 用户自定义传递的参数
 
``` js
     increment (state) 
        state.count++
      },
      decrement (state) {
        state.count--
      }
```   

同理

4.registerAction函数
----------------
``` js
function registerAction (store, type, handler, path = []) {
  const entry = store._actions[type] || (store._actions[type] = [])
  const { dispatch, commit } = store
  entry.push(function wrappedActionHandler (payload, cb) {
    let res = handler({
      dispatch,
      commit,
      getters: store.getters,
      state: getNestedState(store.state, path),
      rootState: store.state
    }, payload, cb)
    if (!isPromise(res)) {
      // 返回 Promise
      res = Promise.resolve(res)
    }
    if (store._devtoolHook) {
      return res.catch(err => {
        store._devtoolHook.emit('vuex:error', err)
        throw err
      })
    } else {
      return res
    }
  })
}
```  
被处理成
```js
_actions: {
    increment: [function wrappedActionHandler(payload){increment({
      dispatch,
      commit,
      getters: store.getters,
      state: getNestedState(store.state, path),
      rootState: store.state
    }){...}}, ...],
    decrement: [function wrappedActionHandler(payload){decrement({
      dispatch,
      commit,
      getters: store.getters,
      state: getNestedState(store.state, path),
      rootState: store.state
    }, payload){...}}, ...],
    ...
}
```
第一个参数传入一个对象,所以才能使用commit方法
```js
const actions = {
  increment: ({ commit }) => commit('increment'),
  decrement: ({ commit }) => commit('decrement')
}
```

5.commit和dispatch
---------------

commit为同步,dispatch为异步,当所有执行完毕返回Promise
```js
commit (type, payload, options) {
    // check object-style commit
    if (isObject(type) && type.type) {
      options = payload
      payload = type
      type = type.type
    }
    const mutation = { type, payload }
    const entry = this._mutations[type]
    if (!entry) {
      console.error(`[vuex] unknown mutation type: ${type}`)
      return
    }
    this._withCommit(() => {
      entry.forEach(function commitIterator (handler) {
        handler(payload)
      })
    })
    if (!options || !options.silent) {
      this._subscribers.forEach(sub => sub(mutation, this.state))
    }
  }

  dispatch (type, payload) {
    // check object-style dispatch
    if (isObject(type) && type.type) {
      payload = type
      type = type.type
    }
    const entry = this._actions[type]
    if (!entry) {
      console.error(`[vuex] unknown action type: ${type}`)
      return
    }
    return entry.length > 1
      ? Promise.all(entry.map(handler => handler(payload)))
      : entry[0](payload)
  }
```

6.wrapGetters
-----------
```js
// store增加一个 _wrappedGetters 属性
function wrapGetters (store, moduleGetters, modulePath) {
  Object.keys(moduleGetters).forEach(getterKey => {
    const rawGetter = moduleGetters[getterKey]
    if (store._wrappedGetters[getterKey]) {
      console.error(`[vuex] duplicate getter key: ${getterKey}`)
      return
    }
    store._wrappedGetters[getterKey] = function wrappedGetter (store) {
      return rawGetter(
        getNestedState(store.state, modulePath), // local state
        store.getters, // getters
        store.state // root state
      )
    }
  })
}
```
注意 这里的所有 getters 都储存在了全局的一个 _wrappedGetters 对象中，同样属性名是各个 getterKey ,属性值同样是一个函数，执行这个函数，将会返回原始 getter 的执行结果。

7.resetStoreVM
------------

```js
function resetStoreVM (store, state) {
  const oldVm = store._vm

  //...
  Object.keys(wrappedGetters).forEach(key => {
    const fn = wrappedGetters[key]
    // use computed to leverage its lazy-caching mechanism
    computed[key] = () => fn(store)
    Object.defineProperty(store.getters, key, {
      get: () => store._vm[key]
    })
  })

   //...
  store._vm = new Vue({
    data: { state },
    computed
  })

  if (oldVm) {
    // dispatch changes in all subscribed watchers
    // to force getter re-evaluation.
    store._withCommit(() => {
      oldVm.state = null
    })
    Vue.nextTick(() => oldVm.$destroy())
  }
}

```
着重讲一下最重要的
  
``` js
    store._vm = new Vue({
        data: { state },
        computed
      })
```
state是放在vue实例对象的 this.data.state 里面的
这个 new Vue 和你平常的new Vue不一样,这是两个vue实例,数据不共用
之所以挂在在vue是方便使用vue的一些方法nextTick,set等,更方便控制state状态
所以在vue组件里如果不写

    computed:{
         ...mapState(["count"])
     },

就不能直接写 template 里写{{count}} 而必须使用 this.$store.state.count

map辅助函数
--------
normalizeMap
------------
```js
/*
 * 如果map是一个数组 ['type1', 'type2', ...]
 * 转化为[
 *   {
 *     key: type1,
 *     val: type1
 *   },
 *   {
 *     key: type2,
 *     val: type2
 *   },
 *   ...
 * ]
 * 如果map是一个对象 {type1: fn1, type2: fn2, ...}
 * 转化为 [
 *   {
 *     key: type1,
 *     val: fn1
 *   },
 *   {
 *     key: type2,
 *     val: fn2
 *   },
 *   ...
 * ]
 */
function normalizeMap (map) {
  return Array.isArray(map)
    ? map.map(function (key) { return ({ key: key, val: key }); })
    : Object.keys(map).map(function (key) { return ({ key: key, val: map[key] }); })
}
```
normalizeMap 函数接受一个对象或者数组，最后都转化成一个数组形式，数组元素是包含 key 和 value 两个属性的对象。

map
---
```js
export function mapState (states) {
  const res = {}
  normalizeMap(states).forEach(({ key, val }) => {
    res[key] = function mappedState () {
      return typeof val === 'function' //是函数,执行返回结果
        ? val.call(this, this.$store.state, this.$store.getters)
        : this.$store.state[val] // 不是,返回$store.state[val]值
    }
  })
  return res
}

export function mapMutations (mutations) {
  const res = {}
  normalizeMap(mutations).forEach(({ key, val }) => {
    res[key] = function mappedMutation (...args) {
      // 通过commit执行Mutation里的函数和自己用$store.commit()执行结果是一样,定义一个方法,用户使用起来更方便
      return this.$store.commit.apply(this.$store, [val].concat(args))
    }
  })
  return res
}
//这里 getters 同样接受一个数组，同样返回一个对象
export function mapGetters (getters) {
  const res = {}
  normalizeMap(getters).forEach(({ key, val }) => {
    res[key] = function mappedGetter () {
      if (!(val in this.$store.getters)) {
        console.error(`[vuex] unknown getter: ${val}`)
      }
      return this.$store.getters[val]
    }
  })
  return res
}


export function mapActions (actions) {
  const res = {}
  normalizeMap(actions).forEach(({ key, val }) => {
    res[key] = function mappedAction (...args) {
     // 同mapMutations
      return this.$store.dispatch.apply(this.$store, [val].concat(args))
    }
  })
  return res
}

```
map*** 方法，这四种方法可以都返回一个对象，所以在vue里我们能够使用...
``` js
    computed:{
         ...mapState(["count"])
     }
```
小结
---
<!-- == -->
【单一状态树】
vuex 使用单一状态树——用一个对象就包含了全部的应用层级状态。至此它便作为一个“唯一数据源 (SSOT)”而存在。这也意味着，每个应用将仅仅包含一个 store 实例。单一状态树能够直接地定位任一特定的状态片段，在调试的过程中也能轻易地取得整个当前应用状态的快照

每一个 Vuex 应用的核心就是 store（仓库）。“store”基本上就是一个容器，它包含着应用中大部分的状态 (state)。Vuex 和单纯的全局对象有以下两点不同：

　　1、Vuex 的状态存储是响应式的。当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新

　　2、不能直接改变 store 中的状态。改变 store 中的状态的唯一途径就是显式地提交 (commit) mutation。这样使得可以方便地跟踪每一个状态的变化，从而能够实现一些工具帮助更好地了解应用
Vuex 背后的基本思想，借鉴了 Flux、Redux和 The Elm Architecture


参考
[vuex 2.0源码解读（一）][2]
[Flux 架构入门教程][3]
[Vue状态管理vuex][4]


  [1]: https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce
  [2]: https://segmentfault.com/a/1190000007108052
  [3]: http://www.ruanyifeng.com/blog/2016/01/flux.html
  [4]: https://www.cnblogs.com/xiaohuochai/p/7554127.html