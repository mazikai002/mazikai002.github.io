---
weight: 1
title: "算法刷题 —— 最长递增子序列(Leetcode 300)"
date: 2024-03-15T11:27:33+08:00
draft: false
author: "mazikai"
authorLink: "https://mazikai002.cn"
description: "最长递增子序列"
images: []
resources:
- name: "featured-image"
  src: "code_logo.png"

tags: ["code"]

lightgallery: true
---


> 最长递增子序列 ~ </br>

```go
package main

func main() {

}

// O(n2)
func lengthOfLIS(nums []int) int {
    res := math.MinInt
    dp := make([]int,len(nums))
    for i := 0 ; i < len(nums) ; i++ {
        dp[i] = 1
    }
    for i := 1 ; i < len(nums) ; i++ {
        for j := 0 ; j < i ; j++ {
            if nums[i] > nums[j] {
                dp[i] = max(dp[j]+1,dp[i])
            }
        }
    }
    for i := 0 ; i < len(nums) ; i++ {
        res = max(res,dp[i])
    }
    return res
}
```
