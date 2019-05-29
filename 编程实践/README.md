# ProgrammingPractices(编程实践)

## 避免双重求值
eval(),Function()构造函数,setTimeout(),setInterval() 四个方法可以允许在程序中提取一个包含代码的字符串来动态执行它。

当我们在javascript代码中执行另一段javascript代码时，会导致双重求值的性能消耗。

每次调用eval()都会创建一个新的解释器/编译器实例(4个方法都是)
Note：优化后的javascript引擎会户那次那些使用了eval且重复运行的代码，但并不是推荐你去用

## 使用Object/Array直接量
使用直接量可以减少代码量，减少文件尺寸，在有些浏览器中比其他方式运行更快。

## 避免重复工作
在浏览器兼容问题上，我们常常会根据不同浏览器而选择使用不同分支的方法。如不进行优化，我们将在每次执行时进行一次分支选择。如
```js
function chooseMethod(){
  if(conditionOne){
    //第一种情况
  }else{
    //第二种情况
  }
}
```

### 延迟加载
信息被使用前不做任何操作。在第一次检测时重写函数为合适的条件下的情况。例如：
```js
function chooseMethod(){
  if(conditionOne){

    chooseMethod = function(){
      //第一种情况的逻辑
    }

  }else{

    chooseMethod = function(){
      //第二种情况的逻辑
    }

  }
}
```
之后调用不会在执行 分支检测(重复的代码)
这种方式第一次耗时较长。适用于一个函数在页面不会立刻调用。

### 条件预加载
在脚本加载期间提前检测，而不会等到函数被调用
```js
let chooseMethod;

if(conditionOne){

  chooseMethod = function(){
    //第一种情况的逻辑
  }
}else{

  chooseMethod = function(){
    //第二种情况的逻辑
  }
}
```

这种方式在脚本加载时耗时，适合于一个函数马上要被使用且在整个页面生命周期中频繁出现。

## 函数节流(throttle)与函数防抖(debounce)
实质：减少代码的执行次数

### 函数防抖 (debounce)
如果一个事件被频繁触发多次，并且触发的时间间隔过短，则防抖函数可以使得对应的事件处理函数只执行最后触发的一次。 函数防抖可以把多个顺序的调用合并成一次。

#### 代码实现
假如有一个`window.resize`事件,用户每调整一点窗口大小都会触发一次,调整较大的幅度就会造成大幅度的性能损坏,我们可以将它改为只在最后的状态执行代码逻辑：
```js
window.addEventListener('resize', (function(){
  let timer = 0;//用于停止将要执行的定时器

  return function(){
   clearTimeout(timer);//停止定时器
   //设置新的定时器
   timer = setTimeout(function(){
     //代码逻辑
    }, 400);
  }
})(), false)
```

#### 应用
1. 手机号、邮箱输入检测
2. 搜索框搜索输入（只需最后一次输入完后，再进行Ajax请求）
3. 窗口大小`resize`（只需窗口调整完成后，计算窗口大小，防止重复渲染）
4. 滚动事件`scroll`（只需执行触发的最后一次滚动事件的处理程序）
5. 文本输入的验证（连续输入文字后发送 AJAX 请求进行验证，（停止输入后）验证一次就好

### 函数节流 (throttle)
如果一个事件被频繁触发多次，节流函数可以按照固定频率去执行对应的事件处理方法。 函数节流保证一个事件一定时间内只执行一次。

#### 代码实现
简单思想实现
```js
function throttle(interval) {
    let timer;

    return function () {
        if(!timer){
          setTimeout(() =>{
            //代码逻辑
          }, interval);
        }
    }
}
```

#### 应用
1. DOM元素的拖拽功能实现（mousemove）
2. 射击游戏的 `mousedown/keydown` 事件（单位时间只能发射一颗子弹）
3. 计算鼠标移动的距离（mousemove）
4. 搜索联想（keyup）
5. 滚动事件scroll，（只要页面滚动就会间隔一段时间判断一次）

## 使用速度快的部分
引擎通常是处理过程最快的部分，运行速度慢的实际上是你的代码，没错。

### 位操作
javascript中数字按照IEEE-754标准以64位格式存储。在位操作中，数字被转换为有符号32位格式。每次运算符会直接操作该32位数以得到结果。尽管需要转换，但与javascript其他数学运算和布尔操作相比要快很多。

*AND 按位与&*
  两个操作数对应位都是1时，在该位返回1

*OR 按位或|*
  两个操作数对应位有1时，在该位返回1

*XOR 按位异或^*
  两个操作数对应位只有一个为1时，在该位返回1

*NOT 按位取反~*
  遇0则返回1，遇1返回0(单个操作数)

代替纯数学运算：在求奇偶数时，可以用`i & 1`代替`i % 2`，可以提升效率

位掩码：用于处理同时存在多个布尔选项的情况。思路即使用单个数字的每一位来判断是否选项成立，从而有效地把数字转化为由布尔值标记组成的数组。掩码中的每个选项的值都等于2的幂。如：
```js
let OPTION_A = 1;// 1
let OPTION_B = 2;// 10
let OPTION_C = 4;// 100
let OPTION_D = 8;// 1000
let OPTION_E = 16;// 10000

let options = OPTION_A | OPTION_C | OPTION_D;//用按位或运算创建一个数字来包含多个设置选项 此处为1101

//通过按位与操作判断一个给定的选项是否可用。如果该选项未设置则运行结果为0，如果已设置则为1

//选项A是否在列表中
if(options & OPTION_A){
  //......
}

//选项B是否在列表中
if(options & OPTION_B){
  //.......
}
```

### 原生方法
原生方法最快  常见数学常量：

常量 | 值
-|-
Math.E | E的值，自然对数的底
Math.LN10 | 10的自然对数
Math.LN2 | 2的自然对数
Math.LOG2E | 以2为底E的对数
Math.LOG10E | 以10为底的E的对数
Math.PI | π常量值
Math.SQRT1_2 | 1/2的平方根
Math.SQRT2 | 2的平方根

方法 | 含义
-|-
Math.abs(num) | 返回num的绝对值
Math.exp(num) | 返回E的指数
Math.log(num) | 返回num的自然对数
Math.pow(num,power) | 返回num的power次幂
Math.sqrt(num) | 返回num的平方根
Math.acos(x) | 返回x的反余弦值
Math.asin(x) | 返回x的反正弦值
Math.atan(x) | 返回x的反正切值
Math.atan2(y,x) | 返回从X轴到(y,x)点的角度
Math.cos(x) | 返回x的余弦值
Math.sin(x) | 返回x的正弦值
Math.tan(x) | 返回x的正切值