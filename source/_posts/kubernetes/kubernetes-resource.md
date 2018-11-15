---
title: "[Kubernetes] kubernetes资源配额详解"
catalog: true
date: 2018-6-24 15:22:24
type: "tags"
subtitle:
header-img: "https://res.cloudinary.com/dqxtn0ick/image/upload/v1542285471/header/building.jpg"
tags:
- Kubernetes
catagories:
- Kubernetes
---

## 1. 资源配额（ResourceQuota）

`ResourceQuota`对象用来定义某个命名空间下所有资源的使用限额，其实包括：

- 计算资源的配额
- 存储资源的配额
- 对象数量的配额

如果集群的总容量小于命名空间的配额总额，可能会产生资源竞争。这时会按照先到先得来处理。
资源竞争和配额的更新都不会影响已经创建好的资源。

### 1.1. 启动资源配额

Kubernetes 的众多发行版本默认开启了资源配额的支持。当在apiserver的`--admission-control`配置中添加`ResourceQuota`参数后，便启用了。 当一个命名空间中含有`ResourceQuota`对象时，资源配额将强制执行。

### 1.2. 计算资源配额

可以在给定的命名空间中限制可以请求的计算资源（[compute resources](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/)）的总量。

| 资源名称        | 描述                                          |
| --------------- | --------------------------------------------- |
| cpu             | 非终止态的所有pod, cpu请求总量不能超出此值。  |
| limits.cpu      | 非终止态的所有pod， cpu限制总量不能超出此值。 |
| limits.memory   | 非终止态的所有pod, 内存限制总量不能超出此值。 |
| memory          | 非终止态的所有pod, 内存请求总量不能超出此值。 |
| requests.cpu    | 非终止态的所有pod, cpu请求总量不能超出此值。  |
| requests.memory | 非终止态的所有pod, 内存请求总量不能超出此值。 |

### 1.3. 存储资源配额

可以在给定的命名空间中限制可以请求的存储资源（[storage resources](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)）的总量。

| 资源名称                                            | 描述                                                         |
| --------------------------------------------------- | ------------------------------------------------------------ |
| requests.storage                                    | 所有PVC, 存储请求总量不能超出此值。                          |
| persistentvolumeclaims                              | 命名空间中可以存在的PVC（[persistent volume claims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)）总数。 |
| .storageclass.storage.k8s.io/requests.storage       | 和该存储类关联的所有PVC, 存储请求总和不能超出此值。          |
| .storageclass.storage.k8s.io/persistentvolumeclaims | 和该存储类关联的所有PVC，命名空间中可以存在的PVC（[persistent volume claims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)）总数。 |

### 1.4. 对象数量的配额

| 资源名称               | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| congfigmaps            | 命名空间中可以存在的配置映射的总数。                         |
| persistentvolumeclaims | 命名空间中可以存在的PVC总数。                                |
| pods                   | 命名空间中可以存在的非终止态的pod总数。如果一个pod的`status.phase` 是 `Failed, Succeeded`, 则该pod处于终止态。 |
| replicationcontrollers | 命名空间中可以存在的`rc`总数。                               |
| resourcequotas         | 命名空间中可以存在的资源配额（[resource quotas](https://kubernetes.io/docs/admin/admission-controllers/#resourcequota)）总数。 |
| services               | 命名空间中可以存在的服务总数量。                             |
| services.loadbalancers | 命名空间中可以存在的服务的负载均衡的总数量。                 |
| services.nodeports     | 命名空间中可以存在的服务的主机接口的总数量。                 |
| secrets                | 命名空间中可以存在的`secrets`的总数量。                      |

例如：可以定义pod的限额来避免某用户消耗过多的Pod IPs。

### 1.5. 限额的作用域

| 作用域         | 描述                                           |
| -------------- | ---------------------------------------------- |
| Terminating    | 匹配 `spec.activeDeadlineSeconds >= 0` 的pod   |
| NotTerminating | 匹配 `spec.activeDeadlineSeconds is nil` 的pod |
| BestEffort     | 匹配具有最佳服务质量的pod                      |
| NotBestEffort  | 匹配具有非最佳服务质量的pod                    |

### 1.6. request和limit

当分配计算资源时，每个容器可以为cpu或者内存指定一个请求值和一个限度值。可以配置限额值来限制它们中的任何一个值。
如果指定了`requests.cpu` 或者 `requests.memory`的限额值，那么就要求传入的每一个容器显式的指定这些资源的请求。如果指定了`limits.cpu`或者`limits.memory`，那么就要求传入的每一个容器显式的指定这些资源的限度。

### 1.7. 查看和设置配额

```shell
# 创建namespace
$ kubectl create namespace myspace

# 创建resourcequota
$ cat <<EOF > compute-resources.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
spec:
  hard:
    pods: "4"
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
EOF
$ kubectl create -f ./compute-resources.yaml --namespace=myspace

# 查询resourcequota
$ kubectl get quota --namespace=myspace
NAME                    AGE
compute-resources       30s

# 查询resourcequota的详细信息
$ kubectl describe quota compute-resources --namespace=myspace
Name:                  compute-resources
Namespace:             myspace
Resource               Used Hard
--------               ---- ----
limits.cpu             0    2
limits.memory          0    2Gi
pods                   0    4
requests.cpu           0    1
requests.memory        0    1Gi
```

### 1.8. 配额和集群容量

资源配额对象与集群容量无关，它们以绝对单位表示。即增加节点的资源并不会增加已经配置的namespace的资源。

## 2. Pod限额（LimitRange） 

> `ResourceQuota`对象是限制某个namespace下`所有Pod(容器)`的资源限额
>
> `LimitRange`对象是限制某个namespace`单个Pod(容器`)的资源限额

