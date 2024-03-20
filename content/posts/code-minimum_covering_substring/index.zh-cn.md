---
weight: 1
title: "算法刷题 —— 最小覆盖子串(Leetcode 76)"
date: 2024-03-20T20:40:00+08:00
draft: false
author: "mazikai"
authorLink: "https://mazikai002.cn"
description: "给你一个字符串 s 、一个字符串 t 。返回 s 中涵盖 t 所有字符的最小子串。如果 s 中不存在涵盖 t 所有字符的子串，则返回空字符串 "" 。

"
images: []
resources:
- name: "featured-image"
  src: "code_logo.png"

tags: ["code"]

lightgallery: true
---

> 给你一个字符串 s 、一个字符串 t 。返回 s 中涵盖 t 所有字符的最小子串。如果 s 中不存在涵盖 t 所有字符的子串，则返回空字符串 "" ~ </br>

```go
func main(){
	var res string
	var s , t string

	// 示例 1
	s , t = "ADOBECODEBANC", "ABC"
	res = minWindow(s,t)
	fmt.Println("res:",res)

	// 示例 2
	s , t = "a", "a"
	res = minWindow(s,t)
	fmt.Println("res:",res)

	// 示例 3
	s , t ="a", "aa"
	res = minWindow(s,t)
	fmt.Println("res:",res)
}

func minWindow(s string, t string) string {
    res := ""
    length := math.MaxInt32
    mp := make(map[byte]int)
    for i := 0 ; i < len(t) ; i++ {
        mp[t[i]]++
    }

    left := 0  // 滑动窗口左边界
    right := 0 // 滑动窗口右边界
    for right < len(s) {
        mp[s[right]]--
        for check(mp) { // 判断当前mp是否有效
            if right - left + 1 < length {
                length = right - left + 1
                res = s[left : right+1]
            }
            mp[s[left]]++
            left++
        }
        right++
    }
    return res
}

func check(mp map[byte]int) bool { // 有效
    for _ , v := range mp {
        if v > 0 {
            return false
        }
    }
    return true
}
```
