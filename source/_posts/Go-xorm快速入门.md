---
title: Go_xorm快速入门
author: GopherTy
top: false
cover: false
toc: true
date: 2019-10-28 20:10:05
coverImg:
password:
summary:
tags:
    - Go
    - Go语言ORM库
    - xorm
categories: 编程语言
reprintPolicy:
---
# Go-xorm 快速入门

## xorm是什么

xorm是一个简单而强大的Go语言ORM库. 通过它可以使数据库操作非常简便。

## 特性

- 支持Struct和数据库表之间的灵活映射，并支持自动同步
- 事务支持
- 同时支持原始SQL语句和ORM操作的混合执行
- 使用连写来简化调用
- 支持使用Id, In, Where, Limit, Join, Having, Table, Sql, Cols等函数和结构体等方式作为条件
- 支持级联加载Struct
- Schema支持（仅Postgres）
- 支持缓存
- 支持根据数据库自动生成xorm的结构体
- 支持记录版本（即乐观锁）
- 内置SQL Builder支持
- 上下文缓存支持

## 驱动支持

目前支持的Go数据库驱动和对应的数据库如下：

- Mysql: [github.com/go-sql-driver/mysql](https://github.com/go-sql-driver/mysql)
- MyMysql: [github.com/ziutek/mymysql/godrv](https://github.com/ziutek/mymysql/godrv)
- Postgres: [github.com/lib/pq](https://github.com/lib/pq)
- Tidb: [github.com/pingcap/tidb](https://github.com/pingcap/tidb)
- SQLite: [github.com/mattn/go-sqlite3](https://github.com/mattn/go-sqlite3)
- MsSql: [github.com/denisenkom/go-mssqldb](https://github.com/denisenkom/go-mssqldb)
- MsSql: [github.com/lunny/godbc](https://github.com/lunny/godbc)
- Oracle: [github.com/mattn/go-oci8](https://github.com/mattn/go-oci8) 

## 安装

```
go get github.com/go-xorm/xorm
```

## 相关文档

[官方文档](http://xorm.io/docs)

[官方文档(中文版)](https://www.kancloud.cn/kancloud/xorm-manual-zh-cn/56013)

[Godoc代码文档](https://godoc.org/github.com/go-xorm/xorm)

## 快速入门

简单示例，更多操作请查看官方文档。

```go
package main

import (
	"fmt"
	"log"
	"os"

	_ "github.com/go-sql-driver/mysql"
	"github.com/go-xorm/xorm"
)

var engine *xorm.Engine

// User 用户表
type User struct {
	ID     int `xorm:"Int  pk notnull autoincr unique id "` // 用户表的主键 后面为 xorm 的 tags，具体实现可查看官方文档。
	Name   string
	Passwd string
}

func main() {
	// 创建日志文件(默认该文件不存在)
	var file *os.File
	defer file.Close()
	if isFileExist("sql.log") {
		file, err := os.Create("sql.log")
		if err != nil {
			fmt.Println("create log file fail", err)
		}
		defer file.Close()
	}

	logger := xorm.NewSimpleLogger(file)                                         // 创建日志对象
	engine, err := xorm.NewEngine("mysql", "root:123@/test?charset=utf8") // 创建数据库引擎
	if err != nil {
		logger.Error("create engine fail ", err)
		os.Exit(1)
	}

	// 创建 User 表
	ok, _ := engine.IsTableExist(&User{})
	if !ok {
		err = engine.CreateTables(&User{})
		if err != nil {
			logger.Error("create  tabel fail ", err)
			os.Exit(1)
		}
	}

	// 开启事务
	session := engine.NewSession()
	defer session.Close()
	err = session.Begin()
	if err != nil {
		logger.Error("start session fail", err)
		os.Exit(1)
	}

	// 插入数据
	ok, _ = engine.IsTableEmpty(&User{})
	if ok {
		user1 := &User{
			ID:     1,
			Name:   "hh",
			Passwd: "123",
		}
		_, err := engine.Insert(user1)
		if err != nil {
			logger.Error("insert data error", err)
			os.Exit(1)
		}
	}

	//使用 sql 语句方式插入数据
	user := &User{}
	_, err = engine.Exec("insert into `user`(name,passwd) values(?,?) ", "hello", "test")
	if err != nil {
		session.Rollback()
		logger.Error("exec  fail", err)
		os.Exit(1)
	}

	// 查询结果
	rows, err := engine.Where("id >= ?", 1).Rows(user)
	if err != nil {
		logger.Error("select  data fail", err)
		os.Exit(1)
	}
	defer rows.Close()
	for rows.Next() {
		err = rows.Scan(user)
		if err != nil {
			logger.Error("range  fail", err)
			os.Exit(1)
		}
		log.Println(user)
	}

	// 提交事务
	err = session.Commit()
	if err != nil {
		logger.Error("session commit fail", err)
		os.Exit(1)
	}
}

// isFileExist
func isFileExist(path string) (b bool) {
	_, err := os.Stat(path) // 获取文件的信息
	if err != nil && os.IsNotExist(err) {
		log.Println("file is not exist", err)
		return true
	}
	return
}

```