`LimitRange`对象用来定义某个`命名空间`下某种`资源对象`的使用限额，其中资源对象包括：`Pod`、`Container`、`PersistentVolumeClaim`。

### 2.1. 为namespace配置CPU和内存的默认值

如果在一个拥有默认内存或CPU限额的命名空间中创建一个容器，并且这个容器未指定它自己的内存或CPU的`limit`， 它会被分配这个默认的内存或CPU的`limit`。既没有设置pod的`limit`和`request`才会分配默认的内存或CPU的`request`。

#### 2.1.1. namespace的内存默认值

```shell
# 创建namespace
$ kubectl create namespace default-mem-example

# 创建LimitRange
$ cat memory-defaults.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
  
$ kubectl create -f https://k8s.io/docs/tasks/administer-cluster/memory-defaults.yaml --namespace=default-mem-example

# 创建Pod,未指定内存的limit和request
$ cat memory-defaults-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: default-mem-demo
spec:
  containers:
  - name: default-mem-demo-ctr
    image: nginx
    
$ kubectl create -f https://k8s.io/docs/tasks/administer-cluster/memory-defaults-pod.yaml --namespace=default-mem-example

# 查看Pod
$ kubectl get pod default-mem-demo --output=yaml --namespace=default-mem-example
containers:
- image: nginx
  imagePullPolicy: Always
  name: default-mem-demo-ctr
  resources:
    limits:
      memory: 512Mi
    requests:
      memory: 256Mi
```

#### 2.1.2. namespace的CPU默认值

```shell
# 创建namespace
$ kubectl create namespace default-cpu-example

# 创建LimitRange
$ cat cpu-defaults.yaml 
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
  - default:
      cpu: 1
    defaultRequest:
      cpu: 0.5
    type: Container
    
$ kubectl create -f https://k8s.io/docs/tasks/administer-cluster/cpu-defaults.yaml --namespace=default-cpu-example    

# 创建Pod，未指定CPU的limit和request
$ cat cpu-defaults-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: default-cpu-demo
spec:
  containers:
  - name: default-cpu-demo-ctr
    image: nginx

$ kubectl create -f https://k8s.io/docs/tasks/administer-cluster/cpu-defaults-pod.yaml --namespace=default-cpu-example

# 查看Pod
$ kubectl get pod default-cpu-demo --output=yaml --namespace=default-cpu-example
containers:
- image: nginx
  imagePullPolicy: Always
  name: default-cpu-demo-ctr
  resources:
    limits:
      cpu: "1"
    requests:
      cpu: 500m
```

#### 2.1.3 说明

1. 如果没有指定pod的`request`和`limit`，则创建的pod会使用`LimitRange`对象定义的默认值（request和limit）
2. 如果指定pod的`limit`但未指定`request`，则创建的pod的`request`值会取`limit`的值，而不会取LimitRange对象定义的request默认值。
3. 如果指定pod的`request`但未指定`limit`，则创建的pod的`limit`值会取`LimitRange`对象定义的`limit`默认值。

