---
layout:     post
title:      kubernetes之存储券
subtitle:   kubernetes之存储券
date:       2020-03-19
author:     ysy
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - k8s


---

##前言
>容器中的文件在磁盘上是临时存放的，这就有一些问题，当容器崩溃是，将重新启动容器，这时上一个容器中的文件在新容器中就会丢失，另外，会存在多容器共享文件，所以k8s提供了存储券(Volume)这个概念


#### 常见的存储券以及使用方法
k8s支持大量的存储类型，包括nfs、aws、gitRepo(关联git地址)、emptyDir等，根绝类型不同，使用场景也不近相同，这个笔记记录怎样使用以及常用的几个存储类型。

##### 使用方法
```
spec.volumes：通过此字段提供指定的存储卷
spec.containers.volumeMounts：通过此字段将存储卷挂接到容器中
```

##### emptyDir的使用与介绍
emptyDir 券存储在支持该节点所使用的介质上，这里的介质可以是磁盘或SSD或网络存储，但是这里也可以通过emptyDir.medium 字段设置为 "Memory"来用作缓存，emptyDir在pod被清除掉时，emptyDir中数据也会被清除。

示例

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-demo
  namespace: default
  labels:
    app: myapp
    tier: frontend
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    ports:
    - name: http
      containerPort: 80
    volumeMounts:
    - name: html
      mountPath: /data/web/html/
  - name: my-busybox
    image: busybox:latest
    volumeMounts:
    - name: html
      mountPath: /data/
    command:
    - "/bin/sh"
    - "-c"
    - "sleep 7200"
  volumes:
  - name: html
    emptyDir: {}
```
这里通过volumes创建了一个为emptyDir的存储券，并在一个pod中的两个容器中同时通过volumeMounts挂在在各自的不同目录下，当一个容器中的目录改变时，查看另一个容器时，发现也会改变

 
##### hostPath的使用与介绍
hostPath类型的存储卷用于将宿主机的文件系统的文件或目录挂接到Pod中，除了需要指定path字段之外，在使用hostPath类型的存储卷时，也可以设置type，type包括DirectoryOrCreate(目录不存在则会创建)、Directory(指定的目录必须存在)

注意：

- 具有相同配置（例如从 podTemplate 创建）的多个 Pod 会由于节点上文件的不同而在不同节点上有不同的行为。

- 当 Kubernetes 按照计划添加资源感知的调度时，这类调度机制将无法考虑由 hostPath 使用的资源。

- 基础主机上创建的文件或目录只能由 root 用户写入。您需要在 特权容器 中以 root 身份运行进程，或者修改主机上的文件权限以便容器能够写入 hostPath 卷。

示例

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-vol-hostpath
  namespace: default
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html/
  volumes:
  - name: html
    hostPath:
      path: /data/pod/volume1
      type: DirectoryOrCreate
```

##### nfs的使用与介绍

在Kubernetes中，可以通过nfs类型的存储卷将现有的NFS（网络文件系统）到的挂接到Pod中。这意味着可以利用nfs来将数据持久化，并且可以在Pod之间共享数据。NFS可以被同时挂接到多个Pod中，并能同时进行写入。

使用nfs首先要有nfs服务器，如果没有可以先部署一台。

首先安装server

```
$ sudo apt-get update
$ sudo apt-get install -y nfs-kernel-server
```
然后配置文件

```
vim /etc/exports

/data/volumes *(insecure,rw,no_root_squash)
```
这里的/data/volumes目录是共享的目录

最后重启服务器

```
$ sudo /etc/init.d/rpcbind restart
$ sudo /etc/init.d/nfs-kernel-server restart
```
可以在pod服务器上查看一下是否能连接到nfs(我的nfs服务器地址是10.122.104.165)

```
sudo showmount -e 10.122.104.165

Export list for 10.122.104.165:
/data/volumes *
```
说明可以链接

可以尝试挂载一下

```
sudo mount -t nfs 10.122.104.165:/data/volumes /mnt
```
nfs服务器搭建好之后就要编写yaml来实现了，与hostpath不同，nfs需要指定一个server，也就是nfs的server

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-vol-nfs
  namespace: default
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html/
  volumes:
  - name: html
    nfs:
      path: /data/volumes
      server: 10.122.104.165
