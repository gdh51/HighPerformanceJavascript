# AlgorithmsAndFlowControl(算法和流程控制)

## 循环

### 循环的类型
+ `for`循环：由初始化、前测条件、后执行体、循环体组成。先初始化代码然后进入前测条件，达到条件时运行循环体
+ `while`循环：由前测条件和循环体构成。
+ `do-while`循环：由循环体和后测条件组成。
+ `for-in`循环、`for-of`循环

### 循环性能
4种循环类型中，只有`for-in`比其他的明显较慢，因为每次迭代操作要同时搜索实例或原型属性。
因此，除非明确需要迭代一个属性**数量未知**的对象，否则应避免使用`for-in`类型循环

其他循环类型与性能无关，可以根据需求而非性能来选择，以下有两个因素选择：
+ 每次迭代处理的事务
+ 迭代的次数
减少其他一个或全部的时间，就可以提升循环的性能。

#### 减少迭代工作量
限制循环中耗时操作的数量。
1. 减少对象成员及数组项的查找次数

2. 使用倒叙循环也有一定的性能优化，因为通过倒叙循环时，只与0比较，比较一次，其他时候两次(倒叙循环：是否为`true`?  其他循环：是否满足条件?是否为`true?`)
   倒叙循环例子：
   ```js
   for (var i = items.length; i--; ){
     process(items[i]);
   }
   ```
   Note：当循环复杂度为O(n)时，减少每次迭代的工作量是最有效的方法;当复杂度大于O(n)，建议着重减少迭代次数

#### 减少迭代次数
Duff's Device：一种循环展开技术，一次迭代执行多次迭代的操作
基本理念：每次循环最多可调用9次`process()`。循环的迭代次数为总数除以8。不能被8整除的余数单独调用一次8次的循环(当然只会执行余数次循环)

原版Duff
```js
var iterators = Math.floor(items.length / 8),
startAt = items.length % 8,
i = 0;

do{
  switch(startAt){
    case 0: process([i++]);
    case 7: process([i++]);
    case 6: process([i++]);
    case 5: process([i++]);
    case 4: process([i++]);
    case 3: process([i++]);
    case 2: process([i++]);
    case 1: process([i++]);
  }
  startAt = 0;
}while (iterators--);
```

改良版Duff
```js
var i = items.length % 8;
while(i){
  process(items[i--]);
}

i = Math.floor(items.length / 8);

while(i){
  process(items[i--]);
  process(items[i--]);
  process(items[i--]);
  process(items[i--]);
  process(items[i--]);
  process(items[i--]);
  process(items[i--]);
  process(items[i--]);
}
```

建议在迭代次数大于1000次使用

### 基于函数的迭代
原生数组的`forEach`等迭代方法要快于循环体

## 条件语句

### if—else与switch对比
基于测试条件的数量判断：较多用`switch`,较少用`if-else`,两者主要区别是易读性,但其实`switch`大多数情况比`if-else`更快,特别是情况更多时.

### 优化if-else
目标：最小化到达正确分支前所需判断的条件数量
+ 判断条件按 大概率先小概率后 的序列排序
+ 嵌套裁分大块的`if-else`语句

### 查找表
当有大量离散值需要测试时，`if-else`与`switch`比使用查找表慢很多。javascript可以用数组或对象模拟查找表(条件语句过多时)

当单个键和单个值之间存在逻辑映射时，查找表具有很大的优势。

## 递归
可能遇到的问题
1. 终止条件不明确或缺少终止条件导致函数长时间运行
2. 调用栈大小限制

### 调用栈限制
javascript支持递归的数量和调用栈大小有直接关系。
由该引发的错误可以被捕获。

### 递归模式
1. 直接递归模式
```js
function recurse(){
  recurse();
}
```
2. 隐伏模式
```js
function first(){
  second();
}

function second(){
  first();
}

first();
```

定位模式错误的第一步是**验证终止条件**。

### 迭代
任何递归能实现的算法同样可以用迭代来实现。优化后的循环替代长时间运行的递归函数可以提升性能，因为运行一个循环比反复调用一个函数的开销要少得多。这是一种避免栈溢出错误的方法。

### Memoization
减少工作量是最好的优化技术。这种方法缓存前一个计算结果供后续计算使用，避免重复工作，例如：
```js
function factorial(n){
  if(n == 1)return 1;
  return n*factorial(n-1);
}
let fact6 = factorial(6);
let fact5 = factorial(5);
let fact4 = factorial(4);
```
当计算5 4 时，实际上重复计算了 2 3次同样的阶乘

利用构造函数来缓存结果,改写`factorial`函数：
```js
function memfactorial(n){
  if(!memfactorial.cache){//没有时创建一个缓存属性
    memfactorial.cache = {
      '0': 1,
      '1': 1
    }
  }

  if(!memfactorial.cache[n]){//没有该值时，利用递归生成该值与之前的值
    memfactorial.cache[n] = n * memfactorial(n);
  }

  return memfactorial.cache[n];
}
```

封装一个利用闭包缓存的普遍适用包装函数：
```js
function memoize(fundamental, cache){
  cache = cache || {};

  return function(arg){
    if(!cache[arg]){
      cache[arg] = fundamental(arg);
    }
    return cache[arg];
  }
}
```
以上的方法比定制的缓存函数性能差一些
