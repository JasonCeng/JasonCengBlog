# 0x00_Windows下Golang环境搭建

## Go下载安装
Go语言官方下载：https://golang.google.cn/
Go语言中文网下载：https://studygolang.com/dl

## 配置环境变量
### 常用环境变量
#### • GOROOT
GOROOT是Go的安装目录，在Windows中，GOROOT的默认位置是C:/go，而在Mac OS或者Linux中GOROOT的默认位置是/usr/local/go，如果Go安装在其他目录，而需要将GOROOT的位置修改为对应的目录。

另外，GOROOT/bin下包含Go为我们提供的工具链，因此应该将GOROOT/bin配置到环境变量PATH中，方便我们在全局中使用Go的工具链。

例如 ： 现在Go的安装目录在D:\WindowsSoftware\Golang，需要在系统变量中添加GOROOT，值为D:\WindowsSoftware\Golang

#### • GOPATH
GOPATH是Go语言的工作目录。

go install/go get和 go的工具等会用到GOPATH环境变量。

GOPATH是作为编译后二进制的存放目的地和import包时的搜索路径。

GOPATH主要包含三个目录: bin、pkg、src

- bin：主要存放可执行文件。

- pkg：存放编译好的库文件, 主要是*.a文件。

- src：下主要存放go的源文件。

此外还需要注意的是不要将GOROOT设置成Go语言的路径，避免出现不必要的冲突。

GOPATH可以设置多个工作区，不过当我们使用go get命令去获取远程库的时候，一般会安装到第一个工作区当中。

每个工作区使用分号，分割即可，如：`export GOPATH=/opt/go;$home/go`

例如 ： 工作区在D:\goProject，需要在系统变量中添加GOPATH，值为D:\goProje

#### • GOBIN
GOBIN是我们在开发程序编译后二进制命令的安装目录。

当我们使用go install命令编译和打包应用程序时，该命令会将编译后的二进制程序打包GOBIN目录，一般我们将GOBIN设置为GOPATH/bin。

例如：在系统变量中找到path，添加值%GOROOT%\bin和%GOPATH%\bin

检测配置是否成功
打开CMD，输入`go env`

### go env windos 修改
示例：`go env -w  GOCACHE=D:\Users\Administrator\AppData\Local\go-build`

### 不常用环境变量
#### • GOOS与GOARCH
GOOS与GOARCH是当需要进行跨平台编译的时候，需要设置的环境变量，这种编译方式叫做交叉编译。

所谓的交叉编译，是指在一个平台上就生成可以在另外一个平台上运行的代码，例如我们可以在32位的Windows操作系统上开发，然后生成可以在64位的Linux操作系统上运行的二进制进程。

GOOS：它的默认值是我们当前的操作系统，例如Windows、Linux，但是需要注意的是Mac OS的对应值是darwin。

GOARCH：表示CPU的架构，如386，amd64，arm等。

可以通过go env来获取当前GOOS与GOARCH的值。
```shell
$ go env GOOS GOARCH
darwin
amd64
```

GOOS与GOARCH的取值范围。

GOOS与GOARCH的值成对出现，而且只能是下面列表对应的值。
```shell
GOOS        GOARCH
------------------
android     arm
darwin      386
darwin      amd64
darwin      arm
darwin      arm64
dragonfly   amd64
freebsd     386
freebsd     amd64
freebsd     arm
linux       386
linux       amd64
linux       arm
linux       arm64
linux       ppc64
linux       ppc64le
linux       mips
linux       mipsle
linux       mips64
linux       mips64le
linux       s390x
netbsd      386
netbsd      amd64
netbsd      arm
openbsd     386
openbsd     amd64
openbsd     arm
plan9       386
plan9       amd64
solaris     amd64
windows     386
windows     amd64
```

编译在64位Linux操作系统上运行的目标程序
```shell
$ CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build main.go
```

编译arm架构Android操作上的目标程序
```shell
$ CGO_ENABLED=0 GOOS=android GOARCH=arm GOARM=7 go build main.go
```

