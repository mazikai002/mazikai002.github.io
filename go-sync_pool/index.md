# Golang 性能提升 —— sync.Pool


>一句话总结：保存和复用临时对象，减少内存分配，降低 GC 压力。</br>

<!--more-->

## 定义

sync.Pool 是 Golang 内置的对象池技术，可用于**缓存临时对象**，避免因频繁建立临时对象所带来的消耗以及对 GC 造成的压力。

注 : sync.Pool 缓存的对象随时可能被无通知的清除，因此不能将 sync.Pool 用于存储持久对象的场景。

## 使用

其实，这个数据类型不难，它只提供了三个对外的方法：New、Get 和 Put。

- New（Pool struct 包含一个 New 字段，这个字段的类型是函数 func() interface{}。当调用 Pool 的 Get 方法从池中获取元素，没有更多的空闲元素可返回时，就会调用这个 New 方法来创建新的元素。如果你没有设置 New 字段，没有更多的空闲元素可返回时，Get 方法将返回 nil，表明当前没有可用的元素）
- Get（如果调用这个方法，就会从 Pool取走一个元素，这也就意味着，这个元素会从 Pool 中移除，返回给调用者。不过，除了返回值是正常实例化的元素，Get 方法的返回值还可能会是一个 nil，所以你在使用的时候，可能需要判断）
- Put（这个方法用于将一个元素返还给 Pool，Pool 会把这个元素保存到池中，并且可以复用。但如果 Put 一个 nil 值，Pool 就会忽略这个值。）

## 示例

```go
package main

import (
"fmt"
"sync"
)

var pool *sync.Pool

type Person struct {
	Name string
}

func initPool() {
	pool = &sync.Pool {
		New: func()interface{} { // 实现 New 函数。对象池中没有对象时，将会调用 New 函数创建。
			return new(Person)
		},
	}
}

func main() {
	initPool()

	p := pool.Get().(*Person) // Get() 用于从对象池中获取对象，因为返回值是 interface{}，因此需要类型转换。
	fmt.Println("首次从 pool 里获取：", p)

	p.Name = "first"
	fmt.Printf("设置 p.Name = %s\n", p.Name)

	pool.Put(p) // Put() 则是在对象使用完毕后，返回对象池。

	fmt.Println("Pool 里已有一个对象：&{first}，调用 Get: ", pool.Get().(*Person))
	fmt.Println("Pool 没有对象了，调用 Get: ", pool.Get().(*Person))
}
```

## 场景

- 高并发场景 : 一个对象会被大量创建
- 稳定创建对象场景 : 不是一次性，也不是间隔很久才创建一次

参考：

## 参考
https://marksuper.xyz/2021/09/02/sync_pool/</br>
https://geektutu.com/post/hpg-sync-pool.html</br>
