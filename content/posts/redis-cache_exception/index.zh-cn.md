---
weight: 1
title: "Redis 缓存异常 —— 缓存雪崩、击穿、穿透"
date: 2023-09-08T22:03:13+08:00
draft: false
author: "mazikai"
authorLink: "https://mazikai002.cn"
description: "缓存雪崩、击穿、穿透"
images: []
resources:
- name: "featured-image"
  src: "redis_logo.png"

tags: ["redis"]

lightgallery: true
---

> 引入缓存层，会有缓存异常的三个常见问题，分别是缓存雪崩、击穿、穿透。 </br>
> 本文总结了这三个问题的定义、原因以及应对方案。 </br>
> 请看如下表格，希望对您有帮助 ~  </br>

<!--more-->
<div align='center' >
  <table>
      <tr>
          <th>缓存异常</th><th>问题描述</th><th>产生原因</th><th>应对方案</th>
      </tr>
      <tr>
          <td rowspan="2">缓存雪崩</td><td rowspan="2">大量的应用请求无法在Redis缓存中进行处理，从而使得大量请求发送到数据库层，导致数据库压力过大甚至宕机。</td><td>大量数据同时过期</td><td>- 均匀设置过期时间，避免同一时间过期</br>- 互斥锁，保证同一时间只有一个应用在构建缓存</br>- 双key策略，主key设置过期时间，备key永久，主key过期时，返回备key的内容</br>- 后台定时刷新</td>
      </tr>
      <tr>
          <td>Redis故障宕机</td><td>- 服务熔断</br>- 请求限流</br>- 搭建Redis高可用集群</td>
      </tr>
      <tr>
          <td>缓存击穿</td><td>一个热点数据的缓存失效，导致大量请求直接落到数据库上，造成数据库压力过大，影响系统性能。</td><td>频繁访问的热点数据过期</td><td>- 互斥锁</br>- 不给热点数据设置过期时间</br>- 后台定时刷新(过期前刷新)</td>
      </tr>
      <tr>
          <td>缓存穿透</td><td>被大量请求缓存和数据库都没有的数据，所有的请求都会直接穿透到数据库，会导致数据库压力过大，甚至垮掉。</td><td>访问的数据既不在缓存，也不再数据库</td><td>- 非法请求限制</br>- 缓存空值</br>- 布隆过滤器</td>
      </tr>
  </table>
</div>


# 参考
https://cloud.tencent.com/developer/article/1924661</br>
https://www.xiaolincoding.com/redis/cluster/cache_problem.html#%E7%BC%93%E5%AD%98%E9%9B%AA%E5%B4%A9</br>