[[toc]]
# 前端api笔记整理合集

 - 获取高度以及其他属性
``` javascript
   getBoundingClientRect()   offsetLeft 
   window.getComputedStyle(test);   currentStyle  getPropertyValue("background-color")  oStyle.getAttribute("backgroundColor"));
   document.createTextNode("CLICK ME");
   
  var test = document.querySelector('.test')
  var oStyle = test.currentStyle? test.currentStyle : window.getComputedStyle(test, null);
  console.log(oStyle.height)
  // console.log(oStyle.getPropertyValue('background-color'))
  if (oStyle.getPropertyValue) {
    console.log("getPropertyValue下背景色：" + oStyle.getPropertyValue("background-color"));
  } else {
    console.log("getAttribute下背景色：" + oStyle.getAttribute("backgroundColor"));
  }
```


- new 的时候发生了什么？

```  javascript
var obj = new Base();
var obj  = {};
obj.__proto__ = Base.prototype;
Base.call(obj);
```


- 单例模式私有变量，私有方法

```  javascript
var single=function (argument) {
    var privateVar=10;
    function privateFun () {
        //私有方法函数
    }
    return {
        publicProerty:true,
        //返回的共有方法可访问私有变量
        publicMethod:function () {
            privateVar++;
            return privateFun()
        }
    }
}
```

闭包的私有变量

```  javascript
function Foo(){
    var n=10;
    this.returnN=function(){
        return n;
    };
};
var newfoo=new Foo();
console.log(newfoo.returnN())//10
```

es6的私有变量

```  javascript
class Foo {
  #a;
  #b;
  #sum() { return #a + #b; }
  printSum() { console.log(#sum()); }
  constructor(a, b) { #a = a; #b = b; }
}
```

判断是否是数组    

```  javascript
Array.isArray([])
[] instanceOf Array
({}).toString.call([]) === '[object Array]'
[].constructor === Array
```


object.create()

```  javascript
object.create = function cc(o){
var F = function(){}
F.prototype = o;
return new F();
}
a2 = object.create(a1) 
a2.__proto__ = a1
```

js  判断是否是继承链上啊

```  javascript
function C(){} 
function D(){} 
var o = new C();
o instanceof C;
instanceof  //不能判断基本类型
null undefined  constructor //报错
Object.prototype.toString.call(null); //万金油判断类型
```


- documnet.ready
  
    ```  javascript
     事件的触发，表示文档结构已经加载完成（不包含图片等非文字媒体文件）。
    ```

- window.onload  
    ```  javascript
    事件的触发，表示页面包含图片等文件在内的所有元素都加载完成。
    ```

- AMD 和 CMD区别

```  javascript
1、AMD推崇依赖前置，在定义模块的时候就要声明其依赖的模块 
2、CMD推崇就近依赖，只有在用到某个模块的时候再去require
```




```  javascript
word-break:break-all;
word-wrap:break-word;
```

    
    线程是CPU调用（执行任务）的最小单位。
    进程是CPU分配资源和调度的单位。

- Boolean(value) 

```  javascript
把给定的值转换成Boolean型。 
当要转换的值是至少有一个字符的字符串、非0数字或对象时，Boolean()函数将返回true。如果该值是空字符串、数字0、undefined或null,它将返回false。
```
- bind 方法 
    > 将第一个参数设置为函数执行的上下文，其他参数依次传递给调用方法（函数的主体本身不执行，可以看成是延迟执行），并动态创建返回一个新的函数， 这符合柯里化特点。
    
    ```  javascript
    var foo = {x: 888};
    var bar = function () {
        console.log(this.x);
    }.bind(foo);               // 绑定
    bar();                     // 888
    ```
- bind的原生实现
```  javascript
Function.prototype.testBind = function(that){
        var _this = this,
            slice = Array.prototype.slice,
            args = slice.apply(arguments,[1]),
            fNOP = function () {},
            bound = function(){
                //这里的this指的是调用时候的环境
                return _this.apply(this instanceof  fNOP ?　this : that||window,
                    args.concat(Array.prototype.slice.apply(arguments,[0]))
                )
            }    
        fNOP.prototype = _this.prototype;
    
        bound.prototype = new fNOP();
    
        return bound;
    }
```
js模块

