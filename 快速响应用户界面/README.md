# ResponsiveInterfaces(快速响应用户界面)

## 浏览器UI线程
用于执行javascript和更新用户界面的进程通常被称为*浏览器UI线程*

UI线程的工作基于一个简单的队列系统，任务会被保存到队列中直到进程空闲。一旦空闲，队列中的下一个任务就被保存到队列中直到进程空闲。一旦空闲，队列中的下一个任务就被重新提取出来并运行。这些任务要么是运行javascript代码，要么执行UI更新(回流和重绘)。

### 浏览器限制
浏览器限制了javascript任务的运行时间，分为：
+ 调用栈大小限制
+ 长时间运行脚本限制：浏览器记录脚本的运行时间，并在达到一定限度时终止它

### 多久才算长
对于javascript：单个javascript操作不超过100ms

有时浏览器在javascript运行时不会把UI更新任务加入队列：如果在某些javascript代码运行时点击按钮，浏览器可能不会把重绘的按钮按下状态的任务或点击按钮启动的新javascript任务加入队列，最终结果就是一个失去响应的UI

有时当脚本执行时，UI不随用户交互而更新，但交互引发的javascript任务被加入队列

## 使用定时器让出时间片段
无法完全保证一些复杂的任务在100ms内完成。这个时候，最理想的方法是让出UI线程的控制权，使UI可以更新———停止执行javascript，之后在执行

### 定时器基础
定时器与UI线程交互的方式有助于把运行时间较长的脚本拆分为较短的片段。调用setTimeout()或setInterval()会告诉javascript引擎先等待一定时间，然后添加一个javascript任务到UI队列。

创建一个定时器会造成UI线程暂停，如同它从一个任务切换到下一个任务。所有定时器代码会重置所有相关的浏览器限制，包括长时间运行脚本定时器。此外，调用栈页在定时器的代码中重置为0.5

Note：setInterval与setTimeout的区别在于如果UI队列中已经存在由用一个setInterval创建的任务，那么后续任务不会被添加到UI队列直接跳过。

### 定时器的精度
在window系统中定时器分辨率为15ms，意味着一个延时15ms的定时器根据最后一次系统时间刷新而转换为0或15.设置定时器延时小于15将导致IE锁定，所以延时的最小值建议为25ms(实际为15或30)以确保至少有15ms延时。
(**实际上浏览器最低定时器延时为40ms,即使你设置了比它小的数值**)

### 利用定时器处理数组
首先是否可用定时器来处理数组必须满足两个条件：
+ 处理过程是否必须同步
+ 数据是否必须按顺序处理

当都为否时可以使用定时器来处理数组：
```js
let todo = items.concat();  //克隆原数组

setTimeout(function test(){
  process(todo.shift());

  if(todo.length > 0){
    setTimeout(test, 40);
  }else{
    callback(items);
  }
})
```

这种方法的缺点就是处理数组的总时间被延长

### 分割任务
我们通常会把一个任务分解成一系列子任务。然后还可以使用上面的数组模式来处理。

### 记录代码运行时间
通过函数前后的new Date来记录时间

### 定时器与性能
间隔在1000ms或以上的低频率的重复定时器不会影响web应用的响应速度。但当多个重复定时器使用较高频率(100ms~200ms)时，响应变得不及时

所以建议使用一个独立的重复定时器执行多个操作。

## Web Workers
Web Workers没有绑定UI线程，所以不能访问浏览器的某些资源(如DOM)

Worker运行环境由如下组成：
+ 一个navigator对象，包括4个属性：app.Name,appVersion,userAgent,platform
+ 一个location对象，同window.location,属性为只读
+ 一个self对象，指向全局worker对象
+ 一个importScripts(),用来加载Worker所用到的外部javascript文件
+ 所有的ES对象，如Object,Array...
+ XMLHttpRequest构造器
+ setInterval与setTimeout
+ close()方法用于立即停止Worker

由于Web Worker有着不同的全局运行环境，所以必须创建一个独立的javascript文件并在实例化Worker时引入该文件的URL。
```js
let worker = new Worker('code.js');
```
代码一旦执行，将为该文件创建一个新的线程和一个新的Worker运行环境。该文件会被异步下载，直到文件下载并执行完成后才会启动此Worker。

### 与Worker通信
页面代码可以通过postMessage()方法给Worker传递数据，它接收一个参数，即需要传递给Worker的数据。
处于javascript文件的Worker则需要通过message事件来监听页面传递的数据。若要通过Worker给页面传递数据，同样也要在页面注册该事件。
```js
let worker = new Worker('code.js');
worker.onmessage = function (event){
  //event.data中保存则传递过来的数据
};
worker.postMessage('A new Message');
```
Note：虽然可以传递各种类型的数据，但由于浏览器兼容性问题，建议序列化为字符串后传递。

### 加载外部文件
Worker通过 importScripts()方法加载外部javascript文件，该方法接收一个或多个javascript文件URL作为参数。importScript()的调用是阻塞式的，直到所有文件加载并执行完成后，脚本才会继续运行，但这种阻塞并不会阻塞UI响应。

### 实际应用
适用于处理纯数据，或者与浏览器UI无关的长时间运行脚本。例如：
+ 编码/解码大字符串
+ 复杂数学运算(包括图像或视频处理)
+ 大数组排序

任何超过100ms的处理都可以考虑下，当然前提是支持worker