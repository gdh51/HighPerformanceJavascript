# Ajax

## 数据传输

### 请求数据
有五种常用技术用于向服务器请求数据：
+ XMLHttpRequest
+ Dynamic script tag insertion 动态脚本注入
+ iframes
+ Comet
+ Multipart XHR

#### XMLHttpRequest
进行Ajax读取数据时，通过监听readyState值等于3，可以与正在传输的服务器响应进行交互。(IE7及其以上)

使用XHR时，POST与GET对比：对于不改变服务器状态，只会获取数据的请求应该使用GET。经过GET请求的数据会被缓存,多次请求同一数据便可以提升性能。
只有当请求的URL加上参数的长度接近或超过2048个字符时，才应该用POST获取数据(因为IE限制URL长度，过长会被截断)。

#### 动态脚本注入(JSONP)
优点：克服XHR限制：不能跨域请求数据。
缺点：不能设置请求的头信息；参数传递只能使用GET方式；不能设置请求的超时处理或重试；因为响应信息作为脚本标签的源码，所以必须是可执行的javascript代码。

同时要注意脚本来源一定要值得信任

#### Multipart XHR
允许客户端只用一个HTTP请求就可以从服务器向客户端传送多个资源。通过在服务器将资源打包成一个由双方约定的字符串分隔的长字符串并发送到客户端，然后通过javascript处理并根据mime-type类型和传入的其他头信息解析出每个资源。

```js
var imageData = imageString.split('\u0001');
var imageElement;

for(let i = 0,len = imageData.length; i < len; i++){
  imageElement = document.createElement('img');
  imageElement.src = 'data:image/jpeg;base64,' + imageData[i];
  document.getElementById('container').appendChild(imageElement);
}
```

这里服务器传回表示三张图片的,base64标码长字符串，并且它们直接用Unicode编码字符1连接,即imageData

图片不是由base64字符串转换为二进制，而是使用data:URL的方式创建，并指定mime-type为images/jpeg

当然此处不限于图片，基本格式为`'data:' + mimeType + ';base64,' + data`

由于MXHR响应消息的体积越来越大，所以可以在每个资源收到时就立刻处理，而不是等到整个响应消息接收完成。可以通过监听readyState值为3来实现。

缺点：无法缓存；IE8以下无readyState和data：URL属性

适用情况：
+ 页面含有大量其他地方用不到的资源(无须缓存),尤其是图片
+ 网站已经在每个页面中使用一个独立打包的javascript或CSS文件以减少HTTP请求

testSide：http://techfoolery.com/mxhr/

### 发送数据
当我们不关心接收数据，只需要将数据发送给服务器时，可以使用以下技术

#### XMLHttpRequest
当使用GET传回数据超过上限时，可以使用POST

当使用XHR发送数据时，GET更快，因为一个GET请求只会向服务器发送一个数据包，而一个POST请求，至少发送两个数据包————一个装载头部信息，一个装载正文

#### Beacons(图像ping)
类似于动态脚本注入。

使用javascript创建一个Image对象，并把src属性设置为服务器上脚本的URL(没有添加到DOM中或创建img元素)
```js
(new Image()).src = url + '?' + someparams.join('&');
```

可以通过Image对象的load与error事件来监听是否成功，在load中可以通过服务器返回的图片的宽度与高度(当然不一定是图片)来通知服务器的状态。不需要在响应中返回数据时，服务器应该返回一个 204 No Content状态码

## 数据格式
当考虑到数据格式时，唯一需要比较的标准就是速度。当考虑数据传输技术时，当然要考虑功能集、兼容性、性能以及方向

### XML
优势：当Ajax才流行时，它有极佳的通用性、格式严格、易于验证
缺点：冗长，结构复杂，语法模糊，要提前知道详细结构

起初的XML使用各种表示属性的标签来存储对应的数据如：
```xml
<user id='1'>
  <email>asdj@ddd</email>
  <realname>Tony</realname>
</user>
```

简化后的XML统一使用一个标签，而要表示的值统一用标签上的属性来定义，这样缩小了体积，加快了响应
```xml
<user id='1' email='asdj@ddd' realname='Tony'></user>
```

总结：高性能Ajax，XML告辞！

### JSON
一种使用Javascript对象和数组直接量编写的轻量级且易于解析的数据格式

可以使用`eval('(' + responseText + ')')`来解析

JSON的简化方案比较多如简化名称，或干脆不要名称，但在接收数据时要处理下，但处理后时间综合比之前快。

#### JSON-P
在动态脚本注入时，JSON数据被当成另一个Javascript文件并作为原生代码执行，为实现这一点，这些数据必须封装在一个回调函数中。JSON-P ———— JSON with Padding(JSON填充)

JSON-P因为回调包装的原因略微增大了文件尺寸，但与其解析性能的提升相比微不足道。因为数据是作为原生的javascript，因此解析速度跟原生javascript一样快。

### HTML
数据量偏大，而且需要较长时间解析，因为将HTML插入DOM需要很多时间。

### 自定义格式
一个数据包含必要的结构以便自己分解出每一个独立的字段。如查询字符串`name1=aa&name2=bb&name3=cc`
(使用String.prototype.split()方法分解)
创建自定义格式时，采用什么分隔符最重要，理想情况下它不应该出现在你的数据中。

它的性能不亚于JSON。当需要在很短时间内向客户端传送大量数据时可以考虑使用。

**总结**：JSON 与 字符串分隔的自定义格式 最优
当数据集很大且对解析时间有要求时，可以按以下选择：
+ JSON-P数据，使用动态脚本注入获取————解析速度快；跨域
+ 字符串分隔的自定义格式，使用XHR或动态脚本注入获取，用split()解析————解析大数据集比JSON-P略快，而且通常文件尺寸小

## Ajax性能指南
当选择了合适的数据传输技术和数据格式时，就开始考虑其他优化技术了

### 缓存数据
最快的Ajax请求就是没有请求。有两种方法可以避免发送不必要的请求：
+ 服务器端：设置HTTP头信息以确保响应会被浏览器缓存
+ 客户端：把获取的信息存储到本地，从而避免再次请求

第一种使用最简单而且好维护，第二种给你最大的控制权

#### 设置HTTP头信息
如果喜欢Ajax响应能够被浏览器缓存，那么必须使用GET方式发出请求并设置HTTP头信息。
Expires头信息会告诉浏览器应该缓存响应多久，它的值为一个日期，过期之后对该URL的任何请求都不再从缓存中获取

Expires头信息格式为：`Expires: Mon, 28 Jul 2014 23:30:00 GMT`

#### 本地数据存储
将响应文本存储到一个对象中，以URL为键值作为索引。(*这个缓存感觉有点奇怪，只在一次会话中多个请求有效*)

`Expires`缓存内容能跨页面和跨会话。
手工管理缓存在废除缓存并获取更新数据时很有用
