# cookie
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
  
