[[toc]]
# 今日头条和美团面试题面经分享

> 昨天，下着小雨去面试了，特地避开雨天，竟然还是雨天，宝宝不想说话。。。 正好2点到，先去字节跳动，一进去就是客厅沙发桌子，旁边有书架和书，摆设一般，一点都感受不出大公司的气质，去了一直等了30分钟，催了前台2次 后来给面试题，先答题，面试官不在旁边 额...这下可以上网搜答案了 不废话，放题
 
##  一· css和html
-   A元素垂直居中
-   A元素距离屏幕左右各边各10px
-   A元素里的文字font—size:20px,水平垂直居中
-   A元素的高度始终是A元素宽度的50% 
   <div class="box"> <div class="Abox">我是居中元素 </div> </div>

``` html{4}
*{
      padding:0;
      margin: 0;
    }
    html,body{
      width: 100%;
      height: 100%;
    }
    .box{
      position: relative;
      background: red;
      width: 100%;
      height: 100%;
    }
    .Abox{
      margin-left:10px;
      width: calc(100vw - 20px);
      height: calc(50vw - 10px);
      position: absolute;
      background: yellow;
      top:50%;
      transform: translateY(-50%);
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 20px;
    }
```

 
## 二· 函数arguments
> 函数中的arguments是数组吗？怎么转数组？

这个灰常简单 array.from ...arguments 他说就三个点吗 我又说加各括号或者[]...我就是表示一下下...


``` js{4}
function cc () { 
    console.log(Array.from(arguments))
    console.log([...arguments]) 
 }
```

##  三· 以下打印结果
   
``` js{4}
if([]==false){console.log(1)};
if({}==false){console.log(2)};
if([]){console.log(3)}
if([1]==[1]){console.log(4)}
```

这个也比较简单 1 3 学好== 和转换不成问题

1和2左右被转成数字   3被转成boolean   4 地址不一样
##  四· 以下打印结果

``` js{4}
async function async1(){
    console.log('async1 start')
    await async2()
    console.log('async1 end')
  }
  async function async2(){
    console.log('async2')
  }
  console.log('script start')
 setTimeout(function(){
    console.log('setTimeout') 
 },0)  
 async1();
 new promise(function(resolve){
    console.log('promise1')
    resolve();
 }).then(function(){
    console.log('promise2')
 })
 console.log('script end')
```

 这个也很简单 promise 优先于 setTimeout 微任务和宏任务 

script start
async1 start
async2
promise1
script end
promise2
async1 end
setTimeout
await等async 后面的加入异步不知道233333
 还有最新V8和旧版本V8展示结果不一样promise2 和async1 end略有互换

``` js{4}
async function asyncFunc() {    
    const result = await otherAsyncFunc();   
    console.log(result);
}
 // 等价于:
function asyncFunc() { 
   return otherAsyncFunc().then(result => {     
   console.log(result);   
 });}
```

##  五· 改动错误 
 此处省略太长就是this和let 的问题
这个只改了let 后来在提示下改了this 不过又被问住坑了
我说箭头函数没有this this只向外边
他说 没有this，this是从哪来的
我没回答，又问this是声明确定还是执行确定
我觉得this就是外面的这跟声明确定还是执行确定有什么关系
在面试官的一再引导下我竟然回答是执行确认，因为我觉得是执行的时候外面的
这真的是说法问题，其实this是继承来的，我只是忘了这一点。就被问懵了
这个面试官说话给人感觉很冲啊，让你经常怀疑自己，哼，大家心里要强大啊
天啊噜竟然能栽倒箭头函数上
##  六· 写bind 
这个bind我在网上看了好几遍
觉得自己懂了
手写代码其实有一部分上网搜了
少写了一行就被发现了，自己作死，然后就被问住了，233333333
七 ·函数节流（此题有坑）
从图上看大概就是100ms内阻止函数运行
觉得如此简单的问题竟然能被问住
我要好好研究研究
不就是节流防抖吗？
我发现好多问题明明知道一问就死
 八 ·从一个无序，不相等的数组中，选取N个数，使其和为M实现算法
这个我竟然看错题了 使其和为M实现算法看成求和，哈哈哈哈哈哈哈哈
其实即便不看错题，我手写也写不出来啊 
这个得在机器上试几边才能写出来啊

最后问了问几个项目问题， 如何提升说去看 你不知道的js 
（都知道一问也被问住了啊，有些事不面不知到啊） 
然后他说不知道让你过还是不过 哈哈哈哈哈哈哈哈哈哈
最后挂了，他说要好好看看vue源码
面试官长得很小只，他扣的很死，经常说的话：你确定你写的能执行？ 
我我我。。。心态不能输 ，能！！！
经过两个小时，面试卒，结束。
对比美团面试官，头条小哥哥很好。 
结束后就立即去美团了
也许是上次美团小哥哥给我的感觉太好，这个美团面试官素质是如此之低，是我见过最差面试官没有之一
这个人一来什么基础都没问，全是问项目
问项目也就算了，问的还都是项目安全问题，大概她做的是安全方面的
问加密算法 问https是怎么回事，真的安全吗？问dns解析怎么回事，dns劫持知道吗？
localhost如何不被篡改，对，全部是围绕安全来的
最后看了看简历，你工作才1年半啊。经验太少了，这个简历是你自己拿着，还是留着
我就不送了，自己出去吧，结果我连门卡都没有，出都出不去。
就这样10分钟把我打发，态度非常恶劣，既然如此你又何必让我来呢？
我这大老远的跑过来就给10分钟的时间。
天黑了，回家。
谢谢大家，技术没有顶峰，要保持一直不断的努力，学无止境，面试题不足以判断一个人的真实能力，大家一起加油ヾ(◍°∇°◍)ﾉﾞ



