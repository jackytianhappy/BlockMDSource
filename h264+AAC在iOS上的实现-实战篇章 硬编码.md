---
title: 直播日记-3 h264+AAC在iOS上的实现-实战篇章 硬编码
date: 2017-03-23 21:44:43
tags: 直播
---
>上篇已介绍了关于在iOS上编码的原理，本篇主要介绍如何实现iOS上的硬编码。通常实现编码的方式有两种，1.利用FFMpeg，利用CPU做视频的编码和解码，俗称软编，软解。但是该方法比较消耗cpu的资源。2.利用系统GPU或者专用处理器来对视频流进行编码，俗称硬编和硬解。<br>
>iOS自8.0之后，系统开开放了硬件编码和解码的功能。即通过Video ToolBox框架来处理。

## Video Toolbox基本数据结构
- CVPixelBuffer:编码前和编码后的图像的数据结构
- CMTime、CMClock和CMTImebase：时间戳相关，时间以64-bit/32-bit的形式出现
- CMBlockBuffer：编码后，结果的图像的数据结构
- CMVideoFormatDescription：图像存储方式，编解码器等格式描述。
- CMSampleBuffer：存放编解码前后的视频退选哪个的容器数据结构。

<font color = 'red'>未完待续</font>