```  javascript
//CommonJS
exports.add = function(a, b) {
  return a + b;
}
var math = require('math');
math.add(2, 3); // 5
//缺点 同步加载假死
```
- AMD
```   javascript
// RequireJS 浏览器端模块化开发的目的
// AMD ”Asynchronous Module Definition”
// math.js
define(function() {
  var add = function(x, y) {
    return x + y;
  }

  return  {
    add: add
  }
})

require([module], callback);
require(['math'], function(math) {
  math.add(2, 3);
})
//特点 说AMD是依赖前置的
//代码在一旦运行到此处，能立即知晓依赖。
//而无需遍历整个函数体找到它的依赖，
//因此性能有所提升，缺点就是开发者必须显式得指明依赖——
//这会使得开发工作量变大，
//比如：当你写到函数体内部几百上千行的时候
//，忽然发现需要增加一个依赖，你不得不回到函数顶端
//来将这个依赖添加进数组。
``` 
- CMD

```  javascript
// Seajs
define(function(require, exports, module) {
  var a = require('./a');
  a.doSomething();
  var b = require('./b');
  b.doSomething();
})
//特点 依赖就近
// 这是一种牺牲性能来换取更多开发便利的方法。
```
- UMD  兼容
    
- ES6

```  javascript
// 导出变量
export var a = 1;
export var b = 2;
export var c = 3;
export default c;
// 导出变量
var a = 1;
var b = 2;
var c = 3;
export {a, b, c}
// 导出函数
function foo(){}
function bar(){}
export {foo, bar as bar2}

import * as m from './foo';
```

- filter去重

```  javascript
[1,2,3,1,'a',1,'a'].filter(function(ele,index,array){
    return index===array.indexOf(ele)
})
```
原生js ajax

```  javascript
//步骤一:创建异步对象
var ajax = new XMLHttpRequest();
//步骤二:设置请求的url参数,参数一是请求的类型,参数二是请求的url,可以带参数,动态的传递参数starName到服务端
ajax.open('get','getStar.php?starName='+name);
//步骤三:发送请求
ajax.send();
//步骤四:注册事件 onreadystatechange 状态改变就会调用
ajax.onreadystatechange = function () {
   if (ajax.readyState==4 &&ajax.status==200) {
    //步骤五 如果能够进到这个判断 说明 数据 完美的回来了,并且请求的页面是存在的
　　　　console.log(xml.responseText);//输入相应的内容
  　　}
}
//post
var xhr = new XMLHttpRequest();
//设置请求的类型及url
//post请求一定要添加请求头才行不然会报错
xhr.setRequestHeader("Content-type","application/x-www-form-urlencoded");
 xhr.open('post', '02.post.php' );
//发送请求
xhr.send('name=fox&age=18');
xhr.onreadystatechange = function () {
    // 这步为判断服务器是否正确响应
  if (xhr.readyState == 4 && xhr.status == 200) {
    console.log(xhr.responseText);
  } 
};
// readyState的几种状态：
// 0：初始化，XMLHttpRequest对象还没有完成初始化
// 1：载入，XMLHttpRequest对象开始发送请求
// 2：载入完成，XMLHttpRequest对象的请求发送完成，已收到全部响应内容但尚未解析
// 3：解析，XMLHttpRequest对象开始解析服务器的响应内容
// 4：完成，XMLHttpRequest对象读取服务器响应结束

IE浏览器
xmlHttp=new ActiveXObject("Msxml2.XMLHTTP");
```
- 函数防抖

```  javascript
// 防抖
var canRun = true;
document.getElementById("throttle").onscroll = function(){
    if(!canRun){
        // 判断是否已空闲，如果在执行中，则直接return
        return;
    }

    canRun = false;
    setTimeout(function(){
        console.log("函数节流");
        canRun = true;
    }, 300);
};
// 将若干个函数调用合成一次，并在给定时间过去之后仅被调用一次。
```
- 节流

```  javascript
// 节流函数允许一个函数在规定的时间内只执行一次。
// 它和防抖动最大的区别就是，节流函数不管事件触发有多频繁，都会保证在规定时间内一定会执行一次真正的事件处理函数。
var throttle = function(func,delay){
    var prev = Date.now();
    return function(){
        var context = this;
        var args = arguments;
        var now = Date.now();
        if(now-prev>=delay){
            func.apply(context,args);
            prev = Date.now();
        }
    }
}
```

