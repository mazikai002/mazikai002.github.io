# 算法刷题 —— 快速排序


> 快速排序 ~ </br>

```go
func main() {
	a := []int{4, 1, 2, 4, 7}
	res := IsContainNumTotalK(a, 20)
	fmt.Println(res)
}

func quickSort(nums []int, left, right int) {
	if left >= right { // 下标重叠不用排序
		return
	}
	pivot := partition(nums,left,right)
	quickSort(nums,left,pivot-1)
	quickSort(nums,pivot+1,right)
}

func partition(nums []int, left, right int) int {
	i, j := left, right
	for i < j {
		for i < j && nums[j] >= nums[left] {
			j--
		}
		for i < j && nums[i] <= nums[left] {
			i++
		}
		nums[i], nums[j] = nums[j], nums[i]
	}
	nums[i], nums[left] = nums[left], nums[i]
	return i
}
```

