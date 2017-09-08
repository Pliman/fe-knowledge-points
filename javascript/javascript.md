#### Javascript和EMACScript的关系是什么?NodeJS呢？
简单来说EMACScript是标准，Javascript是对EMAC标准的实现。但Javascript不仅仅包含对EMACScript标准的实现，它还包括了对DOM（文档对象模型）
和BOM（浏览器对象模型）标准的实现。其中DOM标准由W3C制定和维护，而BOM则是由各个浏览器厂商根据DOM标准实现。
  
NodeJS也实现了EMACScript，但由于NodeJS和Javascript运行的宿主环境不同，因此它没有实现DOM和BOM。相对的，它提供了对OS、FILE、NET、DATABASE
的支持。
  
一句话，无论是浏览器（javascript），还是服务端（nodejs）运行的js，它们的语言基础都是EMACScript。在这个基础上分别衍生出了对不同宿主环境的支持。

- 相关阅读
    
  [TC39，EMACScript， And the Future of Javascript](https://ponyfoo.com/articles/tc39-ecmascript-proposals-future-of-javascript)
    
  [How to read EMACScript Standard](https://cauu.github.io/2017/07/How-to-read-ECMAScript-Specification/)
  
#### 如何判断JS中this的指向？

#### 什么是闭包？如何理解JS中的闭包？

#### JS中如何实现继承？

#### ==和===有什么区别？
#### 跨域方法介绍
- CORS是一个W3C标准，全称是"跨域资源共享"（Cross-origin resource sharing）它允许浏览器跨源服务器发请求，从而克服了ajax只能同源使用的限制。CORS 需要客户端和服务器同时支持。目前，所有浏览器都支持该机制。cors 涉及简单请求，预检请求，附带身份凭证(cookie)的请求这些概念   
 
	[HTTP访问控制（CORS） - HTTP | MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)  
 
	[跨域资源共享CORS 详解- 阮一峰的网络日志 - 阮一峰的个人网站](http://www.ruanyifeng.com/blog/2016/04/cors.html)   
 
- JSONP它的基本思想是，网页通过添加一个`<script>`元素，向服务器请求JSON数据，这种做法不受同源政策限制，服务器收到请求后，将数据放在一个指定名字的回调函数里传回来，JSONP 只支持 GET 请求，而且需要服务器端来配合，如果服务器端返回一段恶意代码，客户端也会毅然决然的执行……JSONP 是一个伟大的创造，但还不够好。   
 
	[跨域请求之JSONP 和CORS | I sudo X](https://isudox.com/2016/09/24/cross-site-jsonp-and-cors/ "跨域请求之JSONP 和CORS | I sudo X")  
 
	[5 分钟彻底明白JSONP](https://tonghuashuo.github.io/blog/jsonp.html "5 分钟彻底明白JSONP")  
 
- 其他跨域方法整理  
 
	[前端跨域整理](https://zhuanlan.zhihu.com/p/24101549 "前端跨域整理")   
 
	[同源策略](http://javascript.ruanyifeng.com/bom/same-origin.html "同源策略")    