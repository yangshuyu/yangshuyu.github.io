---
layout:     post
title:      kubernetes之ingress controller
subtitle:   kubernetes之ingress controller
date:       2019-11-20
author:     ysy
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - k8s


---

##前言
>当我们想要内部的集群暴露给外部可以访问时，可以使用NodeIP类型的服务把应用暴露给外部用户使用，但是当应用越来越多，以及需要使用七层负载均衡https，NodeIP类型就不适合，所以k8s就提供了一个模块-ingress

#### Ingres与NodeIP对比
NodeIP优缺点

	优点：
	1、结构简单，容易理解
	缺点：
	1、一个app需要占用一个主机端口，管理紊乱，数据较多不适合
	2、L4(TCP/UDP)转发，无法根据http path进行转发

Ingress优缺点

	优点：
	1、可以实现7层识别，了一跟觉url header、path进行转发
	2、方便卸载https
	缺点：
	1、复杂度提升
	2、加了一层解析
	
		

#### Ingress实现
实现
Ingress是一个模块，现在可以实现的方式有很多，例如[ingress-nginx](https://kubernetes.github.io/ingress-nginx/)、[traefik](https://github.com/containous/traefik)，这里是使用的 ingress-nginx， [git地址](https://github.com/kubernetes/ingress-nginx)

##### 第一步 
需要拉取ingress代码，并且进入文件夹 ingress-nginx->deploy->static，最后直接生成该文件夹下的所有配置（先生成命名空间文件），其中包括ingress的命名空间等

```
kubectl apply -f namespace.yaml
kubectl apply -f ./
```
查看一下
kubectl get ns

```
NAME              STATUS   AGE
default           Active   45h
ingress-nginx     Active   16h
kube-node-lease   Active   45h
kube-public       Active   45h
kube-system       Active   45h

```

##### 第二步 创建pods与svc
这里的服务并不是为了通过ingress调用服务，而是ingress通过服务来获取pods的更新（添加-删除-重建），当pods更新时候，service会把更新信息同步到ingress上，由ingress直接找到pods，这样会减少一层service解析

首先，编写pods与service的yaml, deploy-demo.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: myapp
  namespace: default
spec:
  selector:
    app: myapp
    release: canary
  ports:
  - name: http
    targetPort: 80
    port: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      release: canary
  template:
    metadata:
      labels:
        app: myapp
        release: canary
    spec:
      containers:
      - name: myapp
        image: ikubernetes/myapp:v2
        ports:
        - name: http
          containerPort: 80
```
执行yaml文件

```
kubectl apply -f deploy-demo.yaml
```

查看service与pods

```
NAME                             READY   STATUS    RESTARTS   AGE
myapp-deploy-798dc9b584-dq5gd    1/1     Running   0          22h
myapp-deploy-798dc9b584-f9pjw    1/1     Running   0          22h
myapp-deploy-798dc9b584-whnp9    1/1     Running   0          22h

```

```
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP             2d5h
myapp        ClusterIP   10.96.232.187   <none>        80/TCP              22h
```


##### 第三步 添加与pods的server相关联的ingress

首先，编写ingress的yaml

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-myapp
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: myapp.magedu.com
    http:
      paths:
      - path:
        backend:
          serviceName: myapp
          servicePort: 80
```
注意：

1、metadata的annotations一定要标明是nginx，要不然会有问题 

2、ingress的namespace要与所属的pods在同一个

3、host指向的域名是我们刚才pod的域名，一会要在hosts文件指定一下




##### 第四步 创建ingress的svc

因为ingress实际上也相当于一个pod，所以也要有一个service与外部通信

这个官网上有执行的代码

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/provider/baremetal/service-nodeport.yaml
```
也可以下载下来查看一下yaml文件

```
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/provider/baremetal/service-nodeport.yaml
```
文件内容我们增加一个了一个nodePort

```
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  type: NodePort
  ports:
    - name: http
      port: 80
      targetPort: 80
      protocol: TCP
      nodePort: 30080
    - name: https
      port: 443
      targetPort: 443
      protocol: TCP
      nodePort: 30443
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---

```
这里我们使用的namespace是ingress-nginx

并且添加了nodePort为30080与30443

执行一下yaml文件并查看service

```
kubectl get svc -n ingress-nginx

NAME            TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx   NodePort   10.96.252.14   <none>        80:30080/TCP,443:30443/TCP   3d17h
```

##### 第五步 hosts中添加域名并访问

```
172.17.0.1 myapp.magedu.com

curl myapp.magedu.com:30080
Hello MyApp | Version: v2 | <a href="hostname.html">Pod Name</a>

```
这个输出说明ingress-nginx布置成功了


#### Ingress卸载https

当需要使用https来进行通信时，因为iptables和ipvs是工作在4层的，所以没有办法对7层https进行解析，如果需要提供https服务（需要在后端提供服务的pod上配置ssl证书，这样一来在kubernetes内部也是通过https进行访问）就比较麻烦，这个时候就需要用ingress
来卸载https

##### 第一步 生成证书与secret

因为是本地测试，所以可以生成一个简单的证书

```
openssl genrsa -out tls.key 2048

openssl req -new -x509 -key tls.key -out tls.crt -subj /C=CN/ST=BeiJing/L=BeiJing/O=DevOps/CN=tomcat-magedu.com
```
注意：这里的CN一定要与我们要访问的域名相同

生成完证书之后，生成一个secret

```
kubectl create secret tls tomcat-ingress-secret --cert=tls.crt --key=tls.key
```
可以查看secret

```
kubectl get secret

NAME                    TYPE                                  DATA   AGE
default-token-vmbjv     kubernetes.io/service-account-token   3      5d23h
tomcat-ingress-secret   kubernetes.io/tls                     2      125m
```

###### 第二步 生成带有rules的ingress

编写yaml

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-tomcat-tls
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  tls:
  - hosts:
    - tomcat.magedu.com
    secretName: tomcat-ingress-secret
  rules:
  - host: tomcat.magedu.com
    http:
      paths:
      - path:
        backend:
          serviceName: tomcat
          servicePort: 8080
```
rules包括hosts列表和刚才生成的sccret名称

然后生成该ingress

```
kubectl create -f ingress-tomcat-tls.yaml
```
因为我们前面的ingress的svc指定的30443端口会请求443端口，所以我们访问一下


```
curl https://tomcat.magedu.com:30443 --insecure
```
因为我们的证书不受信任，所以加上--insecure，最后访问成功，设置成功。