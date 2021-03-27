# golang与HTTP服务

## 一个golang实现的最简单的Http服务器程序
```go
package main
 
import (
    "fmt"
    "net/http"
)
 
func IndexHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "hello world")
}
 
func main() {
    http.HandleFunc("/", IndexHandler)
    http.ListenAndServe("127.0.0.1:8000", nil)
}
```

## Golang简单实现HTTP服务器返回json数据
```
package main
import (
        "io"
        "net/http"
		"log"
		"encoding/json"
)
type Data struct{
	Name string
	Age int
}
type Ret struct{
	Code int
	Param string
	Msg string
	Data []Data
}
func HelloServer(w http.ResponseWriter, req *http.Request) {
	data := Data{Name: "why", Age: 18}
 
	ret := new(Ret)
	id := req.FormValue("id")
	//id := req.PostFormValue('id')
 
	ret.Code = 0
	ret.Param = id
	ret.Msg = "success"
	ret.Data = append(ret.Data, data)
	ret.Data = append(ret.Data, data)
	ret.Data = append(ret.Data, data)
	ret_json,_ := json.Marshal(ret)
	
	io.WriteString(w, string(ret_json))
}
 
func main() {
		http.HandleFunc("/hello", HelloServer)
        err := http.ListenAndServe(":8080", nil)
        if err != nil {
                log.Fatal("ListenAndServe: ", err)
        }
}
```

## golang对HTTP的处理
### (一)获取GET参数
```go
query := req.URL.Query()
get_act := query["act"][0]
fmt.Println(get_act)
```

### (二)获取POST参数

 

#### 2.1 获取单个POST字段值
```go
post_act := req.PostFormValue("act")
fmt.Println(post_act)
```

#### 2.2 获取多个POST字段值

如果我们在提交的时候，有两个一样input-name怎么办呢？如下的html代码
```html
<form method="POST" action="login?act=in">
<input type="text" name="username" class="text" />
<input type="text" name="username" class="text" />
<input type="submit" name="login_button" value="登录">
</form>
```

这时候就要把post传输的参数当成数组，通过PostFormValue就只能获得第一个input的内容。怎么能两个都获取到呢？
```go
post_act := req.PostForm["username"]
fmt.Println(post_act)
```

参考：

https://blog.csdn.net/lengyuezuixue/article/details/79094323

https://blog.csdn.net/why444216978/article/details/102261982

https://studygolang.com/articles/5903