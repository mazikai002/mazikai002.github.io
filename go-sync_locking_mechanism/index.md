# Golang 并发编程 —— 锁机制


> 在日常生活中，锁是为了保护一些东西，比如门锁、密码箱锁，可以理解对资源的保护。</br>
> 在编程世界里，锁也是为了保护资源，比如文件锁 : 同一时间只也许一个用户修改文件。</br>
> 本文就梳理了一下 Golang 中 sync 包中的锁机制。希望对您有帮助 ~</br>


<!--more-->

## sync.Mutex(互斥锁)

这是一个标准的互斥锁，平时用的也比较多，用法也非常简单，lock用于加锁，unlock用于解锁，配合defer使用，完美。

## sync.RWMutex(读写锁)

读写锁是互斥锁的升级版，它最大的优点就是支持多读，但是读和写、以及写与写之间还是互斥的，所以比较适合读多写少的场景。

```go
func (rw *RWMutex) Lock()            // 申请写锁
func (rw *RWMutex) Unlock()	         // 释放写锁
func (rw *RWMutex) RLock()           // 申请读锁
func (rw *RWMutex) RUnlock()         // 释放读锁
func (rw *RWMutex) RLocker() Locker  // 返回一个实现了Lock()和Unlock()方法的Locker接口
```

注 : 读性能上面比 Mutex 要好。

## sync.Map(并发安全的 Map )

标准库里面实现的 sync.Map，是一个自带锁的map，使用起来方便省心。
```go
package main

var m sync.Map

func main() {
    m.Store("1", 1)
    m.Store("2", 1)
    m.Store("3", 1)
    m.Store(4, "5") // 注意类型

    load, ok := m.Load("1")
    if ok {
        fmt.Printf("%v\n", load)
    }

    load, ok = m.Load(4)
    if ok {
        fmt.Printf("%v\n", load)
    }
}
```

## sync.Once
sync.Once的功能就是保证只执行一次，也算是一种锁，通常可以用于只能执行一次的初始化操作，比如说单例模式里面的懒汉模式可以用到。
```go
package main

import "sync"

var once sync.Once

func main() {
    doOnce()
    doOnce()
    doOnce()
}

func doOnce() {
    once.Do(func() {
        println("one")
    })
}
```

## sync.Cond(条件锁)
这个一般称之为条件锁，就是当满足某些条件下才起作用的锁，啥个意思呢？举个例子，当我们执行某个操作需要先获取锁，但是这个锁必须是由某个条件触发的，其中包含三种方式：
- 等待通知: wait,阻塞当前线程，直到收到该条件变量发来的通知。
- 单发通知: signal,让该条件变量向至少一个正在等待它的通知的线程发送通知，表示共享数据的状态已经改变。
- 广播通知: broadcast,让条件变量给正在等待它的通知的所有线程都发送通知。

```go
package main
import (
    "sync"
    "time"
)

var cond = sync.NewCond(&sync.Mutex{})

func main() {
    for i := 0; i < 10; i++ {
        go func(i int) {
            cond.L.Lock()
            cond.Wait() // 等待通知,阻塞当前goroutine
            println(i)
            cond.L.Unlock()
        }(i)
    }

    // 确保所有协程启动完毕
    time.Sleep(time.Second * 1)

    cond.Signal()
    cond.Signal()
    cond.Signal()

    // 确保结果有时间输出
    time.Sleep(time.Second * 1)
}
```

## sync.WaitGroup(一般用来控制协程执行顺序)
```go
package main

import "sync"

var wg sync.WaitGroup

func main() {
    for i := 0; i < 10; i++ {
        wg.Add(1) // 计数+1
        go func() {
            println("1")
            wg.Done() // 计数-1，相当于wg.add(-1)
        }()
    }
    wg.Wait() // 阻塞带等待所有协程执行完毕
}
```

## sync.Pool
这是一个池子，但是却是一个不怎么可靠的池子，sync.Pool 初衷是用来保存和复用临时对象，以减少内存分配，降低CG压力。</br>
说它不可靠是指放进 Pool 中的对象，会在说不准什么时候被GC回收掉，所以如果事先 Put 进去 100 个对象，下次 Get 的时候发现 Pool 是空也是有可能的。
```go
package main

import (
    "fmt"
    "sync"
)

type User struct {
    name string
}

var pool = sync.Pool{
    New: func() interface{} {
        return User{
            name: "default name",
        }
    },
}

func main() {
    pool.Put(User{name: "name1"})
    pool.Put(User{name: "name2"})

    fmt.Printf("%v\n", pool.Get()) // {name1}
    fmt.Printf("%v\n", pool.Get()) // {name2}
    fmt.Printf("%v\n", pool.Get()) // {default name} 池子已空，会返回New的结果
}
```
从上述输出结果可以看到，Pool就像是一个池子，我们放进去什么东西，但不一定可以取出来（如果中间有GC的话就会被清空），如果池子空了，就会使用之前定义的New方法返回的结果。

注 : 为什么这个池子会放到sync包里面呢？那是因为它有一个重要的特性就是协程安全的，所以其底层自然也用到锁机制。


## 参考
https://wangbjun.site/2020/coding/golang/locker.html</br>
