#### 什么是Middleware?

如果熟悉Node，对这个概念一不会陌生。middleware，从本质上来说，都是使用**装饰者模式**封装一个function。

例如，我们有3个middleware m1,m2,m3，他们都会接收一个function作为参数，并且返回一个基于该function的新的function。此时，如果我们希望使用他们装饰我们的function foo，我们可以这么做:

```

function foo(params) {

//do something

}

function m1(func) {    //m2, m3与此类似

    return function(params) {

        //do other things

         func(params);

    }

}

var newFunc = m3(m2(m1(foo)));  //最简单的包装方法



//如果middleware的数量不固定

function applyMiddleware(func, middlewares) {

    var newFunc = func;

    for(let i in middlewares) {

        newFunc = middlewares[i](newFunc);

    }

    return newFunc;

}

```

#### Middleware在Redux中的应用

Redux中，正好有一个applyMiddleware函数，我们来看看它是如何实现的:

```

import compose from './compose';

export default function applyMiddleware(...middlewares) { 

    return (next) => (reducer, initialState) => {

        var store = next(reducer, initialState);

        var dispatch = store.dispatch;

        var chain = [];



        var middlewareAPI = {

            getState: store.getState,

            dispatch: (action) => dispatch(action)

        };        

        chain = middlewares.map(middleware => middleware(middlewareAPI));

        dispatch = compose(...chain)(store.dispatch);

        

        return {

            ...store,

            dispatch

        }

    }

}

```

第一行用到了ES6的新特性：rest参数，将传入函数的所有middleware打包成一个数组middlewares。

下文的next实际上对应的就是createStore函数。因为之后的middleware会对dispatch函数进行封装，因此，此处不可避免的需要用创建store。

13行又用到了ES6的另一个新特性：扩展运算。它用在函数调用时，好比rest参数的逆运算，将传入的数组解构成一个以逗号分隔的参数列表。

除了上述几个点，这个函数的核心实际上就两句话:

```

chain = middlewares.map(middleware => middleware(middlewareAPI));

dispatch = compose(...chain)(store.dispatch)

```

首先使用传入的middleware封装原始的dispatch函数，此时新的函数形如:

```

function createNewDispatch(next) {

    return (action) => {

        //...do something...

    }

}

```

接着compose函数会通过上面提到的链式写法，将函数[柯里化](http://blog.jobbole.com/77956/)，最终返回给我们一个如下函数：

```

function newDispatch(action) {

    //...new dispatch...

}

```

#### 自定义middleware

当我们希望定义自己的middleware时，仿照上面的写法即可:

```

function myMiddleware(store) {

    return (next) => (action) {

    }

}

```

或

```

var myMiddleware = (store) => (next) => (action) {

    //...do something...

    next(action); 

}

```
