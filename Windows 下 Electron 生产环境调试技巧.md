#electron

大家都知道，electron 是多进程架构，也就是由主进程和渲染进程组成的，其对应的其实就是 nodejs 和 chromium 两大部分所以我们的调试方法也要从这两方面入手。

## 渲染进程调试

渲染进程，对应的是 chromium 里的每一个标签页。

我们调试的目标可以等效为调试浏览器中的一个网页，所以可以直接通过 f12 或者通过代码打开 devtools，就可以愉快地调试 dom 和 performance 等关键指标。

```js
const mainWindow = new BrowserWindow();
mainWindow.loadURL("file://path/of/index.html");
mainWindow.webContents.openDevTools();
```

在生产环境下，我们一般会把 devtools 禁用掉。不过，我们可以通过读取环境变量的方式让应用允许打开 devtools：

```js
webPreferences: {
    devTools: isDev || runmode !== 'pro',
}
```

另外，打开了 devtools 后，如果渲染进程内禁用了 f5 刷新，我们也可以在 devtool 内按 f5 刷新渲染进程的页面， 不受按键事件影响。

这样的方式其实也不是很安全，最好的方法还是做好 log 机制，力求通过 log 可以还原我们的渲染进程逻辑。

## 主进程调试

主进程则对应的是 nodejs 的进程，我们可以通过命令行启动应用，带上 `—-inspect` 或 `—-inspect-brk` 开启主进程调试模式：

![lWtvOU|675](https://raw.githubusercontent.com/raoooool/blog-images/main/lWtvOU.png)

然后在 chromium 系浏览器打开 devtools：

![8RtMpZ|325](https://raw.githubusercontent.com/raoooool/blog-images/main/8RtMpZ.png)

就可以针对 nodejs 进程 debug，断点、单步跨步调试以堆栈信息一应俱全：

![JC29Ev](https://raw.githubusercontent.com/raoooool/blog-images/main/JC29Ev.png)

如果要查看 electron 本身的运行日志，可以在命令行启动时添加 `—-enable-logging` ，electron 就会在 stderr 打印运行信息，方便排查问题。

![lyYVzz|500](https://raw.githubusercontent.com/raoooool/blog-images/main/lyYVzz.png)

除了运行日志，进程崩溃日志也是需收集的。electron 提供了 crashpad api，原理是另外启动一个 crashpad 进程监控主进程和渲染进程的运行情况，如果出现异常，可以通过设置的 url 上传 dump 文件，sentry electron sdk 就是以这个 api 为基础实现的。

![U0bWs4|475](https://raw.githubusercontent.com/raoooool/blog-images/main/U0bWs4.webp)

当然，除上传到日志服务器，我们还可以选择保存到本地，通过 crashpad 的设置 `xxx`，就可以在 appdata/romaing/appname/crashpad/reports 里找到生成的 dump 文件。

![aJhW4w|475](https://raw.githubusercontent.com/raoooool/blog-images/main/aJhW4w.png)

## Windows 调试

经过上述生成的 dump 文件，应该如何分析？
