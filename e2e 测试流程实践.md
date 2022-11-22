## 什么是 e2e 测试

端到端测试（end-to-end test），区别于单元测试（unit test），e2e 更像是一种黑盒测试，测试者模拟用户的行为，不需要了解系统内部实现。

其实也就是目前测试同学工作（以及面向开发的冒烟测试）的自动化版本。

## 为什么要实施 e2e 测试

在我看来，e2e 测试的好处有以下几点：

1. 我们可以经常听到很多人在抱怨“自测用例一过就要过一天”，现在的情况可能会变成“自测用例一写就要写一天”。但这两者是有本质区别的，“Dont Repeat Yourself”，后者的工作可以说是一劳永逸。
2. 对重构重新充满信心。这个就不用多说了，懂得都懂。在重构一个大型系统的时候，如果有完善的 e2e 测试，至少可以保证你重构完之后，主流程上不会出现大的问题。
3. 显著减少测试 bug。
4. 自身发展，可以快速了解一个第三方库的用法。

## 如何实施 e2e 测试

目前前端 e2e 测试的原理大同小异，就是通过无头浏览器去模拟用户的各种行为，比如点击、滚动等等。目前业内做得比较好也比较完善的 cypress，它封装了类似 jquery 的接口，让我们很方便地去对 dom 做各种操作。当然，这里并不是说 e2e 测试等于 cypress，e2e 测试只是一种概念，用单元测试框架 jest 也可以实现 e2e 测试。

而目前应用实践的 e2e 测试类型有以下三种：

1. Web 前端应用
2. CLI
3. 组件库

接下来我们一一探索。

### 安装 cypress

cypress 安装会带上一个无头浏览器环境，在国内下载会非常慢，我们可以通过指定环境变量，使用 taobao 地址下载指定版本：

```shell
CYPRESS_INSTALL_BINARY=https://registry.npmmirror.com/-/binary/cypress/10.9.0/darwin-x64/cypress.zip yarn add -D cypress
```

### Web 前端应用

这里以超能云平台 Web 端举例。首先，官方文档里也有提到，完整类型的 Web 应用自动化测试有很大一部分工作在登录页面。

### 组件库

#### 准备工作

这里以 @px/ui 举例。在组件 e2e 测试中，cypress 会启动一个浏览器运行时沙箱，我们通过它提供的 mount 方法去挂在需要测试的组件。这个浏览器沙箱的环境是需要我们配置的。这里我们使用 vite 进行配置。vite 有一个比较有意思的特性，那就是在开发服务器的启动中，vite 会执行一个仅在开发环境下生效的预打包操作，官方解释的原因有以下两点：

1. 需要提前把 commonjs 模块转换为 esm
2. 这是因为一个 esm 导出项可能特别特别多，vite 会提前把这些模块打包成一个模块，以减少开发过程中服务器的频繁请求

不过在 monorepo 中，预打包不会解析来自工作区的依赖，所以必须要保证我们的依赖是 esm 格式的，否则会报运行时错误。因为 @px/ui 依赖了 @px/common，@px/common 是一个由 ts 编译的 commonjs 包，所以需要做一些特殊处理：

```js
// cypress.config.ts

export default defineConfig({
  component: {
    devServer: {
      framework: "react",
      bundler: "vite",
      viteConfig: viteDefineConfig({
        plugins: [react()],
        optimizeDeps: {
          include: ["@px/common"],
        },
        build: {
          commonjsOptions: {
            include: [/@px\/common/],
          },
        },
      }),
    },
  },
});
```

还需要统一配置一些环境，比如引入全局组件样式和引入 ConfigProvider：

```js
// cypress/support/component.ts

import "./commands";
import "@px/ui/dist/pxui.css";
import "./app.css";
import { ConfigProvider } from "@px/ui";
import zhCN from "antd/lib/locale/zh_CN";
import { mount } from "cypress/react";
import { createElement } from "react";

Cypress.Commands.add("mount", (component, config) => {
  return mount(
    createElement(ConfigProvider, { locale: zhCN }, component),
    config
  );
});
```

然后我们在 CLI 输入：

```shell
yarn cypress open
```

就可以打开 cypress 可视化界面愉快地玩耍了。

#### 快照测试

快照测试是最适合组件库的测试，因为不像应用一样有很多干扰项，可以节省很多写重复代码的时间。

通常的快照测试是通过保存 html 结构，cypress 也有这样的插件提供：[cypress-plugin-snapshots](https://github.com/meinaart/cypress-plugin-snapshots)。但这个插件是社区提供的，并且已经长期没人维护了。在这个库的 [issue](https://github.com/meinaart/cypress-plugin-snapshots/issues/220) 里有人提到，目前 cypress 的快照测试，推荐使用 [cypress-plugin-visual-regression-diff](https://github.com/FRSOURCE/cypress-plugin-visual-regression-diff) 。

`cypress-plugin-visual-regression-diff` 的原理是通过调用 `cy.snapshot()` 在测试目录下生成捕获的图片，后面每次都通过图片的像素对比来确定组件快照是否有变化。

参考 t-design 的测试，我们可以在每个组件测试之前都进行一遍正常组件挂载的快照测试，至少可以保证组件最基本的渲染是没有问题的。

这里是快照测试的[最佳实践](https://docs.cypress.io/guides/tooling/visual-testing#Best-practices)参考。

#### 测试覆盖率

通过 `istanbul` babel 插件打包出含有覆盖率计数代码的包，再运行一次 cypress 测试，即可得到测试覆盖率文件 .nycout 和相对应的覆盖率文件，这时候我们可以选择通过直接打开 html 文件，可以很直观地看到覆盖率和**代码覆盖范围**，适合开发的时候使用；在命令行中，调用 yarn nyc report 可以直接在控制台得到覆盖率结果，方便在 ci 环境下使用。

[![xHOky6.md.png](https://s1.ax1x.com/2022/11/02/xHOky6.md.png)](https://imgse.com/i/xHOky6)

#### 开发流程

因为 TDD 流程团队内部都还没达成共识，建议直接开始写组件，然后再写测试用例。测试用例统一放在组件文件夹下的 `__test__` 文件夹中，以 `组件名.ct.tsx` 格式命名。

在大部分时候，完善组件的 Demo，在通过引入 Demo 组件的形式进行快照测试是比较好的选择，具体可以参考 `Layout` 组件的例子。

组件库会在提交推送（push）的时候跑一遍测试，用例全部通过才允许提交。因为 vscode git 管理的一些原因，目前只能支持命令行内输入 `git push` 提交到远程仓库。

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h7ksqx45cvj315n09lq4a.jpg)

## 踩坑

### 1. 组件测试基座需要设置为 1280 \* 720

cypress run versus cypress open is not same，因为 headless 测试的时候通过 `Xvfb` 进行无界面自动化测试，最大限制为 1280 \* 720。

### 2. Vscode git pre-push 问题频出

Vscode git 模块默认日志输出 stderr，如果需要查看 pre-push 的打印内容需要在项目 vscode 配置中加上 `"git.commandsToLog": ["push"]`，vscode 才会输出 stdout。

待解决的问题：通过 vscode git 工具推送到远程仓库会报无法找到 cypress app 的错误。
