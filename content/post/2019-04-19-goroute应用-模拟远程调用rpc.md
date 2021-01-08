---
title: goroute应用-模拟远程调用RPC
author: qstudy
type: post
date: 2019-04-19T14:08:51+00:00
url: /archives/150
views:
  - 530
categories:
  - golang
tags:
  - rpc

---
### 

### 在本例中，使用通道代替socket实现RPC过程，客户端和服务器运行在同一个进程，服务器和客户端在两个goroute中。

##### 应用知识：_**goroute，select，RPC过程**_

#### 

#### 1、客户端请求和接收封装

下面的代码封装了向服务器请求数据，等待服务器返回数据，如果请求方超时，利用select，该函数还会处理超时逻辑。如下：

<!--more-->

<pre class="lang:default decode:true">func RPCClient(ch chan string, req string) (string, error) {
	ch &lt;- req
	select {
	case ack := &lt;-ch:
		return ack, nil
	case &lt;-time.After(time.Second):
		return "", errors.New("Time out")
	}
}</pre>

&nbsp;

<!--more-->

#### 2、服务器接收和反馈数据

服务器接收到客户端的任意数据后，先打印再通过通道返回给客户端一个固定的字符串（hello），表示服务器已经收到请求。

<!--more-->

<pre class="lang:default decode:true ">func RPCServer(ch chan string) {
	for {
		data := &lt;-ch
		fmt.Println("server received:", data)
		//time.Sleep(time.Second * 2)
		ch &lt;- "roger"
	}

}</pre>

注：为了模拟服务端相应超时，可以用time.Sleep()函数让goroute执行暂停2秒，这样会触发客户端“Time out”

&nbsp;

<!--more-->

#### 3、主流程

主流程中会创建一个无缓冲的字符串格式通道。将通道传给服务器的RPCServer()函数，这个函数会并发执行，使用RPCClient函数通过ch对服务器发出RPC请求同时接收服务器反馈数据或者等待超时。

<!--more-->

<pre class="lang:default decode:true">func main() {
	ch := make(chan string)
	go RPCServer(ch)

	recv, err := RPCClient(ch, "hi")

	if err != nil {
		fmt.Println(err)
	} else {
		fmt.Println("client received", recv)
	}
}</pre>

&nbsp;

<!--more-->

#### 4、输出结果如下：

<pre class="lang:default decode:true ">server received: hi
client received helllo</pre>

&nbsp;