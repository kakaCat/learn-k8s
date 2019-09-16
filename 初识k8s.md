# 1.概述

## 1.1 编写目的

本文介绍k8s单机版部署步骤、运行步骤。

## 1.2名词解释

## 1.3 参考资料

<https://www.cnblogs.com/spll/p/10075781.html>

<https://www.58jb.com/html/152.html>

<https://www.cnblogs.com/berry-ma/p/9265871.html>

<https://www.jianshu.com/p/d27141e18398>

# 2 安装部署

## 2.1 硬件环境

阿里云 内存2g ，硬盘40g

## 2.2 软件环境

centos7 64位系统

## 2.3安装部署

### 2.3.1 准备工作

#### 2.3.1.1 yum 换阿里源

- 备份原有的源

  mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

- 下载阿里的源（主要centos版本，本文用centos7 下载centos7源）

  wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

- 运行yum makecache生成缓存

  yum clean all

  yum makecache

#### 2.3.1.2 docker 1.13.1 换阿里源

- /etc/docker/daemon.json

  ```
  {
  "registry-mirrors": [
  "https://kfwkfulq.mirror.aliyuncs.com",
  "https://2lqq34jg.mirror.aliyuncs.com",
  "https://pee6w651.mirror.aliyuncs.com",
  "https://registry.docker-cn.com",
  "http://hub-mirror.c.163.com"
  ],
  "dns": ["8.8.8.8","8.8.4.4"]
  }
  ```

#### 2.3.1.3 下载ca授权

- 下载ca授权

  wget http://mirror.centos.org/centos/7/os/x86_64/Packages/python-rhsm-certificates-1.19.10-1.el7_4.x86_64.rpm

  

  rpm2cpio python-rhsm-certificates-1.19.10-1.el7_4.x86_64.rpm | cpio -iv --to-stdout ./etc/rhsm/ca/redhat-uep.pem | tee /etc/rhsm/ca/redhat-uep.pem

### 2.3.2 关闭防火墙

- 开机启动有效

  systemctl disable firewalld

- 关闭防火墙

   systemctl stop firewalld 

### 2.3.3 docker etcd kubernets 配置及其启动

- 下载软件

  yum install -y etcd kubernetes docker

- 修改docker 配置

  Docker配置文件/etc/sysconfig/docker, OPTIONS=’--selinux-enabled=false --insecure-registry gcr.io’

- 修改apiservce

  Kubernetes apiservce配置文件/etc/kubernetes/apiserver,把–admission_control参数钟的ServiceAccount删除

- 软件启动

  systemctl start etcd

  systemctl start docker

  systemctl start kube-apiserver

  systemctl start kube-controller-manager

  systemctl start kube-scheduler

  systemctl start kubelet

  systemctl start kube-proxy

### 2.3.4 验证

- docker 验证

  docker image

- Kubernetes 验证

  kubectl get pods -o wide 

# 3 通过k8s部署软件

## 3.1 nginx

- nginx-deployment.yaml

  ```
  apiVersion: extensions/v1beta1
  kind: Deployment
  metadata:
    name: nginx-deployment
    labels:
      app: nginx
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx:1.7.9
          ports:
          - containerPort: 80
  ```

- 创建pod

  ```
  kubectl create -f nginx-deployment.yaml
  ```

- 验证

  kubectl get pod 

- nginx-service.yaml 

  ```shell
  piVersion: v1
  kind: Service
  metadata:
    name: nginx-service
    labels:
      app: nginx
  spec:
    ports:
    - port: 88
      targetPort: 80
    selector:
      app: nginx
    type: NodePort
  ```

- 创建service

  ```
  kubectl create -f nginx-service.yaml 
  ```

- 验证

  kubectl get service

- 查看映射端口号 访问IP:

## 3.2 tomcat

- kube-tomcat-dep.yaml

  ```shell
  apiVersion: v1
  kind: ReplicationController
  metadata:
    name: tomcat-demo
  spec:
    replicas: 1
    selector:
      app: tomcat-demo
    template:
      metadata:
      labels:
        app: tomcat-demo
      spec:
        containers:
        - name: tomcat-demo
          image: tomcat
          ports:
          - containerPort: 8080
  ```

- 创建pod

  ```shell
  kubectl create -f kube-tomcat-dep.yaml
  ```

- 验证

  kubectl get pod 

- kube-tomcat-svc.yaml

  ```shell
  apiVersion: v1
  kind: Service
  metadata:
    name: tomcat-demo
  spec:
    type: NodePort
    ports:
     - port: 8080
       nodePort: 30001
    selector:
      app: tomcat-demo
  ```

- 创建service

  ```cpp
  kubectl create -f tomcat-demo-svc.yaml
  ```

- 验证

  kubectl get service

- 访问 ip:30001

# 4 k8s常用命令

## 4.1 基础命令

kubectl get po # 查看目前所有的pod
kubectl get rs # 查看目前所有的replica set
kubectl get deployment # 查看目前所有的deployment
kubectl describe po ｛app:name｝ # 查看指定pod的详细状态
kubectl describe rs ｛app:name｝ # 查看指定pod replica set的详细状态
kubectl describe deployment ｛app:name｝ # 查看指定pod deployment的详细状态
