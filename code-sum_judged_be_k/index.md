# 算法刷题 —— 判断 nums 中是否存在某个子序列之和等于 k


> 判断 nums 中是否存在某个子序列之和等于 k ~ </br>

```go
func main() {
	a := []int{4, 1, 2, 4, 7}
	res := IsContainNumTotalK(a, 20)
	fmt.Println(res)
}

func IsContainNumTotalK(nums []int, k int) bool {
	if k == 0 {
		return true
	} else if len(nums) == 0 {
		return false
	}

	for i := 0; i < len(nums); i++ {
		return IsContainNumTotalK(nums[i+1:len(nums)], k-nums[i]) || IsContainNumTotalK(nums[i+1:len(nums)], k)
	}

	return false
}
```

