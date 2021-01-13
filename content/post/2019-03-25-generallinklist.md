---
title: go语言实现通用链表
author: qstudy
type: post
date: 2019-03-24T18:43:15+00:00
url: /archives/81
views:
  - 362
categories:
  - golang

---


实现通用链表



  1. link.go

```go
package main

import "fmt"

type LinkNode struct {
	data interface{}
	next *LinkNode
}

type Link struct {
	head *LinkNode
	tail *LinkNode
}

func (p *Link) InsertHead(data interface{}) {
	node := &LinkNode{
		data: data,
		next: nil,
	}
	if p.tail == nil && p.head == nil {
		p.head = node
		p.tail = node
		return
	}
	node.next = p.head
	p.head = node

}

func (p *Link) InsertTail(data interface{}) {
	node := &LinkNode{
		data: data,
		next: nil,
	}

	if p.tail == nil && p.head == nil {
		p.head = node
		p.tail = node
		return
	}
	p.tail.next = node
	p.tail = node

}

func (p *Link) Trans() {
	q := p.head
	for q != nil {
		fmt.Println(q.data)
		q = q.next
	}

}

func (p *Link) Getlen() {
	q := p.head
	m := 0
	for q != nil {
		m++
		q = q.next
	}
	fmt.Println(m)
}
```

  2. main.go

```go
package main

func main() {
	var l Link
	for i := 1; i &lt; 10; i++ {
		l.InsertTail(i)
		//l.InsertHead(i)
		//l.InsertTail(fmt.Sprintf("str %d", i))

	}

	l.Trans()
	l.Getlen()
}
```