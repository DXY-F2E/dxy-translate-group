# 函数组件和类组件到底哪里不同

> 原文：[How Are Function Components Different from Classes?](https://overreacted.io/how-are-function-components-different-from-classes/)

React里的函数组件和类组件有哪些不同？

一度，两者的区别在于类组件能提供更多的能力（比如局部的状态）。但是有了[Hooks](https://reactjs.org/docs/hooks-intro.html)之后，情况却有所不同了。

也许之前你听说性能也是两者的差别。但是哪个性能更好？不好说。我一直很谨慎地对待这类[结论](https://github.com/ryardley/hooks-perf-issues/pull/2)，因为很多性能测试都是[不全面](https://medium.com/@dan_abramov/this-benchmark-is-indeed-flawed-c3d6b5b6f97f?source=your_stories_page---------------------------)的。性能主要还是在于代码的逻辑而非你选择了函数组件或者类组件。据我们观察，虽然两者的优化策略有所差别，但是整体来说性能区别是微不足道的。

无论哪种情况下，我们都[不推荐](https://reactjs.org/docs/hooks-faq.html#should-i-use-hooks-classes-or-a-mix-of-both)用Hooks重写你现有的组件。除非你有其他原因并且不介意做第一个吃螃蟹的人。因为Hooks还不是很成熟（就像2014年的React），有一些“最佳实践”尚待发掘。

所以我们该怎么办？函数组件和类组件本质上难道没有区别吗？当然有，两者在心智模型上有区别，也就是接下来我要说的**两者之间最大的区别**。早在2015年，函数组件被引入的时候就存在了，只是它常常被忽略。

> **函数组件能够捕获渲染后的值**

