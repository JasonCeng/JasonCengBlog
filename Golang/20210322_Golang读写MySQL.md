# Golang读写MySQL

```go
package main

import (
	"fmt"

	_ "github.com/go-sql-driver/mysql"
	"github.com/jmoiron/sqlx"
)

var (
	user     string = "root"
	password string = "123456"
	host     string = "127.0.0.1"
	port     int    = 33306
	db       string = "test"
	charset  string = "utf8"
)

type Db struct {
	db *sqlx.DB
}

func GetConnect() *Db {
	dsn := fmt.Sprintf("%s:%s@tcp(%s:%d)/%s?charset=%s", user, password, host, port, db, charset)
	db, err := sqlx.Open("mysql", dsn)
	if err != nil {
		fmt.Printf("mysql connect failed, detail is [%v] \n", err.Error())
	}
	fmt.Println("connect success!!!")
	return &Db{db: db}
}

//db:标记数据库
type DataInfo struct {
	Cisno int    `db:"cisno"`
	Amt   string `db:"amt"`
	Date  string `db:"date"`
}

func (curdb *Db) Get(sqlstr string) {
	var Data *DataInfo = new(DataInfo)
	err := curdb.db.Get(Data, sqlstr)
	if err != nil {
		fmt.Printf("Get failed, detail is [%v] \n", err.Error())
	}
	//打印结构体内容
	fmt.Println(Data.Cisno, Data.Amt, Data.Date)
}

func main() {
	db := GetConnect()
	var sqlstr string = "select cisno,amt,date from test.Customer limit 1"
	db.Get(sqlstr)
}
```

## 报错

### (一) Error 1049: Unknown database 'TEST'

原因：端口配置不正确。

### (二) missing destination name field in *package.Struct

原因：在查询时使用了select *, 将查询结果映射到定义好的结构体上，无法正确进行映射。

**sqlx详解**

追踪sqlx的调用链你会找到scanAny函数，而此函数会有一个对比操作,代码如下:
```go
// v.Type() 为你传入的struct的反射类型
// columns 为你查询的数据库列
fields := m.TraversalsByName(v.Type(), columns)
// if we are not unsafe and are missing fields, return an error
if f, err := missingFields(fields); err != nil && !r.unsafe {
    return fmt.Errorf("missing destination name %s in %T", columns[f], dest)
}
```
此段代码会对比你查询的数据库字段和映射的结构体字段,如果结构体中不存在这个字段就会报` "missing destination name"`