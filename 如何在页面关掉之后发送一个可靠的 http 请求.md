> 本文是 [Alex MacArthur](https://css-tricks.com/author/alexmacarthur/) 在 css-tricks 上的[一篇文章](https://css-tricks.com/send-an-http-request-on-page-exit/)的备忘录。

埋点是一个很常见的需求。常见做法是在用户做出一定行为之后提交一个请求，比如用户点击按钮、跳转到其他页面、提交表单等等。考虑下面这个例子：

```html
<a href="/some-other-page" id="link">Go to Page</a>

<script>
  document.getElementById("link").addEventListener("click", (e) => {
    fetch("/log", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        some: "data",
      }),
    });
  });
</script>
```

不复杂的逻辑。当用户点击 a 标签的时候向服务器会发送一个请求，并且这里**不需要知道服务器的响应，只要发送请求就可以了**。

## 问题

这段代码有什么问题吗？其实是有的：**浏览器并不会保证页面离开之后的请求**。

如果某些行为导致页面「中断」，浏览器不会确保页面进程内的请求被成功发出（[这里](https://developers.google.com/web/updates/2018/07/page-lifecycle-api)描述了什么是「中断」和页面的生命周期模型）。请求的可靠性和以下方面息息相关：

1. 网络连接状态
2. 应用性能
3. 外部服务本身的配置

所以，如果某些数据驱动的业务用这些埋点数据来做一些决策，那可能会有非常严重的潜在问题。

## 问题复现

这个问题很容易复现。在另外一个页面里，我们可以通过在 Chrome Devtools 里限制网速为 `slow 3G`。

[![xThd76.md.png](https://s1.ax1x.com/2022/11/01/xThd76.md.png)](https://imgse.com/i/xThd76)

然后点击链接，会发现请求的状态是 `canceled` 了。

[![xThtXR.md.png](https://s1.ax1x.com/2022/11/01/xThtXR.md.png)](https://imgse.com/i/xThtXR)

并且，这个问题如果通过重写 `window.location` 的方式也会出现。也就是说，无论什么时候和什么方式，只要页面被终止，那些未完成的请求就会有被丢弃的风险。

## 请求为什么会被丢弃

根本原因在于，XHR 请求（包括 XMLHttpReqeust 和 fetch）**都是异步非阻塞的**。一旦请求进入队列之后，就交由浏览器来控制这些请求的生命周期了。

从性能的角度来说，无可厚非。毕竟谁也不希望一个请求直接把页面搞崩了。不过这也意味着我们要承担页面进入终止状态的时候，请求会被丢弃的风险。

Google 给出了[状态定义](https://developers.google.com/web/updates/2018/07/page-lifecycle-api#states)：

> 一旦页面进入「终止」状态，它就会开始被浏览器卸载并从内存中清理掉。在这个状态下，不能有新的任务启用，并且过一段时间后正在进行的任务也会被丢弃。

简单来说，就是浏览器被设计成只要页面不存在了，就不应该再执行队列中相对应的任务。

## 解决方案

### 一、异步变同步

在 Chrome 80 前的版本，可以通过给 XMLHttpRequest 选项加上一个配置可以开启把异步的请求变成同步调用，但这违背了请求 api 的设计初衷，所以被废弃了。

取而代之的方法是，通过 `async + await` 处理异步请求：

```js
document.getElementById("link").addEventListener("click", async (e) => {
  e.preventDefault();

  // Wait for response to come back...
  await fetch("/log", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      some: "data",
    }),
  });

  // ...and THEN navigate away.
  window.location = e.target.href;
});
```

这种做法存在几个问题：

1. 牺牲了用户体验，操作会变得卡顿。用户承担了额外的服务器性能和延时成本
2. 局限性很大。在很多场景下，用户跳转行为并不是我们控制的，比如直接点击浏览器的上一页按钮

### 二、让浏览器保留请求

实际上，浏览器提供了让请求保留的接口的方法。比如 fetch 的 `keepalive` 配置：

```js
<a href="/some-other-page" id="link">Go to Page</a>

<script>
  document.getElementById('link').addEventListener('click', (e) => {
    fetch("/log", {
      method: "POST",
      headers: {
        "Content-Type": "application/json"
      },
      body: JSON.stringify({
        some: "data"
      }),
     keepalive: true
    });
  });
</script>
```

加上这个字段之后，让我们再操作一下之前的例子。

[![xTh6cd.md.png](https://s1.ax1x.com/2022/11/01/xTh6cd.md.png)](https://imgse.com/i/xTh6cd)

当 a 标签被点击的时候，原来被 `canceled` 的请求状态变成了 `unknown`。这是正确的，因为我们不需要知道响应的内容。

## 三、使用 beacon

使用 `navigate.sendBeacon` 可以发送一个[信标请求](https://w3c.github.io/beacon/#sec-processing-model)，同样支持页面中断后发送请求：

```js
navigator.sendBeacon(
  "/log",
  JSON.stringify({
    some: "data",
  })
);
```

beacon 请求不允许自定义头，如果需要发送 JSON 数据，使用 Blob：

```html
<a href="/some-other-page" id="link">Go to Page</a>

<script>
  document.getElementById('link').addEventListener('click', (e) => {
    const blob = new Blob([JSON.stringify({ some: "data" })], { type: 'application/json; charset=UTF-8' });
    navigator.sendBeacon('/log', blob));
  });
</script>
```

那么 beacon 和 fetch 相比有什么好处呢？答案是，beacon 的**发送优先级**更低。

在 Chrome Devtools 里可以看到，如果同时发送 beacon 和 fetch 请求，在 Priority 列里，fetch 为 high，beacon 为 low；而在 Type 列中，beacon 是 ping 类型的。

这里是 beacon 请求的[定义](https://www.w3.org/TR/beacon/)：

> ...定义了一种接口，该接口对其他时间关键型的操作的竞争达到最小，同时会确保此类接口仍得到处理并且送到目的地。

如果是和页面数据不相关的请求，优先级越低越好。换句话说，beacon 请求**不会影响用户正常行为的体验**。

### 四、HTML ping 属性

越来越多的浏览器支持 a 标签的 ping 属性，用法如下：

```html
<a href="http://localhost:3000/other" ping="http://localhost:3000/log">
  Go to Other Page
</a>
```

点击了 a 标签之后，会发送一个包含信息头部的请求：

```js
headers: {
  'ping-from': 'http://localhost:3000/',
  'ping-to': 'http://localhost:3000/other'
  'content-type': 'text/ping'
  // ...other headers
},
```

这里技术上和 beacon 请求类似，但有几个要注意的限制：

1. 只能对 a 标签使用，如果是想追踪用户点击按钮或者提交表单之类的行为则无能为力
2. 浏览器支持还不够好，主流浏览器基本已经支持，除了火狐
3. 不能发送任何自定义的数据

如果不受上述限制的影响，那么 ping 属性是一个很好的选择。因为完全不需要写额外的 js 代码。

## 总结

总而言之，其实目前比较可用的方法就是使用 fetch 的 keepalive 字段或者使用 beacon 请求。

什么时候使用 fetch：

1. 需要发送自定义头部
2. 需要发送 GET 请求
3. 需要支持 IE 等上古浏览器

什么时候使用 beacon：

1. 不需要太多定制化
2. 更优雅和简洁的 api
3. 需要确保请求的低优先级以完善用户体验
