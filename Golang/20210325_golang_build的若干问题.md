# golang_build的若干问题

## Q1: `/bin/sh: xxx not found`
问题描述：在Linux上`go build main.go`后，将编译后的可执行文件放到docker中运行出现`/bin/sh: main not found`

解决办法：用Linux上的编译命令:` CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build <go_filename>`
