# PROMETHEUS

http://prometheus.io

Prometheus 是一个很受欢迎的开源监控和警报工具包，它最初是在 SoundCloud 进行构建的。现在是 CNCF 项目，也是该公司在 Kubernetes 之后的第二个托管项目。

作为一个工具包，它与目前为止所描述的其他监视解决方案有很大不同。首先一个主要的区别是，Prometheus 作为一种云服务，是模块化的，可以自行托管，这意味着无论是在本地还是在云端，用户都可以在他们的集群上部署 Prometheus。

值得注意的是，Prometheus 不是将数据推送到云服务，而是安装在每个 Docker 主机上，并通过 HTTP 从 Prometheus 提供的各种输出口获取或“抓取”数据。

其中，一些输出口被官方保留为 Prometheus GitHub 项目的一部分，而另一些则是由外部贡献。有些项目本身暴露了 Prometheus 数据，因此不需要输出口。由于 Prometheus 可高度扩展，用户需要考虑输出方的数量，并根据收集的数据量适当地配置轮询间隔。

Prometheus 的服务器从各种来源检索时间序列数据，并将数据存储在其内部数据存储区中。此外，Prometheus 提供服务发现等功能，这是一种针对特定类型数据的独立推送网关，并且有一个嵌入的查询语言 ( PromQL )，该语言擅长查询多维数据。

同时，它也有一个嵌入式的 Web UI 和 API 。虽然 Prometheus 中的 Web UI 提供了强大的功能，但用户必须对 PromQL 十分了解，因此一些站点更愿意使用 Grafana 作为绘制和查看集群相关指标的接口。

Prometheus 有一个独立的告警管理器，它也具有独特的 UI，并且可以处理存储在 Prometheus 中的数据。和其他告警管理器一样，它可以与各种外部告警服务一起工作，包括电子邮件、Hipchat、Pagerduty、#Slack、OpsGenie、VictorOps 等。

因为 Prometheus 由许多组件组成，输出方需要根据所监控的服务进行选择和安装，所以安装起来比较困难，但是作为免费产品，在价格上 Prometheus 具有无可比拟的优势。

虽然不像 Datadog 或 Sysdig 这样精炼，但是 Prometheus 提供了类似的功能、广泛的第三方软件集成以及一流的云监控解决方案，并且 Prometheus 十分了解 Kubernetes 和其他容器管理架构。

另外，由 Infinityworks 开发的 Rancher Catalog 中的条目使得在使用 Cattle 作为 Rancher 的编排架构时，Prometheus 更容易入门，但由于配置选项的种类繁多，管理员需要花费一些时间才能正确安装和配置。

Infinityworks 提供了一些有用的插件，其中包 prometheus-rancher-exporter，这些插件将 Rancher stack 和从 Rancher API 获得的主机的健康状况发送给 Prometheus 兼容端点。因此，Prometheus 对于那些愿意花更多精力的管理者来说是最强大的监控解决方案之一。