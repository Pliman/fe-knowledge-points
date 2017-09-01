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