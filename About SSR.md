## 什么是 SSR

之前听 ssr 的概念，只有一个大的概念，简单地以为 ssr 就是和旧时代的 jsp、php 一样，在服务器端就把页面渲染好交给浏览器解析。

如今看来，这里至少有两个巨大的认知误区：首先，php 不旧，直到现在还有很多拥簇；实现流程千差万别，可以说 ssr 比之前的模板渲染复杂很多。

ssr 的目标只有一个——增加网站的 seo、提高**首屏**加载速度。也就是说，ssr 完全可以和 csr，甚至 isr（增量式更新）混合使用。也就说，ssr 只管页面一开始加载的那一回事，后面就由 csr 接管了。

ssr 有很多好处，比如可以访问服务器端的数据、key 之类的敏感数据不容易丢失等等。但限制也很多，首当其冲的是不能使用 window 等一系列浏览器 api。

## SSR 流程

为了提高首屏渲染速度（这一点有待考究）和 seo 优化，我们需要在服务器端通过 react 提供的 api 接口，提前渲染 react 组件，然后把一些需要交互的以及无法在服务器端渲染的组件**打上标记**，发送到浏览器后，浏览器的 react 会再次走一遍渲染流程，把需要交互的事件绑定到组件上、渲染浏览器端组件，这个过程被称为**水合**，这样的流程也被称为**同构渲染**。

![图片](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGhyxW0kCGf3ZJtHQamQQMlmXkKgJ5W8w9JeHdawMmkKfjUZSzjxltMOGhFqoFTOBswDibgofOyjECQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

为什么说 react 的 ssr 对于性能的提升有待考究：react 通过一套复杂的算法，在服务端和浏览器端实现了两套渲染流程，虽然 ssr 提高了首屏加载的时候，那么在服务器端渲染的事件又占用了多少呢？

或许 react 的算法真的足够厉害，可以让看似更复杂的同构渲染比直接扔在浏览器里渲染一次的事件更短。

不然 nextjs 也不会默认启用 ssr 了，对吧！

## 其他

总而言之，ssr 对于 ux 来说，实在不敢恭维。nextjs 默认使用 ssr，所以在 nextjs 中构建应用的时候，要注意加上一个 `isBrowser` 的判断；如果真的要突破 ssr 的限制，那就把方法放在 useEffect 里，这些方法在 ssr 阶段不会被执行。

不过对于那些拥有大量静态内容的网站，例如博客、新闻来说，nextjs 的稳定性、完善的工具链、和 react 核心团队的深度合作，似乎已经成为当下唯一最好的选择。

更吸引我的，是未来的 `react server component`（未来其实已来）。简单来说，就是通过流式传输到浏览器再渲染组件。这里可以解决一个很大的痛点，那就是可以在服务器端共享大型依赖，理论上来说，如果一个应用全部都由 server component 组成，那么这个应用就可以是“零依赖”的应用。

---

[React Hooks & SSR](https://ahooks.gitee.io/zh-CN/guide/blog/ssr)  
[React SSR 实现原理：从 renderToString 到 hydrate](https://mp.weixin.qq.com/s/MA6onW57f5LsntgF5mrSHQ)
