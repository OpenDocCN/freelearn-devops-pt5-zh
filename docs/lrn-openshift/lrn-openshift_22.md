# 第二十二章：评估

# 第一章

1.  答案是 1（Docker 容器，Docker 镜像，Docker 注册表）

1.  答案是 2 和 3（私有注册表和公共注册表）

1.  答案是 正确。

1.  答案是 1（Cgroups）

1.  答案是 1 和 4（`docker build -t new_httpd_image .` 和 `docker build -t new_httpd_image ./`）

1.  答案是 错误

# 第二章

1.  答案是 1 和 4（节点和主机）

1.  答案是 1 和 3（Docker 和 Rkt）

1.  答案是 正确

1.  答案是 2 和 3（kubelet 和 kube-proxy）

1.  答案是 1 和 5（JSON 和 YAML）

1.  答案是 错误

# 第三章

1.  答案是 2（CRI）

1.  答案是 1 和 3（Docker 和 Rkt）

1.  答案是 正确

1.  答案是 1（kubectl describe pods/httpd）

1.  答案是 2 和 3（CRI-O 直接与容器运行时通信，且 CRI-O 遵循 OCI 标准）

1.  答案是 正确

# 第四章

1.  答案是 PaaS

1.  答案是 1 和 3（OpenShift Origin 和 OpenShift Enterprise）

1.  答案是 错误

1.  答案是 4（持久存储）

1.  答案是 1 和 2（路由作为入口流量控制和 OpenShift 内部注册表）

1.  答案是 正确

# 第五章

1.  答案是 1（Docker）

1.  答案是 1（8443）

1.  答案是 错误

1.  答案是 2（oc login -u system:admin）

1.  答案是 1 和 3（oc login -u system:admin 和 oc cluster down）

1.  答案是 正确

# 第六章

1.  答案是 1 和 4（主节点和 etcd）

1.  答案是 3（infra）

1.  答案是 2（playbooks/byo/config.yml）

# 第七章

1.  答案是 2（用于生产的 MariaDB 数据库）

1.  答案是 1 和 4（NFS 和 GlusterFS）

1.  答案是 3（任何项目）

1.  答案是 2（1950 M）

1.  答案是 2 和 3（Service 和 Endpoint）

# 第八章

1.  答案是 1 和 4（节点和主机）

1.  答案是 2 和 5（管理员用户和服务用户）

1.  答案是 正确

1.  答案是 2（Route）

1.  答案是 1 和 4（oc 获取 pod 和 oc 获取路由）

1.  答案是 错误

# 第九章

1.  答案是 1 和 4（为了保护应用程序免于在镜像指向的镜像发生更改时意外中断，和 为了在镜像更改时实现自动构建和部署。）

1.  答案是 1 和 3（`oc create configmap my-configmap --from-file=nginx.conf` 和 `oc create -f configmap_definition.yaml`）

1.  答案是 3 和 4（`oc create resourcequota my-quota --hard=cpu=4,services=5` 和 `oc create quota another-quota --hard=pods=8,secrets=4`）

1.  答案是 2 和 4（ConfigMap 和 Service）

1.  答案是 3（${VARIABLE}）

1.  答案是 3（请求）

1.  答案是 3（v2alpha1）

# 第十章

1.  答案是 2 和 4（生成和声明）

1.  答案是 4（默认）

1.  答案是 2 和 4（admin 和 edit）

1.  答案是 3（ProjectRequestLimit）

1.  答案是 1 和 4（anyuid 和 privileged）

1.  答案是 2（数据）

# 第十一章

1.  答案是 4（`vxlan_sys_4789`）

1.  答案是 3（使用 ovs-multitenant 插件，根据需要加入和隔离项目）

1.  答案是 3（来自项目的外部流量的静态 IP）

1.  答案是 3：

```
- type: Allow
 to:
 dnsName: rubygems.org
- type: Allow
 to:
 dnsName: launchpad.net
- type: Deny
 to:
 cidrSelector: 0.0.0.0/0 
```

1.  答案是 5（`web.dev.svc.cluster.local`）

# 第十二章

1.  答案是 2（**Route**）

1.  答案是 1、2、3（Pod、Service、Route）

1.  答案是 3（`oc expose svc httpd --hostname myservice.example.com`）

1.  答案是 1 和 4（oc 获取所有，oc 获取路由）

# 第十三章

1.  答案是 2（openshift）

1.  答案是 1、2、3：

    1.  `oc get template mytemplate -n openshift -o yaml`

    1.  `oc process --parameters -f mytemplate.json`

    1.  `oc describe template mytemplate -n openshift`

1.  答案是 5（以上所有）

# 第十四章

1.  答案是 1, 3（`oc edit bc/redis`，`oc patch bc/redis --patch ...`）

1.  答案是 3 `oc start-build`

1.  答案是 2 `Dockerfile`

# 第十五章

1.  答案是 3, 4（复制控制器、构建配置）

1.  答案是 3（`oc start-build`）

1.  答案是 1, 2, 3:

    1.  `oc status -v`

    1.  `oc status`

    1.  `oc logs build/phpdemo-2`

# 第十六章

1.  答案是 1（buildconfig）

1.  答案是 6（以上所有）

# 第十七章

1.  答案是 2（持续部署）

1.  答案是 1, 2, 4（OpenShift 域特定语言、Jenkins 管道构建策略、Jenkinsfile）

1.  答案是 错

1.  答案是 1（输入）

1.  答案是 1（构建 | 管道）

1.  答案是 错

# 第十八章

1.  答案是 1（虚拟 IP）

1.  答案是 1, 2（虚拟 IP、IP 故障转移）

1.  答案是 对

1.  答案是 1（oc rsync）

1.  答案是 错

# 第十九章

1.  答案是 1（`OpenShift Etcd 键值存储`）

1.  答案是 3（3xOSP - 3xDC）

1.  答案是 对

1.  答案是 1, 2（持久存储复制、应用数据库复制）

# 第二十章

1.  答案是 2（负载均衡器）

1.  答案是 1, 2（故障转移抢占、无状态过滤器）

1.  答案是 错

1.  答案是 **`oc rsync`**

1.  答案是 2（NAT 网关）

# 第二十一章

1.  答案是 1（Docker 替代）

1.  答案是 1, 3（Quay, Tectonik）

1.  答案是 对

1.  答案是 1（CLI 插件）
