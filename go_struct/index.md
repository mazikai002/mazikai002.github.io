# Go 基本语法 —— struct可以比较吗


>struct能不能比较？得分情况！</br>

<!--more-->


struct能不能比较？ 很显然这句话包含了两种情况：</br>
同一个struct的两个实例能不能比较？</br>
两个不同的struct的实例能不能比较？</br>
在分析上面两个问题前，先梳理一下golang中，哪些数据类型是可比较的，哪些是不可比较的：</br>
可比较：Integer，Floating-point，String，Boolean，Complex(复数型)，Pointer，Channel，Interface，Array</br>
不可比较：Slice，Map，Function</br>
同一个struct</br>
同一个struct的两个实例可比较也不可比较，当结构不包含不可直接比较成员变量时可直接比较，否则不可直接比较</br>
```
type S struct {
    Name    string
    Age     int
    Address *int
}

func main() {
    a := S{
        Name:    "aa",
        Age:     1,
        Address: new(int),
    }
    b := S{
        Name:    "aa",
        Age:     1,
        Address: new(int),
    }

      fmt.Println(a == b)
}
```
这段代码会输出false
因为上面的结构体不包含不可比较的成员变量，所以是可以比较的 
```
type S struct {
    Name    string
    Age     int
    Address *int
      Data    []int
}

func main() {
    a := S{
        Name:    "aa",
        Age:     1,
        Address: new(int),
            Data:    []int{1, 2, 3},
    }
    b := S{
        Name:    "aa",
        Age:     1,
        Address: new(int),
            Data:    []int{1, 2, 3},
    }

      fmt.Println(a == b)
}
```
这段代码的输出是：./prog.go:28:14: invalid operation: a == b (struct containing []int cannot be compared)
因为结构体包含了不可比较的成员变量slice，导致代码编译不通过
reflect.DeepEqual
用来对含有不可直接比较的数据类型的结构体实例进行比较
```
type S struct {
    Name    string
    Age     int
    Address *int
      Data    []int
}

func main() {
    a := S{
        Name:    "aa",
        Age:     1,
        Address: new(int),
            Data:    []int{1, 2, 3},
    }
    b := S{
        Name:    "aa",
        Age:     1,
        Address: new(int),
            Data:    []int{1, 2, 3},
    }

      fmt.Println(reflect.DeepEqual(a, b))
}
```
这段代码返回true
DeepEqual函数用来判断两个值是否深度一致。具体比较规则如下：
不同类型的值永远不会深度相等
当两个数组的元素对应深度相等时，两个数组深度相等
当两个相同结构体的所有字段对应深度相等的时候，两个结构体深度相等
当两个函数都为nil时，两个函数深度相等，其他情况不相等（相同函数也不相等）
当两个interface的真实值深度相等时，两个interface深度相等
map的比较需要同时满足以下几个
两个map都为nil或者都不为nil，并且长度要相等
相同的map对象或者所有key要对应相同
map对应的value也要深度相等
指针，满足以下其一即是深度相等
两个指针满足go的==操作符
两个指针指向的值是深度相等的
切片，需要同时满足以下几点才是深度相等
两个切片都为nil或者都不为nil，并且长度要相等
两个切片底层数据指向的第一个位置要相同或者底层的元素要深度相等
注意：空的切片跟nil切片是不深度相等的
其他类型的值（numbers, bools, strings, channels）如果满足go的==操作符，则是深度相等的。要注意不是所有的值都深度相等于自己，例如函数，以及嵌套包含这些值的结构体，数组等
不同的struct
可以比较，也不可以比较，不含有不可比较类型的时候可以通过强转来比较。
```
type T2 struct {
    Name  string
    Age   int
    Arr   [2]bool
    ptr   *int
}

type T3 struct {
    Name  string
    Age   int
    Arr   [2]bool
    ptr   *int
}

func main() {
    var ss1 T2
    var ss2 T3
    // Cannot use 'ss2' (type T3) as type T2 in assignment
    //ss1 = ss2     // 不同结构体之间是不可以赋值的
    ss3 := T2(ss2)
    fmt.Println(ss3==ss1) // true
}
```
这段代码返回true
但是当结构体包含不可比较的成员变量时候，强转比较也会报错
```
type T2 struct {
    Name  string
    Age   int
    Arr   [2]bool
    ptr   *int
    map1  map[string]string
}

type T3 struct {
    Name  string
    Age   int
    Arr   [2]bool
    ptr   *int
    map1  map[string]string
}

func main() {
    var ss1 T2
    var ss2 T3
    
    ss3 := T2(ss2)
    fmt.Println(ss3==ss1)   // 含有不可比较成员变量
}
```
./prog.go:28:14: invalid operation: ss3 == ss1 (struct containing map[string]string cannot be compared)
含有不能比较的结构体互转后可以使用deepequal比较
```
import (
  "fmt"
  "reflect"
)

type T2 struct {
  Name string
  Age  int
  Arr  [2]bool
  ptr  *int
  map1 map[string]string
}

type T3 struct {
  Name string
  Age  int
  Arr  [2]bool
  ptr  *int
  map1 map[string]string
}

func main() {
  var ss1 T2
  var ss2 T3

  ss3 := T2(ss2)
  fmt.Println(reflect.DeepEqual(ss3, ss1)) // 含有不可比较成员变量
}
```
这段代码返回true
问：struct可以作为map的key么
struct必须是可比较的，才能作为key，否则编译时报错
```
type T1 struct {

  Name  string

  Age   int

  Arr   [2]bool

  ptr   *int

  slice []int

  map1  map[string]string

}

type T2 struct {

  Name string

  Age  int

  Arr  [2]bool

  ptr  *int

}

func main() {

  // n := make(map[T2]string, 0) // 无报错

  // fmt.Print(n)                // map[]

  m := make(map[T1]string, 0)

  fmt.Println(m) // invalid map key type T1

}
```

# 参考
https://www.jianshu.com/p/5641648664d8
