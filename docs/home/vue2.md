[[toc]]
# 2.vue双向数据绑定原理


**现代 js 框架存在的根本原因**
-------------------

然而通常人们（自以为）使用框架是因为：
它们支持组件化；
它们有强大的社区支持；
它们有很多（基于框架的）第三方库来解决问题；
它们有很多（很好的）第三方组件;
它们有浏览器扩展工具来帮助调试；
它们适合做单页应用。

**Keeping the UI in sync with the state is hard (UI与状态同步非常困难)**
通过（添加）观察者监测变化，如 Angular 和 Vue.js。应用中状态的属性会被监测，当它们发生变化时，只有依赖了（发生变化）属性的 DOM 元素会被重新渲染。


数据劫持结合发布者-订阅者模式
---------------

**1.属性拦截器-初步数据劫持**

Object.defineProperty()

``` js
let a = {}

Object.defineProperty(a, 'b', {
  enumerable: true,
  configurable: true,
  set (newValue){
    console.log('set')
    value = newValue
  },
  get (){
    console.log('get')
    return value
  }
})
value = a.b
a.b = 1
console.log(a.b)
```

读a.b或者设置a.b时候触发get和set函数
configurable如果为false,那么不可以修改, 不可以删除.
writable给的说明是如果设置为false,不可以采用 数据运算符,进行赋值

**2.想实现一个这样的功能**

当我们试图修改 a 的值时：ins.a = 2，在控制台将会打印 '修改了 a’,
乍一看比较简单
考虑到复杂情况,
比如如何避免收集重复的依赖，如何深度观测，如何处理数组以及其他边界条件等等

``` js
const ins = new Vue({
  data: {
    a: 1
  }
})

ins.$watch('a', () => {
  console.log('修改了 a')
})
```

**3.收集依赖, 起码需要一个”筐“**

``` js
// dep 数组就是我们所谓的“筐”
const dep = []
Object.defineProperty(data, 'a', {
  set () {
    // 当属性被设置的时候，将“筐”里的依赖都执行一次
    dep.forEach(fn => fn())
  },
  get () {
    // 当属性被获取的时候，把依赖放到“筐”里
    dep.push(fn)
  }
})

$watch('a', () => {
  console.log('设置了 a')
})
```
**4.$watch 函数是知道当前正在观测的是哪一个字段的**

``` js
const data = {
  a: 1
}

const dep = []
Object.defineProperty(data, 'a', {
  set () {
    dep.forEach(fn => fn())
  },
  get () {
    // 此时 Target 变量中保存的就是依赖函数
    dep.push(Target)
  }
})

// Target 是全局变量
let Target = null
function $watch (exp, fn) {
  // 将 Target 的值设置为 fn
  Target = fn
  // 读取字段值，触发 get 函数
  data[exp]
}

``` 
明白数据响应系统的整体思路,为接下来真正进入 Vue 源码做必要的铺垫

**4.observer**

observe工厂函数
``` js
    const data = {
      a: 1
    }
    
    const data = {
      a: 1,
      // __ob__ 是不可枚举的属性
      __ob__: {
        value: data, // value 属性指向 data 数据对象本身，这是一个循环引用
        dep: dep实例对象, // new Dep()
        vmCount: 0
      }
    }
```
new Observer(data) 

Observer构造函数-调用this.walk(value)-defineReactive(get和set)

get 调用 dep.depend()  在 get 函数中如何收集依赖

set 调用 dep.notify()   通知更新


**5.dep**

``` js
static target;  // watcher
id;     //记录id不能重复收集
subs;    //数组,sub收集所以的watcher
```

dep.notify()实际上是 watcher里的update() 渲染更新
dep.depend()实际上是 watcher里的update() 渲染更新  把watcher实例对象推入subs

**6.watcher**
vm, vue实例对象
expOrFn, 表达式
cb, 回调函数

当然还在vm上定义了很多其他的computer,watch之类的
收集依赖,要想收集,必须 new watcher()
``` js
get () {
   pushTarget(this)
}

addDep (){
   dep.addSub(this)  // 把watcher自己加入dep.subs数组
}
update(){
  queueWatcher()  //排队渲染
} 
```
**6.总结**
![图片描述][1]


  [1]: https://image-static.segmentfault.com/115/842/1158420349-5b8e2aa1e0d39_articlex



