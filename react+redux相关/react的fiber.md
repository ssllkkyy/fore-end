## Fiber

## react15的问题

> 页面元素很多，需要频繁的刷新，react15会出现掉帧的问题
>
> 原因
>
> 大量的同步计算操作阻碍了页面ui进程的渲染，如果js运算占用主线程，页面就无法得到及时的更新，调用setState的时候，react会遍历所有的节点计算差异然后更新ui，整个过程不能被打断，如果页面元素很多，就会出现掉帧的情况

## 如何解决

> 解决主线程长时间被js运算占用的问题的基本思路就是把运算切割成多个步骤，分批完成，完成一部分就将控制权交给浏览器进行渲染
>
> 之前的react通过递归的方式进行渲染，使用的js引擎自身的调用栈，会一直执行到栈空为止，Fiber实现了自己的一套组件调用栈，以链表的形式遍历组件树，可以灵活暂停，继续和丢弃执行的任务，实现方式是使用了浏览器的`requestIdleCallback`这一api
>
> #### ```window.requestIdleCallback()会在浏览器空闲时期依次调用函数，这就可以让开发者在主事件循环中执行后台或低优先级的任务，而且不会对像动画和用户交互这些延迟触发但关键的事件产生影响。函数一般会按先进先调用的顺序执行，除非函数在浏览器调用它之前就到了它的超时时间。```
>
>  

## React的解决

## React的内部的运作层

> - Virtual DOM 层，描述页面长什么样。
> - Reconciler 层，负责调用组件生命周期方法，进行 Diff 运算等。
> - Renderer 层，根据不同的平台，渲染出相应的页面，比较常见的是 ReactDOM 和 ReactNative。
>
> ### 改动的是Reconciler 层 ，成为Fiber Reconciler ,以前的 Reconciler 被命名为`Stack Reconciler`
>
>  

## 任务优先级

> - synchronous，与之前的Stack Reconciler操作一样，同步执行
> - task，在next tick之前执行
> - animation，下一帧之前执行
> - high，在不久的将来立即执行
> - low，稍微延迟执行也没关系
> - offscreen，下一次render时或scroll时才执行

## Fiber Reconciler 执行过程的阶段

Fiber Reconciler 在执行过程中，会分为 2 个阶段

> - 阶段一，生成 Fiber 树，得出需要更新的节点信息。这一步是一个渐进的过程，可以被打断。
> - 阶段二，将需要更新的节点一次过批量更新，这个过程不能被打断。

## 总结

> 在源码层面是干了一件递归改循环的事情.



## 以上摘抄自  https://segmentfault.com/a/1190000018250127?utm_source=tag-newest

## 我的理解

> 当进行页面首次渲染的时候是直接一次生成的，后续如果有节点更新的时候，就根据diff构建一个新的树，这个树每生成一个新的节点就把控制权交给主线程去检查，如果主线程有优先级更高的任务需要执行的时候就丢弃正在生成的树，在空闲的时候重新执行一遍，如果没有就继续构建树
>
> 在空闲的时候重新执行一遍 得益于 浏览器的`requestIdleCallback`这一api