HTTP1.0 和 HTTP2.0
```  javascript
//HTTP1.0与HTTP 1.1的主要区别 
长连接
节约带宽
HOST域
管道机制
http1.1所有的数据通信是按次序进行的
// http1.0的缺陷
每个请求都需单独建立连接（keep-alive能解决部分问题单不能交叉推送）
每个请求和响应都需要完整的头信息


// http2的优势
多路复用（SDPY协议是HTTP2的基础，它的核心思想是多路复用，仅使用一个连接链传输一个网页中的众多资源：）
压缩头信息（根据资源请求的特性和优先级，调整这些资源请求的优先级。）
请求划分优先级
支持服务器端主动推送 （一方面使用gzip或compress压缩后再发送；另一方面，客户端和服务器
时维护一张头信息表，所有字段都会存入这个表，生成一个索引号，以后就不发送同样字段了，只发送索引号）
而HTTP2的基本协议单位为二进制帧流
SDPY必须建立在SSL协议之上  必须使用https

//Connection: keep-alive。
TCP连接默认不关闭
``` 
- TCP与UDP基本区别
```  javascript
  1.基于连接与无连接
  2.TCP要求系统资源较多，UDP较少； 
  3.UDP程序结构较简单 
  4.流模式（TCP）与数据报模式(UDP); 
  5.TCP保证数据正确性，UDP可能丢包 
  6.TCP保证数据顺序，UDP不保证 
　　
UDP应用场景：
  1.面向数据报方式
  2.网络数据大多为短消息 
  3.拥有大量Client
  4.对数据安全性无特殊要求
  5.网络负担非常重，但对响应速度要求高
```

```  javascript
dns 预取技术 tcp预连接
UI线程是brower线程中的主线程
主线程的目的是保持用户界面的高度相应
```


webpack

```  javascript
webpack 的初衷，require everything, bundle everything.
Gulp 的定位是 Task Runner, 就是用来跑一个一个任务的。
```

数组去重

```  javascript
// ES6
function unique (arr) {
  const seen = new Map()
  return arr.filter((a) => !seen.has(a) && seen.set(a, 1))
}
// or
function unique (arr) {
  return Array.from(new Set(arr))
}
// 普通

function unique(array){ 
    var n = []; //一个新的临时数组 
    
    for(var i = 0; i < array.length; i++){ 
        if (n.indexOf(array[i]) == -1) n.push(array[i]); 
    } 
    return n; 
}
```

原型

```  javascript
function A(){};  
var a = new A();
console.log(A.prototype.constructor==A)
console.log(a.constructor == A)
console.log(a.__proto__ == A.prototype)
console.log(A.__proto__ == Function.prototype)
```


##### reduce
```  javascript
arr.reduce((a,b)=>a+b)
// 求和
console.log(eval(arr.join("+")))
// 求和
arr = [1,2,3,4,5]
cc = arr.reduce(sum,0)
function sum(a,b,index,arr){
  console.log(a,b,index,arr)
  return a+b
}
console.log(cc)
// 1 2 1 [1, 2, 3, 4, 5]
// 3 3 2 [1, 2, 3, 4, 5]
// 6 4 3 [1, 2, 3, 4, 5]
// 10 5 4 [1, 2, 3, 4, 5]
// 15
```

##### sort

```  javascript
arr.sort((a,b)=>a-b)

// 由小到大的排序
var arr = [4,3,6,5,7,2,1];
arr.sort();
arr.sort(function(a,b){
    return a-b;
});
console.log(arr);
//输出结果 [1, 2, 3, 4, 5, 6, 7]
// sort的运算结果是sort后然后比较大小 如果a-b<1 那么 不动如果大于1 那么交换顺序 
// a-b是从小到大
// b-a是从大到小
```
##### 冒泡排序

```  javascript
arr=[6,2,14,15,12,47,26,38,1,3,9,5,4,8,54,23,16,17,18,19,20,21]

for(let i=0; i<arr.length ; i++){
  for(let j=0;j<arr.length-i-1; j++){
    if(arr[j] > arr[j+1]){
      [ arr[j], arr[j+1] ] = [ arr[j+1] , arr[j] ]
    }
  }
}
```

##### Math.max
```  javascript
arr = [1,2,3]
Math.max(...arr)//3
Math.max.apply(null,arr)//3
```

```  javascript
##### 判断是否是回文
var gg = 'abccba'
function palindrome(text){
    return text.split('').reverse().join('') === text?'是回文':'不是回文'
}
console.log(palindrome(gg))
```
js翻转字符串

```  javascript
function reverseString(str) {  
    return (str === '') ? '' : reverseString(str.substr(1)) + str.charAt(0);  
} 
function reverseString(str){
    return str.split('').reverse().join('');
}
```

正则匹配数字
```  javascript
var test='abc345efgabcab'; 
uu = test.replace(/(\d)/g,'[$1]'); 
console.log(uu) // abc[3][4][5]efgabcab
```

递归算法

```  javascript
//二分查找
function binSearch(target, arr, start, end) {
  var start = start || 0; // 允许从什么位置开始,下标
  var end = end || arr.length - 1; // 什么位置结束,下标
  start >= end ? -1 : ''; // 没有找到,直接返回-1
  var mid = Math.floor((start + end) / 2); // 中位下标
  if (target == arr[mid]) {
    return mid; // 找到直接返回下标
  } else if (target > arr[mid]) {
    //目标值若是大于中位值,则下标往前走一位
    return binSearch(target, arr, start, mid - 1);
  } else {
    //若是目标值小于中位值,则下标往后退一位
    return binSearch(target, arr, mid + 1, end);
  }
}
```
js深拷贝

