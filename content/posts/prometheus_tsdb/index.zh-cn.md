---
weight: 1
title: "Prometheus 原理分析 —— 深入TSDB存储"
date: 2024-03-02T11:33:42+08:00
draft: false
author: "mazikai"
authorLink: "https://mazikai002.cn"
description: "深入TSDB"
images: []
resources:
- name: "featured-image"
  src: "prometheus_logo.png"

tags: ["prometheus"]

lightgallery: true
---

> 聊聊 Prometheus 的时序数据库 TSDB ~</br>

<!--more-->

## 一些概念
- 时序数据库 : 专门用来存储随时间变化数据的数据库，如股票价格、传感器数据等
- 时间序列 : 某个变量随时间变化的所有历史
- 样本 : 历史中该变量的瞬时值

关系如图所示 : 
<div align=center>
	<img src="prometheus_concept.png"/>
</div>

## Prometheus 中的时序数据库
要了解Prometheus 中的时序数据库，首先得看一下官方架构图。如图所示:
<div align=center>
	<img src="prometheus_architecture.png"/>
</div>
对Prometheus Server来说，整个数据处理流程分为三个部分 : Retrieval(数据抓取)、TSDB(数据存储及查询)、HTTP server(对外接口服务)。其中TSDB为本文重点关注部分，其他部分后续会进行详解。

## Prometheus 中时序数据存储结构
Prometheus 中时间序列是按照时间戳和值的序列顺序存放的，每条time-series通过指标名称（metrics name）和一组标签集（labelset）命名。如下所示，可以将时间序列理解为一个以时间为Y轴的数字矩阵：
```
 ^
  │   . . . . . . . . . . . . . . . . .   . .   node_cpu{cpu="cpu0",mode="idle"}
  │     . . . . . . . . . . . . . . . . . . .   node_cpu{cpu="cpu0",mode="system"}
  │     . . . . . . . . . .   . . . . . . . .   node_load1{}
  │     . . . . . . . . . . . . . . . .   . .  
  v
    <------------------ 时间 ---------------->
```
上图中的每一个点称为一个样本，样本由两部分组成，源码内容如下(release-2.20版本) : 
```go
type Series struct {
  Metric labels.Labels `json:"metric"`
  Points []Point `json:"values"`
}

type Point struct {
  T int64
  V float64
}

type Sample struct {
  Point
  Metric labels.Labels
}
```
- Metric 为metricName和label内容
- Point 为当前样本时间和浮点数据值

有人很可能会好奇Prometheus中存储的数据不应该为四部分数据么？(指标、标签、时间戳、样本值)，这是因为很多书籍方便给大家的一些基础认知，实际上Prometheus在底层数据存储的时候做了一些处理，把指标这一部分转化成了标签进行处理(具体原因作者还需要找一些认证材料，后续更新)，源码如下(release-2.20版本):
```go
// prometheus/pkg/labels/labels.go 文件
package labels

import (
  "bytes"
  "encoding/json"
  "sort"
  "strconv"

  "github.com/cespare/xxhash"
)

const (
  MetricName = "__name__" // 将指标转化的为标签的内容
  ...
)
```

```go
// promntheus/pkg/textparse/promparse.go 文件
type PromParser struct {
  l       *promlexer
  series  []byte
  text    []byte
  mtype   MetricType
  val     float64
  ts      int64
  hasTS   bool
  start   int
  offsets []int
}

// Metric writes tge labels of the current sample into tge passed labels.
// It returns the string from which the metric was parsed.
function (p *PromParser) Metric(l *labels.Labels) string {
  s := string(p.series)

  *l = append(*l, labels.Label) {
    Name: labels.MetricName, // 将metric指标转化成label标签的证据
    Value: s[:p.offsets[0]-p.start],
  }

  for i := 1 ; i < len(p.offsets); i += 4 {
    a := p.offsets[i] - p.start
    b := p.offsets[i+1] - p.start
    b := p.offsets[i+2] - p.start
    b := p.offsets[i+3] - p.start

    if strings.IndexByte(s[c:d], byte('\\')) >= 0 {
      *l = append(*l, labels.Label{Name: s[a:b],Value: lvalReplacer.Replace(s[c:d])})
      continue
    }
    *l = append(*l, labels.Label{Name: s[a:b], Value:s[c:d]})
  }

  sort.Sort(*l)
  return s
}
```

## TSDB 数据写入
### 磁盘部分
要了解TSDB的数据写入，首先得了解TSDB的目录结构
```
./data
├── 01BKGV7JBM69T2G1BGBGM6KB12
│   └── meta.json
├── 01BKGTZQ1SYQJTR4PB43C8PD98
│   ├── chunks
│   │   └── 000001
│   ├── tombstones
│   ├── index
│   └── meta.json
├── 01BKGTZQ1HHWHV8FBJXW1Y3W0K
│   └── meta.json
├── 01BKGV7JC0RY8A6MACW02A2PJD
│   ├── chunks
│   │   └── 000001
│   ├── tombstones
│   ├── index
│   └── meta.json
├── chunks_head
│   └── 000001
└── wal
    ├── 000000002
    └── checkpoint.00000001
        └── 00000000
```
默认Prometheus的所有监控数据都会放在 ./data 目录下面，里面存放了 chunks_head、wal、block(01BKGV7JBM69T2G1BGBGM6KB12等) 三种类型的数据。</br>
- block : 每个 block 会存储 2 小时时间窗口内所有 series 指标数据，每个 block 文件名都会使用 github.com/oklog/ulid 这个库生成不重复的文件名，里面包含了 meta.json 文件、index 文件、tombstones 文件、chunks 文件夹，所有指标数据都存放在 chunks 文件夹中，chunks 中包含了多个数据段信息，每个数据段会按照 512MB 分成一个文件存储，被删除的数据会存放在 tombstone 文件夹中。
- chunks_head : 这个文件夹里面也包含了多个 chunks ，当内存的 `head block` 写不下了会将数据存放在这个文件夹下面，并保留对文件的引用。
- wal : 该文件夹里面存放的数据是当前正在写入的数据，里面包含多个数据段文件，一个文件默认最大 128M，Prometheus 会至少保留3个文件，对于高负载的机器会至少保留2小时的数据。wal 文件夹里面的数据是没有压缩过的，所以会比 block 里面的数据略大一些。

### 内存部分
- head block : v2.19之前，最近 2 小时的指标数据存储在内存中，v2.19 引入 head block，最近的指标数据存储在内存中，当内存存满时将数据刷入到磁盘中，并通过一个引用关联刷到磁盘的数据。

## 参考
https://prometheus.io/</br>
https://juejin.cn/post/7001273893780471845</br>