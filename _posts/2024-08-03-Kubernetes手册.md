---
layout: post
title:  "kubernates笔记"
date:   2024-08-01 18:27:14 -0400
category: middle
---



## 1. 简介

> 什么是运维工程师?
>
> 主要职责是 维护并确保整个服务的高可用性, 同时不断优化系统架构提升部署效率,优化资源利用率提高整体的ROI, 保障并不断提升服务的可用性,确保用户数据安全,提升用户体验,用自动化的工具、平台提升如按键在研发生命周期中的工程效率，通过技术手段优化服务架构、性能调优；通过资源优化组合基昂蒂成本、提升ROI
>
>  

### 1.1 场景

> 当你的应用只是跑在一台机器，直接一个 docker + docker-compose 就够了，方便轻松；
> 当你的应用需要跑在 3，4 台机器上，你依旧可以每台机器单独配置运行环境 + 负载均衡器；

但是随着机器数量的增加,管理难度逐渐增加, 这时就可以使用K8S进行管理了!

kubernetes可以轻松管理非常多的机器, 降低集群管理的时间复杂度!

**特性**：

- 高可用，不宕机，自动灾难恢复
- 灰度更新，不影响业务正常运转
- 一键回滚到历史版本
- 方便的伸缩拓展（应用拓展、机器加减），提供负载均衡
- 生态完整

### 1.2 概念

<center>
    <img src="https://s2.loli.net/2022/09/14/AmGidc7p9bkryjz.png" alt="image-20220914184635774" style="zoom:80%;" /> 
    <br/>Kubernetes集群
</center>

> 节点(Node):虚拟机或者物理机

**master节点:**

主节点, 控制平台, 不需要很高的性能, 不跑任务，通常一个即可。也可以开多个主节点来提高集群可用度。

**worker节点:**

工作节点，负责具体工作/任务, 通常都有很多个, 可以不断加机器扩大集群, 工作节点由主节点管理

**Pod:**

是K8S进行调度、管理的最小单位，一个Pod可以包含一个或多个容器，**每个Pod有自己的虚拟ip**，一个工作节点可以有多个Pod，==主节点==会考量负载、自动调度将==Pod==分配到具体哪个==工作节点==运行。





### 1.3 组件

<center>
    <img src="https://s2.loli.net/2022/09/14/dC7zqiAlQLKY3rh.png" alt="image-20220914185756489" style="zoom:80%;" /><br/> k8s组件示意图
</center>
#### 1.3.1 控制平台组件

