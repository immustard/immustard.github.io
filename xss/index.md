# XSS(跨站脚本攻击)


<!--more-->



> 昨天的时候, 部门老大提到session和OnceToken的时候, 说到了xss攻击和csrf攻击. 今天记录一下学习的内容. 



### 什么是XSS攻击

XSS攻击全称跨站脚本攻击(Cross Site Scripting), 是一种在Web应用中的计算机安全漏洞, 它允许恶意Web用户将代码植入到提供给其他用户使用的正常页面中. 

> 之所以缩写是XSS, 是因为如果是CSS会和层叠样式表混淆(Cascading Style Sheets). 

举个简单的🌰, 恶意服务器嵌套了正常服务器中某页面的某form表单. 当用户登录了正常服务器之后, 在恶意服务器上也可以提交这个表单, 甚至拿到更高的权限. 

XSS是**最普遍**的Web应用安全漏洞. 可以做到劫持用户会话, 插入恶意内容, 重定向用户, 使用恶意软件劫持用户浏览器, 繁殖[XSS蠕虫](https://baike.baidu.com/item/XSS蠕虫/22777013), 甚至是破坏网站, 修改路由器配置信息等. 所以XSS攻击的危害还是很严重的. 



### XSS原理

首先看看XSS可以插入哪里: 

1. script标签内容
2. HTML注释内容
3. HTML标签的属性名
4. HTML标签的属性值
5. HTML标签的名字
6. 直接插入到CSS里

看到XSS可以插入到这些地方之后, 就更能理解它的原理. XSS通过一些被特殊对待的文本和标记(🌰, 小于符号`<` 被看做是HTML标签的开始), 使得用户浏览器将这些误认为是插入了正常的内容, 所以就会在用户浏览器中被执行. 也就是说, 当这些特殊字符不能被动态页面检查或检查出现失误时, 就将会产生XSS漏洞. 



### 分类

* **存储型XSS攻击(持久型)**: 最直接的危害类型. 将XSS代码提交存储到服务器端(**数据库**, 内存, 文件系统等). 这样下次请求目标页面时就不用再提交XSS代码, 会从服务器获取. 一般出现在留言, 评论, 博客日志等交互. 

  <center>
      <img style="border-radius: 0.3125em;
      box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
      src="http://mustard_gxg.gitee.io/pic/pictures/2022-02/202202230947946.png" width = "65%" alt=""/>
      <br>
      <div style="color:orange; border-bottom: 1px solid #d9d9d9;
      display: inline-block;
      color: #999;
      padding: 2px;">
        存储型XSS攻击
    	</div>
  </center>

* **反射性XSS攻击(非持久型):** 最普遍的类型. 通过特定手法(🌰email), 诱使用户访问一个包含恶意代码的地址, 当用户点击这些链接的时候, 恶意代码会在用户的浏览器执行. 一般出现在搜索栏, 用户登录口, 常用来窃取客户端Cookies或进行钓鱼欺骗. 

  <center>
      <img style="border-radius: 0.3125em;
      box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
      src="http://mustard_gxg.gitee.io/pic/pictures/2022-02/202202230934435.png" width = "65%" alt=""/>
      <br>
      <div style="color:orange; border-bottom: 1px solid #d9d9d9;
      display: inline-block;
      color: #999;
      padding: 2px;">
        反射型XSS攻击
    	</div>
  </center>

* **DOM-based 型XSS攻击:** 通过/xss修改页面的DOM(Document Object Model)结构, 是纯粹发生在客户端的攻击. 在整个攻击过程中, 服务器响应的页面没有发生变化, 取出和执行恶意代码都由浏览器端完成, 属于前端自身的安全漏洞. 



### 防御手段

**总体思路: 对用户的输入(和URL参数)进行过滤, 对输出进行编码.**

也就是说, 对用户提交的所有内容进行过滤, 对URL中的参数进行过滤, 过滤掉会导致脚本执行的相关内容; 然后对动态输出到页面的内容进行html编码, 使脚本无法在浏览器中执行. 还可以服务端设置会话Cookie的HTTP Only属性, 这样客户端的JS脚本就不能获取Cookie信息了. 

