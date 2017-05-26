---
layout: post
title: Linux下Intel+Nvidia双屏双显卡配置
date: 2017-05-26
tags:
  - linux
  - xserver
---

最近入手了号称图吧标配的昂达1050ti，可能是人品爆发竟然没有传说中的那么大声音，收到之后就开始在linux上折腾了。

残念的是现在的4.9内核还是不支持1050ti，直接亮机就别想啦，只能先接集成显卡，装了私有驱动之后才能成功亮机，为了图方便我就一个屏幕接N卡，一个屏幕接集显，竟然发现虽然BIOS里设置的显示输出是PCI（独显），在启动linux的过程中plymouth主题是显示在接集显的显示器上的，而且虽然桌面是跑在独显的显示器上，切tty的时候独显的显示器会黑掉，tty的界面会在集显的显示器上面显示。这时候我就觉得intel的显卡并不是完全没有工作，它也是能够进行显示输出的。

经过一番搜索，发现在linux也是可以让两个显卡同时工作的，只不过需要自己配置：[Nvidia官方文档](http://download.nvidia.com/XFree86/Linux-x86/375.26/README/randr14.html)

需要编辑 /etc/X11/xorg.conf ：
```
Section "ServerLayout"
    Identifier "layout"
    Screen 0 "nvidia"
    Inactive "intel"
EndSection

Section "Device"
    Identifier "nvidia"
    Driver "nvidia"
    #BusID "<BusID for NVIDIA device here>"
EndSection

Section "Screen"
    Identifier "nvidia"
    Device "nvidia"
    Option "AllowEmptyInitialConfiguration"
EndSection

Section "Device"
    Identifier "intel"
    Driver "modesetting"
EndSection

Section "Screen"
    Identifier "intel"
    Device "intel"
EndSection
```

重启X后执行下面两行命令：
``` bash
xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
```
这时就会发现接intel显卡的显示器已经亮了，作为扩展屏来使用。执行命令`xrandr --listproviders`可以看到如下输出:

```
Providers: number : 2
Provider 0: id: 0x1f0 cap: 0x1, Source Output crtcs: 4 outputs: 4 associated providers: 0 name:NVIDIA-0
Provider 1: id: 0x44 cap: 0x2, Sink Output crtcs: 3 outputs: 2 associated providers: 0 name:modesetting
```
可以看到Nvidia是作为Source Output, Intel作为Sink Output,所以虽然扩展屏接了集显，其显示渲染都是Nvidia完成的.

后来我也测试过Windows，win10是可以直接识别接集显的屏幕并让其作为扩展屏工作的，linux的xserver应该也可以通过检测双显卡来做到/etc/X11/xorg.conf的那些配置，最后由桌面环境来自动配置xrandr，大概是没人提这个需求吧。