```

创建完pod之后，通过在/data/volumes下创建index.html，然后访问myapp的ip，发现访问的网页是刚修改的网页，说明nfs存储券使用成功。

##### pv与pvc
PersistentVolume（PV）是集群之中的一块网络存储。跟 Node 一样，也是集群的资源。PV 跟 Volume (卷) 类似，不过会有独立于 Pod 的生命周期。这一 API 对象包含了存储的实现细节，例如 NFS、iSCSI 或者其他的云提供商的存储系统。

PersistentVolumeClaim (PVC) 是用户的一个请求。他跟 Pod 类似。Pod 消费 Node 的资源，PVCs 消费 PV 的资源。Pod 能够申请特定的资源（CPU 和 内存）；Claim 能够请求特定的尺寸和访问模式（例如可以加载一个读写，以及多个只读实例）

pv是集群的资源，pvc是对这一资源的请求。pv和pvc之间的互动遵循了一套生命周期系统

供应

集群管理员会创建一系列的 PV。这些 PV 包含了为集群用户提供的真实存储资源。他们可利用 Kubernetes API 来消费。

绑定

用户创建一个包含了容量和访问模式的持久卷申请。Master 会监听 PVC 的产生，并尝试根据请求内容查找匹配的 PV，并把 PV 和 PVC 进行绑定。用户能够获取满足需要的资源，并且在使用过程中可能超出请求数量。

如果找不到合适的卷，这一申请就会持续处于非绑定状态，一直到出现合适的 PV。例如一个集群准备了很多的 50G 大小的持久卷，（虽然总量足够）也是无法响应 100G 的申请的，除非把 100G 的 PV 加入集群。

使用

Pod 把申请作为卷来使用。集群会通过 PVC 查找绑定的 PV，并 Mount 给 Pod。对于支持多种访问方式的卷，用户在使用 PVC 作为卷的时候，可以指定需要的访问方式。

一旦用户拥有了一个已经绑定的 PVC，被绑定的 PV 就归该用户所有了。用户的 Pods 能够通过在 Pod 的卷中包含的 PVC 来访问他们占有的 PV。

释放

当用户完成对卷的使用时，就可以利用 API 删除 PVC 对象了，而且他还可以重新申请。删除 PVC 后，对应的卷被视为 “被释放”，但是这时还不能给其他的 PVC 使用。之前的 PVC 数据还保存在卷中，要根据策略来进行后续处理。

回收

PV 的回收策略向集群阐述了在 PVC 释放卷的时候，应如何进行后续工作。目前可以采用三种策略：保留，回收或者删除。保留策略允许重新申请这一资源。在持久卷能够支持的情况下，删除策略会同时删除持久卷以及 AWS EBS/GCE PD 或者 Cinder 卷中的存储内容。如果插件能够支持，回收策略会执行基础的擦除操作（rm -rf /thevolume/*），这一卷就能被重新申请了。

Recycling Policy（回收策略）
当前的回收策略可选值包括：

Retain - 人工重新申请

Recycle - 基础擦除（“rm -rf /thevolume/*”）

Delete - 相关的存储资产例如 AWS EBS，GCE PD 或者 OpenStack Cinder 卷一并删除。
目前，只有 NFS 和 HostPath 支持 Recycle 策略，AWS EBS、GCE PD 以及 Cinder 卷支持 Delete 策略（*其他的都是 Retain 是吧。。*）。

阶段（Phase）

一个卷会处于如下阶段之一：

Available：可用资源，尚未被绑定到 PVC 上
Bound：该卷已经被绑定
Released：PVC 已经被删除，但该资源尚未被集群回收
Failed：该卷的自动回收过程失败。

那么怎么实现一个pv与pvc那

首先创建几个pv，就用我们刚才搭建的nfs服务器

```
cd /data/volumes/
mkdir v{1,2,3}
```
首先窗扇3个文件夹，然后更新我们的/etc/exports文件

```
/data/volumes/v1 *(insecure,rw,no_root_squash)
/data/volumes/v2 *(insecure,rw,no_root_squash)
/data/volumes/v3 *(insecure,rw,no_root_squash)
```
然后在master上创建三个pv

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv001
  labels:
    name: pv001
spec:
  nfs:
    path: /data/volumes/v1
    server: 10.122.104.165
  accessModes: ["ReadWriteMany", "ReadWriteOnce"]
  capacity:
    storage: 2Gi

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv002
  labels:
    name: pv002
spec:
  nfs:
    path: /data/volumes/v2
    server: 10.122.104.165
  accessModes: ["ReadWriteMany", "ReadWriteOnce"]
  capacity:
    storage: 2Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv003
  labels:
    name: pv003
spec:
  nfs:
    path: /data/volumes/v3
    server: 10.122.104.165
  accessModes: ["ReadWriteMany", "ReadWriteOnce"]
  capacity:
    storage: 2Gi
```

然后创建、查看pv

kubectl get pv

```
pv001   2Gi        RWO,RWX        Retain           Bound       default/mypvc                           9d
pv002   2Gi        RWO,RWX        Retain           Available                                           9d
pv003   2Gi        RWO,RWX        Retain           Available                                           9d
```

然后创建pvc

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
  namespace: default
spec:
  accessModes: ["ReadWriteMany"]
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-vol-pvc
  namespace: default
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html/
  volumes:
  - name: html
    persistentVolumeClaim:
      claimName: mypvc
```
然后部署，部署之后我们通过查看pv就回发现其中一个符合尺寸大小的pv已经被占用

```
NAME    CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM           STORAGECLASS   REASON   AGE
pv001   2Gi        RWO,RWX        Retain           Bound       default/mypvc                           9d

```
状态是bound，说明已经绑定成功。