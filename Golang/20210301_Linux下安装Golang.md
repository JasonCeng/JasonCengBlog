# Linux下安装Golang

```shell
$ cd /data
$ wget https://golang.google.cn/dl/go1.16.linux-amd64.tar.gz
$ tar -zxvf go1.16.linux-amd64.tar.gz -C /usr/local/

$ mkdir $HOME/workspace
$ vim /etc/profile

export GOPATH=$HOME/workspace
export PATH=$PATH:/usr/local/go/bin

$ source /etc/profile
```