---
title: '开发模式是如何工作的'
date: 2019-08-06
---

> 原文地址: https://overreacted.io/how-does-the-development-mode-work/
> 原文作者: 
>
>

<details open>
  <summary>如果你的 JavaScript 代码库已经达到的中等复杂度, 你也许可以选择在 development 和 production 打包和运行不同的代码.</summary>

  If your JavaScript codebase is even moderately complex, you probably have a way to bundle and run different code in development and production.
</details>

<details open>
  <summary>
    在 development 和 production 打包和运行不同的代码是强大的. 在开发模式下, React 包含很多警告, 来帮助你在引入 bug 之前找到对应的问题. 但是, 探测这些错误的必要的代码经常会增加包体积和使 app 运行比较缓慢.
  </summary>
  Bundling and running different code in development and production is powerful. In development mode, React includes many warnings that help you find problems before they lead to bugs. However, the code necessary to detect such mistakes often increases the bundle size and makes the app run slower.
</details>

<details open>
  <summary>
    development 下这个缓慢是可以接受的. 实际上, 在 development 下运行代码比较缓慢也许甚至是有裨益的, 因为其部分补偿了在快速的开发机器和平均水平的消费者机器之间的差异.
  </summary>
  The slowdown is acceptable in development. In fact, running the code slower in development might even be beneficial because it partially compensates for the discrepancy between fast developer machines and an average consumer device.
</details>

<details open>
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
  doSomethingDev()
} else {
  doSomethingProd()
}

// in Production
if (false) {
  doSomethingDev()
} else {
  doSomethingProd()
}
```
