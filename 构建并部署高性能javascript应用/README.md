# BuildingAndDeployingHighPerformanceJavascriptApplications(构建并部署高性能javascript应用)

## 合并多个javascript文件
减少页面渲染所需的HTTP请求数。
把部分文件和代码合并成一个外链文件，从而大大降低页面渲染所需的HTTP请求数

## 预处理javascript文件
预处理你的javascript源文件并不会让应用更快，但它允许你做些其他的事情，比如插入测试代码

## Javascript压缩
将javascript代码中与运行无关的部分进行剥离。如注释和空白。

## 构建时处理对比运行时处理
合并、预处理和压缩可以在两个时候进行。
开发高性能应用的一个普遍规则：能在构建时完成的工作，就不要留到运行时去做

## Javascript的HTTP压缩
当浏览器请求一个资源时，它通常会发送一个Accept-Enoding HTTP头来告诉Web服务器它支持哪种编码转换类型。
这个信息主要用来压缩文档以获得更快的下载。可用的值包括：gzip、compress、deflate、identity

在服务器中查看到这些头部后，会选择最合适的编码方式，并通过Content-Encoding HTTP头通知浏览器服务器的决定。

## 缓存Javascript文件
缓存HTTP组件能极大提高网站回访用户的体验，缓存大多数适用于图片，也适用于所有静态内容。

在HTML5中也可以利用离线应用缓存

## 处理缓存问题
问题：当应用升级时，需要确保用户下载到最新的静态内容。可以通过把改动过的静态资源重新命名来解决。

## 使用内容分发网络(CDN)
内容分发网络是在互联网上按地理位置分布计算机网络，它负责传递内容给终端用户。可以极大减少网络延时。