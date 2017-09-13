Javascript中的``this``一直是一个老大难的问题，在ES6中，箭头函数的```this```又有不一样的表现。于是趁着研究箭头函数```this```的机会，好好总结了一下this的用法。

### ES5中的```this```

首先需要明确``this``绑定的基本原则：**``this``绑定发生在函数的调用时（而非声明时）**，也就是所谓的运行时绑定。明确了这一点，我们可以看看实际使用中``this``常常遇到的几种情况:

#### 1. 默认绑定全局变量

```

var a = 1;

function foo() {

    console.log(this.a);
}

foo();  //ouput: 1

```

根据我们上面的原则，函数foo的调用方是全局的，因此我们的this实际上是指向默认全局对象，也就是a。需要注意的是，若我们采用```strict模式```，此时this指向应该是undefined（因为上文中没有出现this的声明）。

#### 2. 对象中的隐式绑定

```

function foo() {

    console.log(this.a);
}

var b = {

    a: 2,

    foo: foo

}

b.foo();    //2



var c = {

    a: 3,

    b: b

}



c.b.foo();   //2

```

此时foo函数的调用者是对象b，因此根据原则，```this```指向对象b。另外，``this``的指向遵循就近原则，因此c.b.foo()的结果是2而不是3。

####3. 显式绑定

```

function foo() {

    console.log(this.a);
}

var obj = {

    a: 2
}

foo.call(obj); // 2

foo.apply(obj); // 2

```

javascript提供了内置函数apply和call，通过调用它们，我们可以强制将函数的``this``对象指定为obj。

在apply和call的基础上，js还实现了bind函数：

```

function foo() {

    console.log(this.a);
}

var obj = {

    a: 5

};

var obj2 = {

    a: 10

};

var bar = foo.bind(obj);

bar();    // 5

bar.apply(obj2); // 5

```

对foo调用bind函数，我们得到一个bar函数。此时bar函数的``this``对象固定指向obj，即使我们对bar函数调用了apply方法，依然无法改变``this``的指向。我们称这种情况为``hard bind``。

####4. ``new``绑定

除了上面几种情况，js中，我们可以通过``new``创建对象，此时也会造成``this``绑定。

```

function foo(a) {

    this.a = a;
}

var obj = {

    a: 5

};

var baz = foo.bind(obj); // 绑定obj

var bar = new baz(2); // new覆盖了绑定

console.log(bar.a);  // 2

```

``new``会创建对象，并让``this``指向当前对象。通过上面的例子，我们还可以发现，``new``绑定的优先级高于``hard bind``，即使baz已经是foo函数调用``bind``的结果，bar.a的输出依然是2。



### 绑定丢失的问题

下面是一个最常见的绑定丢失的场景:

```

function foo() {

    console.log(this.a);

}

var obj = {

    a: 5,

    foo: foo

}

setTimeout(obj.foo, 10); // undefined

```

我们原本期待setTimeout函数在十秒后输出5，然而输出结果却是undefined。原因是什么呢？因为**作为参数传递到setTimeout函数中的obj.foo只是一个指向foo方法的引用**，它的效果其实等同于下面这段代码:

```

setTimeout(function() {

    console.log(this.a);
}, 10);

```

此时，不存在任何this绑定，this指向调用setTimeout的对象，在普通模式下是global变量，在strict模式下是undefined。这也就解释了为什么会出现this绑定丢失的情况。解决办法也简单：可以使用bind函数强制绑定this对象，也可以将this对象作为参数传递给对应的函数。这里推荐第一种方法：

```

setTimeout(obj.foo.bind(obj), 10);  // 5

```

### ES6箭头函数(=>)中的```this```

为了解决绑定丢失的问题，ES6中的箭头函数对``this``绑定做了一定的改进，具体来说，就是:**箭头函数内的``this``始终指向调用它的闭包所绑定的``this``。**说起来有点拗口，下面还是用setTimeout的例子来解释这句话:

```

var obj = {

    a: 5

}

function foo() {

    setTimeout(() => {    

        console.log(this.a);

    }, 10);

}

foo.call(obj); // 5

```

setTimeout对应的闭包是foo（注意js中，闭包只针对函数，Object不是闭包），而foo的``this``对应obj，因此setTimeout中，匿名箭头函数的``this``指向调用它的闭包（也就是foo函数）对应的``this``。可以看到，箭头函数完美解决了匿名函数this绑定丢失的问题。

  

延伸阅读:

[You dont know js: this & object.prototypes](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes/ch2.md)