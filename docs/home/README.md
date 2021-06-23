[[toc]]
# 1.模版渲染

在init.js里调用$mount函数
把el #app获取相应dom传递过去
``` js
    Vue.prototype._init = function (options) {
       ...
       if (vm.$options.el) {
          vm.$mount(vm.$options.el)
        }   
    }
```
 entry-runtime-with-compiler.js里在Vue原型上定义$mount方法

``` js
Vue.prototype.$mount = function (
  el,
  hydrating
) { 
       let template = options.template
       template = idToTemplate(template)
       const { render, staticRenderFns } = compileToFunctions(template , {
        shouldDecodeNewlines, // 对浏览器的怪癖做兼容,布尔值。
        shouldDecodeNewlinesForHref, // 对浏览器的怪癖做兼容,布尔值。
        delimiters: options.delimiters, //vue透传
        comments: options.comments //vue透传
      }, this)
 } 
``` 
![图片描述][1]


 template(64)和 render(72), staticRenderFns(73) 对应图中所示

render是渲染函数，staticrenderfns是静态树的渲染函数

``` js
//render
function anonymous(
) {
with(this){return _c('div',[_c('h1',{staticStyle:{"color":"red"}},[_v("我是选项模板3")]),_v(" "),_c('p',[_v(_s(number))]),_v(" "),_c('p',[_v(_s(message))]),_v(" "),_m(0)])}
}  
//staticrenderfns
function anonymous(
) {
with(this){return _c('div',[_c('div',[_v("\n          1\n          "),_c('div',[_v("11")]),_v(" "),_c('div',[_v("12")])]),_v(" "),_c('div',[_v("2")]),_v(" "),_c('div',[_v("3")]),_v(" "),_c('div',[_v("4")]),_v(" "),_c('div',[_v("5")])])}
}
```


``` html
<!-- template -->
<template id="demo">
    <div>
      <h1 style="color:red">我是选项模板3</h1>
      <p>{{number}}</p>
      <p>{{message}}</p>
      <div>
        <div>
          1
          <div>11</div>  
          <div>12</div>  
        </div>
        <div>2</div>
        <div>3</div>
        <div>4</div>
        <div>5</div>
      </div>
    </div>
  </template>
```

children多的放进了staticrenderfns  其余的放进了render 至于为什么这么做,传送门
[从ast到render过程][2]


compileToFunctions来源
platform文件下 compiler文件 index.js文件

``` js
const { compile, compileToFunctions } = createCompiler(baseOptions)
```
createCompiler来源
core文件下 compiler文件 index.js文件

``` js

export const createCompiler = createCompilerCreator(function baseCompile (
  template,
  options
) {
  // 使用 parse 函数将模板解析为 AST
  const ast = parse(template.trim(), options)
 
  // ast对象进行优化，找出ast对象中所有静态子树
  if (options.optimize !== false) {
    optimize(ast, options)
  }
  // 根据给定的AST生成最终的目标平台的代码
  const code = generate(ast, options)
  return {
    ast,
    render: code.render,
    staticRenderFns: code.staticRenderFns
  }
})
```
createCompilerCreator来源
creact-compiler.js下

``` js
export function createCompilerCreator (baseCompile) {
  return function createCompiler (baseOptions) {
    function compile (
      template,
      options
    ) {
      const finalOptions = Object.create(baseOptions)
    // 执行createCompilerCreator里传来的函数 生生ast等
      const compiled = baseCompile(template, finalOptions)
      compiled.errors = errors
      compiled.tips = tips
      return compiled
    }

    return {
      compile,
      compileToFunctions: createCompileToFunctionFn(compile)
    }
  }
}
```
compiled如下即

``` js
 return {
    ast,
    render: code.render,
    staticRenderFns: code.staticRenderFns
  }
compiled.errors = errors
compiled.tips = tips

```
![图片描述][3]


总之,在$mount原型函数里面$mount
给 vue.options 挂载了 render和staticRenderFns
options.render = render
options.staticRenderFns = staticRenderFns


打印 options

![图片描述][4]


挂在后需要渲染

lifecycle.js

``` js
mountComponent(){
   callHook(vm, 'beforeMount')
   updateComponent = () => {
      vm._update(vm._render(), hydrating)
   }
   // 把更新组件加入依赖
   new Watcher(vm, updateComponent, noop, {
    before () {
      if (vm._isMounted) {
        callHook(vm, 'beforeUpdate')
      }
    }
  }, true /* isRenderWatcher */)
}
```
vm._render()生成传说中的VNode,即虚拟dom

vm._render()执行函数结果


![图片描述][5]


vm._update方法的实现

``` js
 Vue.prototype._update = function (vnode, hydrating) {
   // 判断vnode是否初始化过
    if (!prevVnode) {
      // initial render
      vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */)
    } else {
      // updates
      vm.$el = vm.__patch__(prevVnode, vnode)
    }
  }

```
通过__patch__更新

在 platform/web/runtime/index.js文件里执行了mountComponent方法

``` js
    import { mountComponent } from 'core/instance/lifecycle'
    
  
    Vue.prototype.$mount = function (
      el,
      hydrating
    ) {
      el = el && inBrowser ? query(el) : undefined
      // 调用mountComponent方法
      return mountComponent(this, el, hydrating)
    }
```

最终效果图
![图片描述][6]


  [1]: https://image-static.segmentfault.com/102/060/1020605093-5b8e08d0e9d8d_articlex
  [2]: https://segmentfault.com/a/1190000016277098
  [3]: https://image-static.segmentfault.com/123/522/1235224945-5b8e0acd56e86_articlex
  [4]: https://image-static.segmentfault.com/369/019/3690199183-5b8e0b0d7bef3_articlex
  [5]: https://image-static.segmentfault.com/968/144/968144021-5b8e0bb566f28_articlex
  [6]: https://image-static.segmentfault.com/199/397/1993972749-5b8e0cad9f60c_articlex