**默认Limit和request的动机**

如果命名空间具有`资源配额（ResourceQuota）`, 它为内存限额（CPU限额）设置默认值是有意义的。 以下是资源配额对命名空间施加的两个限制：

- 在命名空间运行的每一个容器必须有它自己的内存限额（CPU限额）。
- 在命名空间中所有的容器使用的内存总量（CPU总量）不能超出指定的限额。

如果一个容器没有指定它自己的内存限额（CPU限额），它将被赋予默认的限额值，然后它才可以在被配额限制的命名空间中运行。

### 2.2. 为namespace配置CPU和内存的最大最小值

#### 2.2.1. 内存的最大最小值

**创建LimitRange**

```shell
# 创建namespace
$ kubectl create namespace constraints-mem-example

# 创建LimitRange
$ cat memory-constraints.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-min-max-demo-lr
spec:
  limits:
  - max:
      memory: 1Gi
    min:
      memory: 500Mi
    type: Container
 
$ kubectl create -f https://k8s.io/docs/tasks/administer-cluster/memory-constraints.yaml --namespace=constraints-mem-example

# 查看LimitRange
$ kubectl get limitrange cpu-min-max-demo --namespace=constraints-mem-example --output=yaml
...
  limits:
  - default:
      memory: 1Gi
    defaultRequest:
      memory: 1Gi
    max:
      memory: 1Gi
    min:
      memory: 500Mi
    type: Container
...
# LimitRange设置了最大最小值，但没有设置默认值，也会被自动设置默认值。
```

**创建符合要求的Pod**

```shell
# 创建符合要求的Pod
$ cat memory-constraints-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: constraints-mem-demo
spec:
  containers:
  - name: constraints-mem-demo-ctr
    image: nginx
    resources:
      limits:
        memory: "800Mi"
      requests:
        memory: "600Mi"
 
$ kubectl create -f https://k8s.io/docs/tasks/administer-cluster/memory-constraints-pod.yaml --namespace=constraints-mem-example

# 查看Pod
$ kubectl get pod constraints-mem-demo --output=yaml --namespace=constraints-mem-example
...
resources:
  limits:
     memory: 800Mi
  requests:
    memory: 600Mi
...
```

**创建超过最大内存limit的pod**

```shell
$ cat memory-constraints-pod-2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: constraints-mem-demo-2
spec:
  containers:
  - name: constraints-mem-demo-2-ctr
    image: nginx
    resources:
      limits:
        memory: "1.5Gi"  # 超过最大值 1Gi
      requests:
        memory: "800Mi"
        
$ kubectl create -f https://k8s.io/docs/tasks/administer-cluster/memory-constraints-pod-2.yaml --namespace=constraints-mem-example

# Pod创建失败，因为容器指定的limit过大
Error from server (Forbidden): error when creating "docs/tasks/administer-cluster/memory-constraints-pod-2.yaml":
pods "constraints-mem-demo-2" is forbidden: maximum memory usage per Container is 1Gi, but limit is 1536Mi.
```

**创建小于最小内存request的Pod**

```shell
$ cat memory-constraints-pod-3.yaml
apiVersion: v1
kind: Pod
metadata:
  name: constraints-mem-demo-3
spec:
  containers:
  - name: constraints-mem-demo-3-ctr
    image: nginx
    resources:
      limits:
        memory: "800Mi"
      requests:
        memory: "100Mi"   # 小于最小值500Mi
        
$ kubectl create -f https://k8s.io/docs/tasks/administer-cluster/memory-constraints-pod-3.yaml --namespace=constraints-mem-example         

# Pod创建失败，因为容器指定的内存request过小
Error from server (Forbidden): error when creating "docs/tasks/administer-cluster/memory-constraints-pod-3.yaml":
pods "constraints-mem-demo-3" is forbidden: minimum memory usage per Container is 500Mi, but request is 100Mi.
```

**创建没有指定任何内存limit和request的pod**

