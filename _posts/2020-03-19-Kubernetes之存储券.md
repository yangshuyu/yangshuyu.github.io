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


