# Golang 基本语法 —— 浅谈 Context


>context为Go中的一个标准库, 专门用来处理多个协程之间的控制问题, 比如协程的取消, 协程运行截止时间, 协程运行的超时时间, 协程之间的数据传输等.</br>

<!--more-->

# context使用规则

1. context要作为函数的第一个参数使用, 不要试图将它放入结构体中
2. 当方法或函数需要一个context时, 不要传入nil, 可以传入context.TODO
3. context可以传入到不同的协程中,** 并且在多个协程中是安全的**
4. 当context传入到子协程中时, 需要在子协程中监控conext.Done()返回的通道, 当收到通知时, 停止当前子协程的运行, 释放资源并返回到上层

# 并发安全？

```
func main()  {
 ctx := context.WithValue(context.Background(), "asong", "test01")
 go func() {
  for {
   _ = context.WithValue(ctx, "asong", "test02")
  }
 }()
 go func() {
  for {
   _ = context.WithValue(ctx, "asong", "test03")
  }
 }()
 go func() {
  for {
   fmt.Println(ctx.Value("asong"))
  }
 }()
 go func() {
  for {
   fmt.Println(ctx.Value("asong"))
  }
 }()
 time.Sleep(10 * time.Second)
}

```

`context`添加的键值对一个链式的，会不断衍生新的`context`，所以`context`本身是不可变的**，因此是线程安全的。**

!https://secure2.wostatic.cn/static/uSBmnegpS63aHSVxPCKCkq/image.png?auth_key=1709437657-jBqxaGan91qXdknbrRbeKZ-0-fb5050a3090ee6e8b1e7750c44241b3c

## Context包 父子关系

**控制是从上至下的，查找是从下至上的。**

context 的实例之间存在父子关系：

- 当父亲取消或者超时，所有派生的子context 都被取消或者超时
- 找 key 的时候，子 context 先看自己有没有，没有则去祖先里面找控制是从上至下的，查找是从下至上的。
- 父context是无法拿到子context的key的值得，只能看到自己的

# context.WithCancel的使用

withcancel 使用demo，我理解就是当前线程用来控制子协程退出的工具

```
package main

import (
  "context"
  "fmt"
  "time"
)

func slowfunc1(ctx context.Context) {
  defer func() {
    fmt.Println("end slowfunc1")
  }()
  fmt.Println("start slowfunc1")
  for {
    select {
    case <-ctx.Done():
      fmt.Println("slowfunc1 end by main")
      return
    default:
    }
  }
}

func slowfunc(ctx context.Context) {
  defer func() {
    fmt.Println("end slowfunc")
  }()
  fmt.Println("start slowfunc")
  go slowfunc1(ctx)
  for {
    select {
    case <-ctx.Done():
      fmt.Println("slowfunc end by main")
      return
    default:
    }
  }
}

func main() {
  fmt.Println("start main ")
  ctx := context.Background()
  ctxWithCancel, cancelfunc := context.WithCancel(ctx)
  defer func() {
    //cancelfunc()
    time.Sleep(2 * time.Second) // 不等待的话子线程来不及打印，但是会退出
    fmt.Println("end main")
    // cancelfunc()
  }()
  go slowfunc(ctxWithCancel)
  time.Sleep(3 * time.Second)
  cancelfunc()
}

```

# context.WithValue的使用

```
func valueFunc()  {
    // 使用context.WithValue创建一个conext
    // 设置要通过context要传递的键值
    ctx := context.WithValue(context.Background(), "key", "value")
 
    go valueFunc2(ctx)
 
    time.Sleep(time.Second)
 }
 
 func valueFunc2(ctx context.Context)  {
 
    // 通过context提供的Value方法来获取数据
    v := ctx.Value("key")
 
    fmt.Println("value: ", v)
 }

```

# 参考
https://juejin.cn/post/7001273893780471845
