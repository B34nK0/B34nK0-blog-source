---
title: 初始Kubernetes
date: 2022-01-26 09:55:40
categories: 运维
tags: kebernetes
---

# 概要
Kubernetes 是一个可移植的、可扩展的开源平台，用于管理容器化的工作负载和服务，可促进声明式配置和自动化.

<!--more-->  

## Kubernetes部署优势

### 服务发现和负载均衡

    Kubernetes 可以使用 DNS 名称或自己的 IP 地址公开容器，如果进入容器的流量很大， Kubernetes 可以负载均衡并分配网络流量，从而使部署稳定。

### 存储编排

    Kubernetes 允许你自动挂载你选择的存储系统，例如本地存储、公共云提供商等。

### 自动部署和回滚

    你可以使用 Kubernetes 描述已部署容器的所需状态，它可以以受控的速率将实际状态 更改为期望状态。例如，你可以自动化 Kubernetes 来为你的部署创建新容器， 删除现有容器并将它们的所有资源用于新容器。

### 自动完成装箱计算

    Kubernetes 允许你指定每个容器所需 CPU 和内存（RAM）。 当容器指定了资源请求时，Kubernetes 可以做出更好的决策来管理容器的资源。

### 自我修复

    Kubernetes 重新启动失败的容器、替换容器、杀死不响应用户定义的 运行状况检查的容器，并且在准备好服务之前不将其通告给客户端。

### 密钥与配置管理

    Kubernetes 允许你存储和管理敏感信息，例如密码、OAuth 令牌和 ssh 密钥。 你可以在不重建容器镜像的情况下部署和更新密钥和应用程序配置，也无需在堆栈配置中暴露密钥。

# Kubernetes集群

![](Kubernetes集群组件.svg)

这里先不详细介绍各个组件的功能，先从简单应用场景入手把K8s用起来

## 部署场景

使用 Kebuctl命令行方式发布java微服务

1、配置kubernetes.yaml
```shell
apiVersion: apps/v1
kind: Deployment
metadata:
  name: system-deployment
  labels:
    app: system
spec:
  selector:
    matchLabels:
      app: system
  template:
    metadata:
      labels:
        app: system
    spec:
      containers:
      - name: system-container
        image: system:1.0-SNAPSHOT
        ports:
        - containerPort: 9080
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 9080
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 3
          failureThreshold: 1
---
apiVersion: v1
kind: Service
metadata:
  name: system-service
spec:
  type: NodePort
  selector:
    app: system
  ports:
  - protocol: TCP
    port: 9080
    targetPort: 9080
    nodePort: 31000
```

在使用kubectl发布时，实际是将yaml转成json，所以也可以通过k8s的api发送json方式来发布。
在yaml配置中，我们设置了要发布的微服务system，其监听端口是9080，而在设置node端口映射时，node端口是31000。通过不同的对外端口可以实现api（服务）版本控制，即灰度发布。

2、发布

```shell
$ kubectl apply -f kubernetes.yaml

deployment.apps/system-deployment created
service/system-service created
```

可以看到我们成功的发布了system-service服务

3、查看服务的运行状态
```shell
$ kubectl wait --for=condition=ready pod -l app=system
pod/system-deployment-85b9978c95-7kf4l condition met
```

当我们看到condition met 说明服务已经是reading状态，可以接收我们的请求

4、测试服务
```shell
$ curl http://$( minikube ip ):31000/system/properties
```
我们使用curl来发送请求，这里的31000端口不是服务监听的端口，而是kubernetes.yaml配置的pod对外端口映射

5、查看服务状态
```shell
$ kubectl get --watch pods

NAME                                    READY   STATUS    RESTARTS   AGE
inventory-deployment-67ffcfc8f6-j4w28   1/1     Running   0          30m
kubernetes-bootcamp-fb5c67579-hh789     1/1     Running   0          50m
system-deployment-85b9978c95-7kf4l      1/1     Running   0          30m
```