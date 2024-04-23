# CSRF攻击


&lt;!--more--&gt;



### 什么是CSRF攻击

CSRF攻击全称跨站请求伪造(Cross-site request forgery), 也被称为**one-click attack**或**session riding**, 通常缩写为CSRF或者XSRF. 尽管CSRF和[XSS攻击](../xss)很像, 但是两者的方式截然不同, 甚至可以说是相左的. XSS利用的是用户对指定网站的信任, CSRF利用的是网站对用户网页浏览器的信任. 我理解的是XSS是攻击者对浏览器下手, CSRF是对用户下手. 

可以这么理解: 攻击者盗用受信任用户的身份, 以该用户的名已发送恶意请求. 



### CSRF攻击的原理

1. 用户1访问受信任的网站A, 输入用户名和密码登录网站A. 
2. 登录成功之后, 网站A产生Cookie信息并返回给浏览器. 
3. 用户1在未退出登录网站A的情况下, **在同一浏览器**, 访问网站B. 
4. 网站B接收到请求后, 携带网站A的Cookie信息, 发出一个请求去访问网站A
5. 网站A在接收到网站B的请求之后, 会以用户1的权限处理该请求, 从而导致用户1的隐私泄漏和财产安全受到威胁. 



### 常见的几种类型

1. `GET`类型的CSRF:

   * 这种类型的CSRF利用非常简单, 只需要一个HTTP请求: 

   ```html
   &lt;img src=http://xxx.com/csrf?user=xxx /&gt;
   ```

   在访问这个`img`的页面后, 成功发出了一次HTTP请求. 

2. `POST`类型的CSRF:

   * 这种类型的CSRF没有`GET`型的大, 利用起来通常使用的是一个自动提交的表单:

   ```html
   &lt;form action=http://xxx.com/csrf.php method=POST&gt;
     &lt;input type=&#34;text&#34; name=&#34;user&#34; value=&#34;xxx&#34; /&gt;
   &lt;/form&gt;
   &lt;script&gt;document.forms[0].submit();&lt;/script&gt;
   ```

3. 其他类型CSRF: 

   ```html
   &lt;img src=http://admin:admin@192.168.1.1 /&gt;
   ```

   在访问这个`img`的页面后, 路由器会给用户一个合法的SESSION, 就可以进行下一步操作了. 



### 防御手段

目前防御CSRF攻击主要有三种策略:

1. **检查`HTTP Referer`字段**

   根据HTTP协议, 在HTTP头中有一个字段叫Referer, 它记录了该HTTP请求的来源地址. 

   &gt; 优势: 简单易行, 网站的普通开发人员不需要操心CSRF的漏洞, 只需要在最后给所有安全敏感的请求统一增加一个拦截器来检查Referer的值就可以了. 特别是对于已有的系统来说, 不需要更改当前系统的任何代码和逻辑, 没有风险. 
   &gt;
   &gt; 劣势: 这种方式是把安全性都依赖于浏览器来保障, 对于一些低版本的浏览器来说, 是可以篡改Referer值的. 对于某些最新的浏览器, 虽然不能篡改Referer的值, 但是用户有些时候会认为Referer值会记录下用户的访问来源, 所以用户自己可以设置浏览器在发送请求的时候不再提供Referer. 这种情况下, 此种方式就会认为是CSRF攻击, 从而拒绝合法用户的访问. 

2. **添加`校验token`**

   CSRF之所以能够成功是因为攻击者获取到了Cookie信息. 如果能够不只是依靠Cookie中的信息来抵御CSRF攻击, 那么就可以防御了. 所以这种方式就是在HTTP请求中以参数的形式加入一个随机产生的`token`, 并且在服务器端建立一个拦截器来验证这个`token`. 

   &gt; 优势: 这种方法比检查Referer更安全一些, `token`可以在用户登录之后产生并放与SESSION中, 然后每次请求时把`token`从SESSION中取出来进行比对. 
   &gt;
   &gt; 劣势: 难以保证`token`本身的安全. 

   &gt; 疑问: 可以做到每次验证`token`成功之后, 产生一个新的`token`是否可以避免此种方法可能产生的漏洞. 

3. **在HTTP头中自定义属性并验证**

   这种方法也是使用`token`进行验证, 但是不是放在参数中, 而是放在HTTP请求头的自定义属性中. 通过`XMLHttpRequest`这个类, 可以一次性给所有该类请求加上`csrftoken`这个HTTP头属性, 并把`token`放进去. 

   &gt; 优势: 不会记录到浏览器的地址栏, 统一管理`token`输入输出, 可以保证`token`的安全性.
   &gt;
   &gt; 劣势: 无法在非异步的请求上实施. 


---

> 作者:   
> URL: https://buli-home.cn/csrf/  

