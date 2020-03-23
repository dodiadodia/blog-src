---
title: "Kube Plugin"
date: 2020-03-20T16:44:40+08:00
draft: false
tags: ["kubectl", "kubernetes"]
isCJKLanguage: true
---

# Kubectl几个比较有用的插件分享

## 介绍

kubectl命令支持插件扩展，具体介绍可以参考下面的链接

https://kubernetes.io/docs/tasks/extend-kubectl/kubectl-plugins/

kubectl的plugin就是一个可执行文件，以kubectl-\*方式命令，放置在PATH路径中，kubectl plugin list的时候就会在PATH路径下递归寻找相应的kubect-\*文件，作为其plugin输出。

github上的这个项目https://github.com/ishantanu/awesome-kubectl-plugins 分享了很多kubectl的plugin，我试用了其中的几个，现在把我觉得有用的几个plugin记录一下

## kubectl krew介绍

kubectl krew是第一个介绍的，其实他就是一个plugin的管理器，和mac上的brew类似，用于管理kubectl插件的，其github地址如下https://github.com/kubernetes-sigs/krew

直接使用下面的代码安装kubectl

```sh
(
  set -x; cd "$(mktemp -d)" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/krew.{tar.gz,yaml}" &&
  tar zxvf krew.tar.gz &&
  KREW=./krew-"$(uname | tr '[:upper:]' '[:lower:]')_amd64" &&
  "$KREW" install --manifest=krew.yaml --archive=krew.tar.gz &&
  "$KREW" update
)
```

然后把kubectl-krew的执行命令所在位置添加到PATH目录，如下

```sh
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
```

其实上面的工作很简单，就是从github的release上下载kubectl-krew的压缩包并解压，然后放置到PATH目录中。这样执行kubectl krew的时候就等于是执行kubectl-krew了，kubectl-krew内部记录了索引，会对plugin的整个生命周期进行管理，比如list search install uninstall等等操作，和brew的理念完全一致。然后通过kubectl krew install就可以安装需要的插件了*（有些插件krew安装的版本比github上还新，有些不太理解）*

下面罗列出一些使用示例：

```sh
[root@dodia1 ~]# kubectl krew list
PLUGIN  VERSION
krew    v0.3.4
```

```sh
[root@dodia1 ~]# kubectl krew search debug
NAME         DESCRIPTION                                      INSTALLED
debug        Attach ephemeral debug container to running pod  no
debug-shell  Create pod with interactive kube-shell.          no
```

```sh
[root@dodia1 ~]# kubectl krew install doctor
Updated the local copy of plugin index.
  New plugins available:
    * rolesum
Installing plugin: doctor
Installed plugin: doctor
\
 | Use this plugin:
 |      kubectl doctor
 | Documentation:
 |      https://github.com/emirozer/kubectl-doctor
 | Caveats:
 | \
 |  | This plugin needs higher privileges on core API group.
 |  | Potentially a ClusterRole that can get cluster-scoped resources.
 |  | Such as nodes / all namespaces etc.
 | /
/
WARNING: You installed plugin "doctor" from the krew-index plugin repository.
   These plugins are not audited for security by the Krew maintainers.
   Run them at your own risk.
```

## kubectl-debug介绍

这个插件经常使用，用于调试k8s环境中的pod，github里这样的项目有好几个，我现在使用的是下面这个项目

https://github.com/aylei/kubectl-debug

其余的项目包括：https://github.com/verb/kubectl-debug

其官网描述如下：

> kubectl-debug 是一个简单的 kubectl 插件, 能够帮助你便捷地进行 Kubernetes 上的 Pod 排障诊断. 背后做的事情很简单: 在运行中的 Pod 上额外起一个新容器, 并将新容器加入到目标容器的 pid, network, user 以及 ipc namespace 中, 这时我们就可以在新容器中直接用 netstat, tcpdump 这些熟悉的工具来解决问题了, 而旧容器可以保持最小化, 不需要预装任何额外的排障工具.

## kubect-pod_dive介绍

另一个我常用的插件就是pod_dive了，这个插件可以直接显示pod资源的树状图

```sh
Air-for-Dodia:~ wenjuntang$ kubectl pod_dive test-nginx-nginx-6c75c78459-5qdl8
[node]      node3 [ready]
[namespace]  ├─┬ default
[type]       │ └─┬ replicaset [deployment]
[workload]   │   └─┬ test-nginx-nginx-6c75c78459 [1 replica]
[pod]        │     └─┬ test-nginx-nginx-6c75c78459-5qdl8 [running]
[containers] │       └── nginx [0 restarts]
            ... 
[siblings]   ├── csi-rbdplugin-k8dk6
             ├── csi-rbdplugin-provisioner-844f8568b4-gjw9s
             ├── csi-rbdplugin-provisioner-844f8568b4-smk2f
             ├── cephfs-provisioner-75545979d8-b25nd
             ├── debug-agent-zzb87
             ├── harbor1-harbor-chartmuseum-557c74b5cb-c8gfg
             ├── harbor1-harbor-core-76b687cd7b-8q99p
             ├── harbor1-harbor-notary-server-8888898c9-tf52w
             ├── harbor1-harbor-registry-567468c485-pbl7g
             ├── jira7-1
             ├── jira7-recovered-1
             ├── jira7-recovered-4
             ├── jira7-recovered-6
             ├── testmemcache-memcached-1
             ├── tomcatest1-85b9df7d58-77xwt
             ├── wangxi-deployment-deployment-1-wangxi-deployment-6c94cd59-dpwpg
             ├── wx-etcd1-1
             ├── ingress-nginx-controller-wg9cl
             ├── appmanager-harbor-79ff98fcd4-kmtkq
             ├── auth-server-kubedb-recovered-0
             ├── calico-node-85dpd
             ├── coredns-8956959d7-kxwfn
             ├── dns-autoscaler-6db59696c5-vh2cd
             ├── dnsmasq-outer-autoscaler-5cf4cdc5f4-qgqw9
             ├── dnsmasq-outer-cf8966c78-rv5hd
             ├── kube-apiserver-node3
             ├── kube-controller-manager-node3
             ├── kube-proxy-zhvcn
             ├── kube-scheduler-node3
             ├── local-volume-provisioner-p9nbv
             ├── mkdir-zfbqr
             ├── nodelocaldns-n7fp7
             ├── elasticsearch-logging-2
             ├── fluentd-es-v2.8.0-vvjtm
             ├── alertmanager-6cbf699fbd-spmz2
             ├── ceph-exporter-dd9658889-m5m8t
             ├── kube-state-metrics-deployment-54d88bbcfc-rdwj4
             ├── monitoring-disk-4mhgw
             ├── monitoring-influxdb-cc5fdb46-lwhdr
             ├── monitoring-smart-rdwsn
             ├── node-directory-size-metrics-8mk76
             ├── prometheus-node-exporter-svfn8
             ├── remote-storage-adapter-dfb9557d6-g72x2
             └── registry-proxy-nxvs9
```