#### • 所有环境变量列表
虽然我们一般虽然配置的环境变量就那么几个，但其实Go语言是提供了非常多的环境变量，让我们可以自由地定制开发和编译器行为。

下面是Go提供的所有的环境变量列表，一般可以划分为下面几大类，大概了解一下就可以了，因为有些环境变量我们可以永远都不会用到。
```shell
GCCGO
GOARCH
GOBIN
GOCACHE
GOFLAGS
GOOS
GOPATH
GOPROXY
GORACE
GOROOT
GOTMPDIR
```

#### • 和cgo一起使用的环境变量
```shell
CC
CGO_ENABLED   // 禁用cgo
CGO_CFLAGS
CGO_CFLAGS_ALLOW
CGO_CFLAGS_DISALLOW
CGO_CPPFLAGS, CGO_CPPFLAGS_ALLOW, CGO_CPPFLAGS_DISALLOW
CGO_CXXFLAGS, CGO_CXXFLAGS_ALLOW, CGO_CXXFLAGS_DISALLOW
CGO_FFLAGS, CGO_FFLAGS_ALLOW, CGO_FFLAGS_DISALLOW
CGO_LDFLAGS, CGO_LDFLAGS_ALLOW, CGO_LDFLAGS_DISALLOW
CXX
PKG_CONFIG
AR
```

#### • 与系统架构体系相关的环境变量
```shell
GOARM
GO386
GOMIPS
GOMIPS64
```

#### • 专用的环境变量
```shell
GCCGOTOOLDIR
GOROOT_FINAL
GO_EXTLINK_ENABLED
GIT_ALLOW_PROTOCOL
```

#### • 所有环境变量列表
```shell
GOEXE
GOHOSTARCH
GOHOSTOS
GOMOD
GOTOOLDIR
```

## 目录结构
#### • 目前流行的项目结构
```shell
├── bin  # 存放编译后的二进制文件
├── pkg  # 存放编译后的库文件
└── src  # 存放源码文件
    └── jasonceng.com  # 使用网站域名区分项目
        └── zengzc      # 作者/部门/机构...
            └── demo1     # 项目
                └── main.go
```

## 推荐编辑器
Go采用的是UTF-8编码的文本文件存放代码的，理论上使用任何一款文件编辑器都可以做Go语言开发，这里主要推荐两个开发工具。
#### • VS Code
Visual Studio Code（简称VS Code）是一个由微软开发的，同时支持Windows、Linux、和macOS系统且开放源代码的代码编辑器，它支持测试，并内置了Git 版本控制功能，同时也具有开发环境功能，例如代码补全（类似于 IntelliSense）、代码片段、和代码重构等，该编辑器支持用户个性化配置，例如改变主题颜色、键盘快捷方式等各种属性和参数，还在编辑器中内置了扩展程序管理的功能。

虽然不如一些IDE功能强大，但是它添加Go扩展插件后已经足够胜任我们日常的Go开发工作了，而且它占用资源较少，所以就算配置较低的电脑也可以使用。

#### • Goland
喜欢用IDE做开发的同学必定不能错过Jetbrains家族的IDE，款款精品，可谓都是IDE中的神兵利器。

下面介绍的就是Jetbrains家族中开发Go语言的Goland。

Goland是付费的，当然网上也会有一些激活码可以直接激活使用，但是个人认为，如果条件允许的话，希望购买正版，若是不愿意付费的话，建议使用VS Code也可满足正常开发需求。

## 编写第一个Go程序
#### • hello.go
```go
go
package main
import (
    "fmt"
)
func main() {
    fmt.Println("Hello Go!")
}
```

#### • 编译
```shell
$ go build hello.go
```
go build命令可以将Go语言程序代码编译成二进制的可执行文件，但是需要我们手动运行该二进制文件.
```shell
$ go run hello.go
```
go run命令则更加方便，它会在编译后直接运行Go语言程序，编译过程中会产生一个临时文件，但不会生成可执行文件，这个特点很适合用来调试程序。

#### • 运行
```shell
$ ./hello
```