几个问题:
0. 如何在js中实现继承?

1. instanceof的工作原理?

2. ES6可以extend build-in类型，然而使用babel转码的ES6代码却不能extend build-in类型，为何？

3. 在babel中，如何extend build-in class?（如Error，Array，Date等类型）

#### Babel转码引发的问题

vip系统的开发过程中，我们希望自定义一些Error类型，以便根据服务端的errorCode抛出不同类型的Error进行处理。

由于前端使用Babel对ES6进行转码，自然而然的，我们采用了新的class语法去继承Error：

```

class LoginError extends Error {

    constructor(message) {
        super(message);

        this.name = this.constructor.name;

        this.message = message;

    }

}



const e = new LoginError('login');

console.log(e);

console.log(e.message);

console.log(e.name);

console.log(e instanceof LoginError);

console.log(e instanceof Error);

```

构建完成后，在浏览器中运行这段代码时，却惊奇的发现输出的结果如下：

```

{ Error: login(...) ... }

Error

login

false

true

```

由于Node 8.x增加了对ES6语法的支持，不信邪的我们在当Node 8.x环境下再次运行这段代码，得到了：

```

{ LoginError: login ... }

LoginError

login

true

true

```

完全符合预期！这意味着ES6应该是支持对Error对象的继承的。

为什么同样的代码，经过Babel转码后却得到了截然不同的结果呢？google后，发现因为一些bug，babel默认不允许extend buildin class：

***

***Extending native classes is not supported by Babel. It was removed in version 5.2.17 (see [this commit](https://github.com/babel/babel/commit/3878bd812c73bdd18b1011be59515dad985940fd))

It was removed because it was not working properly, see the bug: https://phabricator.babeljs.io/T1424

It's unlikely it will be ever added because it's not a feature that can be simulated. We will have to wait for native support in browsers (some already support it now in experimental mode). That also means it will currently behave differently in different browsers. ***

***

#### 解决办法

解决这个问题的办法有两个：

- 在babel的配置文件中添加[babel-plugin-transform-buildin-extend](https://www.npmjs.com/package/babel-plugin-transform-builtin-extend)插件

- 手动实现一个用于extend buildIn对象的函数:

```

function ExtendableBuiltIn(cls){

  function buildIn(){

      cls.apply(this, arguments);

  }

  buildIn.prototype = Object.create(cls.prototype);

  buildIn.__proto__ = cls;



  return buildIn;

}

```

如果我们希望extend Error，现在可以这么做了:

```

class MyError extends ExtendableBuildIn(Error) {

    ....

}

```

上面的代码为什么能起作用？那么我们需要看看：

#### JS中实现继承的方式

我们知道，JS中没有真正的“继承”，但是它的对象，都存在原型链这一概念:

```obj ---> obj.[[Prototype]] ---> obj.[[Prototype]].[[Prototype]] ---> ... ---> Object.prototype ---> null```

因此，我们可以通过原型链的方式，实现代码的复用，来达到所谓“继承”的目的。

##### 1. 原型链继承

```

function Bird(name) {  this.name = name; }

function Chiken() {

}

Chiken.prototype = new Bird('chiken');

Chiken.prototype.fly = function() { console.log('cannot fly'); }

const chiken = new Chiken('jj');

``` 

上面这种方法的缺陷也很明显：所有的chiken对象都共享了同一个bird对象，当我们创建子类对象时，无法调用父类的构造函数。因此，这种方法不能算严格意义上的继承。

##### 2.  使用构造函数继承

```

function Bird(name) {  this.name = name; }

function Chiken(name) {

    Bird.call(this, arguments);

}

Chiken.prototype = Object.create(Bird && Bird.prototype);

Chiken.__proto__ = Bird;

// or 

// Chiken.prototype = new Bird();

```

首先我们通过`Bird.call(this, arguments);`继承了Bird对象的实例属性，接着通过`Chiken.prototype = Object.create(Bird && Bird.prototype)`继承了Bird对象的原型属性和方法。通过这两步，我们就实现了对Bird的继承。

// 问题: 为什么需要添加`Chiken.__proto__ = Bird`？

事实上，ES6中的Class语法就是这种实现方式的语法糖，其中，extend实现了对原型链的继承，super实现了对父对象实例属性的继承。至于如何实现，可以参考[此处](https://segmentfault.com/a/1190000008390268)。

至此，我们的`ExtendableBuildIn`方法也就没什么神秘可言了。它的本质就是通过方法二重新包装传入的cls对象，从而允许我们继承它。

#### instanceof的原理是？

到这里，关于继承的部分已经到了尾声。但是我们最开始提出的问题1依然没有解决：如何保证instanceof操作符能对继承后的对象起作用?如对上面的Chiken对象：

```

new Chiken() instanceof Chiken // true

new Chiken() instanceof Bird // true

```

我们希望 new Chiken() 得到的实例即是一个Chiken对象，又是Bird对象。  

先看看instanceof的实现:

```

function instance_of(L, R) {//L 表示左表达式，R 表示右表达式

 var O = R.prototype;// 取 R 的显示原型

 L = L.__proto__;// 取 L 的隐式原型

 while (true) { 

   if (L === null) 

     return false; 

   if (O === L)// 这里重点：当 O 严格等于 L 时，返回 true 

     return true; 

   L = L.__proto__; 

 } 

}

```

也就是说，instanceof判断是否存在`L.__proto__.__proto__ ....._proto__ === R.prototype`的情况。一旦存在，那么它就会返回true。对于我们这里，我们就希望`new Chiken().__proto__ === Chiken.prototype`同时`new Chiken().__proto__.__proto__ === Bird.prototype`。

在实现继承的时候我们定义了:

`Chiken.prototype = Object.create(Bird.prototype);`

该函数会返回一个对象O，其中: 

`O = {}; O.__proto__ = Bird.prototype`

看到这里，是不是恍然大悟呢？事实上，`Chiken.prototype = new Bird()`也可以达到同样的效果。

延伸阅读:

[ES6 class实现原理](https://segmentfault.com/a/1190000008390268)

[JS实现继承的几种方式](http://www.cnblogs.com/humin/p/4556820.html)

[What's a good way to Extend Error?](https://stackoverflow.com/questions/1382107/whats-a-good-way-to-extend-error-in-javascript)

[Extend Error in ES6](https://stackoverflow.com/questions/31089801/extending-error-in-javascript-with-es6-syntax)