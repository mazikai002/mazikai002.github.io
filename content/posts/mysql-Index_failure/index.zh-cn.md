---
weight: 1
title: "MySQL 优化手段 —— 索引失效"
date: 2023-08-17T15:43:57+08:00
draft: false
author: "mazikai"
authorLink: "https://mazikai002.cn"
description: "索引失效"
images: []
resources:
- name: "featured-image"
  src: "mysql_logo.png"

tags: ["mysql"]

lightgallery: true
---

>聊聊索引失效</br>

<!--more-->

# 总结

**模 :**   模糊查询。like查询以%开头，会导致索引失效。可以有两种方式优化(覆盖索引、最左匹配)

**数 :**   数据类型。编写SQL时要保证索引字段与匹配数据类型一致。

**函 :**   数据处理。索引字段不做函数处理。

**空 :**   null值。唯一索引有null值。

**运 :**   运算。索引字段不做运算。

**左 :**   最左匹配原则。

**回 :**   回表超过临界值，避免回表尽量覆盖索引。

# 参考
https://bbs.huaweicloud.com/blogs/333163