# MapReduce：大规模集群上的简化数据处理

## 摘要
MapReduce是用于处理和生成大型数据集的编程模型以及相关的实现。 用户指定一个处理键/值对以生成一组中间键/值对的map函数，以及一个reduce函数，该reduce函数合并与同一中间键关联的所有中间值。 如该论文所示，许多现实世界中的任务在这种模型中都是可以表达的。

以这种功能风格编写的程序会自动并行化，并在大型商用机器集群上执行。运行时系统负责划分输入数据，安排程序在一组计算机上的执行，处理机器故障以及管理所需的机器间通信的细节。这使没有并行和分布式系统经验的程序员可以轻松利用大型分布式系统的资源。

我们对MapReduce的实现可在大型商用机器集群上运行，并且具有高度可扩展性：典型的MapReduce计算可在数千台机器上处理数TB的数据。 程序员发现该系统易于使用：每天执行数百个MapReduce程序，每天在Google集群上执行多达一千个MapReduce作业。

## 1 简介
在过去的五年中，Google的作者和许多其他人已经实现了数百种特殊用途的计算，这些计算处理大量的原始数据（例如抓取的文档，Web请求日志等），以计算各种派生数据，例如 作为反向索引，Web文档的图形结构的各种表示形式，每个主机抓取的页面数的摘要，给定日期中最频繁查询的集合等。大多数此类计算从概念上讲都是简单明了的。 但是，输入数据通常很大，并且必须在数百或数千台机器上分布计算，以便在合理的时间内完成操作。 如何并行化计算，分配数据以及处理故障的问题共同困扰着用大量复杂代码来掩盖最初的简单计算，以应对这些问题。

为了应对这种复杂性，我们设计了一个新的抽象，该抽象使我们能够表达我们试图执行的简单计算，但在库中隐藏了并行化，容错，数据分发和负载平衡的混乱细节。我们的抽象受到map和reduce的启发，并减少了Lisp和许多其他函数式语言中存在的原语。 我们意识到，大多数计算都涉及对输入中的每个逻辑“记录”应用map操作，以便计算一组中间键/值对，然后对所有共享同一键的值应用reduce操作，为了适当地组合得出的数据。我们使用具有用户指定的map和reduce运算的功能模型，使我们能够轻松地并行进行大型计算，并使用重新执行作为容错的主要机制。

这项工作的主要贡献是一个简单而强大的接口，它能够自动并行化和分配大规模计算，并结合了该接口的实现，从而在大型商用PC集群上实现了高性能。

第2节描述了基本的编程模型，并给出了几个示例。 第3节介绍了针对我们基于集群的计算环境量身定制的MapReduce接口的实现。第4节描述了一些有用的编程模型改进。 第5节对我们执行各种任务的性能进行了度量。第6部分探讨了MapReduce在Google中的用法，包括我们使用它作为重写生产索引系统基础的经验。 第7节讨论相关和未来的工作。

## 2 编程模型
计算采用一个输入键/值对集合，并产生一个键/值对集合输出。MapReduce库的用户把计算表示为两个函数：Map和Reduce。

由用户编写的Map接受一个输入键/值对，并生成一组中间键/值对。MapReduce库将与同一中间键I关联的所有中间值分组在一起，并将它们传递给Reduce函数。

还由用户编写的Reduce函数接受中间键I和该键的一组值。 它将这些值合并在一起，以形成可能较小的一组值。 通常，每个Reduce调用仅产生零个或一个输出值。 中间值通过迭代器提供给用户的reduce函数。 这使我们能够处理太大而无法容纳在内存中的值列表。





https://pdos.csail.mit.edu/6.824/papers/mapreduce.pdf