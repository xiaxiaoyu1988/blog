---
title: CEF中文教程
date: 2018-05-23 11:35:14
tags:
  - CEF
  - Chrome
  - Blink
---
- [CEF中文教程](#cef中文教程)
    - [介绍](#介绍)
    - [开始](#开始)
        - [加载自定义URL](#加载自定义url)
    - [应用组件](#应用组件)
    - [1分钟了解架构](#1分钟了解架构)
    - [源码](#源码)
    - [入口函数](#入口函数)
    - [SimpleApp类](#simpleapp类)
    - [SimpleHandler类](#simplehandler类)
    - [编译步骤](#编译步骤)
        - [基于CMake的通用平台编译步骤](#基于cmake的通用平台编译步骤)
 
本教程翻译自[CEF官方文档](https://bitbucket.org/chromiumembedded/cef/wiki/Tutorial)

# CEF中文教程

## 介绍
本教程详细解释了，如何通过CEF3创建一个简单的应用程序。本教程中的示例程序引用的是CEF项目中的[cefsimple示例程序](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/?at=master)。访问[GeneralUsage](https://bitbucket.org/chromiumembedded/cef/wiki/GeneralUsage.md)可以获取更多有关CEF的信息。

## 开始
CEF提供了一个非常易于开始的示例程序, 可以访问[cef-project](https://bitbucket.org/chromiumembedded/cef-project)查阅相关文档。本教程中使用的示例程序以及CEF3程序来自于CEF项目的主线分支，可能与发布版本略有不同。
### 加载自定义URL
cefsimple程序默认加载的网址是 google.com, 可以通过命令行指定参数的方式加载不同的URL。  

```
    # Load the local file “c:\example\example.html”
    cefsimple.exe --url=file://c:/example/example.html
```
<!-- More -->  

也可以直接修改cefsimple程序源码来加载指定的URL，源码位置在[cefsimple/simple_app.cc](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/simple_app.cc?at=master)  

```
    // Load the local file “c:\example\example.html”
    …
    if (url.empty())
        url = "file://c:/example/example.html";
    …
```

## 应用组件
所有的CEF程序拥有以下几个主要的组件：  
 *  CEF动态链接库(Windows下是libcef.dll, Linux是 libcef.so, OS X是Chromium Embedded Framework.framework)
 *  支持文件(*.pak *.bin等等)
 *  资源文件(html/js/css等等)
 *  可执行文件(例如示例程序中的cefsimple.exe)

这些组件在所有基于CEF开发的程序中都是有的，不管是在调试版本或者是发布版本中。可以查看CEF发布版本中的README.txt, 了解哪些组件是必须的，哪些组件是可选的。下文将详细介绍每个平台的程序布局。

## 1分钟了解架构
下面列出了本教程中最重要的几项内容：
 * CEF是基于多进程的。主进程称为浏览器进程， 其他子进程则用于渲染、插件、GPU等等。
 * Windows和Linux上主进程和子进程可以在同一个可执行程序中，而在OS X中主进程和子进程在不同的可执行文件中
 * CEF的大部分进程都是多线程的。CEF为不同线程的通信提供了接口
 * 有些回调和函数都只能用在特定的进程或者线程中。在使用前，一定要阅读API文档以及接口的注释。

这个[Wiki](https://bitbucket.org/chromiumembedded/cef/wiki/GeneralUsage.md)中可以查看有关以上几点更详细的解释

## 源码
cefsimple程序中初始化CEF以及创建了一个弹出式的浏览器窗口。所有的浏览器窗口关闭时， 程序终止。
程序流程如下：
 * 操作系统首先执行主进程中的入口函数（main或者是wWinMain）
 * 入口函数中
    * 创建处理进程级别回调函数的SimpleAPP实例
    * 初始化CEF并启动CEF消息处理循环
 * CEF初始化完成后，调用SimpleApp::OnContextInitialized().该函数
    *   创建SimpleHandler的单例
    *   通过CefBrowserHost::CreateBrowser()创建一个浏览器窗口
    *   所有的浏览器窗口共享SimpleHandler实例，该实例负责自定义浏览器窗口以及处理浏览器相关回调事件，包括浏览器生命周期、加载状态、标题显示等等
    *   浏览器窗口关闭时，回调函数SimpleHandler::OnBeforeClose()会被调用。当所有浏览器窗口退出后，回调函数OnBeforeClose 接着被调用，然后退出CEF的消息循环，退出程序。

最新的CEF发布版本中，可能文件有更新，但是上面的这些概念是不会修改的。

## 入口函数
程序执行是从浏览器进程中的入口函数开始的。入口函数中负责初始化CEF以及操作系统相关的对象。
例如，在LINUX中，入口函数中初始化了X11相关的错误处理函数，而在OS X中则初始化了必须的cocoa对象。另外的OSX 中，浏览器进程和其他辅助进程都有独立的入口函数。
 * Windows平台的入口函数文件[cefsimple/cefsimple_win.cc](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/cefsimple_win.cc?at=master)
 * LINUX平台下的入口函数文件[cefsimple/cefsimple_linux.cc](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/cefsimple_linux.cc?at=master)
 * MAC OSX平台下
    * 浏览器进程入口函数文件[cefsimple/cefsimple_mac.mm](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/cefsimple_mac.mm?at=master)
    * 其他子进程[cefsimple/process_helper_mac.cc](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/process_helper_mac.cc?at=master)

## SimpleApp类
SimpleApp主要负责进程级别的回调处理。SimpleApp中有一部分公共接口和函数可共享用于所有进程中，也有一些接口和函数只能用于特定的进程中。CefBrowserProcessHandler 仅在浏览器进程中使用。CefRenderProcessHandler 仅能在渲染进程中使用。需要注意的是，GetBrowserProcessHandler必须返回this指针，因为SimpleApp 实现了CefApp 和 CefBrowserProcessHandler.可以访问[ GeneralUsage ](https://bitbucket.org/chromiumembedded/cef/wiki/GeneralUsage.md)查阅更多信息
 * 代码位置: [cefsimple/simple_app.h](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/simple_app.h?at=master), [cefsimple/simple_app.cc](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/simple_app.cc?at=master)

## SimpleHandler类
SimpleApp主要负责浏览器级别的回调处理。这些回调只会在浏览器进程中触发。示例程序中，我们使用了同样的cefclient，但是实际开发过程中可以使用不同的cefclient。可以访问[GeneralUsage](https://bitbucket.org/chromiumembedded/cef/wiki/GeneralUsage.md)查看更多相关信息。
 * 跨平台实现: [cefsimple/simple_handler.h](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/simple_handler.h?at=master), [cefsimple/simple_handler.cc](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/simple_handler.cc?at=master)
 * Windows 平台实现: [cefsimple/simple_handler_win.cc](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/simple_handler_win.cc?at=master)
 * Linux 平台实现: [cefsimple/simple_handler_linux.cc](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/simple_handler_linux.cc?at=master)
 * Mac OS X 平台实现:[] cefsimple/simple_handler_mac.mm](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/simple_handler_mac.mm?at=master)

## 编译步骤
编译步骤在不同平台上略有不同。可以查看CMake文件详细了解不同平台下的编译流程。
所有平台下的编译步骤可以总结为以下几点：
 * 编译libcef_dll_wrapper 静态库
 * 编译程序源码，链接到libcef 动态库和libcef_dll_wrapper 静态库
 * 拷贝资源和库文件到输出目录

### 基于CMake的通用平台编译步骤
 * 安装不同平台下的编译工具链（例如windows 下的visual， mac osx下的xcode， linux下的g++等等，建议安装最新版本）
 * 源码目录创建build目录（mkdir build）
 * 进入build目录，执行cmake命令，-G 参数指定平台（例如 mac osx 下cmake -G "Xcode" .. ）
 * 生成makefile文件后，执行make命令 编译指定模块或者全部模块 (例如 make all; make cefsimple)
 * 各个平台下稍有不同，编译问题大部分都是库不完整，编译工具链不完整，或者路径不正确，官网的源码本身没有问题，各个平台下都能编译通过。