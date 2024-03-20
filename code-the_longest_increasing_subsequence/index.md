# 算法刷题 —— 最长递增子序列(Leetcode 300)



> 最长递增子序列 ~ </br>

```go
package main

func main() {

}

// O(n2)
func lengthOfLIS(nums []int) int {
    res := math.MinInt
    dp := make([]int,len(nums))
    for i := 0 ; i < len(nums) ; i++ {
        dp[i] = 1
    }
    for i := 1 ; i < len(nums) ; i++ {
        for j := 0 ; j < i ; j++ {
            if nums[i] > nums[j] {
                dp[i] = max(dp[j]+1,dp[i])
            }
        }
    }
    for i := 0 ; i < len(nums) ; i++ {
        res = max(res,dp[i])
    }
    return res
}
```

