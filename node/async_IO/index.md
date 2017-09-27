#### 阻塞非阻塞与同步异步的区别
Node是因为其事件驱动，非阻塞的特点而具有轻量及高效优点的Javascript运行平台。因为Js的主要用途就是与用户交互及操作DOM等，如果使用多线程的话，会引入极复杂的同步问题，所以node的设计也是基于单线程的。又因为单线程极大地限制了其CPU 的利用效率及降低了用户体验，所以基于此，提出了异步IO的概念。

然而，对于node 的几个常见概念，例如 异步、非阻塞、回调、事件等这些词语常常会混淆。从实际效果而言，异步和非阻塞均达到了并行IO的目的，然而，对于计算机内核而言，异步/同步与非阻塞/非阻塞是两回事。

下面几篇文章分析介绍了同步异步与阻塞非阻塞之间的关系
- [使用异步I/O大大提高应用程序的性能](https://www.ibm.com/developerworks/cn/linux/l-async/)
- [asynchronous vs non-blocking](https://stackoverflow.com/questions/2625493/asynchronous-vs-non-blocking)
- [知乎：怎样理解阻塞/非阻塞和同步/异步的区别](https://www.zhihu.com/question/19732473)
- [How does a single thread handle ajax in javascript](https://www.quora.com/How-does-a-single-thread-handle-asynchronous-code-in-JavaScript)

#### Node的异步IO实现

Node完成整个异步IO环节的有事件循环、观察者和请求对象以及线程池等。

##### 事件循环
![](http://odf594a9x.bkt.clouddn.com/node-io0.png)

进程启动时，Node便创建一个循环体（Tick），每执行一次Tick，要查看是否有事件需要处理，如果有，就取出对应的事件及相关的回调函数执行，然后再进入下一个循环,直至没有事件处理。

##### 观察者

事件循环中有一个或者多个观察者，判断是否有事件需要执行就是向这些观察者询问的。

这个过程就如同饭馆的厨房，厨房一轮一轮地制作菜肴，但是要具体制作哪些菜肴取决于收银台收到客人的下单。厨房每做完一轮菜肴，就去问收银台接下来有没有要做的菜，如果没有的话，就下班打烊了。在这个过程中，收银台的小妹就是观察者，她收到客人点单就是关联的回调函数。当然，如果饭店经营有方，它可能有多个收银员，就如同事件循环中有多个观察者一样。收到下单就是一个事件，一个观察者里可能有多个事件。
观察者是跟任务队列存在对应的关系，观察者判断该事件是否可以执行，如果可以，就把该事件推入该观察者对应的任务队列中去。然后事件循环会遍历任务队列中的事件进而进行执行。

##### 请求对象

从JavaScript发起调用到内核执行完I/O操作的过渡中，存在一个中间产物，成为请求对象。

```
var fs = require('fs');
fs.open('./test.js','w',function(err,fd){
  //..do something
});

```
![](http://img2.tbcdn.cn/L1/461/1/fad8e5f6433ca965b3fcf282910ba5bc3bff65cf)
![](http://img4.tbcdn.cn/L1/461/1/a9e67142615f49863438cc0086b594e48984d1c9)

调用fs.open时，Node.js会通过process.binding调用C/C++层面的open函数，然后再调用Libuv的具体方法uv_fs_open方法，最后的结果通过回调的方式传回，完成流程。
在uv_fs_open的调用过程中，创建了一个FSReqWrap请求对象，从js层传入的参数和当前方法都封装在这个请求对象中。

然而Node的异步执行分为可两个步骤
1. 组装好请求对象，送入IO线程池等待执行
2. 线程池的IO操作调用完毕后，会将获取的结果储存在req->result属性上，然后调用PostQueuedCompletionStatus()通知IOCP，告知当前对象操作已经完成

#### 非IO的异步API
- setTimeout
- setInterval
- setImmediate
- Process.nextTick

setTimeout或setInterval创建的定时器会被插入到定时器观察者内部的红黑树。每一次Tick执行，会从该红黑树中迭代取出定时器对象，检查是否超过定时时间，如果超过，就形成一个事件，其回调函数将立即执行。setImmediate与Process.nextTick 类似，都是延迟执行。但是Process.nextTick()中的回调函数执行优先于setImmediate，原因在于事件循环中对观察者的检查有先后顺序，process.nextTick属于idle观察者，setImmediate属于check观察者，在每一次循环中，idle观察者先于IO观察者，IO观察者先于check观察者。

最后在补充一张JS的事件执行模型图
![](http://ww4.sinaimg.cn/large/006bEpFbgw1f8jw4zax6dj30so0l2wfo.jpg)
