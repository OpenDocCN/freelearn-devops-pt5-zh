# 管理资源

如果没有指明容器需要多少 CPU 和内存，Kubernetes 只能将所有容器视为平等。这往往导致资源使用分配极不均衡。如果没有资源规格就请求 Kubernetes 调度容器，就像进入一辆由盲人驾驶的出租车一样。

我们已经走了很长一段路，从最初的起点开始，逐步了解了许多 Kubernetes 的基本对象类型和原则。现在我们最缺少的就是资源管理。到目前为止，Kubernetes 一直在盲目地调度我们部署的应用程序。我们从未告诉它这些应用程序需要多少资源，也没有设定任何限制。没有这些，Kubernetes 执行任务的方式就显得非常短视。Kubernetes 看到的很多，但远远不够。我们很快就会改变这一点，给 Kubernetes 配上一副眼镜，让它有更清晰的视野。

一旦我们学会了如何定义资源，我们将进一步确保设定一些限制，确定默认值，并设置配额，以防止应用程序超载集群。

本章是拼图的最后一块。解决了这一部分，我们就可以开始考虑在生产环境中使用 Kubernetes 了。你可能还不能掌握操作 Kubernetes 所需的所有知识，实际上没有人能做到这一点。但你会知道足够多的内容，能够把你引导到正确的方向。

# 创建集群

我们将进行几乎与前几章相同的操作。我们会进入我们克隆的 `vfarcic/k8s-specs` 仓库所在的目录，拉取最新代码，启动 Minikube 集群，等等。这次唯一不同的地方是我们将启用一个额外的插件——Heapster。现在解释它的功能和为什么需要它还为时过早，稍后会详细说明。暂时只需记住，你的集群里很快会有一个叫 Heapster 的东西。如果你还不知道它是什么，认为这是一个悬念引导，稍后会有更多解释。

