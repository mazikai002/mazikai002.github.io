---
weight: 1
title: "算法刷题 —— 交替打印"
date: 2024-03-30T20:50:00+08:00
draft: false
author: "mazikai"
authorLink: "https://mazikai002.cn"
description: "使用两个Goroutine，向标准输出中按顺序按顺序交替打出字母与数字，输出是a1b2c3...z26"
images: []
resources:
- name: "featured-image"
  src: "code_logo.png"

tags: ["code"]

lightgallery: true
---

> 使用两个Goroutine，向标准输出中按顺序按顺序交替打出字母与数字，输出是a1b2c3...z26 ~ </br>

```go
func main() {
	letter , number := make(chan bool), make(chan bool) // 使用两个channel交替实现
	wait := sync.WaitGroup{}

	go func(){
		i := 1 
		for {
			select { // 轮训
			case <- number:
				fmt.Println(i)
				i++
				letter <- true
			}
		}
	}()

	wait.Add(1)
	go fun(wait *sync.WaitGroup) {
		for ch := 'a' ; ch <= 'z' ; ch++ {
			select {
			case <- letter :
				if ch > 'z' {
					wait.Done()
					return
				}
				fmt.Println("%c",ch)
				number <- true
			}
		}
	}(&wait)

	number <- true // 第一次打印触发
	wait.Wait() // 主程序等待字母打印完成
}
```

select 语法使用
```go
select {
    case communication clause  :
       statement(s);      
    case communication clause  :
       statement(s);
    /* 你可以定义任意数量的 case */
    default : /* 可选 */
       statement(s);
}
```

每个case都必须是一个通信</br>
所有channel表达式都会被求值</br>
所有被发送的表达式都会被求值</br>
如果任意某个通信可以进行，它就执行；其他被忽略</br>
如果有多个case都可以运行，Select会随机公平地选出一个执行。其他不会执行</br>
否则：</br>
如果有default子句，则执行该语句</br>
如果没有default字句，select将阻塞，直到某个通信可以运行；Go不会重新对channel或值进行求值</br>