```shell
$ cat memory-constraints-pod-4.yaml
apiVersion: v1
kind: Pod
metadata:
  name: constraints-mem-demo-4
spec:
  containers:
  - name: constraints-mem-demo-4-ctr
    image: nginx

$ kubectl create -f https://k8s.io/docs/tasks/administer-cluster/memory-constraints-pod-4.yaml --namespace=constraints-mem-example

# 查看Pod
$ kubectl get pod constraints-mem-demo-4 --namespace=constraints-mem-example --output=yaml
...
resources:
  limits:
    memory: 1Gi
  requests:
    memory: 1Gi
...
```

容器没有指定自己的 CPU 请求和限制，所以它将从 LimitRange 获取默认的 CPU 请求和限制值。

#### 2.2.2. CPU的最大最小值

**创建LimitRange**

```shell
# 创建namespace
$ kubectl create namespace constraints-cpu-example

# 创建LimitRange
$ cat cpu-constraints.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-min-max-demo-lr
spec:
  limits:
  - max:
      cpu: "800m"
    min:
      cpu: "200m"
    type: Container
    
$ kubectl create -f https://k8s.io/docs/tasks/administer-cluster/cpu-constraints.yaml --namespace=constraints-cpu-example

# 查看LimitRange
$ kubectl get limitrange cpu-min-max-demo-lr --output=yaml --namespace=constraints-cpu-example
...
limits:
- default:
    cpu: 800m
  defaultRequest:
    cpu: 800m
  max:
    cpu: 800m
  min:
    cpu: 200m
  type: Container
...

```

**创建符合要求的Pod**

```shell
$ cat cpu-constraints-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: constraints-cpu-demo
spec:
  containers:
  - name: constraints-cpu-demo-ctr
    image: nginx
    resources:
      limits:
        cpu: "800m"
      requests:
        cpu: "500m"
        
$ kubectl create -f https://k8s.io/docs/tasks/administer-cluster/cpu-constraints-pod.yaml --namespace=constraints-cpu-example

# 查看Pod
$ kubectl get pod constraints-cpu-demo --output=yaml --namespace=constraints-cpu-example
...
resources:
  limits:
    cpu: 800m
  requests:
    cpu: 500m
...
```

**创建超过最大CPU limit的Pod**

```shell
$ cat cpu-constraints-pod-2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: constraints-cpu-demo-2
spec:
  containers:
  - name: constraints-cpu-demo-2-ctr
    image: nginx
    resources:
      limits:
        cpu: "1.5"
      requests:
        cpu: "500m"
        
$ kubectl create -f https://k8s.io/docs/tasks/administer-cluster/cpu-constraints-pod-2.yaml --namespace=constraints-cpu-example

# Pod创建失败，因为容器指定的CPU limit过大
Error from server (Forbidden): error when creating "docs/tasks/administer-cluster/cpu-constraints-pod-2.yaml":
pods "constraints-cpu-demo-2" is forbidden: maximum cpu usage per Container is 800m, but limit is 1500m.
```

**创建小于最小CPU request的Pod**

```shell
$ cat cpu-constraints-pod-3.yaml
apiVersion: v1
kind: Pod
metadata:
  name: constraints-cpu-demo-4
spec:
  containers:
  - name: constraints-cpu-demo-4-ctr
    image: nginx
    resources:
      limits:
        cpu: "800m"
      requests:
        cpu: "100m"
        
$ kubectl create -f https://k8s.io/docs/tasks/administer-cluster/cpu-constraints-pod-3.yaml --namespace=constraints-cpu-example

# Pod创建失败，因为容器指定的CPU request过小
Error from server (Forbidden): error when creating "docs/tasks/administer-cluster/cpu-constraints-pod-3.yaml":
pods "constraints-cpu-demo-4" is forbidden: minimum cpu usage per Container is 200m, but request is 100m.
```

**创建没有指定任何CPU limit和request的pod**

```shell
$ cat cpu-constraints-pod-4.yaml
apiVersion: v1
kind: Pod
metadata:
  name: constraints-cpu-demo-4
spec:
  containers:
  - name: constraints-cpu-demo-4-ctr
    image: vish/stress
    
$ kubectl create -f https://k8s.io/docs/tasks/administer-cluster/cpu-constraints-pod-4.yaml --namespace=constraints-cpu-example    

# 查看Pod
kubectl get pod constraints-cpu-demo-4 --namespace=constraints-cpu-example --output=yaml
...
resources:
  limits:
    cpu: 800m
  requests:
    cpu: 800m
...
```

