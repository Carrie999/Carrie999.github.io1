[[toc]]
# 谈一谈节流和防抖

代码只有敲一遍看到结果才会印象深刻，so实践一遍吧
``` html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <style>
    #box{
      width: 100%;
      height: 300px;
      background: pink;
    }
  </style>
</head>
<body>

  <div id="box"></div>  
  <script>

  function throFun(){
    console.log('执行真正的函数')
  }
    // 这里可以点击粉色区域，更换函数debounce感受一下不同
  var thro = throttle (throFun,1000);

  box.addEventListener('click' , function(){
    thro();
  })

  //函数节流
  function throttle (func,delay){
    var prev = Date.now();
    return function(){
      console.log('被触发')
        var context = this;
        var args = arguments;
        var now = Date.now();
        if(now-prev>=delay){
            func.apply(context,args);
            prev = Date.now();
        }
    }
  }
  // 函数节流给人的感受是上次执行时间和第二次执行时间相隔一定秒。否则不会被执行真正的函数
  // 上面函数返回是一个闭包，prev不会被清除掉，并且每次prev都会更新为最后一次执行的时间
  // 要么不会执行，要么就立即执行不会等待
  

    
  //函数防抖
  function debounce(method,time){
    var timer = null;
    return function(){
      console.log('被触发')
      var context = this;
      //在函数执行的时候先清除timer定时器;
      
      console.log(timer)
      clearTimeout(timer);
      timer = setTimeout(function(){
        method.call(context);
      },time);
    }
  }
  //防抖给人的感受是执行后等待一定时间才会被执行
  //如果在等待的时间内又被触发，以最后的为准 因为上次的被clear掉了
  //总之一定会等待规定时间秒才会被执行
  

  </script>
</body>
</html>
```
