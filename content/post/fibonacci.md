---
title: "Fibonacci"
date: 2021-04-04T23:28:44+08:00
draft: 
categories:
  - 算法
tags:
  - LeetCode
---
509. 斐波那契数

```go
package main

import "fmt"

//动态规划
func fib(n int) int {
	if n < 2 {
		return n
	}
	nums := make([]int, n+1)
	nums[0], nums[1] = 1, 1
	for i := 2; i < n; i++ {
		// nums[i] = (nums[i-1] + nums[i-2]) % 1000000007
		nums[i] = nums[i-1] + nums[i-2]
	}
	return nums[n]
}
//递归
func fib01(n int) int {
	if n < 2 {
		return 2
	}
	return fib01(n-1) + fib(n-2)
}

func main() {
	var n int
	fmt.Scanln(&n)
	fmt.Println(fib(n))
	fmt.Println("the fib01 is ", fib01(n))

}
```

