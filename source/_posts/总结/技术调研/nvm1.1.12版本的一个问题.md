---
title: 技术调研-nvm1.1.12版本的一个问题
abbrlink: '1840e493'
date: 2024-07-08 09:05:01
categories:
  - 总结
tags:
  - 技术调研
---

# 背景

最近没有时间更新网站，是在开发一个桌面端的应用工具；桌面端的工具使用 Node+vue 开发的，但是在使用 node+nvm 的时候会有下面的一种情况；

因为`nvm`的版本在[github](https://github.com/coreybutler/nvm-windows)的版本为`1.1.12`也是最新版本的；但是在最新版本的使用的时候，会报错；

```js
const { exec, spawn } = require("chaild_process");

const process = exec("nvm ls", { shell: "cmd" });

// or

spawn("nvm", ["ls"], { shell: "cmd" });
```

这一段代码就是就是通过`node`操作`nvm`，但针对于`1.1.11`版本是可行的，但是`1.1.12`会提示`NVM for Windows should be run from a terminal such as CMD or PowerShell.`也就是只能通过终端打开；

![图片](https://wangxiaoze-view.github.io/picx-images-hosting/images/nvm_20240425095509.png)

然后在看了一下[源码](https://github.com/coreybutler/nvm-windows)，发现`nvm1.1.12`是`go`语言编写的；于是我在源码中找到了这样一段代码：

```go
// 入口文件
func main() {
 args := os.Args
 detail := ""
 procarch := arch.Validate(env.arch)

  // 这里判断了一下终端的条件
 if !isTerminal() {
  alert("NVM for Windows should be run from a terminal such as CMD or PowerShell.", "Terminal Only")
  os.Exit(0)
 }

 // Capture any additional arguments
 if len(args) > 2 {
  detail = args[2]
 }
 // .....
}


func isTerminal() bool {
 fileInfo, err := os.Stdout.Stat()
 if err != nil {
  return false
 }
 return (fileInfo.Mode() & os.ModeCharDevice) != 0
}
```

关于`isTerminal()`的解释大致是：

1. 通过 os.Stdout.Stat()获取标准输出的信息。
2. 如果获取信息时出现错误，则返回 false。
3. 判断获取到的信息中是否包含 os.ModeCharDevice 模式，若包含则表示输出为字符设备，即终端设备，返回 true；否则返回 false。

因为`node`直接操作`nvm`不是在终端唤起的，于是我就在`github`添加了一个[issues](https://github.com/coreybutler/nvm-windows/issues/1126)

不久后官方就提出了解释：

> This is due to a change that identifies the terminal. The change was added in 1.1.12 and has been nothing but a pain for everyone. It will be reverted in the next release, but it may be a little while before the next release (our code signing cert was locked when trying to automate the code signing process and I'm trying to get it unlocked).

> Revert to 1.1.11 for the time being. The changes between 1.1.11 and 1.1.12 are primarily just debugging. Most people won't experience a difference between the two.

大致意思就是:

**这是由于标识终端的更改。更改是在 1.1.12 添加的，对每个人来说都是一个痛苦。它将在下一个版本中恢复，但可能需要一段时间才能下一个版本（我们的代码签名证书在尝试自动化代码签名过程时被锁定，我正在尝试解锁它）。暂时回到 1.1.11。1.1.11 和 1.1.12 之间的变化主要只是调试。大多数人不会体验到两者之间的区别。**

官方说是要在下一个版本会将它修复，具体什么时候会修复只能等着了；

目前来说想要通过`node`操作`nvm`只能将 `1.1.12`降版本，然后重新安装`nvm1.1.11`了；
