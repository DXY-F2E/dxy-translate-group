---
title: '开发模式是如何工作的'
date: 2019-08-06
---

> 原文地址: https://overreacted.io/how-does-the-development-mode-work/
>
> 原文作者: 
>
> 

<details>
  <summary>如果你的 JavaScript 代码库已经达到的中等复杂度, 你也许可以选择在 development 和 production 打包和运行不同的代码.</summary>

  If your JavaScript codebase is even moderately complex, you probably have a way to bundle and run different code in development and production.
</details>

<details>
  <summary>
    在 development 和 production 打包和运行不同的代码是强大的. 在开发模式下, React 包含很多警告, 来帮助你在引入 bug 之前找到对应的问题. 但是, 探测这些错误的必要的代码经常会增加包体积和使 app 运行比较缓慢.
  </summary>
  Bundling and running different code in development and production is powerful. In development mode, React includes many warnings that help you find problems before they lead to bugs. However, the code necessary to detect such mistakes often increases the bundle size and makes the app run slower.
</details>

<details>
  <summary>
    development 下这个缓慢是可以接受的. 实际上, 在 development 下运行代码比较缓慢也许甚至是有裨益的, 因为其部分补偿了在快速的开发机器和平均水平的消费者机器之间的差异.
  </summary>
  The slowdown is acceptable in development. In fact, running the code slower in development might even be beneficial because it partially compensates for the discrepancy between fast developer machines and an average consumer device.
</details>

<details>
  <summary>
    生产环境下, 我们不想要付出任何的这种消耗. 因此, 在生产环境下我们省略了这些检查. 这是如何工作的? 让我们来看一看.
  </summary>
  In production we don’t want to pay any of that cost. Hence, we omit these checks in production. How does that work? Let’s take a look.
</details>

-------

<details>
  <summary>
    在开发环境运行不同的代码的确切的方式依赖你的 JavaScript 构建流程(以及是否有一个). 在 Facebook 它看起来像这样:
  </summary>
  The exact way to run different code in development depends on your JavaScript build pipeline (and whether you have one). At Facebook it looks like this:
</details>

```js
if (__DEV__) {
  doSomethingDev()
} else {
  doSomethingProd()
}
```

<details open>
  <summary>
    这里, `__DEV__` 并不是一个真正的变量. 这是一个常量, 当 modules 为了浏览器被一起拼接时, 其将会被替换. 结果看起来像这样:
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

// in Production
if (false) {
  doSomethingDev()
} else {
  doSomethingProd() // 👈
}
```

<details>
  <summary>
    在生产环境下, 你还要在代码之上运行一个缩小器(minifier)(比如 terser). 大多数的 JavaScript 缩小器(minifier) 会做一些有限制的 <a href="https://zh.wikipedia.org/wiki/死碼刪除">死码删除</a>, 比如移除 <code>if (false)</code> 分支, 因此在生产环境你将会看到:
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
    虽然你可能没有使用 `__DEV__` 魔法变量, 如果你使用像 webpack 之类的流行的 JavaScript 打包工具, 这里很可能有一些约定你可以遵循. 例如, 表达相同模式是非常普遍的, 像这样:
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
    <b>当你使用打包器从 npm import 一些例如 React, Vue 的库的时候, 它们正是使用了这种模式.</b> (单文件 <code>&lt;script&gt;</code> 标签构建为开发环境和生产环境提供了不同的 <code>.js</code> 和 <code>.min.js</code> 文件.)
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
    替换的是, 整个 <code>process.env.NODE_ENV</code> 表达式都会在构建时期被替换成一个字符串字面量, 就好像我们的魔法 <code>__ENV__</code> 变量:
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
    因为整个表达式是固定的 (<code>'production' !== 'production'</code>保证为 <code>false</code>), 一个压缩器同样可以移除其他的分支.
  </summary>
  Because the whole expression is constant (<code>'production' !== 'production'</code> is guaranteed to be <code>false</code>), a minifier can also remove the other branch.
</details>

```js
// In production(after minification):
doSomethingProd();
```

<details>
  <summary>
    恶作剧完成
  </summary>
  Mischief managed.
</details>
