---
title: kubernetes dashboard安装
date: 2019-04-13 10:40:50
tags: k8s
categories: 运维
---

* 安装 k8s dashboard
* 改用NodePort方式
* 配置HTTPS证书

------

<!-- more -->

## 安装k8s dashboard

执行以下命令：

```shell
kubectl apply -f http://mirror.faasx.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml

```

## 改用NodePort方式

由于目前k8s测试集群部署在阿里云上，部署在公网上，所以临时使用NodePort方式，可以外网访问。

执行以下命令：

```shell
kubectl -n kube-system edit service kubernetes-dashboard
```

在以下的`yaml`配置文件中，修改`type: ClusterIP`为`type: NodePort`，然后保存：

```yaml
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
...
  name: kubernetes-dashboard
  namespace: kube-system
  resourceVersion: "343478"
  selfLink: /api/v1/namespaces/kube-system/services/kubernetes-dashboard-head
  uid: 8e48f478-993d-11e7-87e0-901b0e532516
spec:
  clusterIP: 10.100.124.90
  externalTrafficPolicy: Cluster
  ports:
  - port: 443
    protocol: TCP
    targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```

然后检查Dashboard是否已经可以访问：

```shell
$ kubectl -n kube-system get service kubernetes-dashboard
NAME                   CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes-dashboard   10.100.124.90   <nodes>       443:31707/TCP   21h
```

如上所示，我们可以在外部通过`http://cluster-ip:31707`进行访问，但是由于证书问题，我们无法访问，我们需要重新制作有效的证书，才可以访问。

## 配置HTTPS证书

通过以下步骤制作证书：

```shell
openssl genrsa -des3 -passout pass:x -out dashboard.pass.key 2048

openssl rsa -passin pass:x -in dashboard.pass.key -out dashboard.key

rm dashboard.pass.key

openssl req -new -key dashboard.key -out dashboard.csr

openssl x509 -req -sha256 -days 365 -in dashboard.csr -signkey dashboard.key -out dashboard.crt

kubectl create secret generic kubernetes-dashboard-certs --from-file=$HOME/certs -n kube-system（如果already exists，先删除，kubectl delete secret kubernetes-dashboard-certs -n kube-system）
```

然后重新配置：

```shell
kubectl apply -f http://mirror.faasx.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```

通过以上配置，我们就可以在外部通过`http://cluster-ip:31707`进行访问。

## 参考链接

<https://www.cnblogs.com/RainingNight/p/deploying-k8s-dashboard-ui.html>

<https://github.com/kubernetes/dashboard/wiki/Accessing-Dashboard---1.7.X-and-above>

<https://github.com/kubernetes/dashboard/wiki/Certificate-management>

<https://github.com/kubernetes/dashboard/wiki/Installation#recommended-setup>