本章中的所有命令都可以在 [`13-resource.sh`](https://gist.github.com/cc8c44e1e84446dccde3d377c131a5cd) ([`gist.github.com/vfarcic/cc8c44e1e84446dccde3d377c131a5cd`](https://gist.github.com/vfarcic/cc8c44e1e84446dccde3d377c131a5cd)) Gist 中找到。

```
cd k8s-specs

git pull

minikube start --vm-driver=virtualbox

kubectl config current-context

minikube addons enable ingress

minikube addons enable heapster  
```

现在已经拉取了最新的代码，集群也在运行，插件已启用，我们可以继续并探索如何定义容器的内存和 CPU 资源。

# 定义容器的内存和 CPU 资源

到目前为止，我们还没有指定容器应该使用多少内存和 CPU，也没有设定它们的限制。如果我们这样做，Kubernetes 的调度器将能更好地了解这些容器的需求，从而做出更好的决策，决定将 Pods 放置在哪些节点上，以及当容器“行为异常”时应该如何处理。

让我们来看一下修改后的 `go-demo-2` 定义：

```
cat res/go-demo-2-random.yml  
```

规范几乎与我们之前使用的相同。唯一新增的条目是在`resources`部分。

输出，仅限于相关部分，如下所示：

```
... 
apiVersion: apps/v1beta2 
kind: Deployment 
metadata: 
  name: go-demo-2-db 
spec: 
  ... 
  template: 
    ... 
    spec: 
      containers: 
      - name: db 
        image: mongo:3.3 
        resources: 
          limits: 
            memory: 200Mi 
            cpu: 0.5 
          requests: 
            memory: 100Mi 
            cpu: 0.3 
... 
apiVersion: apps/v1beta2 
kind: Deployment 
metadata: 
  name: go-demo-2-api 
spec: 
  ... 
  template: 
    ... 
    spec: 
      containers: 
      - name: api 
        image: vfarcic/go-demo-2 
        ... 
        resources: 
          limits: 
            memory: 100Mi 
            cpu: 200m 
          requests: 
            memory: 50Mi 
            cpu: 100m 
... 
```

我们在`resources`部分指定了`limits`和`requests`条目。

CPU 资源以`cpu`单位进行度量。`cpu`单位的确切含义取决于我们托管集群的位置。如果服务器是虚拟化的，那么一个`cpu`单位相当于一个**虚拟化处理器**（**vCPU**）。当在裸机上运行且启用了超线程时，一个`cpu`等于一个超线程。为了简化，我们假设一个`cpu`资源等于一个 CPU 处理器（尽管这并不完全准确）。

如果一个容器设置为使用`两个`CPU，而另一个设置为使用`一个`CPU，则后者的处理能力将仅为前者的一半。

CPU 值可以被分割。在我们的示例中，`db`容器的 CPU 请求设置为`0.5`，这相当于半个 CPU。这个值也可以表示为`500m`，即五百毫 CPU。如果你再看看`api`容器的 CPU 规格，你会看到它的 CPU 限制设置为`400m`，请求为`200m`，它们分别等于`0.4`和`0.2`个 CPU。

内存资源的模式与 CPU 类似。主要区别在于单位。内存可以表示为**K**（**千字节**）、**M**（**兆字节**）、**G**（**千兆字节**）、**T**（**太字节**）、**P**（**拍字节**）和**E**（**艾字节**）。我们也可以使用二的幂次方单位`Ki`、`Mi`、`Gi`、`Ti`、`Pi`和`Ei`。

如果我们回到`go-demo-2-random.yml`定义文件，我们会看到`db`容器的限制设置为**200Mi**（**两百兆字节**），请求设置为**100Mi**（**一百兆字节**）。

我们已经提到过`limits`和`requests`好几次，但仍然没有解释它们各自的含义。

限制表示容器不应超过的资源量。假设我们将限制定义为上限，当达到该上限时，表示出现了问题，并且是一种防止单个恶性容器因内存泄漏或类似问题而占用过多资源的方式。

如果容器是可重启的，Kubernetes 会重启超出内存限制的容器。否则，它可能会终止该容器。请记住，如果容器属于一个 Pod（所有 Kubernetes 管理的容器都是如此），即使它被终止，也会被重新创建。

与内存不同，CPU 限制永远不会导致容器终止或重启。相反，容器将不会允许在较长时间内消耗超过 CPU 限制的资源。

请求代表预期的资源利用率。Kubernetes 使用它们来决定如何根据节点的实际资源使用情况，将 Pods 放置在集群中。

如果容器超出了其内存请求，且节点的内存不足，Pod 可能会被驱逐。这种驱逐通常会导致 Pod 被调度到另一节点，只要该节点有足够的可用内存。如果由于资源不足，Pod 无法调度到任何节点，它将进入待处理状态，直到某个节点的资源被释放，或者集群中新增一个节点。

如果仅仅讨论 `资源` 的理论，而没有实际的例子，可能会让人感到困惑。因此，我们将继续并创建在 `go-demo-2-random.yml` 文件中定义的资源：

```
kubectl create \
 -f res/go-demo-2-random.yml \
 --record --save-config

kubectl rollout status \
 deployment go-demo-2-api  
```

我们创建了资源并等待 `go-demo-2-api` Deployment 部署完成。后续命令的输出应该如下所示：

```
deployment "go-demo-2-api" successfully rolled out  
```

让我们描述 `go-demo-2-api` Deployment，并查看其 `limits` 和 `requests`：

```
kubectl describe deploy go-demo-2-api  
```

限制为 `limits` 和 `requests` 的输出如下：

```
... 
Pod Template: 
  ... 
  Containers: 
    ... 
   Limits: 
      cpu:    200m 
      memory: 100Mi 
    Requests: 
      cpu:    100m 
      memory: 50Mi 
... 
```

我们可以看到，`limits` 和 `requests` 对应于我们在 `go-demo-2-random.yml` 文件中定义的内容。这个结果并不令人惊讶。

让我们描述形成集群的节点（尽管这里只有一个）。

```
kubectl describe nodes  
```

限制为资源相关条目的输出如下：

```
... 
Capacity: 
 cpu:     2 
 memory:  2048052Ki 
 pods:    110 
... 
Non-terminated Pods: (12 in total) 
  Namespace          Name                         CPU Requests CPU Limits Memory Requests Memory Limits 
  ---------          ----                         ------------ ---------- --------------- ------------- 
  default            go-demo-2-api-...            100m (5%)    200m (10%) 50Mi (2%)       100Mi (5%) 
  default            go-demo-2-api-...            100m (5%)    200m (10%) 50Mi (2%)       100Mi (5%) 
  default            go-demo-2-api-...            100m (5%)    200m (10%) 50Mi (2%)       100Mi (5%) 
  default            go-demo-2-db-...             300m (15%)   500m (25%) 100Mi (5%)      200Mi (10%) 
  kube-system        default-http-...             10m (0%)     10m (0%)   20Mi (1%)       20Mi (1%) 
  kube-system        heapster-...                 0 (0%)       0 (0%)     0 (0%)          0 (0%) 
  kube-system        influxdb-grafana-...         0 (0%)       0 (0%)     0 (0%)          0 (0%) 
  kube-system        kube-addon-manager-minikube  5m (0%)      0 (0%)     50Mi (2%)       0 (0%) 
  kube-system        kube-dns-54cccfbdf8-...      260m (13%)   0 (0%)     110Mi (5%)      170Mi (8%) 
  kube-system        kubernetes-dashboard-...     0 (0%)       0 (0%)     0 (0%)          0 (0%) 
  kube-system        nginx-ingress-controller-... 0 (0%)       0 (0%)     0 (0%)          0 (0%) 
  kube-system        storage-provisioner          0 (0%)       0 (0%)     0 (0%)          0 (0%) 
Allocated resources: 
  (Total limits may be over 100 percent, i.e., overcommitted.) 
  CPU Requests CPU Limits  Memory Requests Memory Limits 
  ------------ ----------  --------------- ------------- 
  875m (43%)   1110m (55%) 430Mi (22%)     690Mi (36%) 
... 
```

`容量` 表示节点的总体容量。在我们的例子中，`minikube` 节点有 2 个 CPU、2GB 的内存，最多可运行 110 个 Pods。这些是硬件或在我们案例中，由 Minikube 创建的虚拟机大小所强加的上限。

接下来是 `未终止的 Pods` 部分。它列出了所有具有 CPU 和内存限制及请求的 Pods。例如，我们可以看到 `go-demo-2-db` Pod 的内存限制设置为 `100Mi`，即占总容量的 `5%`。类似地，我们可以看到并非所有 Pods 都指定了资源。例如，`heapster-snq2f` Pod 的所有值都设置为 `0`。Kubernetes 无法适当处理这些 Pods。然而，由于这是一个演示集群，我们可以宽容对待，忽略资源未指定的问题。

最后，`已分配资源` 部分提供了所有 Pods 的汇总值。例如，我们可以看到 CPU 限制为 `55%`。限制值甚至可以超过 `100%`，这并不一定是需要担心的事情。并非所有容器都会超过请求的内存和 CPU 值。即使发生这种情况，Kubernetes 也会知道如何处理。

真正重要的是，请求的内存和 CPU 总量在容量的限制范围内。然而，这也引出了一个有趣的问题。我们到目前为止定义资源的依据是什么？

# 测量实际的内存和 CPU 消耗

我们是如何得出当前内存和 CPU 值的？为什么将 MongoDB 的内存设置为`100Mi`？为什么不是`50Mi`或`1Gi`？现在承认我们有的值是随机的是令人尴尬的。我猜测基于`vfarcic/go-demo-2`镜像的容器需要比 Mongo 数据库更少的资源，因此它们的值相对较小。这是我定义资源的唯一标准。

在您对我为资源放置随机值感到不满之前，您应该知道我们没有任何支持我们的指标。任何人的猜测与我的一样好。

了解一个应用程序使用多少内存和 CPU 的唯一方法是检索指标。我们将使用 Heapster ([`github.com/kubernetes/heapster`](https://github.com/kubernetes/heapster)) 来实现这一目的。

Heapster 收集和解释各种信号，如计算资源使用情况，生命周期事件等。在我们的情况下，我们只关注我们集群中正在运行的容器的 CPU 和内存消耗。

在创建集群时，我们启用了`heapster`插件，并且 Minikube 将其部署为系统应用程序。不仅如此，它还部署了 InfluxDB ([`github.com/influxdata/influxdb`](https://github.com/influxdata/influxdb)) 和 Grafana ([`grafana.com/`](https://grafana.com/))。前者是 Heapster 存储数据的数据库，后者可以通过仪表板可视化数据。

您可能倾向于认为 Heapster、InfluxDB 和 Grafana 可能是您监控需求的解决方案。我建议不要做出这样的决定。我们仅使用 Heapster，因为它作为 Minikube 插件 readily 可用。将 Heapster 开发为监控工具的想法在很大程度上已被放弃。它的主要重点是作为一些 Kubernetes 特性所需的内部工具。相反，我建议结合使用 Prometheus ([`prometheus.io/`](https://prometheus.io/)) 和 Kubernetes API 作为指标的来源，以及 Alertmanager ([`prometheus.io/docs/alerting/alertmanager/`](https://prometheus.io/docs/alerting/alertmanager/)) 作为您的警报需求。然而，这些工具不在本章的范围内，所以您可能需要从它们的文档中学习，或者等待本书的续集出版（暂定名为*高级 Kubernetes*）。

仅使用 Heapster 作为快速检索指标的一种方法。探索 Prometheus 和 Alertmanager 的组合，以满足您的监控和警报需求。

现在我们澄清了 Heapster 的作用及其非作用，我们可以继续确认它确实在我们的集群内运行。

```
kubectl --namespace kube-system \
    get pods 
```

输出如下：

```
NAME                         READY STATUS  RESTARTS AGE
default-http-backend-...     1/1   Running 0        59m
heapster-...                 1/1   Running 0        59m
influxdb-grafana-...         2/2   Running 0        59m
kube-addon-manager-minikube  1/1   Running 0        59m
kube-dns-54cccfbdf8-...      3/3   Running 0        59m
kubernetes-dashboard-...     1/1   Running 0        59m
nginx-ingress-controller-... 1/1   Running 0        59m
storage-provisioner          1/1   Running 0        59m  
```

正如你所见，`heapster` 和 `influxdb-grafana` Pods 正在运行。

我们将仅探索足够多的 Heapster 来检索我们需要的数据。为此，我们需要访问其 API。然而，Minikube 没有暴露其端口，这将是我们要做的第一件事：

```
kubectl --namespace kube-system \ 
    expose rc heapster \ 
    --name heapster-api \ 
    --port 8082 \ 
    --type NodePort 
```

我们需要找出为我们创建了哪个`NodePort`。为此，我们需要熟悉服务的 JSON 定义：

```
kubectl --namespace kube-system \ 
    get svc heapster-api \  
    -o json 
```

我们正在寻找`spec.ports`数组中的`nodePort`条目。检索并将其输出分配给`PORT`变量的命令如下：

```
PORT=$(kubectl --namespace kube-system \ 
    get svc heapster-api \ 
    -o jsonpath="{.spec.ports[0].nodePort}") 
```

我们使用`jsonpath`输出仅检索`spec.ports`数组中的第一个（也是唯一的）条目的`nodePort`。

让我们尝试一次非常简单的 Heapster API 查询。

```
BASE_URL="http://$(minikube ip):$PORT/api/v1/model/namespaces/default/pods" 

curl "$BASE_URL" 
```

`curl`请求的输出如下：

```
[ 
  "go-demo-2-api-796db5987d-dm69g", 
  "go-demo-2-db-bf6f5b486-p9vhj", 
  "go-demo-2-api-796db5987d-5t84b", 
  "go-demo-2-api-796db5987d-99nh6" 
 ] 
```

我们实际上不需要 Heapster 来检索 Pod 列表。我们真正需要的是其中一个 Pod 的指标。为此，我们需要它的名称。

我们将使用一个类似的命令来检索 Heapster 的服务端口。

```
DB_POD_NAME=$(kubectl get pods \ 
    -l service=go-demo-2 \ 
    -l type=db \ 
    -o jsonpath="{.items[0].metadata.name}") 
```

我们检索了所有标签为`service=go-demo-2`和`type=db`的 Pod，并将输出格式化，以便从第一个项中检索`metadata.name`。该值被存储为`DB_POD_NAME`变量。

现在我们可以查看 Pod 中`db`容器的可用指标。

```
curl "$BASE_URL/$DB_POD_NAME/containers/db/metrics"  
```

输出如下：

```
[
 "memory/rss",
 "cpu/usage_rate",
 "cpu/request",
 "memory/usage",
 "memory/major_page_faults_rate",
 "cpu/limit",
 "memory/page_faults",
 "memory/major_page_faults",
 "uptime",
 "memory/limit",
 "cpu/usage",
 "memory/page_faults_rate",
 "memory/working_set",
 "restart_count",
 "memory/request"
]

```

如你所见，大多数可用的指标与内存和 CPU 相关。

让我们看看内存使用是否确实与我们为`go-demo-2-db`部署定义的内存资源相匹配。提醒一下，我们将内存请求设置为`100Mi`，内存限制设置为`200Mi`。

检索`db`容器内存使用情况的请求如下：

```
curl "$BASE_URL/$DB_POD_NAME/containers/db/metrics/memory/usage"  
```

输出仅限于几个条目，如下所示：

```
{ 
  "metrics": [ 
   ... 
   { 
    "timestamp": "2018-02-01T20:24:00Z", 
    "value": 38334464 
   }, 
   { 
    "timestamp": "2018-02-01T20:25:00Z", 
    "value": 38342656 
   } 
  ], 
  "latestTimestamp": "2018-02-01T20:25:00Z" 
 } 
```

我们可以看到内存使用量大约是 38MB。与我们设置的`100Mi`相比，这是一个相当大的差距。当然，这项服务并没有承受真实的生产负载，但由于我们正在模拟一个“真实”的集群，我们假装`38Mi`确实是在“真实”条件下的内存使用情况。这意味着我们通过分配一个几乎是实际使用量三倍的值，过高地估计了请求。

那 CPU 怎么样呢？我们在这方面也犯了如此巨大的错误吗？提醒一下，我们将 CPU 请求设置为`0.3`，限制为`0.5`。

```
curl "$BASE_URL/$DB_POD_NAME/containers/db/metrics/cpu/usage_rate"  
```

输出仅限于几个条目，如下所示。

```
{ 
  "metrics": [ 
   ... 
   { 
    "timestamp": "2018-02-01T20:25:00Z", 
    "value": 5 
   }, 
   { 
    "timestamp": "2018-02-01T20:26:00Z", 
    "value": 4 
   } 
  ], 
  "latestTimestamp": "2018-02-01T20:26:00Z" 
 } 
```

如我们所见，CPU 使用量大约为`5m`或`0.005` CPU。我们再次在资源规范上犯了一个巨大的错误。我们的值大约高出六十倍。

我们期望（资源请求和限制）与实际使用之间的这种偏差可能会导致调度极度不平衡，产生不良后果。我们将很快修正资源配置。现在，我们将探讨一下如果资源量低于实际使用情况会发生什么。

# 探索资源规范与资源使用之间差异的影响

让我们看看稍微修改过的`go-demo-2`定义：

```
cat res/go-demo-2-insuf-mem.yml  
```

与之前的定义相比，差异仅在于`go-demo-2-db`部署中`db`容器的`resources`。

输出仅限于相关部分，如下所示：

```
apiVersion: apps/v1beta2 
kind: Deployment 
metadata: 
  name: go-demo-2-db 
spec: 
  ... 
  template: 
    ... 
    spec: 
      containers: 
      - name: db 
        image: mongo:3.3 
        resources: 
          limits: 
            memory: 10Mi 
            cpu: 0.5 
          requests: 
            memory: 5Mi 
            cpu: 0.3 
```

内存限制设置为`10Mi`，请求为`5Mi`。由于我们已经从 Heapster 的数据中知道 MongoDB 大约需要`38Mi`，这次内存资源远低于实际使用量。

让我们看看当应用新配置时会发生什么：

```
kubectl apply \  
    -f res/go-demo-2-insuf-mem.yml \
    --record 

kubectl get pods 
```

我们应用了新配置并检索了 Pods。输出如下：

```
    NAME              READY STATUS    RESTARTS AGE
    go-demo-2-api-... 1/1   Running   0        1m
    go-demo-2-api-... 1/1   Running   0        1m
    go-demo-2-api-... 1/1   Running   0        1m
    go-demo-2-db-...  0/1   OOMKilled 2        17s

```

在你的情况下，状态可能不是`OOMKilled`。如果是这样，请再等待一段时间并重新检索 Pods。状态最终应更改为`CrashLoopBackOff`。

如您所见，`go-demo-2-db` Pod 的状态是`OOMKilled`（**内存溢出终止**）。Kubernetes 检测到实际使用量远超限制，且将 Pod 标记为终止候选者。容器随后被终止。Kubernetes 稍后会重新创建已终止的容器，但很快会发现内存使用仍然超出限制。如此反复循环，直到继续下去。

如果节点有足够的可用内存，容器可以超过其内存请求。另一方面，容器不能使用超过限制的内存。当发生这种情况时，它将成为终止的候选者。

让我们描述一下部署并查看`db`容器的状态：

```
kubectl describe pod go-demo-2-db  
```

限制输出的相关部分如下：

```
...
Containers:
 db:
 ...
 Last State:     Terminated
 Reason:       OOMKilled
 Exit Code:    137
 ...
Events:
 Type    Reason  Age             From              Message
 ----    ------  ----            ----              -------
 ...
 Warning BackOff 3s (x8 over 1m) kubelet, minikube Back-off restarting failed container

```

我们可以看到`db`容器的最后状态是`OOMKilled`。当我们查看事件时，可以看到到目前为止，该容器因`BackOff`原因已重启了八次。

让我们通过另一个更新的定义来探索另一种可能的情况：

```
cat res/go-demo-2-insuf-node.yml  
```

如之前所述，更改仅影响`go-demo-2-db`部署中的`resources`部分。相关部分的输出如下：

```
apiVersion: apps/v1beta2 
kind: Deployment 
metadata: 
  name: go-demo-2-db 
spec: 
  ... 
  template: 
    ... 
    spec: 
      containers: 
      - name: db 
        image: mongo:3.3 
        resources: 
          limits: 
            memory: 8Gi 
            cpu: 0.5 
          requests: 
            memory: 4Gi 
            cpu: 0.3 
```

这次，我们指定请求的内存是节点总内存（2GB）的两倍。内存限制甚至更高。

让我们应用这个更改并观察会发生什么：

```
kubectl apply \  
    -f res/go-demo-2-insuf-node.yml \ 
    --record 

kubectl get pods 
```

后续命令的输出如下：

```
NAME                           READY STATUS  RESTARTS AGE
go-demo-2-api-796db5987d-8wbk4 1/1   Running 0        8m
go-demo-2-api-796db5987d-w6mnx 1/1   Running 0        8m
go-demo-2-api-796db5987d-wtz4q 1/1   Running 0        9m
go-demo-2-db-5d5c46bc7c-d676j  0/1   Pending 0        13s  
```

这次，Pod 的状态是`Pending`。Kubernetes 无法将其放置在集群中的任何地方，并正在等待直到情况有所变化。

尽管内存请求与容器相关，但通常更有意义的是将它们转化为 Pod 的需求。我们可以说，Pod 的请求内存是构成该 Pod 的所有容器请求的总和。在我们的案例中，Pod 只有一个容器，因此它们的请求是相等的。限制也是如此。

在调度过程中，Kubernetes 会将一个 Pod 的请求进行求和，并寻找一个有足够可用内存和 CPU 的节点。如果 Pod 的请求无法满足，Pod 将进入待处理状态，期待某个节点释放资源，或者集群中加入新服务器。由于在我们这种情况下不会发生这样的事情，通过`go-demo-2-db`部署创建的 Pod 将永远处于待处理状态，除非我们再次更改内存请求。

当 Kubernetes 无法找到足够的空闲资源来满足构成 Pod 的所有容器的资源请求时，它会将状态更改为`Pending`。这些 Pod 将保持此状态，直到请求的资源变得可用。

让我们描述一下 `go-demo-2-db` 部署，看看是否能从中获取一些额外的有用信息。

```
kubectl describe pod go-demo-2-db  
```

限制在事件部分的输出如下：

```
...
Events:
 Type    Reason           Age               From              Message
 ----    ------           ----              ----              -------
  Warning FailedScheduling 11s (x7 over 42s) default-scheduler 0/
 1 nodes are available: 1 Insufficient memory.

```

我们可以看到它已经 `FailedScheduling` 了七次，且消息清楚地表明存在 `Insufficient memory`。

我们将恢复到初始定义。尽管我们知道它的资源配置不正确，但我们知道它满足所有要求，且所有 Pod 将成功调度：

```
kubectl apply \  
    -f res/go-demo-2-random.yml \  
    --record 

kubectl rollout status \  
    deployment go-demo-2-db 

kubectl rollout status \ 
    deployment go-demo-2-api 
```

现在所有 Pod 都在运行，我们应该尝试编写更好的定义。为此，我们需要观察内存和 CPU 使用情况，并利用这些信息来决定请求和限制。

# 基于实际使用情况调整资源

我们已经看到了资源使用和资源规格之间差异可能带来的一些影响。调整我们的规格以更好地反映实际的内存和 CPU 使用情况是理所当然的。

我们从数据库开始：

```
DB_POD_NAME=$(kubectl get pods \  
    -l service=go-demo-2 \  
    -l type=db \ 
    -o jsonpath="{.items[0].metadata.name}") 

curl "$BASE_URL/$DB_POD_NAME/containers/db/metrics/memory/usage" 

curl "$BASE_URL/$DB_POD_NAME/containers/db/metrics/cpu/usage_rate" 
```

我们检索到了数据库 Pod 的名称，并使用它获取了 `db` 容器的内存和 CPU 使用情况。因此，我们现在知道内存使用量在 `30Mi` 和 `40Mi` 之间。同样，我们知道 CPU 消耗大约在 `5m` 附近。

让我们拿 `api` 容器的相同指标来看看。

```
API_POD_NAME=$(kubectl get pods \ 
    -l service=go-demo-2 \ 
    -l type=api \ 
    -o jsonpath="{.items[0].metadata.name}") 

curl "$BASE_URL/$API_POD_NAME/containers/api/metrics/memory/usage" 

curl 
 "$BASE_URL/$API_POD_NAME/containers/api/metrics/cpu/usage_rate" 
```

正如预期的那样，`api` 容器使用的资源甚至比 MongoDB 还少。它的内存在 `3Mi` 和 `7Mi` 之间。它的 CPU 使用率低到 Heapster 将其四舍五入为 `0m`。

具备了这些知识，我们可以继续更新我们的 YAML 定义。然而，在此之前，我需要澄清一些事情。

我们收集的指标基于一些什么也不做的应用程序。一旦它们开始承载实际负载并开始处理生产规模的数据，指标将会发生剧烈变化。你需要的是一种方法来预测应用程序在生产环境中将使用多少资源，而不是在简单的测试环境中。你可能倾向于进行压力测试，以模拟生产环境。这样做是有意义的，但不一定能得到与实际生产环境相似的行为。

复制生产环境和模拟真实用户行为是很困难的。压力测试能够帮助你走一半的路。剩下的一半，你需要在生产环境中监控你的应用，并且，除了其他事情外，还需要根据实际情况调整资源。你应该考虑许多额外的因素，但目前，我想强调的是，什么都不做的应用并不是衡量资源使用的好标准。不过，我们将假设当前运行的应用程序承受的是类似生产环境的负载，且我们收集的指标代表了这些应用在生产中的表现。

简单的测试环境无法反映生产环境中资源的使用情况。压力测试是一个好的开始，但并不是一个完整的解决方案。只有生产环境提供真实的度量数据。

让我们来看看一个更能代表应用程序资源使用的新定义。

```
cat res/go-demo-2.yml  
```

输出，仅限于相关部分如下：

```
apiVersion: apps/v1beta2 
kind: Deployment 
metadata: 
  name: go-demo-2-db 
spec: 
  ... 
  template: 
    ... 
    spec: 
      containers: 
      - name: db 
        image: mongo:3.3 
        resources: 
          limits: 
            memory: "100Mi" 
            cpu: 0.1 
          requests: 
            memory: "50Mi" 
            cpu: 0.01 
... 
apiVersion: apps/v1beta2 
kind: Deployment 
metadata: 
  name: go-demo-2-api 
spec: 
  ... 
  template: 
    ... 
    spec: 
      containers: 
      - name: api 
        image: vfarcic/go-demo-2 
        ... 
        resources: 
          limits: 
            memory: "10Mi" 
            cpu: 0.1 
          requests: 
            memory: "5Mi" 
            cpu: 0.01 
```

这样就好多了。资源请求仅略高于当前使用量。我们将内存限制设置为请求值的两倍，以便应用程序在偶尔（且短暂）需要额外内存时，有充足的资源。CPU 限制远高于请求值，主要是因为我太不好意思将 CPU 限制设置为不到十分之一的 CPU。无论如何，关键在于请求值接近实际使用情况，而限制值较高，以便在资源使用暂时激增时，应用程序有些喘息空间。

现在剩下的就是应用新的定义：

```
kubectl apply \  
    -f res/go-demo-2.yml \  
    --record 

kubectl rollout status \ 
    deployment go-demo-2-api 
```

`deployment "go-demo-2-api"` 已成功发布，我们可以进入下一个主题。

# 探索服务质量（QoS）合同

当我们向 Kubernetes API 发送请求创建一个 Pod（直接或通过其中一个控制器）时，它会启动调度过程。接下来会发生什么，或者更准确地说，它决定在哪个地方运行 Pod，主要取决于我们为组成该 Pod 的容器定义的资源。简而言之，Kubernetes 会决定在某个节点上部署 Pod，只要该节点有足够的可用内存。

当定义了内存请求时，Pod 将获得它们请求的内存。如果某个容器的内存使用量超过请求的数量，或者其他 Pod 需要这些内存，那么托管该容器的 Pod 可能会被终止。请注意，我说的是 Pod *可能* 被终止。是否发生这种情况取决于其他 Pod 的请求和集群中可用的内存。另一方面，超出内存限制的容器总是会被终止（除非是暂时的情况）。

CPU 请求和限制有些不同。超过指定 CPU 资源的容器不会被终止。相反，它们会被限制。

现在我们稍微了解了 Kubernetes 的终止机制，我们应该注意到（几乎）没有什么是随机发生的。当没有足够的资源来满足所有 Pod 的需求时，Kubernetes 会销毁一个或多个容器。决定销毁哪一个是绝非随机的。谁将成为不幸的那个，取决于分配的 **服务质量**（**QoS**）。优先级最低的会最先被销毁。

由于这可能是你第一次听说 QoS，我们将花一些时间来解释它们是什么以及如何工作。

Pods 是 Kubernetes 中最小的单元。由于几乎所有东西都最终作为 Pod 存在（无论以何种方式），因此 Kubernetes 承诺为集群中运行的所有 Pods 提供特定的保证也就不足为奇了。每当我们向 API 发送请求以创建或更新 Pod 时，Pod 会被分配一个 QoS 类别。这些 QoS 用于做出决策，例如在哪里调度 Pod 或是否驱逐它。

我们不会直接指定 QoS。相反，它们是根据我们在资源请求和限制上的决策来分配的。

目前，有三种 QoS 类别可用。每个 Pod 都可以拥有 *Guaranteed*、*Burstable* 或 *BestEffort* QoS。

**Guaranteed QoS** 只分配给那些为所有容器设置了 CPU 请求和限制以及内存请求和限制的 Pods。我们使用上一个定义创建的 Pods 符合这一标准。然而，还有一个必须满足的必要条件。每个容器的请求和限制值必须相同。不过，这里有个陷阱。当容器仅指定限制时，请求会自动设置为与限制相同的值。换句话说，当容器没有指定请求时，只要它们的限制已定义，它们将拥有 Guaranteed QoS。

我们可以总结出保证 QoS 的标准如下：

+   必须设置内存和 CPU 限制

+   内存和 CPU 请求必须设置为与限制相同的值，或者可以留空，在这种情况下，它们默认为限制值（我们稍后会详细探讨）。

分配了 Guaranteed QoS 的 Pods 是最高优先级，除非它们超出限制或不健康，否则永远不会被杀掉。当出现问题时，它们是最后一个被终止的。只要它们的资源使用量在限制范围内，Kubernetes 总是会选择在资源使用超出容量时终止具有其他 QoS 分配的 Pods。

让我们继续下一个 QoS。

**Burstable QoS** 分配给不符合 Guaranteed QoS 标准但至少有一个容器定义了内存或 CPU 请求的 Pods。

具有 Burstable QoS 的 Pods 保证最小（请求的）内存使用量。如果有更多资源可用，它们可能会使用更多资源。如果系统处于压力状态并需要更多可用内存，则具有 Burstable QoS 的容器比那些具有 Guaranteed QoS 的容器更容易被杀掉，前提是没有具有 BestEffort QoS 的 Pods。你可以将具有该 QoS 的 Pods 看作中等优先级。

最后，我们到达了最后一个 QoS。

**BestEffort QoS** 分配给不符合 Guaranteed 或 Burstable 标准的 Pods。它们是由没有定义任何资源的容器组成的 Pods。符合 BestEffort 条件的 Pods 中的容器可以使用任何它们需要的可用内存。

当需要更多资源时，Kubernetes 将开始终止位于 BestEffort QoS 的 Pods 中的容器。它们的优先级最低，当需要更多内存时，它们是最先被终止的。

让我们来看一下我们的 `go-demo-2-db` Pod 被分配了哪种 QoS。

```
kubectl describe pod go-demo-2-db  
```

输出，限制为相关部分，如下所示：

```
... 
Containers: 
  db: 
    ... 
    Limits: 
      cpu:    100m 
      memory: 100Mi 
    Requests: 
      cpu:    10m 
      memory: 50Mi 
... 
QoS Class:       Burstable 
... 
```

该 Pod 被分配了可突发的 QoS。它的限制与请求不同，因此不符合保证的 QoS。由于它的资源已设置，并且不符合保证的 QoS，Kubernetes 将其分配为第二优先级的 QoS。

现在，让我们看一下稍作修改的定义：

```
cat res/go-demo-2-qos.yml  
```

输出，限制为相关部分，如下所示：

```
apiVersion: apps/v1beta2 
kind: Deployment 
metadata: 
  name: go-demo-2-db 
spec: 
  ... 
  template: 
    ... 
    spec: 
      containers: 
      - name: db 
        image: mongo:3.3 
        resources: 
          limits: 
            memory: "50Mi" 
            cpu: 0.1 
          requests: 
            memory: "50Mi" 
            cpu: 0.1 
... 
apiVersion: apps/v1beta2 
kind: Deployment 
metadata: 
  name: go-demo-2-api 
spec: 
  ... 
  template: 
    ... 
    spec: 
      containers: 
      - name: api 
        image: vfarcic/go-demo-2 
... 
```

这一次，我们指定了 `cpu` 和 `memory` 对容器的 `requests` 和 `limits` 应具有相同的值，以便通过 `go-demo-2-db` Deployment 创建的容器。因此，它应该被分配为 `Guaranteed` QoS。

`go-demo-2-api` Deployment 的容器没有任何 `resources` 定义，因此将被分配为 BestEffort QoS。

让我们确认这两个假设（不说是猜测）是否确实正确。

```
kubectl apply \  
    -f res/go-demo-2-qos.yml \ 
    --record 

kubectl rollout status \ 
    deployment go-demo-2-db 
```

我们应用了新的定义并输出了 `go-demo-2-db` Deployment 的滚动更新状态。

现在我们可以描述通过 `go-demo-2-db` Deployment 创建的 Pod，并检查其 QoS。

```
kubectl describe pod go-demo-2-db  
```

输出，限制为相关部分，如下所示：

```
Containers: 
  db: 
    ... 
    Limits: 
      cpu:    100m 
      memory: 50Mi 
    Requests: 
      cpu:    100m 
      memory: 50Mi 
... 
QoS Class: Guaranteed 
... 
```

内存和 CPU 的限制与请求相同，因此 QoS 为 `Guaranteed`。

让我们检查通过 `go-demo-2-api` Deployment 创建的 Pods 的 QoS。

```
kubectl describe pod go-demo-2-api  
```

输出，限制为相关部分，如下所示：

```
... 
QoS Class:       BestEffort 
... 
QoS Class:       BestEffort 
... 
QoS Class:       BestEffort 
... 
```

通过 `go-demo-2-api` Deployment 创建的三个 Pods 没有任何资源定义，因此它们的 QoS 被设置为 `BestEffort`。

我们将不再需要之前创建的对象，所以我们在进入下一个主题之前会先将它们删除。

```
kubectl delete \
 -f res/go-demo-2-qos.yml
```

# 在命名空间内定义资源默认值和限制

我们已经学习了如何利用 Kubernetes 命名空间在一个集群内创建多个子集群。当与 RBAC 结合使用时，我们可以创建命名空间并授予用户使用这些命名空间的权限，而不暴露整个集群。然而，仍然有一件事缺失。

我们可以假设，创建一个 `test` 命名空间，并允许用户在不允许其访问其他命名空间的情况下创建对象。尽管这样比允许所有人完全访问集群要好，但这种策略并不能防止人们将整个集群宕机或影响在其他命名空间中运行的应用程序的性能。我们缺少的那一块拼图就是在命名空间级别上的资源控制。

我们已经讨论过每个容器都应该定义资源的 `limits` 和 `requests`。这些信息帮助 Kubernetes 更高效地调度 Pods。它还为 Kubernetes 提供了可以用来决定是否驱逐或重启 Pod 的信息。然而，虽然我们可以指定 `resources`，并不意味着我们必须强制定义它们。我们应该有能力设置默认的 `resources`，以便在我们忘记显式指定时自动应用。

即使我们定义了默认的 `resources`，我们也需要一种设置限制的方式。否则，所有具有部署 Pod 权限的人都有可能运行请求资源超过我们愿意提供的应用程序。

总而言之，我们下一步的任务是定义默认请求和限制，并指定可以为 Pod 定义的最小和最大值。

我们将从创建一个 `test` 命名空间开始。

```
kubectl create namespace test  
```

创建了一个 playground 命名空间后，我们可以查看一个新的定义。

```
cat res/limit-range.yml  
```

输出如下：

```
apiVersion: v1 
kind: LimitRange 
metadata: 
  name: limit-range 
spec: 
  limits: 
  - default: 
      memory: 50Mi 
      cpu: 0.2 
    defaultRequest: 
      memory: 30Mi 
      cpu: 0.05 
    max: 
      memory: 80Mi 
      cpu: 0.5 
    min: 
      memory: 10Mi 
      cpu: 0.01 
    type: Container 
```

我们指定资源应该是 `LimitRange` 类型。它的 `spec` 有四个 `limits`。

对于没有指定资源的容器，将应用 `default` 限制和 `defaultRequest` 条目。如果容器没有内存或 CPU 限制，它将被分配到 `LimitRange` 中设置的值。`default` 条目用作限制，`defaultRequest` 条目用作请求。

当容器确实定义了资源时，它们将根据 `LimitRange` 中指定的 `max` 和 `min` 阈值进行评估。如果容器不符合标准，将不会创建承载容器的 Pod。

我们很快将看到四个 `limits` 的实际实现。目前，下一步是创建 `limit-range` 资源：

```
kubectl --namespace test create \  
    -f res/limit-range.yml \  
    --save-config --record 
```

我们创建了 `LimitRange` 资源。

让我们描述创建资源的 `test` 命名空间。

```
kubectl describe namespace test  
```

输出仅限于相关部分如下。

```
...
Resource Limits
 Type      Resource Min  Max  Default Request Default Limit  Max  Limit/Request Ratio
 ----      -------- ---  ---  --------------- ------------- ----- ------------------
 Container cpu      10m  500m 50m             200m          -
 Container memory   10Mi 80Mi 30Mi            50Mi          - 
```

我们可以看到 `test` 命名空间拥有我们指定的资源限制。我们设置了五个可能的值中的四个。`maxLimitRequestRatio` 没有被设置，我们将简要描述它。当设置了 `MaxLimitRequestRatio` 时，容器的请求和限制资源必须同时非零，并且限制除以请求必须小于或等于枚举值。

让我们再看一个 `go-demo` 定义的变体：

```
cat res/go-demo-2-no-res.yml  
```

值得注意的是，没有一个容器定义了任何资源。

接下来，我们将创建在 `go-demo-2-no-res.yml` 文件中定义的对象。

```
kubectl --namespace test create \  
    -f res/go-demo-2-no-res.yml \ 
    --save-config --record 

kubectl --namespace test \ 
    rollout status \ 
    deployment go-demo-2-api 
```

我们在 `test` 命名空间内创建了对象，并等待成功部署 `deployment "go-demo-2-api"`。

让我们描述我们创建的一个 Pod：

```
kubectl --namespace test describe \  
    pod go-demo-2-db 
```

输出仅限于相关部分如下：

```
... 
Containers: 
  db: 
    ... 
    Limits: 
      cpu:     200m 
      memory:  50Mi 
    Requests: 
      cpu:        50m 
      memory:     30Mi 
... 
```

即使我们没有明确指定 `go-demo-2-db` Pod 内 `db` 容器的资源，资源已经被设置。`db` 容器被分配了 `test` 命名空间的 `default` 限制作为容器的限制。类似地，`defaultRequest` 限制被用作容器的请求。

当我们尝试创建未指定资源的容器宿主 Pod 时，将会受到命名空间限制的影响。

我们仍然应该定义容器资源，而不是依赖命名空间的默认限制。毕竟，它们只是在有人忘记定义资源时的备选项。

让我们看看当资源被定义但不符合命名空间 `min` 和 `max` 限制时会发生什么。

我们将使用之前使用的相同`go-demo-2.yml`。

```
cat res/go-demo-2.yml  
```

输出，限定为相关部分，如下所示：

```
... 
apiVersion: apps/v1beta2 
kind: Deployment 
metadata: 
  name: go-demo-2-db 
spec: 
  ... 
  template: 
    ... 
    spec: 
      containers: 
      - name: db 
        image: mongo:3.3 
        resources: 
          limits: 
            memory: "100Mi" 
            cpu: 0.1 
          requests: 
            memory: "50Mi" 
            cpu: 0.01 
... 
apiVersion: apps/v1beta2 
kind: Deployment 
metadata: 
  name: go-demo-2-api 
spec: 
  ... 
  template: 
    ... 
    spec: 
      containers: 
      - name: api 
        ... 
        resources: 
          limits: 
            memory: "10Mi" 
            cpu: 0.1 
          requests: 
            memory: "5Mi" 
            cpu: 0.01 
... 
```

关键点在于，两个部署（Deployments）的`resources`都已定义。

让我们创建这些对象并获取事件。这些将帮助我们更好地理解发生了什么。

```
kubectl --namespace test apply \
 -f res/go-demo-2.yml \
 --record

kubectl --namespace test get events -w  
```

后一个命令的输出，限定为相关部分，如下所示：

```
    ... Error creating: pods "go-demo-2-db-868dbbc488-s92nm" is forbi
 dden: maximum memory usage per Container is 80Mi, but limit is 100Mi.
    ...
    ... Error creating: pods "go-demo-2-api-6bd767ffb6-96mbl" is 
 forbidden: minimum memory usage per Container is 10Mi, but request is 
 5Mi.
    ...

```

我们可以看到我们被禁止创建这两个 Pod。事件之间的区别在于是什么原因导致 Kubernetes 拒绝了我们的请求。

`go-demo-2-db-*` Pod 无法创建，因为其`每个容器的最大内存使用量为 80Mi，但限制为 100Mi`。另一方面，我们被禁止创建`go-demo-2-api-*` Pods，因为`每个容器的最小内存使用量为 10Mi，但请求为 5Mi`。

`test`命名空间中的所有容器必须遵守`min`和`max`限制。否则，我们将被禁止创建它们。容器的限制不能高于命名空间的`max`限制。另一方面，容器资源请求不能小于命名空间的`min`限制。

如果我们将命名空间的限制视为上下阈值，我们可以说容器请求不能低于它们，而容器限制不能高于它们。

按下*Ctrl* + *C*键以停止观察事件。

如果我们直接创建 Pod 而不是通过部署（Deployments），可能更容易观察到`max`和`min`限制的效果。

```
kubectl --namespace test run test \  
    --image alpine \ 
    --requests memory=100Mi \ 
    --restart Never \  
    sleep 10000 
```

我们尝试创建一个内存请求设置为`100Mi`的 Pod。由于命名空间的限制为`80Mi`，API 返回了错误信息，表示`Pod "test" 无效`。尽管`max`限制是针对容器的`limit`，但在没有限制的情况下使用了内存请求。

我们将进行类似的练习，但这次只设置`1Mi`作为内存请求。

```
kubectl --namespace test run test \ 
    --image alpine \ 
    --requests memory=1Mi \  
    --restart Never \ 
    sleep 10000 
```

这次，错误稍有不同。我们可以看到`pods "test" 被禁止：每个容器的最小内存使用量为 10Mi，但请求为 1Mi`。我们请求的值低于`test`命名空间的`min`限制，因此我们被禁止创建 Pod。

在进入下一个主题之前，我们将删除`test`命名空间。

```
kubectl delete namespace test  
```

# 为命名空间定义资源配额

资源默认值和限制是防止恶意或意外部署 Pod 的良好第一步，这些 Pod 可能对集群产生不良影响。然而，任何具有在命名空间中创建 Pod 权限的用户都可以超载系统。即使将`max`值设置为合理较小的内存和 CPU 量，用户也可能部署成千上万，甚至数百万个 Pod，从而“消耗”掉所有可用的集群资源。这种效果可能并非出于恶意，而是意外发生的。例如，一个 Pod 可能附加到一个自动扩展的系统上，并且没有定义上限，结果它可能会扩展到太多副本。还有很多其他方式可能导致情况失控。

我们需要做的是通过配额来定义命名空间的边界。

通过配额，我们可以保证每个命名空间获得其应有的资源份额。与适用于每个容器的`LimitRange`规则不同，`ResourceQuota`基于资源总消耗定义命名空间的限制。

我们可以使用`ResourceQuota`对象来定义命名空间中可以消耗的总计算资源（内存和 CPU）。我们还可以用它来限制存储使用或命名空间中可以创建的某一类型对象的数量。

让我们来看看在 Minikube 集群中我们拥有的资源。它很小，甚至不是真正的集群。然而，它是我们唯一拥有的（暂时），所以请发挥你的想象力，假装它是“真实的”。

我们的集群有 2 个 CPU 和 2GB 内存。现在，假设这个集群仅用于开发和生产目的。我们可以使用`default`命名空间用于生产，并创建一个`dev`命名空间用于开发。我们可以假设生产环境应该消耗集群的所有资源，减去分配给`dev`命名空间的资源，而`dev`命名空间则不应超过特定的资源限制。

事实是，2 个 CPU 和 2GB 内存的情况下，我们能够分配给开发者的资源并不多。尽管如此，我们还是会尽量慷慨一些。我们将为请求分配 500MB 和 0.8 个 CPU。我们还将通过设置 1 个 CPU 和 1GB 内存的限制来允许资源使用时偶尔的突发需求。此外，我们可能希望将 Pod 的数量限制为 10 个。最后，为了降低风险，我们将拒绝开发者使用 NodePort 的权限。

这不是一个不错的计划吗？我想象在这一刻，你正在点头表示同意，所以我们继续前进，创建我们讨论过的配额。

让我们来看看`dev.yaml`的定义：

```
cat res/dev.yml  
```

输出如下：

```
apiVersion: v1
kind: Namespace
metadata:
 name: dev

---

apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev
  namespace: dev
spec:
  hard:
    requests.cpu: 0.8
    requests.memory: 500Mi
    limits.cpu: 1
    limits.memory: 1Gi
    pods: 10
    services.nodeports: "0"  
```

除了创建`dev`命名空间，我们还创建了一个`ResourceQuota`。它指定了一组`hard`限制。请记住，它们是基于聚合数据的，而不是像 LimitRanges 那样按容器计算的。

我们设置了请求配额为`0.8`个 CPU 和`500Mi`内存。类似地，限制配额设置为`1`个 CPU 和`1Gi`内存。最后，我们指定`dev`命名空间最多可以有`10`个 Pod，且不能有 NodePort。这就是我们制定和定义的计划。现在让我们创建这些对象并探索其效果。

```
kubectl create \ 
 -f res/dev.yml \ 
 --record --save-config  
```

我们可以从输出中看到`namespace "dev"`和`resourcequota "dev"`都已创建。为了安全起见，我们将描述新创建的`devquota`。

```
kubectl --namespace dev describe \ 
    quota dev 
```

输出如下：

```
Name:               dev
Namespace:          dev
Resource            Used  Hard
--------            ----  ----
limits.cpu          0     1
limits.memory       0     1Gi
pods                0     10
requests.cpu        0     800m
requests.memory     0     500Mi
services.nodeports  0     0
```

我们可以看到硬性限制已设置，并且当前没有使用。这是预料之中的，因为我们没有在`dev`命名空间中运行任何对象。让我们通过创建已经过于熟悉的`go-demo-2`对象来让它变得有趣一些。

```
kubectl --namespace dev create \
 -f res/go-demo-2.yml \ 
 --save-config --record

kubectl --namespace dev \ 
 rollout status \ 
 deployment go-demo-2-api  
```

我们从`go-demo-2.yml`文件中创建了对象，并等待`go-demo-2-api`部署完成。现在我们可以重新查看`dev`配额的值：

```
kubectl --namespace dev describe \ 
    quota dev 
```

输出如下：

```
Name:              dev
Namespace:         dev
Resource           Used  Hard
--------           ----  ----
limits.cpu         400m  1
limits.memory      130Mi 1Gi
pods               4     10
requests.cpu       40m   800m
requests.memory    65Mi  500Mi
services.nodeports 0     0  
```

从 `Used` 列中，我们可以看到例如我们目前正在运行 `4` 个 Pod，并且仍然低于 `10` 的限制。其中一个 Pod 是通过 `go-demo-2-db` 部署创建的，另外三个是通过 `go-demo-2-api` 创建的。如果你总结这些 Pod 中容器的资源配置，你会看到这些值与已使用的 `limits` 和 `requests` 匹配。

到目前为止，我们还没有达到任何配额。让我们尝试打破至少一个配额。

```
cat res/go-demo-2-scaled.yml  
```

仅限相关部分的输出如下：

```
... 
apiVersion: apps/v1beta2 
kind: Deployment 
metadata: 
  name: go-demo-2-api 
spec: 
  replicas: 15 
  ... 
```

`go-demo-2-scaled.yml` 的定义几乎与 `go-demo-2.yml` 相同。唯一的不同是 `go-demo-2-api` 部署的副本数增加到十五个。正如你所知道的，这将导致通过该部署创建十五个 Pod。

我敢肯定你能猜到如果我们应用新的定义会发生什么，但我们还是会继续这样做。

```
kubectl --namespace dev apply \
 -f res/go-demo-2-scaled.yml \
 --record
```

我们应用了新的定义。我们将给 Kubernetes 一些时间来完成工作，然后再查看它将生成的事件。因此，深呼吸，从一数到你笔记本电脑上处理器的数量。以我为例，应该是“一 Mississippi，二 Mississippi，三 Mississippi”，一直数到十六 Mississippi。

```
kubectl --namespace dev get events  
```

在 `dev` 命名空间内生成的一些事件的输出如下：

```
...
... Error creating: pods "go-demo-2-api-..." is forbidden: exceeded quota: dev, requested: limits.cpu=100m,pods=1, used: limits.cpu=1,pods=10, limited: limits.cpu=1,pods=10
    13s         13s          1         go-demo-2-api-6bd767ffb6.150f5   1f4b3a7ed3f         ReplicaSet                          Warning .
 .. Error creating: pods "go-demo-2-api-..." is forbidden: exceeded quota: dev, requested: limits.cpu=100m,pods=1, used: limits.cpu=1,pods=10, limited: limits.cpu=1,pods=10
... 
```

我们可以看到我们达到了命名空间配额施加的两个限制。我们达到了 CPU 的最大值（`1`）和 Pods 的最大值（`10`）。因此，ReplicaSet 控制器被禁止创建新的 Pod。

我们应该能够通过描述 `dev` 命名空间来确认达到的硬限制。

```
kubectl describe namespace dev  
```

仅限于 `Resource Quotas` 部分的输出如下：

```
... 
Resource Quotas 
 Name:               dev 
 Resource            Used   Hard 
 --------            ---    --- 
 limits.cpu          1      1 
 limits.memory       190Mi  1Gi 
 pods                10     10 
 requests.cpu        100m   800m 
 requests.memory     95Mi   500Mi 
 services.nodeports  0      0 
... 
```

如事件所示，`limits.cpu` 和 `pods` 资源在 `User` 和 `Hard` 列中的值是相同的。因此，我们将无法再创建更多的 Pod，也无法为已运行的 Pod 增加 CPU 限制。

最后，让我们来看一下 `dev` 命名空间中的 Pod。

```
kubectl get pods --namespace dev  
```

以下是前面命令的输出：

```
NAME              READY STATUS  RESTARTS AGE
go-demo-2-api-... 1/1   Running 0        3m
go-demo-2-api-... 1/1   Running 0        3m
go-demo-2-api-... 1/1   Running 0        5m
go-demo-2-api-... 1/1   Running 0        3m
go-demo-2-api-... 1/1   Running 0        5m
go-demo-2-api-... 1/1   Running 0        3m
go-demo-2-api-... 1/1   Running 0        3m
go-demo-2-api-... 1/1   Running 0        3m
go-demo-2-api-... 1/1   Running 0        5m
go-demo-2-db-...  1/1   Running 0        5m  
```

`go-demo-2-api` 部署成功创建了九个 Pod。再加上通过 `go-demo-2-db` 创建的 Pod，我们达到了十个的上限。

我们确认了限制和 Pod 配额有效。我们将在进行下一次验证之前恢复到先前的定义（即不会达到任何配额的定义）。

```
kubectl --namespace dev apply \
 -f res/go-demo-2.yml \
 --record

kubectl --namespace dev \
 rollout status \
 deployment go-demo-2-api  
```

后者命令的输出应表明 `deployment "go-demo-2-api" 已成功发布`。

让我们来看一下另一个稍微修改过的 `go-demo-2` 对象定义：

```
cat res/go-demo-2-mem.yml  
```

仅限相关部分的输出如下：

```
...
apiVersion: apps/v1beta2
kind: Deployment
metadata:
 name: go-demo-2-db
spec:
 ...
 template:
 ...
 spec:
 containers:
 - name: db
 image: mongo:3.3
 resources:
 limits:
 memory: "100Mi"
 cpu: 0.1
 requests:
 memory: "50Mi"
 cpu: 0.01
...
apiVersion: apps/v1beta2
kind: Deployment
metadata:
 name: go-demo-2-api
spec:
 replicas: 3
 ...
 template:
 ...
 spec:
 containers:
 - name: api
 ...
 resources:
 limits:
 memory: "200Mi"
 cpu: 0.1
 requests:
 memory: "200Mi"
 cpu: 0.01
...  
```

`go-demo-2-api` 部署的 `api` 容器的内存请求和限制都设置为 `200Mi`，而数据库的内存请求为 `50Mi`。知道 `dev` 命名空间的 `requests.memory` 配额为 `500Mi`，简单的数学运算足以得出结论，我们无法运行 `go-demo-2-api` 部署的所有三个副本。

```
kubectl --namespace dev apply \
 -f res/go-demo-2-mem.yml \
 --record  
```

就像之前一样，我们应该等一会儿，再查看 `dev` 命名空间的事件。

```
kubectl --namespace dev get events \
 | grep mem  
```

输出仅限于其中一个条目的部分，如下所示：

```
... Error creating: pods "go-demo-2-api-..." is forbidden: exceeded quota: dev, requested: requests.memory=200Mi, used: requests.memory=455Mi, limited: requests.memory=500Mi
```

我们达到了 `requests.memory` 配额。因此，至少一个 Pod 的创建被禁止。我们可以看到，我们请求创建了一个内存请求为 `200Mi` 的 Pod。由于当前内存请求的汇总为 `455Mi`，创建该 Pod 会超过分配的 `500Mi`。

让我们仔细看看命名空间。

```
kubectl describe namespace dev  
```

输出仅限于 `Resource Quotas` 部分，如下所示：

```
...
Resource Quotas
 Name:               dev
 Resource            Used   Hard
 --------            ---    ---
 limits.cpu          400m   1
 limits.memory       510Mi  1Gi
 pods                4      10
 requests.cpu        40m    800m
 requests.memory     455Mi  500Mi
 services.nodeports  0      0
...  
```

确实，已使用的内存请求量为 `455Mi`，这意味着我们可以创建额外的 Pod，最多 `45Mi`，而不是 `200Mi`。

在我们探索最后一个定义的配额之前，我们将再次恢复到 `go-demo-2.yml`。

```
kubectl --namespace dev apply \ 
 -f res/go-demo-2.yml \ 
 --record

kubectl --namespace dev \
 rollout status \
 deployment go-demo-2-api  
```

唯一未验证的配额是 `services.nodeports`。我们将其设置为 `0`，因此我们不应该允许暴露任何节点端口。让我们确认这一点是否确实正确。

```
kubectl expose deployment go-demo-2-api \
 --namespace dev \
 --name go-demo-2-api \ 
 --port 8080 \
 --type NodePort  
```

输出如下所示：

```
Error from server (Forbidden): services "go-demo-2-api" is forbidden: exceeded quota: dev, requested: services.nodeports=1, used: services.nodeports=0, limited: services.nodeports=0
```

所有配额按预期工作。但还有其他配额。我们没有时间探索我们可以使用的所有配额的示例。相反，我们将它们全部列出，供将来参考。

我们可以将配额分为几个组：

**计算资源配额** 限制计算资源的总和。它们如下所示：

| **资源名称** | **描述** |
| --- | --- |
| `cpu` | 在所有非终止状态的 Pod 中，CPU 请求的总和不能超过此值。 |
| `limits.cpu` | 在所有非终止状态的 Pod 中，CPU 限制的总和不能超过此值。 |
| `limits.memory` | 在所有非终止状态的 Pod 中，内存限制的总和不能超过此值。 |
| `memory` | 在所有非终止状态的 Pod 中，内存请求的总和不能超过此值。 |
| `requests.cpu` | 在所有非终止状态的 Pod 中，CPU 请求的总和不能超过此值。 |
| `requests.memory` | 在所有非终止状态的 Pod 中，内存请求的总和不能超过此值。 |

**存储资源配额** 限制存储资源的总和。我们尚未深入探索存储（除了几个本地示例），所以你可能需要保留下面的列表以备将来参考：

| **资源名称** | **描述** |
| --- | --- |
| `requests.storage` | 在所有持久卷声明中，存储请求的总和不能超过此值。 |
| `persistentvolumeclaims` | 命名空间中可以存在的持久卷声明的总数。 |
| `[PREFIX]/requests.storage` | 与存储类名称相关的所有持久卷索赔的总存储请求不能超过此值。 |
| `[PREFIX]/persistentvolumeclaims` | 与存储类名称相关的所有持久卷索赔的总数，可存在于命名空间中。 |
| `requests.ephemeral-storage` | 命名空间中所有 Pod 的本地临时存储请求总和不能超过此值。 |
| `limits.ephemeral-storage` | 命名空间中所有 Pod 的本地临时存储限制的总和不能超过此值。 |

请注意，`[PREFIX]`应替换为`<storage-class-name>.storageclass.storage.k8s.io`。

**对象计数配额** 限制了给定类型对象的数量。它们如下：

| **资源名称** | **描述** |
| --- | --- |
| `configmaps` | 可存在于命名空间中的配置映射总数。 |
| `persistentvolumeclaims` | 可存在于命名空间中的持久卷索赔总数。 |
| `pods` | 可存在于命名空间中的非终端状态下的 Pod 总数。如果 status.phase 为（Failed，Succeeded），则 Pod 处于终止状态。 |
| `replicationcontrollers` | 可存在于命名空间中的复制控制器总数。 |
| `resourcequotas` | 可存在于命名空间中的资源配额总数。 |
| `services` | 可存在于命名空间中的服务总数。 |
| `services.loadbalancers` | 可存在于命名空间中的负载均衡器类型的服务总数。 |
| `services.nodeports` | 可存在于命名空间中的节点端口类型的服务总数。 |
| `secrets` | 可存在于命名空间中的秘密总数。 |

# 现在怎么办？

那不是一次很好的体验吗？

Kubernetes 在整个集群中广泛依赖可用资源。但它不能变出魔法。我们需要通过定义我们预期容器将消耗的资源来帮助它。

尽管 Heapster 不是收集指标的最佳解决方案，但它已经在我们的 Minikube 集群中可用，我们用它来了解我们的应用程序使用多少资源，通过这些信息，我们优化了我们的资源定义。没有指标，我们的定义就是纯粹的猜测。当我们猜测时，Kubernetes 也需要猜测。一个稳定的系统是基于事实而不是想象的可预测系统。Heapster 帮助我们将假设转化为可衡量的事实，然后我们将这些事实提供给 Kubernetes，在其调度算法中使用。

资源定义的探索引导我们进入**服务质量**（**QoS**）。尽管 Kubernetes 决定使用哪种 QoS，但如果我们要优先考虑应用程序及其可用性，了解决策过程中使用的规则是至关重要的。

所有这些将引导我们走向确保集群安全、稳定和强大的战略的顶点。将集群划分为命名空间并采用 RBAC 还远远不够。RBAC 可以防止未经授权的用户访问集群，并为我们信任的用户提供权限。然而，RBAC 并不能阻止用户通过过多的部署、过大的应用程序或不准确的资源配置，意外（或故意）将集群置于危险之中。只有将 RBAC 与资源默认值、限制和配额结合使用，我们才能期望构建一个容错且稳健的集群，能够可靠地托管我们的应用程序。

我们几乎学完了所有 Kubernetes 的核心对象和原理。现在是时候迁移到一个“真实”的集群了。我们即将最后一次删除 Minikube 集群（至少在本书中是最后一次）。

```
minikube delete
```

# Kubernetes 资源管理与 Docker Swarm 的对比

资源管理可以分为几个类别。我们需要定义一个容器预期使用多少内存和 CPU，以及它的资源限制。这些信息对调度器做出“智能”决策至关重要，尤其是在计算容器部署位置时。在这一点上，Kubernetes 和 Docker Swarm 没有本质上的区别。两者都使用请求的资源来决定容器的部署位置，并通过资源限制来决定何时驱逐容器。在这一点上，它们基本相同。

我们如何知道应该为每个容器分配多少内存和 CPU？这是我听到过无数次的问题。答案很简单。收集指标，评估它们，调整资源，休息一下，然后重复。我们从哪里收集指标？随你选择。Prometheus 是一个不错的选择。它从哪里获取指标？嗯，这取决于你使用的是哪个调度器。如果是 Docker Swarm，你需要运行一堆导出器。或者，你也许足够大胆，尝试暴露 Docker 内部指标的实验性功能，采用 Prometheus 格式。你甚至可能充满热情，认为这些指标足以满足你所有的监控和警报需求。也许，当你读到这篇文章时，这个功能已经不再是实验性的了。另一方面，Kubernetes 拥有一切，甚至更多。你可以使用 Heapster，或者你可能会发现它限制太多，于是配置 Prometheus 直接从 Kubernetes API 获取指标。Kubernetes 暴露了大量的数据，远比你可能需要的要多。你可以获取内存、CPU、IO、网络和无数其他指标，从而做出智能决策，不仅是关于容器所需资源的决策，还可以做出关于更多方面的决策。

为了澄清一点，无论你是运行 Kubernetes 还是 Docker Swarm，你都可以获得相同的度量指标。主要的区别在于 Kubernetes 通过其 API 暴露这些指标，而 Swarm 则需要在使用其有限的度量指标与设置像 cAdvisor 和 Node Exporter 这样的导出器之间做出抉择。很可能，你会发现你需要同时使用 Swarm API 的度量指标和导出器的数据。Kubernetes 提供了一个更为强大的解决方案，尽管你可能仍然需要一个或两个导出器。但总的来说，Kubernetes 能通过其 API 提供你几乎所有需要的度量指标，这无疑是一个非常方便的功能。

坦率地说，我们从两个调度器中获取度量指标的方式差异并不是特别重要。如果这就是资源管理的故事结束之处，我会得出结论：两种解决方案大致是一样好的。但故事还在继续。这也是相似之处结束的地方。更确切地说，这是 Docker Swarm 结束的地方，而 Kubernetes 才刚刚开始。

Kubernetes 允许我们定义资源的默认值和限制，并将其应用于没有指定资源的容器。它还允许我们指定资源配额，以防止资源被意外或恶意地过度使用。配额和名称空间相结合提供了非常强大的保障。它们为我们提供了设计真正容错系统的一些手段，通过防止恶意容器、失控的扩展和人为错误将集群拖入停滞。不要一秒钟都认为配额是构建强大系统所需的唯一因素。它不是拼图中的唯一一块，但无论如何，它是一个重要的部分。

名称空间与配额相结合非常重要。我甚至可以说它们是至关重要的。如果没有它们，我们将不得不为每个小组、团队或部门创建一个集群。或者，我们可能不得不进一步收紧流程，防止我们的团队无法充分利用容器编排工具的优势。如果目标是为我们的团队提供自由，而不牺牲集群稳定性，那么 Kubernetes 显然比 Docker Swarm 更具优势。

这场战斗 Kubernetes 赢了，但战争依然在继续。

好吧，我有点夸张了，使用了*战斗*和*战争*这些词。它并不是一个冲突，两个社区正在增加合作，分享想法和解决方案。两个平台正在融合。不过，目前来看，Kubernetes 在资源管理方面相较 Docker Swarm 确实占有明显优势。
