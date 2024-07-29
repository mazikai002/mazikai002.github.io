# 算法刷题 —— 数组中的第K个最大元素


> 数组中的第K个最大元素 —— 堆排序 ~ </br>

```go
func main() {
	res := []int{4, 1, 2, 3, 7}
	res := quickSort(res)
	fmt.Println(res)
}

// 最小堆
type Heap struct {
	arr []int // 堆存储数据的区域
	size int  // 堆的大小
}

func(hp *Heap) Add(num int) {
	if len(hp.arr) < hp.size { // 堆内元素没满
	


	} else { // 堆内元素满了
		if num > hp.arr[0] {
			hp.arr[0] = num
			hp.down(0)
		}
	}
}

func(hp *Heap) down(i int) {
	leftChild , rightChild := 2*i+1 , 2*i+2 // 左右孩子下标
	cur := i // cur指向i、leftChild、leftChild三者中的最小值下标
	if leftChild < len(hp.arr) && hp.arr[cur] > hp.arr[leftChild] {
		cur = leftChild
	}
	if rightChild < len(hp.arr) && hp.arr[cur] > hp.arr[rightChild] {
		cur = rightChild
	}
	if cur != i {
		hp.arr[i] , hp.arr[cur] = hp.arr[cur] , hp.arr[i]
		Down(cur)
	}
}
```

