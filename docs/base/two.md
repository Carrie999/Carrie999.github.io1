[[toc]]
# js reduce sort 排序

##### reduce
``` javascript
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
##### Math.max
```  javascript
arr = [1,2,3]
Math.max(...arr)//3
Math.max.apply(null,arr)//3
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
