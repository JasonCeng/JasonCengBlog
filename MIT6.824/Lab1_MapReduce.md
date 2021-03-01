# Lab1 MapReduce

## 简介
在本实验中，您将构建一个MapReduce系统。 您将实现一个工作程序进程，该进程调用应用程序Map和Reduce函数并处理读写文件，以及一个主进程，该进程将任务分发给工作程序并应对失败的工作程序。 您将构建类似于MapReduce论文的内容。

## 合作策略
您必须编写您在6.824中提交的所有代码，但我们作为分配的一部分提供给您的代码除外。 不允许您查看其他任何人的解决方案，也不允许您查看前几年的解决方案。 您可以与其他学生讨论作业，但不能查看或复制彼此的代码。 制定此规则的原因是，我们相信您将自己设计和实施实验室解决方案，从而从中学到最多的知识。

请不要发布您的代码或将其提供给现在或将来的6.824学生。 github.com仓库默认是公开的，因此除非您将仓库设为私有，否则请不要在其中放置代码。 您可能会发现使用MIT的GitHub很方便，但是请确保创建一个私有存储库。

## 软件
您将在Go中实现此实验室（及所有实验室）。 Go网站包含许多教程信息。 我们将使用Go 1.13版为您的实验室评分； 您也应该使用1.13。 您可以通过运行go版本来检查您的Go版本。

我们建议您在自己的计算机上进行实验，以便可以使用已经熟悉的工具，文本编辑器等。 或者，您可以在Athena上的实验室工作。

### Linux
根据您的Linux发行版本，您也许可以从软件包存储库中获取最新版本的Go，例如 通过运行apt install golang。 否则，您可以从Go的网站手动安装二进制文件。 首先，确保您正在运行64位内核（uname -a应提及“ x86_64 GNU / Linux”），然后运行：
```shell
$ wget -qO- https://dl.google.com/go/go1.13.6.linux-amd64.tar.gz | sudo tar xz -C /usr/local
```
您需要确保`/usr/local/go/bin`在您的`PATH`上。

## 开始
您将使用git（版本控制系统）获取初始实验室软件。 要了解有关git的更多信息，请参阅[Pro Git书](https://git-scm.com/book/en/v2)或[git用户手册](https://mirrors.edge.kernel.org/pub/software/scm/git/docs/user-manual.html)。 要获取6.824实验软件：
```shell
$ git clone git://g.csail.mit.edu/6.824-golabs-2020 6.824
$ cd 6.824
$ ls
Makefile src
$
```
我们在src/main/mrsequential.go中为您提供了一个简单的顺序mapreduce实现。 它可以在一个过程中运行map和reduce。 我们还为您提供了两个MapReduce应用程序：mrapps/wc.go中的单词计数和mrapps/indexer.go中的文本索引器。 您可以按如下顺序运行字数统计：
```shell
$ cd ~/6.824
$ cd src/main
$ go build -buildmode=plugin ../mrapps/wc.go
$ rm mr-out*
$ go run mrsequential.go wc.so pg*.txt
$ more mr-out-0
A 509
ABOUT 2
ACT 8
...
```
mrsequential.go将其输出保留在文件mr-out-0中。 输入来自名为pg-xxx.txt的文本文件。

随时从mrsequential.go借用代码。 您还应该在mrapps/wc.go上查看一下MapReduce应用程序代码的样子。

## 你的工作
您的工作是实现一个分布式MapReduce，它由两个程序（主程序和工作程序）组成。 只有一个主进程，一个或多个工作进程并行执行。 在真实的系统中，工作人员将在一堆不同的机器上运行，但是对于本实验，您将全部在单个机器上运行。 工人将通过RPC与主服务器对话。 每个工作进程都会向主服务器索要任务，从一个或多个文件读取任务的输入，执行任务，并将任务的输出写入一个或多个文件。 主进程应注意一个工作进程是否在合理的时间内没有完成其任务（在本实验中，使用十秒钟），并将同一任务交给另一个工作进程。

我们给了您一些代码，让您开始。 主机和工作程序的“主”例程位于main/mrmaster.go和main/worker.go中； 不要更改这些文件。 您应该将实现放在mr/master.go，mr/worker.go和mr/rpc.go中。

这是在单词计数MapReduce应用程序上运行代码的方法。 首先，确保单词计数插件是全新构建的：
```shell
$ go build -buildmode=plugin ../mrapps/wc.go
```

在master中，运行主程序。
```shell
$ rm mr-out*
$ go run mrmaster.go pg-*.txt
```
mrmaster.go的pg-*。txt参数是输入文件； 每个文件对应一个“拆分”，是一个Map任务的输入。

在一个或多个其他窗口中，运行一些工作程序：
```shell
$ go run mrworker.go wc.so
```

当工作人员和主管完成后，请查看mr-out- *中的输出。 完成实验后，输出文件的排序后的并集应与顺序输出匹配，如下所示：
```shell
$ cat mr-out-* | sort | more
A 509
ABOUT 2
ACT 8
...
```