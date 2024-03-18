---
weight: 1
title: "Golang 并发编程 —— 锁机制"
date: 2024-02-06T19:51:20+08:00
draft: false
author: "mazikai"
authorLink: "https://mazikai002.cn"
description: "锁机制"
images: []
resources:
- name: "featured-image"
  src: "featured-image.jpg"

tags: ["golang"]

lightgallery: true
---

> 在日常生活中，锁是为了保护一些东西，比如门锁、密码箱锁，可以理解对资源的保护。</br>
> 在编程世界里，锁也是为了保护资源，比如文件锁 : 同一时间只也许一个用户修改文件。</br>
> 本文就梳理了一下 Golang 中 sync 包中的锁机制。希望对您有帮助 ~</br>


<!--more-->

## sync.Mutex(互斥锁)

这是一个标准的互斥锁，平时用的也比较多，用法也非常简单，lock用于加锁，unlock用于解锁，配合defer使用，完美。

## sync.RWMutex(读写锁)

读写锁是互斥锁的升级版，它最大的优点就是支持多读，但是读和写、以及写与写之间还是互斥的，所以比较适合读多写少的场景。



## 参考
https://wangbjun.site/2020/coding/golang/locker.html</br>