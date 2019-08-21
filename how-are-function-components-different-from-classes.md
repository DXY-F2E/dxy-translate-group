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

所以类组件的表现为什么是这样的？

让我们仔细看下```showMessage```这个方法：

```javascript
class ProfilePage extends React.Component {
  showMessage = () => {
    alert('Followed ' + this.props.user);
  };
```

这个方法从```this.props.user```中读取用户。在React中，props是不可变的，所以它们不会改变。**但是，```this```一直都是可变的。**

事实上，这就是```this```在class中的目的。React通过经常的更新this来保证你在render和生命周期方法中总能读取到最新版本的数据。

所以，当我们请求期间，this.props改变了，```showMessage```方法就会从’太新‘的props中读取用户信息。

这其实展露了一个关于UI本质的现象。如果概念上来说UI是应用当前状态的一种映射，**那处理事件其实也是渲染的一部分结果 - 就像视觉上的渲染输出一样**。我们的处理事件其实”属于“某个特定props和state生成的特定渲染。

但是，延时任务打破了这种关联。我们的```showMessage```回调不再和特定的渲染绑定，所以就”失去“了其正确的props。正是从this中读取这个行为，切断了两者之间的关联。

---

假设函数组件不存在，我们会怎么解决这类问题？

我们想沿着props丢失的地方，以某种方式修复正确的props生成的渲染和showMessage回调之间的关联。

其中一种方法就是尽早地从```this.props```中读取信息，然后显示地传递给延时任务的回调函数。

```javascript
class ProfilePage extends React.Component {
  showMessage = (user) => {
    alert('Followed ' + user);
  };

  handleClick = () => {
    const {user} = this.props;
    setTimeout(() => this.showMessage(user), 3000);
  };

  render() {
    return <button onClick={this.handleClick}>Follow</button>;
  }
}
```

上面这种方法可以奏效。但是，随着时间的推移，这样的代码会变得很啰嗦，并且容易出错。如果需要更多属性怎么办？如果还要访问state呢？如果```showMessage```又调用了别的函数，而这个函数又从props或者state读取了某些属性，我们又会遇到同样的问题。所以我们还是得传递this.props和this.state给每个在showMessage中被调用的函数。

这么做很容易降低效率。而且开发人员很容易忘记，也很难保证一定会按照上面的方法来避免问题，所以经常要去解决这类bug。

同样的，把`alert`代码内联到`handleClick`也解决不了这个问题。我们希望以更细的粒度来组织代码，同时也希望能读取和特定渲染对应的props和state值。**这个问题其实非React才有，-- 任何一个存在类似`this`这类可变数据对象的UI框架都会有**。

可能有人会说，我们是不是可以在构造函数中绑定这些方法？

```javascript
class ProfilePage extends React.Component {
  constructor(props) {
    super(props);
    this.showMessage = this.showMessage.bind(this);
    this.handleClick = this.handleClick.bind(this);
  }

  showMessage() {
    alert('Followed ' + this.props.user);
  }

  handleClick() {
    setTimeout(this.showMessage, 3000);
  }

  render() {
    return <button onClick={this.handleClick}>Follow</button>;
  }
}
```

这样其实根本没什么用。记住，问题在于我们读取`this.props`的时机太晚了 - 和我们用的语法没什么关系！不过，如果我们借助JavaScript的闭包，就可以完全解决这个问题。

平时我们总是避免闭包。因为随着时间推移，闭包中的可变对象会变得很难维护。但是在React中props和state都是不可变的(至少我们是这么推荐的！)。这样就避免了搬起石头砸自己的脚。

这意味着如果你将每次渲染的props或者state关住，你就可以保证他们对应的都是特定渲染的结果：

```javascript
class ProfilePage extends React.Component {
  render() {
    // Capture the props!
    const props = this.props;

    // Note: we are *inside render*.
    // These aren't class methods.
    const showMessage = () => {
      alert('Followed ' + props.user);
    };

    const handleClick = () => {
      setTimeout(showMessage, 3000);
    };

    return <button onClick={handleClick}>Follow</button>;
  }
}
```

你在渲染的时候就”捕获“了props：

![Capturing Pokemon](https://overreacted.io/pokemon-fa483dd5699aac1350c57591770a49be.gif)

这样一来，任何内部的代码(包括`showMessage`)就能保证读取到的都是这次渲染的props。React也就”动不了我们的奶酪”了。

然后我们想加多少函数就能加多少，并且它们都能读取到正确的props或者state。闭包简直是救星！

但是上面的[例子](https://codesandbox.io/s/oqxy9m7om5)看起来有些奇怪。既然都有类了，我们为什么还要在render函数里定义函数，而不直接定义类的方法？

事实上，我们可以通过拿掉“class”的外壳来简化代码：

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

和之前的例子一样，这里的`props`也能被捕获 - 因为React将它作为参数传递。不同于`this`，React不会对这里的`props`对象对任何修改。

如果你在函数定义中将`props`解构，看起来会更加明显。

```javascript
function ProfilePage({ user }) {
  const showMessage = () => {
    alert('Followed ' + user);
  };

  const handleClick = () => {
    setTimeout(showMessage, 3000);
  };

  return (
    <button onClick={handleClick}>Follow</button>
  );
}
```

当父组件以不同的props去渲染`ProfilePage`组件时，React会重新调用`ProfilePage`这个函数。但是我们之前已经触发的事件处理函数仍然“属于”上一次渲染，并且`user`的值以及引用它的`showMessage`回调也“属于”上一次。一切都是原封不动。

这也就是为什么，在这个[demo](https://codesandbox.io/s/pjqnl16lm7)的函数式版本中，点击关注Sophie，然后切换到Sunil，却仍然提示`Followed Sophie`:

![Demo of correct behavior](https://overreacted.io/fix-84396c4b3982827bead96912a947904e.gif)

---

现在，我们理解了React中函数式组件和类组件之间最大的区别就是：

> 函数式组件可以捕获渲染的值。

**在Hooks中，这个规则也同样适用于state。** 假如有下面的例子：

```javascript
function MessageThread() {
  const [message, setMessage] = useState('');

  const showMessage = () => {
    alert('You said: ' + message);
  };

  const handleSendClick = () => {
    setTimeout(showMessage, 3000);
  };

  const handleMessageChange = (e) => {
    setMessage(e.target.value);
  };

  return (
    <>
      <input value={message} onChange={handleMessageChange} />
      <button onClick={handleSendClick}>Send</button>
    </>
  );
}
```

(这里是[在线Demo](https://codesandbox.io/s/93m5mz9w24))

尽管这不是一个很好的聊天应用界面，但是它也能说明同样的问题：如果我发送了一条消息，应用应该清楚地知道我发送了哪条消息。这个函数组件中的`message`捕获了特定渲染的state，而浏览器所点击的事件处理函数也是渲染结果的一部分。所以`message`就是我点击“Send”时输入的值。