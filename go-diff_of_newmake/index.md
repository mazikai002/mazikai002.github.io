# Golang 基本语法 —— var、new、make区别及使用


>make和new是内置函数，不是关键字，var是关键字。</br>
>make 只能用来分配及初始化类型为 slice、map、chan 的数据，而 new 可以分配任意类型的数据。</br>
>new 分配返回的是指针，即类型 *Type。make 返回引用，即 Type。</br>
>new 分配的空间被清零。make 分配空间后，会进行初始化。</br>

<!--more-->


一：对于值类型的变量，我们通过var 声明(包括结构体)，系统会默认为他分配内存空间，并赋该类型的零值。如下，我们声明一个int类型变量i，输出为0。

```
var i int
fmt.Println(i) //i=0

```

而如果我们声明一个指针类型的变量，系统不会为他分配内存(应该是不会为这个指针分配一个内存给他指向)，默认就是nil。此时如果你想直接使用，那么系统会抛异常。

```
var j *int
fmt.Println(j)  //nil类型，因为指针的0值就是nil
fmt.Println(&j) //0xc00000e028 这是指针变量的地址
fmt.Println(*j) //因为这个指针没有指向任何内存,报错
*j = 10  //invalid memory address or nil pointer dereference，

```

空指针还没指向任何内存，是不能使用的。

二：那么要想使用，此时就需要new出场啦。

```
var j *int
j = new(int)  // j里面的内容指向一块分配好的内存地址，地址里面设置int的零值：0
fmt.Println(j) // 0xc000018040
fmt.Println(*j) //0
*j = 10
fmt.Println(*j)

```

声明指针类型变量后，通过new为他分配指向的内存，有了内存空间，这个变量就可以自由的使用了。

来看一下new函数的签名：

![new函数签名](new函数.png)

它只接受一个参数，这个参数是一个类型，**分配好内存后，返回一个指向该类型内存地址的指针。这块内存上存的是类型的零值。**这个类型也能是指针。

```
package main

import (
  "fmt"
)

func main(){
  a := new(*int)
  fmt.Println(a)
  fmt.Println(*a)
  *a = new(int)
  fmt.Println(*a)
  fmt.Println(**a)
}

0xc0000b2018
<nil>
0xc0000b8010
0

```

- *但是，**实际在工程使用中，通常是直接声明指针使用，不需要new操作。

三：make和new不同，make用于map, slice,chan 的内存创建，因为他们三个是引用类型，直接返回这三个类型本身。make签名是：

```
func make(t Type, size ...IntegerType) Type

```

make 是分配内存并初始化，初始化并不是置为零值。

与new一样，它的第一个参数也是一个类型，但是不一样的是，make**返回的是传入的类型，而不是指针**！

```
var c chan int //声明管道类型变量c，此时c还是nil，不可用；
fmt.Printf("%#v \\n",c) //(chan int)(nil)
c = make(chan int)
fmt.Printf("%#v", c) //(chan int)(0xc000062060)

声明管道类型变量c，此时c还是nil，不可用；
通过make来分配内存并初始化，c就获得了内存可以使用了。
所以，我们在使用map, slice,chan 的时候，需要先对他们用make初始化，然后在进行操作。

```

总结：

var  分配内存，赋0值，返回类型

new 分配内存，赋0值，返回类型指针（指向刚才分配内存的指针）

make给map，channel，slice分配内存，不赋0值，返回类型

Var指针类型一般是var+new；Var三种引用类型时候，一般是var+make

# make和new的区别

1. new 和 make都是用来创建和分配内存的，可能在栈上，也可能在堆上，是内存逃逸分析后确定创建在哪里的。
2. `new` 返回的是**指针**，指向新分配的类型的零值；`make` 返回的是**值**，这个值已经被初始化（例如，一个新的切片，它的元素已经被分配了内存）。
3. `new` 可以用于任何类型，而 `make` 只能用于切片、映射和通道。
4. `make` 创建的切片、映射或通道已经准备好使用，而 `new` 创建的值需要进一步初始化。

# 为什么要给三种引用类型单独实现一个make 函数？

这是因为slice, map和chan的底层结构上要求在使用slice，map和chan的时候必须初始化，如果不初始化，那slice，map和chan的值就是零值，也就是nil。我们知道：

1. map如果是nil，是不能往map插入元素的，插入元素会引发panic
2. chan如果是nil，往chan发送数据或者从chan接收数据都会阻塞
3. slice会有点特殊，理论上slice如果是nil，也是没法用的。但是[append函数](https://www.zhihu.com/search?q=append%E5%87%BD%E6%95%B0&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22:%22answer%22,%22sourceId%22:2245768201%7D)处理了nil slice的情况，可以调用append函数对nil slice做扩容。但是我们使用slice，总是会希望可以自定义长度或者容量，这个时候就需要用到make。

那么，可以使用new来初始化引用类型吗？可以的！

# 参考
https://huweicai.com/</br>
https://draveness.me/golang/docs/part2-foundation/ch05-keyword/golang-make-and-new/</br>
https://blog.wu-boy.com/2021/06/what-is-different-between-new-and-make-in-golang/</br>
https://fivezh.github.io/2020/03/15/golang-new-make/</br>
