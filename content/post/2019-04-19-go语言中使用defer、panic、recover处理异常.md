---
title: go语言中使用defer、panic、recover处理异常
author: qstudy
type: post
date: 2019-04-19T14:59:56+00:00
url: /archives/152
views:
  - 417
categories:
  - golang
tags:
  - go异常

---
#### go语言中的异常处理，没有try...catch等，而是使用defer、panic、recover来处理异常。

##### _知识点：defer、panic、recover，异常处理_

1、首先，panic 是用来表示非常严重的不可恢复的错误的。在Go语言中这是一个内置函数，如果在程序中遇到异常，或者调用panic函数，程序会立即退出（除非recover）。如下代码：

```go
package main

import "fmt"

func main() {
	a := 10
	b := 0
	c := a / b

	fmt.Println(c)
}
```
  

程序的输出如下：
  
 ```bash
demo06 go run main.go
panic: runtime error: integer divide by zero

goroutine 1 [running]:
main.main()
        /Users/qstudy/myfiles/project/src/go_dev/day11/demo06/main.go:8 +0x11
exit status 2
```
2、defer能保证在函数结束最后执行该方法（有问题的或者引发panic的函数),但是有个条件：必须在错误出错之前进行拦截，在错误出现后进行错误捕获，如果在定义的方法中defer定义的方法如果在panic后面，defer定义的方法就无法执行到。如下代码：

```go
package main

import "fmt"

func test(i int) {
	var arr [10]int
	// 错误拦截必须配合defer使用，通过匿名函数使用，在错误之前引用
	defer func() {
		err := recover()
		if err != nil {
			fmt.Println(err)
		}
	}()

	arr[i] = 123
	fmt.Println(arr)
}

func main() {
	i := 10
	test(i)
	fmt.Println("hello world")
}
```
  
执行后输出如下：
  
```bash
runtime error: index out of range
hello world</pre>
```

从以上结果可以看出，如果panic被recover捕获接收到，panic后的方法还是能继续执行的。

来自一本书书上面的总结：

“当在一个函数执行过程中调用panic()函数时，正常的函数执行流程将立即终止，但函数中  
之前使用**defer**关键字延迟执行的语句将正常展开执行，之后该函数将返回到调用函数，并导致逐层向上执行panic流程，直至所属的goroutine 中所有正在执行的函数被终止。错误信息将被报告，包括在调用panic()函数时传入的参数，这个过程称为错误处理流程。”
