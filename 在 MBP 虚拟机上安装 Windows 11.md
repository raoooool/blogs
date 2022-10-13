因为项目原因，需要在 Windows 环境下测试一下 Electron 的表现，于是就记录一下在 Mac 虚拟机上安装 Windows 的体验，总体来说难度不大。

我电脑的情况：`2020 款 Macbook Pro，16G，512G`。

## 前期准备

目前市面上比较有名的 Mac 虚拟机方案主要有以下三种：

|          | VMware   | PD       | VirtualBox |
| -------- | -------- | -------- | ---------- |
| 性能     | 中       | 高 ✅    | 低         |
| 维护性   | 商业软件 | 商业软件 | 开源       |
| 收费情况 | 中等     | 高       | 免费 ✅    |

虽然这样看可能会觉得，VirtualBox 也还可以。不过我试了一下，装最新版的 Windows 11 基本卡到不能用，VMware 表现却出奇的好，结合使用费用等情况，VMware 是不二之选。

### 资源准备

首先需要下载并安装 VMware，根据提示一步步安装就好，这里推荐一个网站：https://macwk.com/soft/vmware-fusion

然后再到微软官网下载最新的 Windows 11 镜像：https://www.microsoft.com/en-us/software-download/windows11

## 开始安装

### 新建虚拟机

在 VMware 里新建一个虚拟机，选中下载好的 Windows 11 镜像。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h666c97oh4j20hs0et74s.jpg)

然后选择 `Windows 10 或更高版本`，然后再按照默认选项继续。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h666day4kpj20hs0etdg2.jpg)

继续。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h666e62yc5j20hs0etmxo.jpg)

在这一步建议选择`自定设置`，选择内存增加到 `8192 MB`，其他不变。然后就可以启动虚拟机了。

### 安装 Windows

启动之后会自动进入到 Windows 安装页面。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h666hvta4gj20l40hg74e.jpg)

#### 修改注册表

重点来了，因为微软对运行 Windows 11 的设备有一定的限制，要对虚拟机做点手脚。

在安装界面按下 `shift + f10`，在弹出的命令行里输入 `regedit` 进入注册表。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h666ke8g9ij20si0net96.jpg)

按以下步骤操作：

- 定位到 `HKEY_LOCAL_MACHINE\SYSTEM\Setup`
- `右键`，`New`，`Key`
- 命名为 `LabConfig`
- 右键点击 `LabConfig` => `DWORD(32-bit) Value`
- 命名为 `BypassTPMCheck`
- 双击它，把值从 0 改成 1
- 重复上面两个操作，分别命名为 `BypassSecureBootCheck`、`BypassRamCheck`

最后，会得到这样的一个结果：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h666os3rp0j20si0nemy7.jpg)

#### 继续安装

然后就可以继续 Windows 11 的安装了，剩下的过程和正常安装操作系统一样，没有什么特别的地方。

### 配置 VMware

安装好了之后，你会发现进入 Windows 会很卡顿，这时候要安装 `VMware Tools`。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h666rnsdyij20xr0n8ta0.jpg)

选择`虚拟机`菜单，选择`安装 VMware Tools`（我这里是重新安装），会发现在资源管理器里插入了一个 DVD 光驱。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h666uf1x7ij20v70hjmzo.jpg)

运行一下 `setup.exe` 然后等待自动安装即可。

### 善后

我这边习惯把 Windows 11 单独全屏放在一个显示器里使用，但每次切换桌面 VMware 都会重新自动设置 Windows 分辨率，在这里把 VMware 的全局显示设置改成下面这样就好了。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h666yeug8mj20w40arq4f.jpg)

## 参考

https://geekflare.com/windows-11-in-virtual-box/
