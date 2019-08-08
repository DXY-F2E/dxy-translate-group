# 函数组件和类组件到底哪里不同

> 原文：[How Are Function Components Different from Classes?](https://overreacted.io/how-are-function-components-different-from-classes/)

React里的函数组件和类组件有哪些不同？

一度，两者的区别在于类组件能提供更多的能力（比如局部的状态）。但是有了[Hooks](https://reactjs.org/docs/hooks-intro.html)之后，情况却有所不同了。

也许之前你听说性能也是两者的差别。但是哪个性能更好？不好说。我一直很谨慎地对待这类[结论](https://github.com/ryardley/hooks-perf-issues/pull/2)，因为很多性能测试都是[不全面](https://medium.com/@dan_abramov/this-benchmark-is-indeed-flawed-c3d6b5b6f97f?source=your_stories_page---------------------------)的。性能主要还是在于代码的逻辑而非你选择了函数组件或者类组件。据我们观察，虽然两者的优化策略有所差别，但是整体来说性能区别是微不足道的。

无论哪种情况下，我们都[不推荐](https://reactjs.org/docs/hooks-faq.html#should-i-use-hooks-classes-or-a-mix-of-both)用Hooks重写你现有的组件。除非你有其他原因并且不介意做第一个吃螃蟹的人。因为Hooks还不是很成熟（就像2014年的React），有一些“最佳实践”尚待发掘。

所以我们该怎么办？函数组件和类组件本质上难道没有区别吗？当然有，两者在心智模型上有区别，也就是接下来我要说的**两者之间最大的区别**。早在2015年，函数组件被引入的时候就存在了，只是它常常被忽略。

> **函数组件能够捕获渲染后的值**

下面让我们一起来理解一下这句话。

---

**注意: 本文只是描述函数组件和类组件两者在React编程模型中的区别，不涉及两者之间的取舍。有关函数组件接受度更高的问题，可以参考[Hooks FAQ](https://reactjs.org/docs/hooks-faq.html#adoption-strategy)。**

---

假如有如下组件：

```javascript
function ProfilePage(props) {
  const showMessage = () => {
    alert('Followed ' + props.user);
  };

  const handleClick = () => {
    setTimeout(showMessage, 3000);
  };

  return (
    <button onClick={handleClick}>Follow</button>
  );
}
```

它有一个按钮，通过setTimeout来模拟网络请求，然后显示一个确认弹窗。如果props.user是Dan，3秒后弹窗就会显示‘关注了Dan’，很简单。

（注意我在例子中虽然使用了箭头函数和普通函数，但是两种声明方式没什么区别。```function handleClick()```也可以正常工作。）

如果用类组件会怎么写呢？最简单的改写方式差不多是这样：

```javascript
class ProfilePage extends React.Component {
  showMessage = () => {
    alert('Followed ' + this.props.user);
  };

  handleClick = () => {
    setTimeout(this.showMessage, 3000);
  };

  render() {
    return <button onClick={this.handleClick}>Follow</button>;
  }
}
```

我们一般会觉得这两段代码功能是一样的。人们可能在这两种模式之间来回重构，但是却并未注意到这两种模式分别意味着什么。

**事实上，这两段代码之间的差别是很精妙的。** 仔细看看，你能察觉它们之间的差别吗？反正我是花了很久才看出来。

**前方剧透，如果你想自己琢磨，这里有一个[例子](https://codesandbox.io/s/pjqnl16lm7)。** 接下来我将解释两者的差别以及为什么它这么重要。

---

在解释之前，我想强调一点：我描述的差别和React Hooks本身无关。因为刚才的例子都没用到Hooks！

这个差别只关乎函数组件和类组件。你应该会想要弄明白如果你经常用到函数组件的话。

---

我们会通过一个React应用里常见的Bug来说明这个差别。

打开这个[示例](https://codesandbox.io/s/pjqnl16lm7)，里面有一个表示当前用户的下拉框以及上面实现的两个关注组件 -- 每个都渲染了一个关注按钮。

按照下面的顺序，分别操作两个的按钮：
1. **点击**其中一个关注按钮
1. 在3秒内**改变**下拉框中选择的用户
1. **观察**弹窗中出现的文字

你会看到一个奇怪的现象：
* 函数组件中，点击关注Dan，然后切换到Sophie，弹窗仍然显示‘关注了Dan’
* 类组件中，弹窗却显示‘关注了 Sophie’：

![Demonstration of the steps](https://overreacted.io/bug-386a449110202d5140d67336a0ade5a0.gif)

在这个例子里，表现正确的应该是第一个。**如果我点击关注了一个人，然后切换到另一个人，我的组件应该知道我最后关注了谁。** 类组件的表现很明显不正确。

(但是你真的应该关注[Sophie](https://mobile.twitter.com/sophiebits)。)

---