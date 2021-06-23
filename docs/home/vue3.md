[[toc]]
# 3.AST到VNode过程


``` HTML
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
```
options.render为

``` js
function anonymous( ) { 
    with(this){
      return _c(
        'div',
        [_c('h1',{staticStyle:{"color":"red"}},[_v("我是选项模板3")])
        ,_v(" "),_c('p',[_v(_s(number))]),
        _v(" "),
        _c('p',[_v(_s(message))]),
        _v(" "),
        _m(0)]
        )
    } 
  }

```
对应

     <h1 style="color:red">我是选项模板3</h1>
      <p>{{number}}</p>
      <p>{{message}}</p>
      
options.staticRenderFns为 [0] 
``` js
 function anonymous() {
   with(this){
     return _c('div',[
              _c('div',[_v("\n          1\n          "),
                _c('div',[_v("11")]),_v(" "),
                _c('div',[_v("12")])
              ]),_v(" "),
              _c('div',[_v("2")]),_v(" "),
              _c('div',[_v("3")]),_v(" "),
              _c('div',[_v("4")]),_v(" "),
              _c('div',[_v("5")])
            ])
    }
  }
```
对应的template为

``` js
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
```
render-helpers 下 index.js
``` js   
    export function installRenderHelpers (target) {
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
    }
```
render.js 
``` js   
    function renderMixin (Vue) {
       //把_v,_m等方法挂载到vue原型上
     installRenderHelpers(Vue.prototype)
    }
    function initRender (vm) {
     vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false)
    }
```
这样this._c就是执行createElement
this._m就是执行renderStatic
_v就是执行createTextVNode

在vdom下create-element.js
``` js   

    tag  // 标签
    data    // 关于这个节点的data值，包括attrs,style,hook等
    children  // 子vdom节点
    context  // vue实例对象
    function _createElement(context,tag,data,children,normalizationType) {
      vnode = new VNode(
            tag, data, children,
            undefined, undefined, context
      )
     return vnode
    }
```

创建vdom对象

vnode

``` js
class VNode {
    constructor (
    tag,
    data,     // 关于这个节点的data值，包括attrs,style,hook等
    children, // 子vdom节点
    text,     // 文本内容
    elm,      // 真实的dom节点
    context,  // 创建这个vdom的上下文
    componentOptions,
    asyncFactory
  ) {
    this.tag = tag
    this.data = data
    this.children = children
    this.text = text
    this.elm = elm
    this.ns = undefined
    this.context = context
    this.fnContext = undefined
    this.fnOptions = undefined
    this.fnScopeId = undefined
    this.key = data && data.key
    this.componentOptions = componentOptions
    this.componentInstance = undefined
    this.parent = undefined
    this.raw = false
    this.isStatic = false
    this.isRootInsert = true
    this.isComment = false
    this.isCloned = false
    this.isOnce = false
    this.asyncFactory = asyncFactory
    this.asyncMeta = undefined
    this.isAsyncPlaceholder = false
  }
}

```
下面dom分别依次执行,先渲染里面然后再渲染外层
```js
    1.[_c('h1',{staticStyle:{"color":"red"}},[_v("我是选项模板3")])
      <h1 style="color:red">我是选项模板3</h1>
    2._c('p',[_v(_s(number))]
     <p>{{number}}</p>
    3._c('p',[_v(_s(message))])
    <p>{{message}}</p>
    4._m(0)是缓存渲染数
    function anonymous() {
       with(this){
         return _c('div',[
                  _c('div',[_v("\n          1\n          "),
                    _c('div',[_v("11")]),_v(" "),
                    _c('div',[_v("12")])
                  ]),_v(" "),
                  _c('div',[_v("2")]),_v(" "),
                  _c('div',[_v("3")]),_v(" "),
                  _c('div',[_v("4")]),_v(" "),
                  _c('div',[_v("5")])
                ])
        }
      }
    
    vdom顺序依次为
    (1)<div>11</div>  
    (2)<div>12</div> 
    (3)<div> 
         1
         <div>11</div>  
         <div>12</div>  
       </div>
    (4)<div>2</div>
    (5)<div>3</div>
    (6)<div>4</div>
    (7)<div>5</div>
    (8)<div>
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
    (9)<div>
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
```
        
![图片描述][1]


![图片描述][2]

最后一次执行最外面的
![图片描述][3]


render-static.js给m[0]静态树做了缓存处理

``` js
renderStatic (
  index,
  isInFor
) {
//缓存处理
 const cached = this._staticTrees || (this._staticTrees = [])
// staticRenderFns被执行
tree = cached[index] = this.$options.staticRenderFns[index].call(
    this._renderProxy, // 代理可以理解为vue实例对象,多了一些提示处理
    null,
    this // for render fns generated for functional component templates
)
markStatic(tree, `__static__${index}`, false)
return tree
}

function markStatic (
  tree,
  key,
  isOnce
) {
function markStaticNode (node, key, isOnce) {
  node.isStatic = true  //静态树为true
  node.key = key  // `__static__${index}` 标志
  node.isOnce = isOnce // 是否是v-once
}
markStaticNode(node, key, isOnce)
}
```
然后又重新执行
``` js
    function anonymous(
    ) {
    with(this){return _c('div',[_c('div',[_v("\n          1\n          "),_c('div',[_v("11")]),_v(" "),_c('div',[_v("12")])]),_v(" "),_c('div',[_v("2")]),_v(" "),_c('div',[_v("3")]),_v(" "),_c('div',[_v("4")]),_v(" "),_c('div',[_v("5")])])}
    }
 ```   
 打印render()结果 
静态树有了isStatic和key值
 ![图片描述][4]
    


  [1]: https://image-static.segmentfault.com/928/319/928319458-5b8f5d6f2bfb2_articlex
  [2]:https://image-static.segmentfault.com/413/588/4135884249-5b8f5d7901e4e_articlex
  [3]: https://image-static.segmentfault.com/292/392/292392337-5b8f5d8298809_articlex
  [4]: https://image-static.segmentfault.com/154/407/1544078691-5b8f5dffd8b18_articlex
**补充个小问题 render函数里number 还是变量是什么时候变成数字的**
因为function有个with(this){}改变了作用域,当this.render()执行时候,那么this就是指得vue, $options.data 已经通过defineProperty代理到了vue下,访问this.data就是访问$options.data 所以number就是数字啦 