## 核心概念

首先，要贯彻一个思想：

> 唯有 State 改变才会引入 React 重渲染

也就是说，其实 `props` 和 `context` 的改变并不会导致重渲染。其实是他们的父级组件 `state` 改变了，导致所有后代组件都进行了一次重渲染。考虑一个这样的组件结构：

```jsx
import React from "react";

function App() {
  return (
    <>
      <Counter />
      <footer>
        <p>Copyright 2022 Big Count Inc.</p>
      </footer>
    </>
  );
}

function Counter() {
  const [count, setCount] = React.useState(0);

  return (
    <main>
      <BigCountNumber count={count} />
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </main>
  );
}

function BigCountNumber({ count }) {
  return (
    <p>
      <span className="prefix">Count:</span>
      {count}
    </p>
  );
}

export default App;
```

当 `count` 变量改变时，`Couter` 的后代 `BingCountNumber` 也会被重新渲染。但父级组件 `App` 是不会被重新渲染的，也包括 `App` 的后代之一 `footer`。

## React 渲染机制

我们可以把每次渲染都当作 React 在拍一张照片（snapshot），这张照片描述了最后 html 生成的结构。当 `count` 变量改变时，React 会重新拍摄一张照片并进行比对，然后只**更新有变化的地方**。

但如上节所说，只要某个组件的 state 发生变化，那么不管其后代组件有没有 props，React 会把它们都更新个遍。

为什么 React 要这么设计呢？因为，React 并不知道你的组件是不是「**每次渲染都能输出一样的结果**」，就比如：

```jsx
function CurrentTime() {
  const now = new Date();
  return <p>It is currently {now.toString()}</p>;
}
```

你会发现，这个组件无论 props 是什么，每次渲染的结果都是不一样的。这种组件被称为「**副作用组件**」，与之相反的概念是「**纯组件**」，也就是**在相同输入下，输出也相同**的组件。

React 开发团队在渲染结果方面一直都是秉承小心谨慎的态度，宁愿牺牲一些性能也不愿意冒风险渲染出不正确的页面（比如 React 在捕获到错误的时候会默认白屏）。

## 创建纯组件

如果我们可以自信地认为自己写的组件就是纯组件，每次渲染输出的结果都是相同的。那么可以使用 `React.memo` 方法包装起来：

```jsx
function Decoration() {
  return <div className="decoration">⛵️</div>;
}
export default React.memo(Decoration);
```

这样在它的父级组件的重渲染的时候，只要 props 没变化，那么这个组件就不会被重新渲染。

那为什么 React 不把这个行为内置到框架中呢？这个逻辑感觉不是更直观，更理所当然吗？原因是**性能**。

作为开发者我们总是高估了「重渲染」的成本，其实 React 经过这么长时间的升级，已经把重渲染这一步优化得很好了。如果一个组件，没有很耗时的计算，但却有很多 props 依赖，那么其实新旧 props 的对比的性能损耗反而会比重新渲染更高。

反之，如果一个组件拥有比较复杂的计算逻辑，但 props 数量不多，那还是建议使用 `React.memo` 来减少无谓的计算。

> 不过，这个现象可能会在未来被改变，React 官方已经在测试「自动 memo」相关的功能了。详情可以看[黄玄的演讲](https://www.youtube.com/watch?v=lGEMwh32soc)。

## Context 的影响

说完 props，再来说说 context：

```jsx
const GreetUser = React.memo(() => {
  const user = React.useContext(UserContext);
  if (!user) {
    return "Hi there!";
  }
  return `Hello ${user.name}!`;
});
```

其实 context 可以被看作隐性的 props。也就是说，上面的代码可以看成下面这样：

```jsx
const GreetUser = React.memo(({ user }) => {
  if (!user) {
    return "Hi there!";
  }
  return `Hello ${user.name}!`;
});
```

那么这个组件的重渲染规则就和上面说的 props 如出一辙。

## 调试

使用` React Developer Tools` 可以调试重渲染现象。在开发者工具中选择 `Profiler` 面板，可以录制一段活动来分析组件的详细渲染情况。

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h7bqiwch77j30ic0a674s.jpg)

或者可以实时高亮被重渲染的组件。

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h7bqjh8l1gj30im08mq39.jpg)

## 例外

有时候可能会碰到一种情况：明明包裹了 `React.memo` 并且 props 也「没有改变」，子组件还是被重渲染了：

```jsx
function App() {
  const dog = {
    name: "Spot",
    breed: "Jack Russell Terrier",
  };
  return <DogProfile dog={dog} />;
}
```

这是因为 `React.memo` 对于 props 的比较实际上是浅比较，也就是说如果 props 是一个 Object，那只会比较其内存的引用地址。

在 👆 上面的那个例子中，每次 App 渲染都会创建一个新的 `dog` 对象并传入 `DogProfile`，那么不管 `DogProfile` 有没有包裹 `React.memo`，都会进行重渲染。

## 总结

1. 只有 state 变化会导致 React 组件重渲染。
2. 使用 `React.memo` 创建纯组件
3. 不要过度优化
4. `React.memo` 只对 props 进行浅比较

---

本文源自：https://www.joshwcomeau.com/react/why-react-re-renders/
