# Cookie
- 设置：Set-Cookie
- 生命周期：  
  1.会话期Cookie：浏览器关闭之后会自动删除，仅在会话期内有效。不需要指定过期时间（Expires）或有效期（Max-Age）。但是有些浏览器提供了会话恢复功能，这种情况下即使关闭了浏览器，会话期Cookie也会被保留下来，就好像浏览器从来没有关闭一样，这会导致Cookie的生命周期无限延长  
  2.持久性Cookie的生命周期取决于过期时间和有效期指定的一段时间
- 限制访问Cookie  
  有两种方法可以确保Cookie被安全发送，并且不会被意外的参与者或脚本访问：Secure属性和HttpOnly属性
  - 标记Secure的cookie只应通过被HTTPS协议加密过的请求发送给服务端
  - JavaScript`Document.cookie`API无法访问带有HttpOnly属性的cookie，此类cookie仅作用于服务器。此预防措施有助于缓解跨站脚本攻击(XSS)
- Cookie的作用域  
  Domain和Path标识定义了COokie的作用域：即允许Cookie应该发送给哪些URL
  - Domain: 指定了哪些主机可以接受cookie，如果不指定，默认为origin，**不包含子域名**。如果指定了Domain，则一般包含子域名
  - Path：指定了主机下的哪些路径可以接受cookie（该URL路径必须存在于请求URL中）。以（"/"）作为路径分隔符，子路径也会被匹配
  - SameSite：cookie允许服务器要求某个cookie在跨站请求时不会被发送，从而可以阻止跨站请求伪造攻击。它有三个值：
    1.**None**：浏览器会在同站请求、跨站请求下继续发送cookie，不区分大小写  
    2.**Strict**：浏览器将只在访问相同站点时发送cookie  
    3.**Lax**：与**Strict**类似，但用户从外部站点导航至URL时除外（例如通过连接）。在新版本浏览器中为默认选项。
  
# 缓存
- 概述  
良好的缓存策略可以降低资源的重复加载挺高王爷的整体加载速度，通常浏览器缓存策略分为两种：强缓存和协商缓存  
1、基本原理  
 - 浏览器在加载资源时，根据请求头的`expires`和`cache-control`判断是否命中强缓存，是则直接从缓存读取资源，不会发送请求到服务器
 - 如果没有命中强缓存，浏览器一定会发送一个请求到服务器，通过`last-modified`和`etag`验证资源是否命中协商缓存，如果命中，服务器会将这个请求返回，但是不会返回这个资源的数据，依然从缓存中读取资源
 - 如果前两者都没有命中，直接从服务器加载资源  
2、相同点  
如果命中，都是从客户端缓存中加载资源，而不是从服务器加载资源数据  
3、不同点  
强缓存不发送请求到服务器，协商缓存会发请求到服务器

- 强缓存
通过**Expires**和**Cache-Control**两种响应头实现  
1、Expires  
Expires是http1.0提出的一个表示资源过期时间的header，它描述的是一个绝对时间，由服务器返回。Expires受限于本地时间，如果修改了本地时间，可能会造成缓存失败  
2、Cache-Control  
出现于http/1.1，优先级高于Expires，表示的是相对时间
```
Cache-Control:max-age=31350000
```
`Cache-control:no-store`：不缓存数据到本地
`Cache-Control:public`：可以被所有用户缓存（多用户共享），包括终端和CDN等中间代理服务器
`Cache-Control:private`：只能被终端浏览器缓存（而且是私有缓存），不允许中间缓存服务器进行缓存

- 协商缓存  
如果命中协商缓存则请求响应返回的http状态码为304并且会显示一个Not Modified的字符串  
协商缓存是利用`【Last-Modified,If-Modified-Since】`和`【ETag,If-None-Match】`这两对header来管理的  
1、Last-Modified，If-Modified-Since  
`Last-Modified`表示本地文件最后修改日期，浏览器会在request header加上`If-Modified-Since`（删除给返回的`Last-Modified`）的值，询问服务器在该日期后资源是否有更新，有更新的话就会将新的资源发送回来  
但是如果在本地打开缓存文件，就会造成Last-Modified被修改，所以出现了ETag  
2、ETag, If-None-Match  
`ETag`就像一个指纹，资源变化都会导致ETag变化，跟最后修改时间没有关系，`ETag`可以保证每一个资源是唯一的  
`If-None-Match`的header会将上次返回的`ETag`发送给服务器，询问该资源的`ETag`是否有更新，有变动就会发送新的资源回来  
`ETag`的优先级比`Last-Modified`高，使用`ETag`的原因：
  - 一些文件也许会周期性的更改，但是它的内容并不改变，这个时候我们并不希望客户端认为这个文件被修改了，而重新获取
  - 某些文件修改非常频繁，If-Modified-Since能见查到的粒度是s级的，这种修改无法判断
  - 某些服务器不能精确的得到文件的最后修改时间

- 整体流程图
![RUNOOB 图标](https://user-images.githubusercontent.com/25027560/38223505-d8ab53da-371d-11e8-9263-79814b6971a5.png)

- 几种状态码的区别
  - 200：强缓存Expires/Cache-COntrol失效时，返回新的资源文件
  - 200(from cache)：强缓存Expires/Cache-control两者都存在，未过期，Cache-Control优先Expires时，浏览器从本地获取资源成功
  - 304(Not Modified)：协商缓存Last-Modified/ETag没有过期时，服务端返回状态码304
注：现在的200(form cache)变成了from disk cache(磁盘缓存)和from memory cache(内存缓存)两种

# HTTP协议特点
- 支持客户/服务器模式
- 简单快速：客户端向服务器请求服务时，只需传动请求方式和路径
- 灵活：HTTP允许传输任意类型的数据对象
- 无连接：限制每次连接只处理一个请求，服务器处理完客户端的请求，并受到客户端的应答后，即断开连接，采用这种方式可以节省传输时间
- 无状态：协议对于事务处理没有记忆能力