```  javascript
function deepClone(obj){
    var newObj = obj.constructor === Array ? []:{};
    if(typeof obj !== 'object'){
        return
    }else{
        for(var i in obj){
            if(obj.hasOwnProperty(i)){
                newObj[i] = typeof obj[i] === 'object'?deepClone(obj[i]):obj[i];
            }
        }
    }
    return newObj
}
```
自定义事件
 
```  javascript
  //自定义事件
  var box = document.getElementById('box')
  // var evt = new Event("look", {"bubbles":true, "cancelable":false});
  // box.addEventListener('look',function (argument) {
  //   console.log('我是look监听事件')
  // })
  var event = new CustomEvent("cat", {"detail":{"hazcheeseburger":true}})
  box.addEventListener("cat", function(e) { console.log(e.detail) })
  // box.dispatchEvent(event)
  setTimeout(()=>{
    box.dispatchEvent(event);
  },1000)
```
相对窗口 offsetTop  
相对视口 getBoundingClientRect()  dom

map和foreach的区别

```  javascript
arr = [1,2,3,4,5]
arr.forEach(function(element, index, array) {
  console.log(element, index)
});
cc = arr.map(function(element, index, array) {
  return element*2
})
var numbers = [1, 4, 9];
var roots = numbers.map(Math.sqrt);  [1, 2, 3];
console.log(cc) // [2, 4, 6, 8, 10]
// Map和forEach的区别
// 1、map速度比foreach快
// 2、map会返回一个新数组，不对原数组产生影响,foreach不会产生新数组，foreach返回undefined
// 3、map因为返回数组所以可以链式操作，foreach不能
// 4, map里可以用return ,而foreach里用return不起作用，foreach不能用break，会直接报错
```

正则匹配金额

```  javascript
  function toS(num){
        let reg =/\B(?=(\d{3})+\b)/g; //B表示匹配非单词边界的元字符，而/b表示匹配单词边界
        console.log(num.replace(reg,","))
    }

    toS(10000000000+"");//1,000,000,000
```
js == 比较
```
如果一个值是Object，另一个是number或者string，会把Object利用 valueOf()或者toString()转换成原始类型再进行比较
在数值运算里，会优先调用valueOf()，如a + b；
在字符串运算里，会优先调用toString()，如alert(c)。
[]+[] toString()
""
[]-[]  先转成valueOf 然后转成number
0 
{}+{}  toString()
"[object Object][object Object]"
{}-{}  先转成valueOf 然后转成number
NaN
{}==0  先转成valueOf 然后转成number
console.log([0,1]=='0,1')
先调用valueOf  然后调用toString
```
https://www.cnblogs.com/huaan011/p/6634349.html

[标记清除](https://www.cnblogs.com/Leo_wl/p/3269590.html)
[树的遍历](https://blog.csdn.net/yimingsilence/article/details/54783208)

事件委托
```  javascript
<ul id="ul1">
    <li>111</li>
    <li>222</li>
    <li>333</li>
    <li>444</li>
  </ul>
  <script>
    window.onload = function(){
    　　var oUl = document.getElementById("ul1");
    　　oUl.onclick = function(ev){
           //父元素
          console.log(ev.currentTarget);
    　　　　var ev = ev || window.event;
    　　　　var target = ev.target || ev.srcElement;
    　　　　if(target.nodeName.toLowerCase() == 'li'){
    　　　　　　　  console.log(target.innerHTML);
    　　　　}
    　　}
    }
  </script> 
```


JS如何实现多重继承？

```  javascript
js转化abcDefGhi，返回：abc_def_ghi
function tranvert(str){
    reg = /([A-Z])+/g;
    return str.replace(reg,'_$1').toLocaleLowerCase()
}
console.log(tranvert('getYourName'))
```
 二分法
```  javascript
// 循环二分法
function bfind(array,target){
  let left = 0
  let right = array.length -1
  while(left <= right){
    mid = Math.floor((left + right)/2);
    if(target > array[mid]){
      left = mid+1;
      continue;
    }else if(target == array[mid]){
      return mid;
    }else {
      right = mid-1;
      continue;
    }
  }
}

//递归二分法
function bfind(array,target,left,right){
  var mid = Math.floor((left + right)/2);
  if(target > array[mid]){
    return bfind(array,target,mid+1,right)
  }else if(target < array[mid]){
    return bfind(array,target,left,mid-1)
  }else {
    return mid
  }
}
console.log(bfind([1,2,3,4,5,6],4,0,5))
```














