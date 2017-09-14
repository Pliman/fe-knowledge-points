开门见山，闭包就是一个作用域，一个可以读取外部作用域内变量的作用域。
最简单的说就是，一个可以读取外部变量的函数。

#### 什么叫做一个可以读取外部变量的函数？

```
　var n=999;
　　function f1(){
　　　　alert(n);
　　}
　　f1(); // 999
```
f1里访问了外部作用域定义的变量 n, 就这么简单。

概念确实很简单，但是你以为这就结束了，那是不可能的。下面将逐一举例，帮助你理解闭包。

##### 如何读取外部局部变量？

```
　　function f1(){
　　　　var n=999;
　　　　function f2(){
　　　　　　alert(n); 
　　　　}
　　　　return f2;
　　}
　　var result=f1();
　　result(); // 999
```

函数f1的内部定义了一个函数f2。并且将这个内部的函数f2返回出来。

f2就是一个可以读取外部作用域变量的函数——闭包。


你以为这又结束了么，不可能的。先拿两个小题试试？

- 题目1：
```
　　var name = "The Window";
　　var object = {
　　　　name : "My Object",
　　　　getNameFunc : function(){
　　　　　　return function(){
　　　　　　　　return this.name;
　　　　　　};
　　　　}
　　};
　　alert(object.getNameFunc()());
```
返回的是"The Window"。

- 题目2：
```
　　var name = "The Window";
　　var object = {
　　　　name : "My Object",
　　　　getNameFunc : function(){
　　　　　　var that = this;
　　　　　　return function(){
　　　　　　　　return that.name;
　　　　　　};
　　　　}
　　};
　　alert(object.getNameFunc()());
```
返回的是"My Object"

#### 以上只是帮你了解闭包的基础，下面开始稍详细的讲解闭包。

##### 闭包的特点之一: 即使外部函数已经返回，当前函数仍然可以引用外部函数所定义的变量，这意味着，你可以返回一个内部函数，并在稍后调用他。

```
function sandwichMaker() {
    var magicIngredient = "peanut butter";
    function make(filling) {
        return magicIngredient + " and" + filling;
    }
    return make;
}

var f = sandwichMaker();
f("bananas"); // "peanut butter and jelly"
```

调用f实际上是调用make函数。但即使sandwichMaker这个外部函数已经返回，内部的make函数仍能记住外部函数中magicIngredient的值。

这是如何工作的？JavaScript函数值包含了比调用它们时执行所需要的代码还要多的信息。而且函数值还在内部存储它们可能会引用的定义，在起封闭作用域。

上面的例子中，每当make函数被调用时，其代码都能引用到外部的变量，因为该闭包存储了它们，包括外部函数的参数。

##### 闭包的另一个特性：闭包可以更新外部变量的值。因为闭包存储的是外部变量的引用，而不是它们的值得副本。下面举个例子来帮你明白。

```
function box() {
    var val = undefined;
    return {
        set: function(newVal) {val = newVal;},
        get: funciton() { return val; },
        type: function() {return typeof val;}
    };
}
var b = box();
b.type(); // "undefined"
b.set(98.6);
b.get(); // 98.6
b.type(); // "number"
```

该例子产生了三个闭包的对象：set, get 和 type 属性。共享访问 val 变量。set更新val的值，随后调用get和type查看更新结果。

你以为又就此结束了？你可能并没有记住或理解前面的东西。出于慈悲之心，再举个例子帮你回忆一下。

```
function wrapElements(a) {
    var result = [], i, n;
    for (i = 0, n = a.length; i < n; i++>) {
        result[i] = function() { return a[i]; };
    }
    return result;
}

var wrapped = wrapElements([10, 20, 30, 40, 50]);
var f = wrapped[0];
f(); // ?
```

输出的是什么?既然这问了当然不是输出10，而是undefined值，不信run一波？

不明白的同学们此时的感受也许是，城市套路深...

但是先别急着回农村，且听下解释原因。

##### JavaScript会为每一个绑定到该作用域的变量在内存中分配一个槽（slot）,wrapElements函数绑定了三个局部变量:result, i, n。在循环每次迭代中，循环都为嵌套函数分配了一个闭包。而他们存储的是变量i的引用。每次创建函数后变量i的值都发生了变化，因此这些创建出来的内部函数最终看到的是变量i最后的值。所以记住，闭包存储的是外部变量的引用而不是值。

 所以所有由wrapElement函数创建的闭包都引用在循环之前创建的变量i的同一个共享槽。由于每次循环迭代都递增变量i知道运行结束，因此我们调用任何一个闭包时，他都会查找数组的索引5并返回undefined值。

那么怎么解决这个问题呢？把i的声明放在for循环头部而不是外部怎么样？像这样：
```
for(var i = 0, n = a.length; i < n; i++>)
```
只需要这么一个小小的改动就可以让同学们明白，城市套路深...但是先别急着回农村...

var的声明被提升到函数的顶部，这就是变量声明提升（所以使用let更好）。

到底如何解决这种问题呢，也就是怎么立即保存外部变量的值而不是引用呢？

##### 办法1：通过创建一个嵌套函数并立即调用它来强制创建一个局部作用域。

```
function wrapElements(a) {

var result = [];
for (var i = 0, n = a.length; i < n; i ++>) {
    (function() {
        var j = i;
        result[i] = function() { return a[j]; }
    })();
    return result;
}
}
```

##### 或者将作为形参的局部变量绑定到立即调用的函数，并将其作为实参传入。
```
function wrapElements(a) {
    var result = [];
    for (var i = 0, n = a.length; i < n; i ++>) {
        (function(j) {
            result[i] = function() { return a[j]; };
        })(i)
    }
    return result;
}
```

以上的做法在使用了this时要注意，这种做法会改变this的含义。

#### 闭包的总结:

1. 函数可以引用定义在其外部的作用域的变量。
2. 闭包比创建他们的函数有更长的声明周期。
3. 闭包在内部存储器外部变量的引用，并且能读写这些变量，由于闭包会使得函数中的变量都被保存在内存中，内存消耗很大，所以不能滥用闭包。

最后，以上的头四个例子摘自博客：http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html

后面的例子和大部分解释来自《Effective JavaScript》一书，不要感谢我，我只是知识大自然的搬运工。