容器没有指定自己的 CPU 请求和限制，所以它将从 LimitRange 获取默认的 CPU 请求和限制值。

#### 2.2.3. 说明

LimitRange 在 namespace 中施加的最小和最大内存（CPU）限制只有在创建和更新 Pod 时才会被应用。改变 LimitRange 不会对之前创建的 Pod 造成影响。

Kubernetes 都会执行下列步骤：

- 如果容器没有指定自己的内存（CPU）请求（request）和限制（limit），系统将会为其分配默认值。
- 验证容器的内存（CPU）请求大于等于最小值。
- 验证容器的内存（CPU）限制小于等于最大值。

## 3. Resource Quality of Service 

### 3.1. 资源QoS简介

`request`值表示容器保证可被分配到资源。`limit`表示容器可允许使用的最大资源。Pod级别的`request`和`limit`是其所有容器的request和limit之和。

### 3.2. Requests and Limits

Pod可以指定`request`和`limit`资源。其中`0 <= request <=`[`Node Allocatable`](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/node/node-allocatable.md) & `request <= limit <= Infinity`。调度是基于`request`而不是`limit`，即如果Pod被成功调度，那么可以保证Pod分配到指定的 `request`的资源。Pod使用的资源能否超过指定的`limit`值取决于该资源是否可被压缩。

#### 3.2.1. 可压缩的资源

- 目前只支持CPU
- pod可以保证获得它们请求的CPU数量，它们可能会也可能不会获得额外的CPU时间(取决于正在运行的其他作业)。因为目前CPU隔离是在容器级别而不是pod级别。

#### 3.2.2. 不可压缩的资源

- 目前只支持内存
- pod将获得它们请求的内存数量，如果超过了它们的内存请求，它们可能会被杀死(如果其他一些pod需要内存)，但如果pod消耗的内存小于请求的内存，那么它们将不会被杀死(除非在系统任务或守护进程需要更多内存的情况下)。

### 3.3. QoS 级别

在机器资源超卖的情况下（limit的总量大于机器的资源容量），即CPU或内存耗尽，将不得不杀死部分不重要的容器。因此对容器分成了3个`QoS`的级别：`Guaranteed`,` Burstable`,  `Best-Effort`，三个级别的优先级依次递减。

当CPU资源无法满足，pod不会被杀死可能被短暂控制。

内存是不可压缩的资源，当内存耗尽的情况下，会依次杀死优先级低的容器。Guaranteed的级别最高，不会被杀死，除非容器使用量超过limit限值或者资源耗尽，已经没有更低级别的容器可驱逐。

#### 3.3.1. Guaranteed

所有的容器的`limit`值和`request`值被配置且两者相等（如果只配置limit没有request，则request取值于limit）。

例如：

```yaml
# 示例1
containers:
  name: foo
    resources:
      limits:
        cpu: 10m
        memory: 1Gi
  name: bar
    resources:
      limits:
        cpu: 100m
        memory: 100Mi
# 示例2
containers:
  name: foo
    resources:
      limits:
        cpu: 10m
        memory: 1Gi
      requests:
        cpu: 10m
        memory: 1Gi

  name: bar
    resources:
      limits:
        cpu: 100m
        memory: 100Mi
      requests:
        cpu: 100m
        memory: 100Mi
```

#### 3.3.2. Burstable

如果一个或多个容器的limit和request值被配置且两者不相等。

例如：

```yaml
# 示例1
containers:
  name: foo
    resources:
      limits:
        cpu: 10m
        memory: 1Gi
      requests:
        cpu: 10m
        memory: 1Gi

  name: bar
  
# 示例2
containers:
  name: foo
    resources:
      limits:
        memory: 1Gi

  name: bar
    resources:
      limits:
        cpu: 100m

# 示例3
containers:
  name: foo
    resources:
      requests:
        cpu: 10m
        memory: 1Gi

  name: bar
```

#### 3.3.3. Best-Effort

所有的容器的`limit`和`request`值都没有配置。

例如：

```yaml
containers:
  name: foo
    resources:
  name: bar
    resources:
```

参考：

https://kubernetes.io/docs/concepts/policy/resource-quotas/

https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/

https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/

https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-constraint-namespace/

https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-constraint-namespace/

https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/

https://github.com/kubernetes/community/blob/master/contributors/design-proposals/node/resource-qos.md
