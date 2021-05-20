---
title: golang中的map
author: qstudy
type: post
date: 2019-11-16T06:09:14+00:00
url: /archives/207
views:
  - 348
love:
  - 1
categories:
  - golang

---
## golang中map学习

### 哈希表

哈希表是一个无序的key/value对的集合，其中所有的key都是不同的，然后通过给定的key可以在常数时间复杂度内检索、更新或删除对应的value；

### golang中的map

在Go语言中，一个map就是一个哈希表的引用，map类型可以写为map[K]V，其中K和V分别对应key和value。map中所有的key都有相同的类型，所有的value也有着相同的类型，但是key和value之间可以是不同的数据类型。

  1. 在向map中插入数据之前，必须要创建一个map,下面是两种常见的方法 
  ```go
    mp:=make(map[string]int)
    mp["liming"]=12
    mp["xiaoming"]=21
  ```
    
  ```go
   mp:=map[string]int{
   "liming":12
   "xiaoming":21
   }
  ```

  2. map中的元素并不是一个变量，对map不能进行取址；并可能随着元素的增加，会改变地址容纳更多的元素;</p> 
  3. Map的迭代顺序是不确定的，并且不同的哈希函数实现可能导致不同的遍历顺序。在实践中，遍历的顺序是随机的，每一次遍历的顺序都不相同;

  4. map不能直接进行相等比较，唯一例外的是和nil进行比较；判断两个map是否含有相同的key，value;可以通过一个循环来实现；

  5. 下面这种语句要理解，在这种场景下，map的下标语法将产生两个值；第二个是一个布尔值，用于报告元素是否真的存在。布尔变量一般命名为ok，特别适合马上用于if条件判断部分；  
  `if age, ok := ages["bob"]; !ok { /* ... */ }`

  6. 上面的3，4可用下面的代码进行验证： 
```go
package main
import "fmt"

func listMap(a map[string]int){
for k,v :=range a{
    fmt.Println(k,v)
}
}

func equal (a,b map[string]int) bool{
if len(a)!=len(b){
    return false
}
for k,av :=range a{
    if bv,ok:=b[k];!ok||av!=bv{
        return false
    }
}
return true
}

func main() {
a:=map[string]int{
    "liming":12,
    "stanni":18,
    "monica":22,
    "zhangqiamg":55,
}
b:=map[string]int{
    "liming":12,
    "stanni":18,
    "monica":22,
    "zhangqiamg":55,
}
listMap(a)
listMap(b)
fmt.Println(equal(a,b))

}
```
### golang中对map进行排序
golang的map是无序的，不管是按照 key 还是value 默认都不排序, 如果要为 map 排序，需要将 key 拷贝到一个切片，对切片排序，然后使用切片的 for-range 方法打印出map中所有的 key 和 value。例如:
```go
package main

import "fmt"

// func listmap(mp map[string]int) {
// 	for k, v := range mp {
// 		fmt.Println(k, v)
// 	}
// }

func main() {
	mp := map[string]int{
		"lq": 9,
		"xw": 10,
		"mm": 8,
	}
	// listmap(mp)

	s := []string{}
	for k := range mp {
		s = append(s, k)
	}

	for _, n := range s {
		fmt.Println(n, mp[n])
	}

}

```