---
weight: 1
title: "算法刷题 —— 判断图中是否有环"
date: 2024-03-15T11:27:33+08:00
draft: false
author: "mazikai"
authorLink: "https://mazikai002.cn"
description: "判断图中是否有环"
images: []
resources:
- name: "featured-image"
  src: "code_logo.png"

tags: ["code"]

lightgallery: true
---


> 判断图中是否有环 ~ </br>

```go
package main

func main() {

}

type Graph struct {
	grips []grip
}

type grip struct {
	inDegree int
	pointTo  []grip
}

func (graph Graph) IsExistRing() bool {
	inDegreeMap := make(map[*grip]int) // map放结构体可以放指针，指针可以比较
	for _, g := range graph.grips {
		inDegreeMap[&g] = g.inDegree // 一定会有一个入度为0的节点，如果没有就说明环
	}

	length := len(inDegreeMap)
	for length != 0 {
		for key, value := range inDegreeMap { // key = 1

			if value == 0 {
				for _, g := range key.pointTo { // 遍历指向的节点进行一个入度的减减 
					g.inDegree--
					inDegreeMap[&g]-- // 这个地方得同步数据
				}
				delete(inDegreeMap, key) // 删除入度为0的节点
				length = len(inDegreeMap)
			}
		}
		if len(inDegreeMap) == length { // 整个map中的数据和上一次循环周期相等
			return true
		}
	}
	return false
}
```
