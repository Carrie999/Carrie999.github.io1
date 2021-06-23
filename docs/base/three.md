[[toc]]
# 手写bind的 Polyfill

- 手写bind函数  Polyfill
- 来来来戳我[官方文档](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)

实现效果一

``` javascript
function foo(){
  this.b = 100;
  console.log(this.a)
  return this.a
}
var func = foo.bind({a:"1"})

func();  // 1 
new func(); // undefined
```
实现效果二
```  javascript
//函数科里化
function add(a, b, c) {
    var i = a+b+c;
    console.log(i);
    return i;
}

var func = add.bind(undefined, 100);//给add()传了第一个参数a
func(1, 2);//103，继续传入b和c

var func2 = func.bind(undefined, 200);//给func2传入第一个参数，也就是b，此前func已有参数a=100
func2(10);//310,继续传入c，100+200+10

``` 
官网Polyfill
```  javascript
if (!Function.prototype.bind) {
  Function.prototype.bind = function(oThis) {
  //这个最简单this必须是函数，不是就throw Error
    if (typeof this !== 'function') {
      throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
    }
    
    var aArgs   = Array.prototype.slice.call(arguments, 1),
        fToBind = this,
        fNOP    = function() {},
        fBound  = function() {
          return fToBind.apply(this instanceof fNOP
                 ? this
                 : oThis,
                 aArgs.concat(Array.prototype.slice.call(arguments)));
        };

    fNOP.prototype = this.prototype; 
    fBound.prototype = new fNOP();

    return fBound;
  };
}
```
第一步

```  javascript
 // 先清楚this是指谁
 foo.bind({a:'1'})
 Function.prototype.bind = function(oThis) {
     fToBind = this
 }
 // 这里 fToBind是指的调用bind的函数也就是foo
 // othis指的是bind传入的对象也可以是函数对象这是指的是{a:'1'}

```  javascript
第二步

```  javascript
var aArgs = Array.prototype.slice.call(arguments, 1)
//首先要明白arguments没有slice方法 要调用array原型上的slice才可以
//这里截的是从第二个参数开始 也就是数组下标为1到最后
var func = add.bind(undefined, 100);
// 此时 aArgs= [100]
同理类推 var func = add.bind(undefined, 100，200); 
//此时 aArgs= [100，200]
```
第三步

```  javascript
    fToBind =this;
    fNOP= function() {},
    fBound  = function() {
      return fToBind.apply(省略);
    };

    fNOP.prototype = this.prototype; 
    fBound.prototype = new fNOP();
    
    foo.bind({a:'1'})
    先清楚原型关系
    fNOP是一个函数，fToBind是foo
    fNOP.prototype = this.prototype; 
    // foo的原型赋值给fNOP的原型
    // 也就是说foo函数原型上的方法全都给了fNOP
    // fNOP能都调用foo函数原型上的方法和属性
    // fNOP在 foo.prototype的这条原型链上
    var dd = new fNOP() 
    dd.__proto__ == fNOP.prototype 成立
    dd.__proto__ == foo.prototype 成立
    dd instanceof fNOP 成立
    dd instanceof foo 成立
    但是 dd 获取不到foo本身的方法和属性
    
    fBound.prototype = new fNOP();
    fBound.prototype就是上面例子中的dd
    
    fBound又在fNOP的这条链上
    var ff = new fBound()
    ff.__proto__ == fBound.prototype 成立
    ff.__proto__.__proto__ == fNOP.prototype 成立
    ff instanceof fBound 成立
    fBound.prototype instanceof fNOP 成立
    ff instanceof fNOP 成立 
    //注意 fBound instanceof fNOP 不成立
    
```  javascript
第四步
最后

```  javascript
fBound  = function() {
    return fToBind.apply(this instanceof fNOP? this: oThis,
        aArgs.concat(Array.prototype.slice.call(arguments)));
};
return fBound;
上面说了 fToBind 就是 foo 
foo 调用了 apply
这里有必要解释一下apply
        //apply的作用和指向
        function Animal(name){
      this.name = 'Animal';
        }
        function Cat(name){
          this.name = 'Tom';
          Animal.apply(this); 
        }
        var cat = new Cat()
        console.log(cat.name) //Animal
        Animal.apply(this); 
        这里在Cat里的apply就是cat调用了Animal的属性和方法
        this全部指向Cat
        如果把上下位置互换就会被重写
        
        
        function Cat(name){
          Animal.apply(this); 
          this.name = 'Tom';
        }
        // 其实这句话就相当于
        function Cat(name){
          this.name = 'Animal';
          this.name = 'Tom';
        }
        
        // 参数传递
        function add(a,b,c){
          console.log(a)
          console.log(b)
          console.log(c)
        }
        function test(){
      add.apply(this,[10,1])
        }
        test();
        打印出  
        10 1 undefined
        // 从调用的第二个起往参数传值
        // 相当于解构赋值
        [a,b,c] = [10,1]
        a =10 
        b =1
        c = undefined
        // call就是单个，apply是数组 
        add.call(this,10,1)
        // apply和call调用的时候只是返回add里面的return
        // 如果add里面没有return就什么也不返回
        // 这个被面试官给坑了，说理解的不深（理解的不深
        // 就被面试官忽悠了，他说apply返回啥，我说返回对象不对不对函数。。。题外话）
// 理解了apply 回到正题
fBound  = function() {
    return fToBind.apply(this instanceof fNOP? this: oThis,
        aArgs.concat(Array.prototype.slice.call(arguments)));
};
//我少抄了一句话就是这句直接
//fBound  = fToBind.apply(this instanceof fNOP? this: oThis,
//aArgs.concat(Array.prototype.slice.call(arguments)));

return fBound;

// 为什么是返回 return 因为
function foo(){
  this.b = 100;
  console.log(this.a)
  return this.a
}
var func = foo.bind({a:"1"})
func();  // 1  
cc = func() 
console.log(cc) // 1


// 只有 return fToBind.apply()  cc才会取得 this.a
fToBind.apply(this instanceof fNOP? this: oThis,
        aArgs.concat(Array.prototype.slice.call(arguments)));

// 回到最初
function foo(){
  this.b = 100;
  console.log(this.a)
  return this.a
}
var func = foo.bind({a:"1"})

func();  // 1 
new func(); // undefined
// 上面说过 fBound又在fNOP的这条链上
var ff = new fBound()
ff.__proto__ == fBound.prototype 成立
ff.__proto__.__proto__ == fNOP.prototype 成立
ff instanceof fBound 成立
fBound.prototype instanceof fNOP 成立
ff instanceof fNOP 成立 
//ff就是new func()的结果
// bind最后返回了 fBound
// 如果单纯得调用func(); 那么this就是window
// window instanceof fNOP 不成立 apply 把this 
// 指向了oThis也就是{a:"1"}
// 所以this.a 返回的是1 
// 如果new func(); this就是指得这个对象 
//此时this 就是上面例子中的ff那么this instanceof fNOP 成立 

this指得是 new fBound()的this
fBound  = function() {
  return fToBind.apply(this instanceof fNOP
         ? this
         : oThis,
         aArgs.concat(Array.prototype.slice.call(arguments)));
};
// 就是指得是 fBound里面的this

在这里可以看作是apply继承
fBound 把foo里面所有的属性名和方法都继承了
所以（new func()).b 能够访问到
至于最后
aArgs.concat(Array.prototype.slice.call(arguments))
new func()可能传参
把bind第二个参数后面的和new func()传进来的参数打包成一个数组

```












