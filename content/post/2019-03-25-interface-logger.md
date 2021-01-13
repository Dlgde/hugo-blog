---
title: 接口应用-可扩展输出方式的日志系统
author: qstudy
type: post
date: 2019-03-25T15:40:56+00:00
url: /archives/83
views:
  - 350
categories:
  - golang

---
##### 利用接口实现一个可扩展的日志系统：



1、日志对外接口：

本例中定义一个日志写入的接口(LogerWriter),要求写入设备必须遵守这个接口协议(实现这个接口)才能被日志器(Logger)注册。日志器有ResgisterWriter()和Log()两个方法，ResgisterWriter()方法将日志写入器(LogWriter)注册到日志器中，log()方法进行日志的输出，这个函数会将日志写入到所有已经注册的日志写入器(LogWriter)中。本例中简单实现文件和输出到终端这两种方式。



```go 
package main

type LogWriter interface {
	Write(data interface{}) error
}

type Logger struct {
	writeList []LogWriter
}

func (l *Logger) RegisterWriter(writer LogWriter) {
	l.writeList = append(l.writeList, writer)
}

func (l *Logger) Log(data interface{}) {
	for _, writer := range l.writeList {
		writer.Write(data)
	}
}

func NewLogger() *Logger {
	return &Logger{}
}
```

2、文件写入器：

```go
package main

import (
	"errors"
	"fmt"
	"os"
)

type fileWriter struct {
	file *os.File
}

func (f *fileWriter) SetFile(filename string) (err error) {
	if f.file != nil {
		f.file.Close()
	}
	f.file, err = os.Create(filename)
	return err
}
func (f *fileWriter) Write(data interface{}) error {
	if f.file == nil {
		return errors.New("file not created")
	}
	str := fmt.Sprintf("%v\n", data)

	_, err := f.file.Write([]byte(str))

	return err
}

func newFileWriter() *fileWriter {
	return &fileWriter{}
}
```

3、命令行写入器：

```go
package main

import (
	"fmt"
	"os"
)

type consoleWriter struct {
}

func (f *consoleWriter) Write(data interface{}) error {
	str := fmt.Sprintf("%v\n", data)
	_, err := os.Stdout.Write([]byte(str))
	return err

}

func newConsolerWriter() *consoleWriter {
	return &consoleWriter{}
}
```

4、main.go

```go 
package main

import "fmt"

func createLogger() *Logger {
	l := NewLogger()
	cw := newConsolerWriter()

	l.RegisterWriter(cw)

	fw := newFileWriter()
	if err := fw.SetFile("log.log"); err != nil {
		fmt.Println(err)
	}

	l.RegisterWriter(fw)

	return l
}

func main() {
	l := createLogger()

	l.Log("hello")
}
```