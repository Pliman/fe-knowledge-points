组件的实现参考了一个原生的js实现的下拉加载的例子。

有了前人踩过的坑，我们不再需要大费周张的使用原生js造一遍轮子。

此处就直奔主题，看看如何用React实现同样的功能。

#### Debounce & Throttle

实现组件之前，我们需要理解js中常见的函数调用频度控制的方法：`Debounce和Throttle`。

`Throttle`顾名思义就是函数截流，它主要用来**控制两次函数调用之间的时间间隔**。举个例子，如果我们有一个函数`onResize`需要在窗口大小调整时触发:

```

function onResize() {
    console.log(new Date().getTime());

}

document.addEventListener('onresize', onResize);

```

如果不使用throttle，当窗口缩放时，`onResize`函数会以极高的频率被调用，如果在低版本的ie浏览器下，它很可能造成浏览器假死，因此，我们需要控制`onResize`函数调用的频率，保证两次`onResize`函数之间的调用间隔大于某个值:

```

    function throttle(func, deltaX) {

        let lastCalledAt = new Date().getTime();

        return function() {

            if(new Date().getTime() - lastCalledAt >= deltaX) {

                func();

                lastCalledAt = new Date().getTime();

            }

        }

    }



    let throttleResize = throttle(onResize, 400);



    document.addEventListener('onresize', throttleResize);

```

与`Throttle`不同，`Debounce`意味着**当事件发生时，我们不会立即激活回调，而是等待一定的时间间隔，如果相同的事件没有再次被出发，则激活回调**。还是上面的resize问题，如果使用debounce，可以这样实现:

```

function debounce(func, timer) {
    let timeoutRef = null;

    return function() {

        if(timeoutRef) {

            clearTimeout(timeoutRef);

        }

        timeoutRef = setTimeout(func, timer);

    }

}



let debResize = debounce(onResize, 400);

document.addEventListener('onresize', debResize);

```

当然，这篇文章不会深入去探究生产环境上的throttle和debounce函数的实现，如果希望进一步了解这两个方法，可以看看[js魔法堂：函数截流/去抖](http://www.cnblogs.com/fsjohnhuang/p/4147810.html)

#### 对Scroll事件的处理

既然我们花了这么大的篇幅来介绍debounce和throttle事件，在实际生产中，我们是不是会采用他们两者中的某一个来处理滚动事件呢?答案是否定的。

在安卓设备上，当用户滚动页面时，滚动事件会以极高的频率被触发；然而，在ios设备上，情况则大相径庭：滚动事件只会在滚动动画停止时触发。

考虑到scroll事件在这两种设备上的差异性，此处我们放弃了在滚动事件里注册我们的回调函数，转而将其放在一个定时触发的onScrollHandler函数中。当onScrollHandler函数被调用时，我们会判断当前ScrollTop的位置是否发生变化，如果发生变化且位于屏幕底部，我们就会调用ajax回调函数，加载新的内容。实际上，通过setInverval函数，我们也实现了对滚动事件的截流，保证了滚动事件的触发频次不会过载。因此，你也可以把这种方法看做是一个变相的`throttle`。

简单看看scrollHandler的实现:

```

  isScrolling() {

    const { scrollTop } = document.body;

    const { clientHeight, scrollHeight } = document.documentElement;



    return scrollTop + clientHeight + 40 >= scrollHeight;

  }



  handleScroll(e, force) {

    if(!force && this.lastScrollY === window.scrollY) {

      return;

    } else {

      this.lastScrollY = window.scrollY;

    }



    if(this.isScrolling()) {

      this.props.getNextPage();

    }

  }

```

然后我们在componentDidMount函数中，定时调用该函数:

```

componentDidMount() {

    setInterval(handleScroll, 100);

}

```

#### LazyLoad

优化了滚动事件，我们还能做什么呢？

如果我们的列表中存在大量图片，当我们下拉刷新列表时，浏览器会同时下载所有的图片文件。然而，对于一个真实的用户，他并不是每次都会将列表一拉到底，这时，列表底部很多图片都成了无意义的流量消耗和带宽占用，同时因为这些图片的加载，反而拖累了用户可视范围内的图片加载速度。对于这些无意义的图片加载，我们可以通过懒加载来进行避免。

懒加载，顾名思义就是当图片出现或接近可视区域时，我们才加载它。那么，要实现懒加载，首先就需要获取当前可视区域。

如何获取可以参考下图:



获取了可视区域之后，我们只需要定时检查图片是否接近可视区域。如果接近，则对img的src属性赋值:

```

  /**@todo 

   * 判断img是否进入了可视区域

   * 若已经进入可视区域，则懒加载

   */

  isVisible() {

    let { img } = this.refs;

    let { offsetTop, offsetHeight } = img;

    let { innerHeight, scrollY } = window;

    let bottomViewPort = scrollY + innerHeight + 200;

    let topViewPort = scrollY - 200;;



    if(offsetTop + offsetHeight > topViewPort && 

      offsetTop < bottomViewPort) {

        this.loadImg();



        return true;

    } 



    setTimeout(this.isVisible, 100);



    return false;

  }

```

#### 样式

除了上面的技巧，一个简单的trick也可以优化用户的感知：在scrollList的图片加载时添加一个简单的淡入效果就可以让用户觉得内容的加载速度变快了。（对我来说似乎免疫了）

这个用简单的css样式就可以实现：

```

{

    opacity: 0,  

    transition: opacity 0.25s ease-in-out;

}

```

当元素进入可视范围时，将他的opacity改为1即可。

#### Cache

除了上面这些方法，淘宝h5还采用了DOM回收和Cache方法来优化性能。虽然DOM节点并非耗能大户，但如果节点足够多，依然会对网站的流畅性带来影响，淘宝就采用了下图所示的方法对DOM节点进行回收和缓存:



当用户向下滑动时，最顶部的DOM节点内容会被缓存，通过节点会被销毁（回收）。一个新的节点会被添加到列表最底部。这样保证了屏幕中节点的数量始终可控。考虑到这项技术的实现和带来的收益并不成正比，因此大部分情况下可以不考虑。

#### 墓碑元素

如果用户向下滑动的速度够快，我们可以添加一些空白的占位元素，来保证用户滑动的流畅性。这个的实现可以参考facebook。由于它更多考虑的是对用户体验而非性能的优化，此处就不赘述。

#### 总结

总的来说，对一个长度无限的下拉加载列表优化方法主要就是以下几种：图片懒加载 + 优化onscroll方法 + css淡入效果 + cache。在实际项目中，可以酌情选择一种或几种方法。



延伸阅读: 

[无限下拉加载](https://juejin.im/post/58b545f0b123db005734634e)

[setTimeout vs setInterval](http://stackoverflow.com/questions/729921/settimeout-or-setinterval)

[What is offsetHeight, clientHeight, scrollHeight](http://stackoverflow.com/questions/22675126/what-is-offsetheight-clientheight-scrollheight)

