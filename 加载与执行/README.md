# LoadingAndExecution(加载与执行)

## 脚本位置

首要规则：**将脚本放在底部**

浏览器在解析到`<body>`标签之前，不会渲染页面任何部分。把**脚本放在页面顶部将会导致明显的渲染延迟**，通常表现为显示空白页面。

`<script>`的**下载和解析**过程会阻塞`dom`的渲染，尽管一些浏览器允许并行加载`Javascript`文件。

所以推荐将所有`script`标签尽可能放到`<body>`标签底部

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>Document</title>
  <link rel='stylesheet' href='file2.css' type='text/css'>
</head>
<body>
  <script src='file1.js'></script>
</body>
</html>
```

## 组织脚本

总结:**减少页面的外链脚本数量可以改善性能**

浏览器每遇到一个`script`标签都会初始下载并阻塞页面渲染，减少`script`标签数量有助于改善该情况。

一般情况下，浏览器遇到`<link>`标签引用的`css`脚本时，会异步下载该`css`脚本，然后待`dom`解析完毕后，在进行样式计算，但如果遇到下面的情况则不同。

**Note：将*脚本*放在引用*外链样式表*`<link>`标签后会导致*脚本*阻塞，然后去等待*样式表*下载完成并解析。(为了确保*脚本*在执行时能获得最精准的样式信息)**

## 无阻塞的脚本

减少`Javascript`文件大小并限制`HTTP`请求数：
因为下载单个较大的`Javascript`文件只产生一次`HTTP`请求但会锁死浏览器一大段时间。为了避免这种情况，需要向页面逐步加载`Javascript`文件。要达到无阻塞可以通过页面加载完成后加载`Javascript`代码(`load`事件、`defer`属性、`async`属性)

+ 拥有`defer`属性的`<script>`标签会在解析到该标签时开始下载，直到DOM渲染加载完成后开始执行(`load`事件前,该属性只针对有`src`属性的脚本)。

+ 具有`async`的`<script>`标签，会在解析到该标签位置时，异步进行下载脚本，待脚本下载完毕后立即执行，执行时就会阻塞`dom`渲染。

## 动态脚本元素

基于DOM创建的`<script>`标签在该元素被添加到页面时开始下载，且这种形式的脚本的下载执行不会阻塞页面其他进程。

Note：将`<script>`添加进`head`标签比添加进`body`更安全，因为IE可能会在`body`中内容没有全部加载完成时抛出操作已中止的错误信息。

使用动态脚本节点下载文件时，返回的代码会立即执行。但当代码只包含供其他脚本调用的接口时，必须跟确定脚本何时下载就绪。(可以通过`<script>`元素的`load`事件来监听`IE需8+`,`IE 8-10`中会触发一个`readystatechange`事件,`<script>`元素会提供一个`readyState`属性,当其为`complete`时所有数据已准备就绪)

`readyState`共有5个值：

+ `uninitialized`：初始状态
+ `loading`：开始下载
+ `loaded`：下载完成
+ `interactive`：数据完成下载但尚不可用
+ `complete`：所有数据已准备就绪

`IE`在标识最终状态的`readyState`时并不一致，有时到达`loaded`状态却不会到达`complete`，有时不经过`loaded`就到达`complete`，最好同时检查两个属性当一个触发时便删除事件处理器(确保不会执行2次)。

### 动态加载后的执行顺序

在所有浏览器中只有部分浏览器能保证脚本按照指定顺序执行，其他浏览器会按照从服务器返回的顺序下载和执行代码。如要确保加载的顺序，可以使用调用回调的形式来加载。

```js
function loadScript(url, callback){
    let script = document.createElement('script');
    script.type = 'text/Javascript';

    script.onload = function(){
        callback();
    }

    script.src = url;
    document.head.appendChild(script);
}

new Promise((resolve, reject){
    loadScript(url, resolve);
}).then(()=>{
    loadScript(url, ()=>null)
}).then().....
```

## XMLHttpRequest脚本注入

通过异步请求加载`<script>`脚本。

+ 优点：不用立即执行。
+ 局限性：必须与请求页面同域。

## 推荐的无阻塞模式

1. 先添加动态加载所需的代码(如上面的`loadScript`函数)
2. 加载初始化页面所需的剩下代码(在利用`loadScript`函数加载脚本)

## 惰性加载

惰性加载是一种设计模式，使用这种设计模式，可以推迟事物的创建或者某些初始化工作直到必须要完成创建或初始化工作时为止，共有三种实现方式：

+ 虚拟代理模式：先加载一个具体的根页面，在需要其具体功能时再加载其功能。
+ 惰性初始化模式：先检查一个对象是否存在，如果不存在则创建，存在则使用现有的。
+ 值持有模式：比如我们常见的单例模式
+