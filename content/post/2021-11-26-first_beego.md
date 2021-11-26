---
title: "First_beego"
date: 2021-11-26T23:27:26+08:00
draft: 
url: /2021/11/26/
categories:
  - golang
tags:
  - beego
---

### 使用beego创建api项目，实现增删改查，简单记录

#### 1.mac安装beego和bee工具

```shell
go get -u github.com/beego/beego/v2
go get -u github.com/beego/bee/v2
```

* 如果需要bee设置为全局变量，添加zshrc全局alias，在~/.zshrc中编辑添加alias bee=xxx/bee
* [beego官方文档](https://beego.vip/)

#### 2.使用bee工具创建项目

```shell
bee api app_delivery -conn="root:root****@tcp(127.0.0.1:3306)/test-beego
```

* 创建后orm.RegisterDataBase("default", "mysql", beego.AppConfig.String("sqlconn"))有报错，通过参数接收下beego.AppConfig.String("sqlconn"）,因为有两个返回值

#### 3.models层user.go实现代码

```go
package models

import (
	"fmt"
	"github.com/beego/beego/v2/client/orm"
	beego "github.com/beego/beego/v2/server/web"
	_ "github.com/go-sql-driver/mysql"
)

func init() {
	orm.Debug = true
	orm.RegisterDriver("mysql", orm.DRMySQL)
	db,_ := beego.AppConfig.String("sqlconn")
	orm.RegisterDataBase("default", "mysql", db)
	orm.RegisterModel(new(User))
	orm.RunSyncdb("default", false, true)
}


type User struct {
	Id           int       `orm:"column(id);pk"`
	Name         string    `orm:"column(name)"`
	UserId        string    `orm:"column(User_id)"`
}

func AddUser(user *User) error{
	o:= orm.NewOrm()
	_, err := o.Insert(user)
	return err
}

func GetUser(user_id string) (*User, bool){
	o := orm.NewOrm()
	User := User{UserId: user_id}
	err := o.Read(&User,"User_id")
	if err == orm.ErrNoRows {
		fmt.Println("User not find")
		return &User, false
	} else if err == orm.ErrMissPK {
		fmt.Println("cant not find prikey")
		return &User, false
	} else {
		return &User, true
	}
}

func DeleteUser(user_id string) bool {
	o := orm.NewOrm()
	User := User{UserId: user_id}
	num, err := o.Delete(&User,"user_id")
	if err == nil {
		fmt.Println("delete num:", num)
		return true
	} else {
		return false
	}
}

func GetUserAll() []User{
	o := orm.NewOrm()
	qs:= o.QueryTable("user")
	var users []User
	count, err := qs.All(&users)
	if err != nil{
		fmt.Println("get all User failed")
	}
	fmt.Println("the count User is ",count)
	return users
}

```



#### 4.controllers层user.go实现代码

```go
package controllers

import (
	"test-beego/models"
	"encoding/json"
	"fmt"
	beego "github.com/beego/beego/v2/server/web"

)

type UserController struct{
	beego.Controller
}

type Result struct {
	ErrNo    int           `json:"errno"`
	ErrMsg   string        `json:"errmsg"`
	Data     interface{}   `json:"data"`
}


func (d *UserController) CreateUser(){
	var user models.User
	var r Result
	defer func() {
		d.Data["json"] = r
		d.ServeJSON()
	}()
	err := json.Unmarshal(d.Ctx.Input.RequestBody, &user)
	if err !=nil{
		r.ErrMsg = fmt.Sprintf(" json unmarshal failed")
	}
	err :=models.AddUser(&User)
	if err != nil{
		r.ErrMsg=fmt.Sprintf("add user failed")
	}
	r.Data = user
}

func (d *UserController) GetUser(){
	var r Result
	defer func() {
		d.Data["json"] = r
		d.ServeJSON()
	}()
	r.Data,_ = models.GetUser(d.GetString(":user_id"))
	fmt.Println(d.GetString(":user_id"))
}

func (d *UserController) DeleteUser(){
	var r Result
	defer func() {
		d.Data["json"] = r
		d.ServeJSON()
	}()
	DelUserId := d.GetString(":user_id")
	IsTrue:=models.DeleteUser(DelUserId)
	if !IsTrue {
		r.ErrMsg =fmt.Sprintf("delete User failed")
	}
	var arry []string
	arry = append(arry, DelUserId)
	mp:= make(map[string][]string)
	mp["success_list"]= arry
	r.Data = mp
}

func (d *UserController) GetUserAll(){
	var r Result
	defer func() {
		d.Data["json"] = r
		d.ServeJSON()
	}()
	r.Data = models.GetUserAll()
}
//TODO 懒得写了
func (c *UserController) EditTask(){

}
```



#### 5.路由router.go代码

```go
package routers

import (
	"app_delivery/controllers"
	beego "github.com/beego/beego/v2/server/web"
)

func init() {
	ns := beego.NewNamespace("/v1")
	beego.AddNamespace(ns)
	beego.Router("/api/users", &controllers.DeliveryController{}, "post:CreateUser")
	beego.Router("/api/users/?:user_id", &controllers.DeliveryController{}, "get:GetUser")
	beego.Router("/api/users/?:user_id", &controllers.DeliveryController{}, "delete:DeleteUser")
	beego.Router("/api/users/getall", &controllers.DeliveryController{}, "get:GetUserAll")
}

```



#### 6. Main.go主要代码

```go
package main

import (
	_ "app_delivery/routers"
	beego "github.com/beego/beego/v2/server/web"
	_ "github.com/go-sql-driver/mysql"
)

func main() {
	if beego.BConfig.RunMode == "dev" {
		beego.BConfig.WebConfig.DirectoryIndex = true
		beego.BConfig.WebConfig.StaticDir["/swagger"] = "swagger"
	}
	beego.Run()
}

```

