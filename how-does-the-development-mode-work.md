---
title: '开发模式是如何工作的'
date: 2019-08-06
---

> 原文地址: https://overreacted.io/how-does-the-development-mode-work/
>
> 原文作者:  [Dan Abramov](https://mobile.twitter.com/dan_abramov)
>

<details>
  <summary>如果你的 JavaScript 代码库已经达到的中等复杂度, 你也许可以选择在 development 和 production 打包和运行不同的代码.</summary>

  If your JavaScript codebase is even moderately complex, you probably have a way to bundle and run different code in development and production.
</details>

<details>
  <summary>
    在 development 和 production 打包和运行不同的代码是强大的. 在 development 模式下, React 包含很多警告, 来帮助你在引入 bug 之前找到对应的问题. 但是, 探测这些错误的必要的代码经常会增加包体积和使 app 运行比较缓慢.
  </summary>
  Bundling and running different code in development and production is powerful. In development mode, React includes many warnings that help you find problems before they lead to bugs. However, the code necessary to detect such mistakes often increases the bundle size and makes the app run slower.
</details>

<details>
  <summary>
    在开发环境中, 这个缓慢是可以接受的. 实际上, 在开发环境中运行代码比较缓慢也许甚至是有裨益的, 因为其部分补偿了在快速的开发机器和平均水平的消费者机器之间的差异.
  </summary>
  The slowdown is acceptable in development. In fact, running the code slower in development might even be beneficial because it partially compensates for the discrepancy between fast developer machines and an average consumer device.
</details>

<details>
  <summary>
    生产环境中, 我们不想要付出任何的这种消耗. 因此, 在生产环境中我们省略了这些检查. 这是如何工作的? 让我们来看一看.
  </summary>
  In production we don’t want to pay any of that cost. Hence, we omit these checks in production. How does that work? Let’s take a look.
</details>

-------

<details>
  <summary>
    在开发环境中运行不同的代码的确切方式依赖你的 JavaScript 构建流程(以及是否有一个). 在 Facebook 它看起来像这样:
  </summary>
  The exact way to run different code in development depends on your JavaScript build pipeline (and whether you have one). At Facebook it looks like this:
</details>

```js
if (__DEV__) {
  doSomethingDev();
} else {
  doSomethingProd();
}
```

<details>
  <summary>
    这里, `__DEV__` 并不是一个真正的变量.  当 modules 为了浏览器被一起拼接时, 其将会被替换为一个常量, . 结果看起来像这样:
  </summary>
  Here, `__DEV__` isn’t a real variable. It’s a constant that gets substituted when the modules are stitched together for the browser. The result looks like this:
</details>

```js
// in Development:
if (true) {
  doSomethingDev()  // 👈
} else {
  doSomethingProd()
}

// in Production:
if (false) {
  doSomethingDev()
} else {
  doSomethingProd() // 👈
}
```

<details>
  <summary>
    在生产环境中, 你还要在代码上运行一个压缩器(minifier)(比如 terser). 大多数的 JavaScript 压缩器(minifier) 会做一些有限形式的 <a href="https://zh.wikipedia.org/wiki/死碼刪除">死码删除</a>, 比如移除 <code>if (false)</code> 分支, 因此在生产环境中你将只会看到:
  </summary>
  In production, you’d also run a minifier (for example, terser) on the code. Most JavaScript minifiers do a limited form of dead code elimination, such as removing if (false) branches. So in production you’d only see:
</details>

```js
// In production (after minification)
doSomethingProd()
```

<details>
  <summary>
    <em>(注意, 主流的 JavaScript 工具对于有效的死码移除是非常有限的, 不过这是另一个话题了.)</em>
  </summary>
  <em>(Note that there are significant limits on how effective dead code elimination can be with mainstream JavaScript tools, but that’s a separate topic.)</em>
</details>

<details>
  <summary>
    虽然你可能没有使用 `__DEV__` 魔法常量, 如果你使用像 webpack 之类的流行的 JavaScript 打包工具, 这里或许有一些约定你可以遵循. 例如, 像这样的表达相同模式是常见的:
  </summary>
  While you might not be using a __DEV__ magic constant, if you use a popular JavaScript bundler like webpack, there’s probably some other convention you can follow. For example, it’s common to express the same pattern like this:
</details>

```js
if (process.env.NODE_ENV === 'production') {
  doSomethingDev()
} else {
  doSomethingProd()
}
```

<details>
  <summary>
    <b>当你使用打包器从 npm 导入一些例如 React, Vue 的库的时候, 它们正是使用了这种模式.</b> (单文件的 <code>&lt;script&gt;</code> 标签构建为开发环境和生产环境提供了不同的 <code>.js</code> 和 <code>.min.js</code> 文件.)
  </summary>
  <b>That’s exactly the pattern used by libraries like React and Vue when you import them from npm using a bundler.</b> (Single-file <code>&lt;script&gt;</code> tag builds offer development and production versions as separate <code>.js</code> and <code>.min.js</code> files.)
</details>

<details>
  <summary>
    这个特殊的约定最初来源于 Node.js. 在 Node.js 中, 有一个全局的 <code>process</code> 变量, 其暴露系统的的环境变量作为 <code>process.env</code> 对象的属性. 但是, 当你在前端的代码库中看到这个模式, 这里通常没有任何真正的 <code>process</code> 变量介入. 🤯
  </summary>
  This particular convention originally comes from Node.js. In Node.js, there is a global <code>process</code> variable that exposes your system’s environment variables as properties on the <code>process.env</code> object. However, when you see this pattern in a front-end codebase, there isn’t usually any real <code>process</code> variable involved. 🤯
</details>

<details>
  <summary>
    代替的是, 整个 <code>process.env.NODE_ENV</code> 表达式都会在构建时期被替换成一个字符串字面量, 就好像我们的魔法 <code>__ENV__</code> 变量:
  </summary>
  Instead, the whole <code>process.env.NODE_ENV</code> expression gets substituted by a string literal at the build time, just like our magic <code>__ENV__</code> variable:
</details>

```js
// In development:
if ('develpment' !== 'production') {
  doSomethingDev(); // 👈
} else {
  doSomethingProd();
}

// In production:
if ('production' !== 'production') {
  doSomethingDev();
} else {
  doSomethingProd(); // 👈
}
```

<details>
  <summary>
    因为整个表达式是固定的 (<code>'production' !== 'production'</code> 保证为 <code>false</code>), 压缩器也能移除其他的分支.
  </summary>
  Because the whole expression is constant (<code>'production' !== 'production'</code> is guaranteed to be <code>false</code>), a minifier can also remove the other branch.
</details>

```js
// In production(after minification):
doSomethingProd();
```

<details>
  <summary>
    恶作剧完成.
  </summary>
  Mischief managed.
</details>

---

<details>
  <summary>
    注意, 带有更复杂的表达式的这个将不会工作:
  </summary>
  Note that this wouldn’t work with more complex expressions:
</details>

```js
let mode = 'production';
if (mode !== 'production') {
  // 🔴 not guaranteed to be eliminated
}
```

<details>
  <summary>
    JavaScript 静态分析工具还不是太智能, 因为该门语言的动态天性.当它们看到诸如 <code>mode</code> 这种变量而不是像 <code>false</code> 或者是 <code>'production' !== 'production'</code> 这种静态表达式的时候, 它们往往会放弃.
  </summary>
  JavaScript static analysis tools are not very smart due to the dynamic nature of the language. When they see variables like <code>mode</code> rather than static expressions like <code>false</code> or <code>'production' !== 'production'</code>, they often give up.
</details>

<details>
  <summary>
    相似的, 当你使用顶层 import 语句的时候, 在穿越模块边界的死码移除在 JavaScript 中通常也不会正常工作:
  </summary>
  Similarly, dead code elimination in JavaScript often doesn’t work well across the module boundaries when you use the top-level import statements:
</details>

```js
import {someFunc} from 'some-module';

if (false) {
  someFunc()
}
```

<details>
  <summary>
    因此你需要以非常机械的方式写书写代码, 使得条件绝对静态, 并且确保你想要移除的所有代码都在其中.
  </summary>
  So you need to write code in a very mechanical way that makes the condition definitely static, and ensure that all code you want to eliminate is inside of it.
</details>

<details>
  <summary>
    为了让这些工作, 你的打包器需要去做 <code>process.env.NODE</code> 的替换, 并且需要知道你要在哪种模式下构建该项目.
  </summary>
  For all of this to work, your bundler needs to do the <code>process.env.NODE_ENV</code> replacement, and needs to know in which mode you want to build the project in.
</details>

<details>
  <summary>
    几年以前, 忘了配置环境在过去很普遍. 你会经常看到一个项目在 development 模式下部署到生产环境.
  </summary>
  A few years ago, it used to be common to forget to configure the environment. You’d often see a project in development mode deployed to production.
</details>

<details>
  <summary>
    这很沮丧, 因为它使得网站加载和运行比较慢.
  </summary>
  That’s bad because it makes the website load and run slower.
</details>

<details>
  <summary>
    在最近的两年, 这个情况已经很大的改善. 例如, webpack 增加了一个简单的 <code>mode</code> 选项来替代手动的配置 <code>process.env.NODE_ENV</code> 的替换. 当站点在开发模式下的时候, React Devtools 现在也展示一个红色的 icon, 使其容易发现和甚至是<a href="https://mobile.twitter.com/BestBuySupport/status/1027195363713736704">报告</a>.
  </summary>
  In the last two years, the situation has significantly improved. For example, webpack added a simple <code>mode</code> option instead of manually configuring the <code>process.env.NODE_ENV</code> replacement. React DevTools also now displays a red icon on sites with development mode, making it easy to spot and even <a href="https://mobile.twitter.com/BestBuySupport/status/1027195363713736704">report</a>.
</details>

![C0A26828-419A-413A-B8EE-58A615109FEC.png](https://i.loli.net/2019/09/05/c9H6jgnyqdKNZxD.jpg)

<details>
  <summary>
    像 Create React App, Next/Nuxt, Vue CLI, Gatsby 这样的观点鲜明的设置工具通过分开开发打包和生产打包成为两个单独的命令使其更难去弄乱.(例如, <code>npm start</code> 和 <code>npm run build</code>) 特别的是, 只有生产打包可以部署, 所以开发者再也不会犯这种错误了.
  </summary>
  Opinionated setups like Create React App, Next/Nuxt, Vue CLI, Gatsby, and others make it even harder to mess up by separating the development builds and production builds into two separate commands. (For example, <code>npm start</code> and <code>npm run build</code>.) Typically, only a production build can be deployed, so the developer can’t make this mistake anymore.
</details>

<details>
  <summary>
    这里经常有一个观点, 也许 production 模式需要成为默认, development 模式需要选择性加入. 就个人而言, 我没有发现这个观点是有说服力的. 从 development 模式的警告中获益很多的人一般是刚刚接触库. 他们大概是不知道如何开启的, 并且常常会忽略一些可以在早期就被感知到的一些 bug.
  </summary>
  There is always an argument that maybe the production mode needs to be the default, and the development mode needs to be opt-in. Personally, I don’t find this argument convincing. People who benefit most from the development mode warnings are often new to the library. They wouldn’t know to turn it on, and would miss the many bugs that the warnings would have detected early.
</details>

<details>
  <summary>
    是的, 性能问题是不好的. 但是, 向终端用户提供破损的, 古怪的体验也是如此. 例如, <a href="https://reactjs.org/docs/lists-and-keys.html#keys">React key warning</a> 帮助避免如发送一个消息给错误的人或买了一个错误的产品这类 bug. 开发过程中关掉该警告, 对你和你的用户都会陷入重大的风险中.如果这个默认是关掉的, 随后你发现了开关并且将其打开, 你将拥有太多的警告需要去清理. 因此大部分的人将关掉开关. 这就是为什么其需要从开始就是开启的, 而不是在之后再开启.
  </summary>
  Yes, performance issues are bad. But so is shipping broken buggy experiences to the end users. For example, the <a href="https://reactjs.org/docs/lists-and-keys.html#keys">React key warning</a> helps prevent bugs like sending a message to a wrong person or buying a wrong product. Developing with this warning disabled is a significant risk for you and your users. If it’s off by default, then by the time you find the toggle and turn it on, you’ll have too many warnings to clean up. So most people would toggle it back off. This is why it needs to be on from the start, rather than enabled later.
</details>

<details>
  <summary>
    最后, 即使开发警告是可选入的, 并且开发者知道如何在早起的开发中开启它们, 我们将回到最初的问题, 一些人将会意外的在部署生产的时候保持它们是开启的!
  </summary>
  Finally, even if development warnings were opt-in, and developers knew to turn them on early in development, we’d just go back to the original problem. Someone would accidentally leave them on when deploying to production!
</details>

<details>
  <summary>我们又回到了原点.</summary>
  And we’re back to square one.
</details>

<details>
  <summary>个人而言, 我相信, 工具显示和使用正确的模式取决于你是在调试还是在部署. 除了 web 浏览器, 几乎所有其他环境(无论 mobile, desktop, 或者是 server)都已经有一个方式去加载和区分开发和生产构建, 这已经存在了数十年了.</summary>
  Personally, I believe in tools that display and use the right mode depending on whether you’re debugging or deploying. Almost every other environment (whether mobile, desktop, or server) except the web browser has had a way to load and differentiate development and production builds for decades.
</details>

<details>
  <summary>也许是时候 JavaScript 环境视该区别为第一类需求, 而不是由库提出并依赖特别的约定.</summary>
  Instead of libraries coming up with and relying on ad-hoc conventions, perhaps it’s time the JavaScript environments see this distinction as a first-class need.
</details>

-----------

<details>
  <summary>足够的哲学!</summary>
  Enough with the philosophy!
</details>

<details>
  <summary>让我们再看看这段代码:</summary>
  Let’s take another look at this code:
</details>

```js
if (process.env.NODE_ENV !== 'production') {
  doSomethingDev();
} else {
  doSomethingProd();
}
```

<details>
  <summary>你也许会疑惑: 如果这里在前端代码中没有真正的 <code>process</code>  对象, 为什么像 React 和 Vue 这样的库会在 npm 构建的时候依赖它?</summary>
  You might be wondering: if there’s no real <code>process</code> object in front-end code, why do libraries like React and Vue rely on it in the npm builds?
</details>

<details>
  <summary>再次阐明: 你可以在 browser 中使用 &lt;script&gt; 标签加载, React 和 Vue 都提供, 不依赖此(译者注: 指的是上述的 process 对象). 但是你需要手动的在开发环境中的 .js 和 生产环境中的 .min.js 文件中选择. 接下来的只是关于通过一个打包器从 npm 导入来使用 React 和 Vue.</summary>
  (To clarify this again: the &lt;script&gt; tags you can load in the browser, offered by both React and Vue, don’t rely on this. Instead you have to manually pick between the development .js and the production .min.js files. The section below is only about using React or Vue with a bundler by importing them from npm.)
</details>

<details>
  <summary>就像在程序中的许多东西, 该特殊的约定也多数是历史原因. 我们继续使用它是因为现在他已经广泛的被不同的工具适配. 切换到其他是昂贵的并且没有太多收益.</summary>
  Like many things in programming, this particular convention has mostly historical reasons. We are still using it because now it’s widely adopted by different tools. Switching to something else is costly and doesn’t buy much.
</details>

<details>
  <summary>那么其背后的历史是什么?</summary>
  So what’s the history behind it?
</details>

<details>
  <summary>
    在 import 和 export 语法被标准化之前的许多年, 有几种竞争的方式去传达在模块之间的关系. node.js 使 require() 和 module.export 流行起来, 被称为 <a href="https://en.wikipedia.org/wiki/CommonJS">CommonJS</a>.
  </summary>
  Many years before the import and export syntax was standardized, there were several competing ways to express relationships between modules. Node.js popularized require() and module.exports, known as <a href="https://en.wikipedia.org/wiki/CommonJS">CommonJS</a>.
</details>

<details>
  <summary>
    早期在 npm 发布的代码是为了 Node.js 写的. <a href="https://expressjs.com/">Express</a> 曾(也许现在也是?)是 Node.js 最流程的服务端框架, 并且其 <a href="https://expressjs.com/en/advanced/best-practice-performance.html#set-node_env-to-production">使用 <code>NODE_ENV</code> 环境变量</a> 去开启 production 模式. 一些其他包也采用了该约定.
  </summary>
  Code published on the npm registry early on was written for Node.js. <a href="https://expressjs.com/">Express</a> was (and probably still is?) the most popular server-side framework for Node.js, and it <a href="https://expressjs.com/en/advanced/best-practice-performance.html#set-node_env-to-production">used the <code>NODE_ENV</code> environment variable</a> to enable production mode. Some other npm packages adopted the same convention.
</details>

<details>
  <summary>
    像 browserify 之类的早期 JavaScript 打包器想要在前端项目之上使用 npm 的代码成为可能. (是的, <a href="https://blog.npmjs.org/post/101775448305/npm-and-front-end-packaging">回到</a> 了几乎没有人在前端中使用 npm的时期! 你能想象吗?) 所以他们将已经存在于 Node.js 生态的相同约定扩展进了前端的代码中.
  </summary>
  Early JavaScript bundlers like browserify wanted to make it possible to use code from npm in front-end projects. (Yes, <a href="https://blog.npmjs.org/post/101775448305/npm-and-front-end-packaging">back then</a> almost nobody used npm for front-end! Can you imagine?) So they extended the same convention already present in the Node.js ecosystem to the front-end code.
</details>

<details>
  <summary>
    最早的 "envify" 转换器在 2013 被发布. React 差不多也是在同时期开源, 并且在那个时期, npm 和 browserify 似乎是打包前端的 CommonJS 代码最好的解决方案.
  </summary>
  The original “envify” transform was released in 2013. React was open sourced around that time, and npm with browserify seemed like the best solution for bundling front-end CommonJS code during that era.
</details>

<details>
  <summary>
    React 从一开始就提供 npm 构建(除 <code>&lt;script&gt;</code> 标签构建外). 随着 React 流行, 使用 CommonJS 模块编写模块化 JavaScript 并通过 npm 发送前端代码也流行起来.
  </summary>
  React started providing npm builds (in addition to <code>&lt;script&gt;</code> tag builds) from the very beginning. As React got popular, so did the practice of writing modular JavaScript with CommonJS modules and shipping front-end code via npm.
</details>

<details>
  <summary>
    React 需要移除在 production 模式中移除 development-only 的代码. Browserify 已经给这个问题提供了解决方案, 所以 React 也采用了在 npm 的构建中使用 <code>process.env.NODE_ENV</code> 的约定. 经过时间的推移, 许多其他的工具和库, 包括 webpack 和 Vue, 也是这样做的.
  </summary>
  React needed to remove development-only code in the production mode. Browserify already offered a solution to this problem, so React also adopted the convention of using <code>process.env.NODE_ENV</code> for its npm builds. With time, many other tools and libraries, including webpack and Vue, did the same.
</details>

<details>
  <summary>
    到了 2019 年, browserify 已经失去了大量的市场占有率. 但是, 在构建步骤将 <code>process.env.NODE_ENV</code> 替换成 'development' 或者是 'production' 是一种非常流行的约定.
  </summary>
  By 2019, browserify has lost quite a bit of mindshare. However, replacing <code>process.env.NODE_ENV</code> with 'development' or 'production' during a build step is a convention that is as popular as ever.
</details>

<details>
  <summary>
    <i></i>
  </summary>
  <i>(It would be interesting to see how adoption of ES Modules as a distribution format, rather than just the authoring format, changes the equation. Tell me on Twitter?)</i>
</details>

---

<details>
  <summary>
    有一件事情有可能依然在迷惑你, 在Github 上面的 React 源码, 你将看到 __DEV__ 还在作为魔术变量来使用. 但是在 npm 的 React 代码, 其使用 process.env.NODE_ENV. 这是怎么工作的?
  </summary>
  One thing that might still confuse you is that in React source code on GitHub, you’ll see __DEV__ being used as a magic variable. But in the React code on npm, it uses process.env.NODE_ENV. How does that work?
</details>

<details>
  <summary>
    在历史上, 我们在源码中使用 __DEV__ 来满足 Facebook 的源码. 很长时间, React 是直接 copy 进入 Facebook 的代码库, 所以它需要跟随相同的规则. 对于 npm, 我们有一个构建步骤, 其按字面的在发布之前将 __DEV__ 检查替换成 process.env.NODE_ENV !== 'production'.
  </summary>
  Historically, we’ve used __DEV__ in the source code to match the Facebook source code. For a long time, React was directly copied into the Facebook codebase, so it needed to follow the same rules. For npm, we had a build step that literally replaced the __DEV__ checks with process.env.NODE_ENV !== 'production' right before publishing.
</details>

<details>
  <summary>
    这有时候是一个问题. 有时, npm 上工作正常的一个代码模式依赖一些 Node.js 的约定, 但是破坏了 Facebook, 反之亦然.
  </summary>
  This was sometimes a problem. Sometimes, a code pattern relying on some Node.js convention worked well on npm, but broke Facebook, or vice versa.
</details>

<details>
  <summary>
    这意味着, 在 React 源码使用 if (__DEV__) 的时候, 我们实际上对每一个包生成两个 bundle. 一个已经用 __DEV__ = true 预编译, 另一个用 __DEV__ = false 预编译. 在 npm 中的每一个包的入口点决定导出哪一个.
  </summary>
  This means that while the React source code says if (__DEV__), we actually produce two bundles for every package. One is already precompiled with __DEV__ = true and another is precompiled with __DEV__ = false. The entry point for each package on npm “decides” which one to export.
</details>

<details>
  <summary>
    例如:
  </summary>
  For Example:
</details>

```js
if (process.env.NODE_ENV === 'production') {
  module.exports = require('./cjs/react.production.min.js');
} else {
  module.exports = require('./cjs/react.development.js');
}
```

<details>
  <summary>
    并且这里是你的打包器将 <code>'development'</code> 或者是 <code>'production'</code> 作为字符串插入的唯一地方, 并且这里是你的压缩器摆脱 development-only <code>require</code> 的地方.
  </summary>
  And that’s the only place where your bundler will interpolate either <code>'development'</code> or <code>'production'</code> as a string, and where your minifier will get rid of the development-only <code>require</code>.
</details>

<details>
  <summary>
    react.production.min.js 和 react.development.js 两者都不在有任何的 <code>process.env.NODE_ENV</code> 检查. 这是很棒的, 因为当实际在 Node.js 中运行的时候, 访问 <code>process.env</code> 稍许缓慢. 提前在两个模式下编译 bundle 也让我们优化文件体积更加一致, 无论你使用的是哪种打包器和压缩器.
  </summary>
  Both react.production.min.js and react.development.js don’t have any <code>process.env.NODE_ENV</code> checks anymore. This is great because when actually running on Node.js, accessing <code>process.env</code> is somewhat slow. Compiling bundles in both modes ahead of time also lets us optimize the file size much more consistently, regardless of which bundler or minifier you are using.
</details>

<details>
  <summary>
    这才是真正有效的.
  </summary>
  And that’s how it really works!
</details>

<details>
  <summary>
    我希望, 这里会有一个更一流的方式来做, 而不是依赖约定, [but here we are]. 如果模式在所有的 JavaScript 环境中是一流概念, 并且当某些代码没有被认为应该是运行在 development 模式下的时候, 如果浏览器中有一些方式来揭露, 这应该是很棒的.
  </summary>
  I wish there was a more first-class way to do it without relying on conventions, but here we are. It would be great if modes were a first-class concept in all JavaScript environments, and if there was some way for a browser to surface that some code is running in a development mode when it’s not supposed to.
</details>

<details>
  <summary>
    另一方面, 在一个单一的项目中一个约定可以传播至生态是非常迷人的. 在 2010 年 EXPRESS_ENV 成为 NODE_ENV, 并且在 2013 蔓延至前端. 也许该解决方案不是最好的, 但是对于每一个项目, 采用这个的成本是低于说服其他人做一些不同的事情的成本. 这教授了关于采用的自上而下与自下而上的有价值的经验. 了解这种动态如何发挥作用将成功的标准化尝试与失败区分开来.
  </summary>
  On the other hand, it is fascinating how a convention in a single project can propagate through the ecosystem. EXPRESS_ENV became NODE_ENV in 2010 and spread to front-end in 2013. Maybe the solution isn’t perfect, but for each project the cost of adopting it was lower than the cost of convincing everyone else to do something different. This teaches a valuable lesson about the top-down versus bottom-up adoption. Understanding how this dynamic plays out distinguishes successful standardization attempts from failures.
</details>

<details>
  <summary>
    区分 development 和 production 模式是非常有用的技术. 我建议在你的库或者是应用程序中使用它, 做一些在生产环境中做起来太昂贵的这类检查, 但是在开发中去做是非常有价值的(并且往往是极重要的).
  </summary>
  Separating development and production modes is a very useful technique. I recommend using it in your libraries and the application code for the kinds of checks that are too expensive to do in production, but are valuable (and often critical!) to do in development.
</details>

<details>
  <summary>
    正如其他的强大的特性一样, 有一些方法可以滥用它, 这将是我下一篇文章的主题!
  </summary>
  As with any powerful feature, there are some ways you can misuse it. This will be the topic of my next post!
</details>
