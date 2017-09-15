### 类型检测
JS中的数据类型分为两类：基本数据类型和用户自定的数据类型。对数据类型的判断遵循以下两个基本原则即可：
  
- 如果判断的是基本数据类型或javascript内置对象，用`toString`
  
- 如果判断的是自定义类型，用`instanceof`

为什么不用typeof呢？看看下面的例子：
```
typeof null; // object
typeof ''; // string
typeof new String('123') // object
```
typeof null已经是一个众所周知的bug，按下不表。对String类型的判断才是我们的心头大患: 我们希望无论是'123'
还是new String('123')都是同样的string类型，而toString方法正好可以满足这个要求。
  
由于很多对象都重载了toString方法，因此我们用Object.prototype.toString来判断对象的类型:
```
toString = Object.prototype.toString;
toString.call('') // [object String]
toString.call(new String('')) // [object String]
```
如果要更加深入的理解这种用法，推荐大家阅读[lodash](https://github.com/lodash/lodash)中`isXXX`
部分的源码。

相比静态语言，动态语言的类型判断更加灵活。有的时候只要实例满足某些条件，我们也认为它是可接受的
数据类型，这就是所谓的[鸭子类型](http://blog.csdn.net/handsomekang/article/details/40270009)。

关于类型检测的更多细节，还可以读一下[这篇文章](http://harttle.com/2015/09/18/js-type-checking.html)。

### 变量提升
在ES6中，新增了块级作用域，使用let和const声明的变量不存在变量提升的问题，但这并不意味这ES6中就不存在
变量提升了，如:
```
function test() {
  hehe(); // hehe

  function hehe() { console.log('hehe'); }
}
```
要消除变量提升带来的不确定性，建议坚持使用`let/const`的方式定义函数。

变量提升的问题更多的是来自于ES5。由于在ES5中只有函数作用域和全局作用域，因此存在大量变量提升的场景，同时
也导致了自执行函数的广泛使用：
```
(function() {
  // ...
}());
```
[you dont know javascript的scope && closure部分](https://github.com/getify/You-Dont-Know-JS/blob/master/scope%20&%20closures/README.md#you-dont-know-js-scope--closures)
对变量提升也有很好的解释，对这部分还有疑问的同学一定要抽时间读一读。

### 闭包
大成总结的[闭包的文章](../closure/index.md)基本涵盖了闭包问题所需要了解的方方面面，这里就不赘述了。

### 继承

### this的问题
js中的this也是个磨人的话题。对于this，除了了解和掌握它的基本原理，一定要能结合代码熟练应用。
  
对于this，一般遵循这几个基本的原则:
  
- this指向函数运行时所在的对象，而非函数定义时所在的对象；

- 匿名函数或不处于任何对象中的函数this指向全局变量window/global;

- 对于调用了call、apply、bind的函数，指定的this是谁就是谁；

- 箭头函数默认不绑定this，它的this指向定义它所在的作用域对应的this;

下面结合几个面试中的问题来说明上面几点:
1. 全局变量的this指向window对象
  
```
var a = 10;
(function() {
  a = 11;
}());

console.log(global === this); // true
console.log(global.a); // 11
console.log(this.a); 11
```

2. 匿名函数的this指向window/global对象
  
```
var a = 100;
var b = {
  a: 10
};
function test() {
  return function() {
    console.log(this.a);
  };
}

test.call(b)(); // 100
```

3. 箭头函数this指向其所在函数作用域的this;
  
```
var value = 10;
var b = { value: 100 };

function test() {
  return () => {
    console.log(arguments, this.value);
  }
}

let a = test(1);
a.call(b, 2); // [1] 10
```

关于this更进一步的解读参考[神奇的this](../this/index.md)。

### 对象的深/浅拷贝
[推荐阅读](http://jerryzou.com/posts/dive-into-deep-clone-in-javascript/)

### 浮点数的运算

### ES6