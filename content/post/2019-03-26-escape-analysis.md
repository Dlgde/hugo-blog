---
title: go语言变量逃逸(Escape Analysis)
author: qstudy
type: post
date: 2019-03-26T08:06:56+00:00
url: /archives/85
views:
  - 477
categories:
  - golang

---
#### 自动决定变量分配在栈还是堆——Escape Analysis

<!--more-->

在c/c++语言中，需要开发者学习如何进行内存分配，选用什么样的内存分配方式来适应不同的算法 要求，比如，函数局部变量尽量使用栈，全局变量使用堆分配等。go语言将整个过程整合到编译器中，由编译器分析代码的特征和代码生命周期，决定是使用堆还是栈进行内存分配。

看两个简单的例子，看下分析结果，使用 **go run -gcflags "-m -l" main.go，**在go运行时，-gcflags参数是编译参数，其中-m 是表示进行内存分配分析，-l 表示避免程序内联，也就是避免进行陈序优化。

#### 1、是否发生逃逸

<pre class="theme:vs2012-black toolbar:2 lang:go decode:true">package main

import "fmt"

func calc(a, b int) int {
	c := a + b
	var x int
	x = c * 10

	return x
}

func main() {
	m := calc(3, 2)
	fmt.Println(m)
}</pre>

执行go run -gcflags "-m -l" main.go后，输出结果如下：

<pre class="theme:vs2012-black toolbar:2 lang:zsh decode:true ">➜  testEscape go run -gcflags "-m -l" main.go
# command-line-arguments
./main.go:15:13: m escapes to heap
./main.go:15:13: main ... argument does not escape
50</pre>

可以看到“m escapes to heap”，m变量到了堆内存,发生了逃逸,因为m是calc的返回值，但后面 被fmt.Println使用。

#### 2、是否被取址

<pre class="theme:vs2012-black toolbar:2 lang:go decode:true ">package main

import "fmt"

func calc(a, b int) *int {
	c := a + b
	var x int
	x = c * 10

	return &x
}

func main() {
	m := calc(3, 2)
	fmt.Println(*m)
}</pre>

输出如下：

<pre class="theme:vs2012-black toolbar:2 lang:zsh decode:true"># command-line-arguments
./main.go:10:9: &x escapes to heap
./main.go:7:6: moved to heap: x
./main.go:15:14: *m escapes to heap
./main.go:15:13: main ... argument does not escape
50</pre>

我们看到，第三行输出：moved to heap: x，x也move到heap。

<!--more-->

内存分配在栈还是堆上看下面的两个条件：  
1、**是否被取址** ：如果被取址，则分配在堆内存  
2、**是否发生逃逸** ：发生逃逸，则分配在堆内存上面

知道这个有啥用呢，哈哈 ，这样看到闭包就不害怕了