控制平台组件(Control Plane Components)会为集群做出全局决策, 比如资源调度、检测和响应集群事件(eg.当不满足部署的`replicas`字段时, 要启动新的pod）。

为了简单起见, 通常会在同一个机器上启动所有的控制平面组件, 并且不会在该机器上同时运行用户容器.

- **kube-apiserver:**

  API服务器, 公开了Kubernetes API



- **etcd**

  键值数据库, 可以作为保存Kubernetes所有集群数据的后台数据库



- **kube-scheduler**

  调度Pod到哪个节点运行, 考虑因素



- **kube-controller-manager**

  集群空置期



- **cloud-controller-manager**

  与云服务商进行交互



<!--以下控制器都包含对云平台驱动的依赖-->

<!--节点控制器(Node Controller): 用在节点终止响应后检查云提供商以确定节点是否已被删除-->
<!--路由控制器(Route Controller): 用于在底层云基础架构中设置路由-->
<!--服务控制器(Service Controller): 用于CRUD云提供负载均衡器-->

#### 1.3.2 节点组件

节点组件会在每个节点上运行, 负责维护运行的Pod并提供运行环境



#### 1.3.3 插件



















![federation](https://s2.loli.net/2022/09/14/G64yWrlFJQi1CU7.png)

![Kubernetes-Architecture-Diagram](https://s2.loli.net/2022/09/14/MOz6wsFReCrxIKG.png)

![Kubernetes-architecture-diagram-1-1](https://s2.loli.net/2022/09/14/5HGcPF7YOBqeKCW.png)





![federation](https://s2.loli.net/2022/09/14/G64yWrlFJQi1CU7.png)





## 2. 使用
### 2.1 选择方式[本地安装或者使用云服务]

### 2.2 部署应用到集群中

#### 2.2.1 编写Yaml文件

**直接命令运行:**

```shell
kubectl run testapp --image=ccr.ccs.tencentyun.com/k8s-tutorial/test-k8s:v1
```

**Pod**

```yaml
apiVersion:	v1
kind: Pod
metadata: 
  name: test-pod
spec:
# 定义容器, 可以多个
  containers:
    - name: test-k8s
    # 镜像
      images: ccr.ccs.tencentyun.com/k8s-tutorial/test-k8s:v1
```

**Deployment:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  # 部署名字
  name: test-k8s
spec:
  replicas: 2
  # 用来查找关联的 Pod，所有标签都匹配才行
  selector:
    matchLabels:
      app: test-k8s
  # 定义 Pod 相关数据
  template:
    metadata:
      labels:
        app: test-k8s
    spec:
      # 定义容器，可以多个
      containers:
      - name: test-k8s # 容器名字
        image: ccr.ccs.tencentyun.com/k8s-tutorial/test-k8s:v1 # 镜像
```

**Deployment通过label关联起来Pods**

![image-20220916143501657](https://s2.loli.net/2022/09/16/cDRqxmJ5XOdUHeE.png)

#### 2.2.2 部署应用演示

部署一个nodeJS web应用

```shell
# 部署应用
kubectl apply -f app.yaml
# 查看 deployment
kubectl get deployment
# 查看 pod
kubectl get pod -o wide
# 查看 pod 详情
kubectl describe pod pod-name
# 查看 log
kubectl logs pod-name
# 进入 Pod 容器终端， -c container-name 可以指定进入哪个容器。
kubectl exec -it pod-name -- bash
# 伸缩扩展副本
kubectl scale deployment test-k8s --replicas=5
# 把集群内端口映射到节点
kubectl port-forward pod-name 8090:8080
# 查看历史
kubectl rollout history deployment test-k8s
# 回到上个版本
kubectl rollout undo deployment test-k8s
# 回到指定版本
kubectl rollout undo deployment test-k8s --to-revision=2
# 删除部署
kubectl delete deployment test-k8s
```

**Pod 报错解决**



```
networkPlugin cni failed to set up pod "test-k8s-68bb74d654-mc6b9_default" network: open /run/flannel/subnet.env: no such file or directory
```

在每个节点创建文件`/run/flannel/subnet.env`写入以下内容，配置后等待一会就好了

```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.0.1/24
FLANNEL_MTU=1450
FLANNEL_IPMASQ=true
```

**更多命令:**

```
# 查看全部
kubectl get all
# 重新部署
kubectl rollout restart deployment test-k8s
# 命令修改镜像，--record 表示把这个命令记录到操作历史中
kubectl set image deployment test-k8s test-k8s=ccr.ccs.tencentyun.com/k8s-tutorial/test-k8s:v2-with-error --record
# 暂停运行，暂停后，对 deployment 的修改不会立刻生效，恢复后才应用设置
kubectl rollout pause deployment test-k8s
# 恢复
kubectl rollout resume deployment test-k8s
# 输出到文件
kubectl get deployment test-k8s -o yaml >> app2.yaml
# 删除全部资源
kubectl delete all --all
```

更多官网关于 [Deployment](https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/) 的介绍

将 Pod 指定到某个节点运行：[nodeselector](https://kubernetes.io/zh/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector)
限定 CPU、内存总量：[文档](https://kubernetes.io/zh/docs/concepts/policy/resource-quotas/#计算资源配额)

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disktype: ssd
```

**工作负载分类**

- Deployment
  适合无状态应用，所有pod等价，可替代
- StatefulSet
  有状态的应用，适合数据库这种类型。
- DaemonSet
  在每个节点上跑一个 Pod，可以用来做节点监控、节点日志收集等
- Job & CronJob
  Job 用来表达的是一次性的任务，而 CronJob 会根据其时间规划反复运行。



### 2.3 Service

#### 2.3.1 特性

- Service 通过 label 关联对应的 Pod
- Servcie 生命周期不跟 Pod 绑定，不会因为 Pod 重创改变 IP
- 提供了负载均衡功能，自动转发流量到不同 Pod
- 可对集群外部提供访问端口
- 集群内部可通过服务名字访问

![image-20220916144908001](https://s2.loli.net/2022/09/16/O19q7ClFQ4j58iN.png)

#### 2.3.2 创建Service

创建一个Service, 通过标签`test-k8s`跟对应的Pod关联上`service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: test-k8s
spec:
  selector:
    app: test-k8s
  type: ClusterIP
  ports:
    - port: 8080        # 本 Service 的端口
      targetPort: 8080  # 容器端口
```

**应用配置**`kubectl apply -f service.yaml`

**查看服务:** `kubectl get svc`

**查看服务详情:** `kubectl describe svc test-k8s`, 可以发现EndPoints是各个Pod的IP, 也就是他会把流量转发到这些节点

服务的默认类型是ClusterIP, 只能在集群内部访问, 我们可以进入到Pod里面访问:

`kubectl exec -it pod名称 -- bash `

在pod内查看链接`curl http://test-k8s:8080`

如果要在集群外部访问, 可以通过端口转发实现(只适合测试用)

```shell
kubectl port-forward service/test-k8s 8888:8080
```

如果使用minikube, 也可以`minikube service test-k8s`

#### 2.3.3 对外暴露服务

上面我们是通过端口转发的方式可以在外面访问到集群里的服务，如果想要直接把集群服务暴露出来，我们可以使用`NodePort` 和 `Loadbalancer` 类型的 Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: test-k8s
spec:
  selector:
    app: test-k8s
  # 默认 ClusterIP 集群内可访问，NodePort 节点可访问，LoadBalancer 负载均衡模式（需要负载均衡器才可用）
  type: NodePort
  ports:
    - port: 8080        # 本 Service 的端口
      targetPort: 8080  # 容器端口
      nodePort: 31000   # 节点端口，范围固定 30000 ~ 32767
```

应用配置 `kubectl apply -f service.yaml`
在节点上，我们可以 `curl http://localhost:31000/hello/easydoc` 访问到应用
并且是有负载均衡的，网页的信息可以看到被转发到了不同的 Pod

```
hello easydoc 

IP lo172.17.0.8, hostname: test-k8s-68bb74d654-962lh
```

> 如果你是用 minikube，因为是模拟集群，你的电脑并不是节点，节点是 minikube 模拟出来的，所以你并不能直接在电脑上访问到服务

`Loadbalancer` 也可以对外提供服务，这需要一个负载均衡器的支持，因为它需要生成一个新的 IP 对外服务，否则状态就一直是 pendding，这个很少用了，后面我们会讲更高端的 Ingress 来代替它。

#### 2.3.4 多端口

多端口时必须配置name!
```yaml
apiVersion: v1
kind: Service
metadata:
  name: test-k8s
spec:
  selector:
    app: test-k8s
  type: NodePort
  ports:
    - port: 8080        # 本 Service 的端口
      name: test-k8s    # 必须配置
      targetPort: 8080  # 容器端口
      nodePort: 31000   # 节点端口，范围固定 30000 ~ 32767
    - port: 8090
      name: test-other
      targetPort: 8090
      nodePort: 32000
```







### 2.4 StatefulSet

#### 2.4.1 说明

StatefulSet是用来管理有状态的应用, 例如数据库.

不需要存储数据、不需要记住状态的，可以任意扩充副本，每个副本都是一样的，可替代的。

而像数据库、Redis这类有状态的，则不能随意扩充副本

StatefulSet会固定每个Pod的名字



#### 2.4.2 部署

部署StatefulSet类型的MongoDB

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb
spec:
  serviceName: mongodb
  replicas: 3
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
        - name: mongo
          image: mongo:4.4
          # IfNotPresent 仅本地没有镜像时才远程拉，Always 永远都是从远程拉，Never 永远只用本地镜像，本地没有则报错
          imagePullPolicy: IfNotPresent
---
apiVersion: v1
kind: Service
metadata:
  name: mongodb
spec:
  selector:
    app: mongodb
  type: ClusterIP
  # HeadLess
  clusterIP: None
  ports:
    - port: 27017
      targetPort: 27017
```

`kubectl apply -f mongo.yml`



#### 2.4.3 特性

- Service 的 `CLUSTER-IP` 是空的，Pod 名字也是固定的。
- Pod 创建和销毁是有序的，创建是顺序的，销毁是逆序的。
- Pod 重建不会改变名字，除了IP，所以不要用IP直连

```shell
kubectl get svc
kubectl get pods -A
```



EndPoints会多一个hostname!

```shell
kubectl get endpoints mongodb -o yaml
```



访问时, 如果直接使用Service名字连接, 会随机转发请求

要连接指定Pod, 可以使用`pod名字.service名字`

运行一个临时pod连接数据测试下:

```shell
kubectl run mongodb-client --rm --tty -i --restart='Never' --image docker.io/bitnami/mongodb:4.4.10-debian-10-r20 --command -- bash
```



#### 2.4.4 Web连接MongoDB

在集群内部, 我们可以通过服务名字访问到不同的服务

指定连接第一个: mongodb-0.mongodb

![image-20220916163414240](https://s2.loli.net/2022/09/16/vdRuXOWGMySPqV1.png)



```shell
kubectl get all
```



### 2.6 数据持久化-挂载

#### 2.6.1 介绍

`kubernetes`集群不会为你处理数据的存储, 我们呢可以为数据库挂载一个磁盘来确保数据的安全.

选择有:

- 本地磁盘: 可以挂载某个节点的目录, 但这需要限定Pod在这个节点上运行
- 云存储: 不限定节点, 不受集群影响, 需要云服务商提供, 裸机集群是没有的
- NFS: 不限定节点, 不受集群影响

#### 2.6.2 hostPath挂载实例

把节点上的一个目录挂载到Pod, 但是已经不推荐使用了





#### 2.6.3 更高级的抽象

![image-20220916165221590](https://s2.loli.net/2022/09/16/DrjQZouPtliafT2.png)

**Storage Class(SC):**

将存储卷划分为不同的种类, 例如: SSD, 普通磁盘, 本地磁盘:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/aws-ebs
parameters:
  type: io1
  iopsPerGB: "10"
  fsType: ext4
```



**Persistent Volume(PV):**

描述卷的具体信息, 例如磁盘大小，访问模式、文档类型等

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodata
spec:
  capacity:
    storage: 2Gi
  volumeMode: Filesystem  # Filesystem（文件系统） Block（块）
  accessModes:
    - ReadWriteOnce       # 卷可以被一个节点以读写方式挂载
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /root/data
  nodeAffinity:
    required:
      # 通过 hostname 限定在某个节点创建存储卷
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - node2
```

**Persistent Volume Claim(PVC):**

对存储需求的一个申明, 可以理解为一个申请单, 系统根据这个申请单去找一个合适的PV

还可以根据PVC自动创建PV

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodata
spec:
  accessModes: ["ReadWriteOnce"]
  storageClassName: "local-storage"
  resources:
    requests:
      storage: 2Gi
```



**思考:为什么有这么多抽象**

- 更好的分工，运维人员负责提供好存储，开发人员不需要关注磁盘细节，只需要写一个申请单。

- 方便云服务商提供不同类型的，配置细节不需要开发者关注，只需要一个申请单。

- 动态创建，开发人员写好申请单后，供应商可以根据需求自动创建所需存储卷。





#### 2.6.4 本地磁盘示例

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
        image: mongo:5.0
        imagePullPolicy: IfNotPresent
        name: mongo
        volumeMounts:
          - mountPath: /data/db
            name: mongo-data
      volumes:
        - name: mongo-data
          persistentVolumeClaim:
             claimName: mongodata
---
apiVersion: v1
kind: Service
metadata:
  name: mongodb
spec:
  clusterIP: None
  ports:
  - port: 27017
    protocol: TCP
    targetPort: 27017
  selector:
    app: mongodb
  type: ClusterIP
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodata
spec:
  capacity:
    storage: 2Gi
  volumeMode: Filesystem  # Filesystem（文件系统） Block（块）
  accessModes:
    - ReadWriteOnce       # 卷可以被一个节点以读写方式挂载
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /root/data
  nodeAffinity:
    required:
      # 通过 hostname 限定在某个节点创建存储卷
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - node2
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodata
spec:
  accessModes: ["ReadWriteOnce"]
  storageClassName: "local-storage"
  resources:
    requests:
      storage: 2Gi
```

​	

### 2.7 ConfigMap 

数据库链接地址, 这种根据部署环境变化的, 我们不应该写死在代码里.

Kubernetes提供的ConfigMap, 可以方便地进行一些变量的撇脂

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongo-config
data:
  mongoHost: mongodb-0.mongodb
```

`configmap.yaml`

```shell
# 应用
kubectl apply -f configmap.yaml
# 查看
kubectl get configmap mongo-config -o yaml
```



### 2.8 Secret

#### 2.8.1 格式

一些重要数据, 例如密码、Token等我们可以放到Secret中。

`secret.yaml`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongo-secret
# Opaque 用户定义的任意数据，更多类型介绍 https://kubernetes.io/zh/docs/concepts/configuration/secret/#secret-types
type: Opaque
data:
  # 数据要 base64。https://tools.fun/base64.html
  mongo-username: bW9uZ291c2Vy
  mongo-password: bW9uZ29wYXNz
```

```shell
# 应用
kubectl apply -f  secret.yaml
# 查看
kubectl get secret mongo-secret -o yaml
```



#### 2.8.2 使用方法

##### 作为环境变量使用

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
        - name: mongo
          image: mongo:4.4
          # IfNotPresent 仅本地没有镜像时才远程拉，Always 永远都是从远程拉，Never 永远只用本地镜像，本地没有则报错
          imagePullPolicy: IfNotPresent
          env:
          - name: MONGO_INITDB_ROOT_USERNAME
            valueFrom:
              secretKeyRef:
                name: mongo-secret
                key: mongo-username
          - name: MONGO_INITDB_ROOT_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mongo-secret
                key: mongo-password
          # Secret 的所有数据定义为容器的环境变量，Secret 中的键名称为 Pod 中的环境变量名称
          # envFrom:
          # - secretRef:
          #     name: mongo-secret

```

##### 挂载为文件

挂载后, 会在容器中对应路径生成文件, 一个key对应一个文件,内容就是value

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
```





### 2.9 Helm

#### 2.9.1 介绍

`helm`类似npm, pip, docker hub等, 可以理解为一个软件库, 可以方便快速的为我们的集群安装一些第三方软件

使用`Helm`我们可以非常方便地搭建出来MongoDB、MySQL副本集群， YAML文件可以直接使用！

#### 2.9.2 安装

**安装**

```shell
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```



**安装MongoDB:**

```shell
# 安装
helm repo add bitnami https://chart.bitnami.com/bitnami
helm install my-mongo bitnami/mongodb

# 指定密码和架构
helm install my-mongo bitnami/mongodb --set architecture="replicaset",auth.rootPassword="mongopass"

# 删除
helm ls
heml delete my-mongo

# 查看密码
kubectl get secret my-mongo-mongodb -o json
kubectl get secret my-mongo-mongodb -o yaml > secret.yaml

# 临时运行一个包含 mongo client 的 debian 系统
kubectl run mongodb-client --rm --tty -i --restart='Never' --image docker.io/bitnami/mongodb:4.4.10-debian-10-r20 --command -- bash

# 进去 mongodb
mongo --host "my-mongo-mongodb" -u root -p mongopass

# 也可以转发集群里的端口到宿主机访问 mongodb
kubectl port-forward svc/my-mongo-mongodb 27017:27018
```





### 2.10 命名空间

#### 2.10.1 定义

如果一个集群中部署了多个应用, 所有的应用都在一起不便管理, 存在名字冲突等问题

我们可以使用namespace把应用划分到不同的命名空间.



#### 2.10.2 使用

```shell
# 创建命名空间
kubectl create namespace testapp
# 部署应用到指定的命名空间
kubectl apply -f app.yml --namespace testapp
# 查询
kubectl get pod --namespace kube-system
```

**可以使用kubens快速切换namespace**

```shell
# 切换命名空间
kubens kube-system
# 回到上个命名空间
kubens -
# 切换集群
kubectx minikube
```





### 2.11 Ingress

#### 2.11.1 介绍

Ingress为外部访问集群提供了一个统一入口， 避免了对外暴露集群端口；

功能类似Nginx，可以根据域名、路径把请求转发到不同的Service。

可以配置Https

> 跟 **LoadBalancer**有什么区别?
>
> LoadBalancer需要对外暴露端口, 不安全
>
> 无法根据域名, 路径转发流量到不同Service, 多个Service则需要开 多个LoadBalancer;并且LoadBalancer功能单一, 无法配置`https`

![image-20220917141819868](https://s2.loli.net/2022/09/17/CKWUh4ufqzgwMkc.png)

#### 2.11.2 使用

要使用Ingress,需要一个负载均衡器 + Ingress Controller

如果是裸机(bare metal)搭建的集群, 则需要自己安装一个负载均衡插件,例如  [METALLB](https://metallb.universe.tf/)

如果是云服务商, 会自动配置, 否则外部ip会是"pending"状态, 无法使用



`Helm`安装`Nginx`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: simple-example
spec:
  ingressClassName: nginx
  rules:
  - host: tools.fun
    http:
      paths:
      - path: /easydoc
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 4200
      - path: /svnbucket
        pathType: Prefix
        backend:
          service:
            name: service2
            port:
              number: 8080
```



















































## x. kube-proxy

> kube-proxy是运行在集群中的每个节点上的网络代理，实现了Kubernetes Service概念的一部分。
>
> 
>
> Kube-proxy维护节点上的网络规则。这些网络规则允许从集群内部或外部的网络会话到您的Pods进行网络通信。
>
> 
>
> Kube-proxy使用操作系统包过滤层(如果有且可用)。否则，kube-proxy自己转发流量。
>
> 
>
> 这反映了Kubernetes API在每个节点上定义的服务，可以跨一组后端进行简单的TCP、UDP和SCTP流转发或轮询TCP、UDP和SCTP转发。服务集群的ip和端口目前是通过与docker链接兼容的环境变量找到的，这些环境变量指定由服务代理打开的端口。有一个可选插件为这些集群IPs提供集群DNS。用户必须使用apiserver API创建一个服务来配置代理。

### x.1 意义

容器的特点是快速创建、快速销毁，Kubernetes Pod和容器一样只具有临时的生命周期，一个Pod随时有可能被终止或者漂移，随着集群的状态变化而变化，一旦Pod变化，则该Pod提供的服务也就无法访问，如果直接访问Pod地址则无法实现服务的连续性和高可用性，因此显然不能使用Pod地址作为服务暴露端口

解决这个问题的办法和传统数据中心解决无状态服务高可用的思路完全一样，通过负载均衡和VIP（虚拟ip）实现后端真实服务的自动转发、故障转移。

这个负载均衡在Kubernetes中称为Service，VIP即Service Cluster IP，因此可以认为Kubernetes的Service就是一个四层（OSI七层模型）负载均衡，Kubernetes对应的还有七层负载均衡Ingress（后面介绍）

这个Service就是由Kube-proxy实现的，Cluster IP不会因为Pods状态改变而变，需要注意的是这个IP（VIP、Cluster IP）在整个集群中根本不存在，当然也就无法通过IP协议栈、无法路由，底层underlay设备更无法感知这个IP的存在，因此Cluster IP只能是单主机(Host only)作用域可见,这个IP在其它节点和集群外均无法访问.



Kubernetes为了实现在集群所有的节点都能访问Service,Kube-proxy默认会在所有的Node节点都创建这个VIP并且实现负载,所以在部署Kubernetes后发现Kube-proxy是一个DaemonSet.



而Service负载之所以能在Node节点上实现是因为无论Kubernetes使用哪个网络模型,均需要保证如下三个条件:

- 容器之间要求不需要任何NAT能直接通信;
- 容器与Node之间要求不需要任何NAT能直接通信;
- 容器看到自身的IP和外边看到它的IP必须是一样的, 即不存在IP转化的问题

至少第2点



### x.4 IPVS 模式
