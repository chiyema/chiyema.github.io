---
layout:     post
title:      "更改macOS外置键盘的键盘布局"
subtitle:   "modifying the keyboard layout on macOS"
date:       2017-11-13
author:     "Moubei"
tags:
    - macOS
    - keyboard
---

当 MacBook 外接显示器的时候，多数情况也需要外接键盘。但不是所有的键盘都支持 macOS 的布局，而 Windows 键盘的布局也全都相同，所以就会遇到，映射的 Control、Option 及 Command 键与习惯上的位置不符合的情况。本文就是为了解决这一情况。

## macOS 自带的解决方案
macOS 本身就内置了一种定制键盘的功能，位于 Pref->keyboard->modifier Keys，但是可惜的是这个功能并没有得到苹果的重视，从网上别人的反馈以及自己的测试，并没有生效。
![](/assets/img/in-post/post-modifying-the-keyboard-layout-on-macOS/1.png)

## 利用第三方软件 Karabiner 来改变映射

> A powerful and stable keyboard customizer for macOS.—— Introduction to Karabiner

根据 [Karabiner官网](https://pqrs.org/osx/karabiner/)介绍，Karabiner支持目前所有版本(High Sierra 之前)的 macOS。软件本身是免费的，安装后即可使用。

打开 Karabiner-Elements，即可进入软件本体。在第一个标签页 Simple Modification 下可以简单地自定义每个按键，比如交换 Control 和 Command。
![](/assets/img/in-post/post-modifying-the-keyboard-layout-on-macOS/2.png)

在标签页 Function Keys，可以自定义 F1 到 F12 的自定义多媒体功能。
![](/assets/img/in-post/post-modifying-the-keyboard-layout-on-macOS/3.png)

此外其他的标签页还可以实现更为复杂的键盘自定义。这个软件更实用的一点是，以上所有的设置都可以单独对每一个键盘设置，只需要在每个页面的 Target Device 选中特定的键盘。这样就不会将改变内置键盘的布局，非常方便。
