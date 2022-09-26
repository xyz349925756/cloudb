二进制搭建好kubernetes基本可以了解组件的工作原理。kubeadm 部署简单，故障解决十分麻烦二进制部署特别困难，排除问题简单。

# kubernetes常用概念操作实战

## namespace

Namespace是kubernetes系统中的一种非常重要资源，它的主要作用是用来实现**多套环境的资源隔离**或者**多租户的资源隔离**。

系统搭建好会自动生成下面四个命名空间

\#查看namespaces

```bash
[root@master01 ~]# kubectl get namespaces 
NAME                   STATUS   AGE
default                Active   23h   #默认没有指定namesoace的对象都会分配到这里
kube-node-lease        Active   23h   #心跳维护
kube-public            Active   23h   #对外开放
kube-system            Active   23h   #系统命名空间
kubernetes-dashboard   Active   19h   #web 界面管理k8s 自己安装
```

ns = namespaces 简写

\#查看指定namespaces

```bash
[root@master01 ~]# kubectl get ns default 
NAME      STATUS   AGE
default   Active   23h
```

指定输出格式yaml查看default

参数搭配-o 

常见kubernetes参数 wide json yaml

\#查看namespaces default的yaml文件

```bash
[root@master01 ~]# kubectl get ns default -o yaml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: "2021-05-18T01:35:14Z"
  labels:
    kubernetes.io/metadata.name: default
  name: default
  resourceVersion: "211"
  uid: 19723afb-18cd-40ba-b12d-5c048821c7ab
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
```

\#看看json格式是什么样的

```bash
[root@master01 ~]# kubectl get ns default -o json
{
    "apiVersion": "v1",
    "kind": "Namespace",
    "metadata": {
        "creationTimestamp": "2021-05-18T01:35:14Z",
        "labels": {
            "kubernetes.io/metadata.name": "default"
        },
        "name": "default",
        "resourceVersion": "211",
        "uid": "19723afb-18cd-40ba-b12d-5c048821c7ab"
    },
    "spec": {
        "finalizers": [
            "kubernetes"
        ]
    },
    "status": {
        "phase": "Active"
    }
}
```

\#查看那个pod在那台服务器

![image-20210819103222510](kubernetes操作实战.assets\image-20210819103222510.png)

可以详细的看出那个pod在那个节点

\#查看端口映射和default pod

![image-20210819103237968](kubernetes操作实战.assets\image-20210819103237968.png)

```bash
[root@master01 ~]# kubectl get pod -n kube-system -o wide
[root@master01 ~]# kubectl get pod,svc -n default -o wide
```

一般查询命令式命令最好。方便排查问题

yaml适合创建containers

```bash
[root@master01 ~]# kubectl  describe namespaces default
Name:         default
Labels:       kubernetes.io/metadata.name=default
Annotations:  <none>
Status:       Active 
#Active namespaces 正在使用   terminating 正在删除 namespaces
No resource quota.#针对namespaces做的资源限制

No LimitRange resource.#对namespaces每个组件做资源限制
```

### 创建一个namespaces

```bash
[root@master01 ~]# kubectl create namespace dev   #创建一个namespaces
namespace/dev created
[root@master01 ~]# kubectl get namespaces dev   #查看
NAME   STATUS   AGE
dev    Active   17s
[root@master01 ~]# kubectl get namespaces dev -o yaml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: "2021-05-19T01:58:56Z"
  labels:
    kubernetes.io/metadata.name: dev
  name: dev
  resourceVersion: "68683"
  uid: 36ad45bb-6894-4bc2-83d9-2c955f756236
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
[root@master01 ~]# kubectl delete namespaces dev   #删除namespaces dev
namespace "dev" deleted 
[root@master01 ~]# kubectl get namespace dev #查看namespaces dev还在不在
Error from server (NotFound): namespace "dev" not found
[root@master01 ~]#
```

```bash
[root@master01 ~/yaml]# cat namespaces-dev.yaml 
apiVersion: v1
kind: Namespace
metadata:
  name: dev
[root@master01 ~/yaml]# kubectl create -f namespaces-dev.yaml 
namespace/dev created
[root@master01 ~/yaml]# kubectl get ns dev
NAME   STATUS   AGE
dev    Active   14s
[root@master01 ~/yaml]# kubectl delete -f namespaces-dev.yaml 
namespace "dev" deleted
[root@master01 ~/yaml]# kubectl get ns dev
Error from server (NotFound): namespaces "dev" not found
```

## pod

Pod是kubernetes集群进行管理的最小单元，程序要运行必须部署在容器中，而容器必须存在于Pod中。

kubernetes在集群启动之后，集群中的各个组件也都是以Pod方式运行的

```bash
[root@master01 ~/yaml]# kubectl get pod -n kube-system -o wide
```

![image-20210819103521469](kubernetes操作实战.assets\image-20210819103521469.png)

kubernetes没有提供单独运行Pod的命令，都是通过Pod控制器来实现的

> \# 命令格式： kubectl run (pod控制器名称) [参数] 
>
> \# --image 指定Pod的镜像
>
> \# --port  指定端口
>
> \# --namespace 指定namespace

### 创建指定pod

```bash
[root@master01 ~/yaml]# kubectl create -f namespaces-dev.yaml  #创建namespace dev
namespace/dev created
[root@master01 ~/yaml]# kubectl run nginx --image=nginx:latest --port=80 --namespace dev     #创建并运行nginx容器在namespace dev下
pod/nginx created
[root@master01 ~/yaml]# kubectl get ns dev   #查看namespace
NAME   STATUS   AGE
dev    Active   19s
[root@master01 ~/yaml]# kubectl get pods   #没有指定namespace 默认去default下面
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6799fc88d8-vflwp   1/1     Running   1          18h
[root@master01 ~/yaml]# kubectl get pods -n dev   #查询dev下面的pod
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          28s
```

查看dev 命名空间下的nginx 详细信息

```bash
[root@master01 ~/yaml]# kubectl describe pod nginx -n dev
Name:         nginx
Namespace:    dev
Priority:     0
Node:         node01/192.168.1.22
Start Time:   Wed, 19 May 2021 10:13:51 +0800
Labels:       run=nginx
Annotations:  cni.projectcalico.org/podIP: 10.244.196.140/32
              cni.projectcalico.org/podIPs: 10.244.196.140/32
Status:       Running
IP:           10.244.196.140
IPs:
  IP:  10.244.196.140
Containers:
  nginx:
    Container ID:   docker://92986ca85a8c4c607e41415438805c33994d5b847f4d73793586e60df2ad8acb
    Image:          nginx:latest
    Image ID:       docker-pullable://nginx@sha256:df13abe416e37eb3db4722840dd479b00ba193ac6606e7902331dcea50f4f1f2
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 19 May 2021 10:13:59 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-bqdzk (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-bqdzk:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  4m49s  default-scheduler  Successfully assigned dev/nginx to node01
  Normal  Pulling    4m46s  kubelet            Pulling image "nginx:latest"
  Normal  Pulled     4m42s  kubelet            Successfully pulled image "nginx:latest" in 4.069178063s
  Normal  Created    4m42s  kubelet            Created container nginx
  Normal  Started    4m41s  kubelet            Started container nginx
[root@master01 ~/yaml]# kubectl get pod -n dev -o wide  #获取指定pod IP地址
NAME    READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          11m   10.244.196.140   node01   <none>           <none>


[root@master01 ~/yaml]# curl http://10.244.196.140   #查看访问
```

### 删除指定pod

```bash
[root@master01 ~/yaml]# kubectl get pod 
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6799fc88d8-vflwp   1/1     Running   1          19h

[root@master01 ~/yaml]# kubectl delete pod nginx-6799fc88d8-vflwp
pod "nginx-6799fc88d8-vflwp" deleted

[root@master01 ~/yaml]# kubectl get pod  #发现不能删除
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6799fc88d8-vbnsm   1/1     Running   0          9s

[root@master01 ~/yaml]# kubectl get deployments  #因为上一级deployments
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           19h

[root@master01 ~/yaml]# kubectl get deployments -n default  
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           19h
#删除上一级deployments

[root@master01 ~/yaml]# kubectl delete deployments nginx -n default  
deployment.apps "nginx" deleted

[root@master01 ~/yaml]# kubectl get pod -n default 
No resources found in default namespace.

配置操作
[root@master01 ~/yaml]# kubectl create -f pod-nginx.yaml 
pod/nginx created

[root@master01 ~/yaml]# cat pod-nginx.yaml 
apiVersion: v1
kind: Pod
metadata: 
  name: nginx
  namespace: dev
spec:
  containers: 
  - image: nginx:latest
    name: pod
    ports:
    - name: nginx-port
      containerPort: 80
      protocol: TCP
[root@master01 ~/yaml]# kubectl get pod -n dev
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          54s
```

## labs

Label是kubernetes系统中的一个重要概念。它的作用就是在资源上添加标识，用来对它们进行区分和选择。

```bash
[root@master01 ~/yaml]# kubectl get pod -n dev
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          45m


#打标签
[root@master01 ~/yaml]# kubectl label pod nginx version=1.0 -n dev
pod/nginx labeled
[root@master01 ~/yaml]# kubectl get pod nginx -n dev --show-labels 
NAME    READY   STATUS    RESTARTS   AGE   LABELS
nginx   1/1     Running   0          46m   version=1.0


#更新标签
[root@master01 ~/yaml]# kubectl label pod nginx version=2.0 -n dev
error: 'version' already has a value (1.0), and --overwrite is false
[root@master01 ~/yaml]# kubectl label pod nginx version=2.0 -n dev --overwrite 
pod/nginx labeled
[root@master01 ~/yaml]# kubectl get pod -n dev --show-
--show-kind            --show-labels          --show-managed-fields  
[root@master01 ~/yaml]# kubectl get pod -n dev --show-labels #查看标签
NAME    READY   STATUS    RESTARTS   AGE   LABELS
nginx   1/1     Running   0          47m   version=2.0


#筛选
[root@master01 ~/yaml]# kubectl get pod -n dev -l version=2.0 --show-labels 
NAME    READY   STATUS    RESTARTS   AGE   LABELS
nginx   1/1     Running   0          49m   version=2.0
[root@master01 ~/yaml]# kubectl get pod -n dev -l version!=2.0 --show-labels 
No resources found in dev namespace.



#删除标签
[root@master01 ~/yaml]# kubectl label pod nginx version- -n dev
pod/nginx labeled
[root@master01 ~/yaml]# kubectl get pod nginx -n dev --show-labels 
NAME    READY   STATUS    RESTARTS   AGE   LABELS
nginx   1/1     Running   0          51m   <none>



#文件创建
[root@master01 ~/yaml]# cat pod-nginx-labs.yaml 
apiVersion: v1
kind: Pod
metadata: 
  name: nginx-labels
  namespace: dev
  labels: 
    version: "2.0"
    env: "test"
spec:
  containers:
  - image: nginx:latest
    name: pod
    ports:
    - name: nginx-port
      containerPort: 80
      protocol: TCP
[root@master01 ~/yaml]# kubectl create -f pod-nginx-labs.yaml 
pod/nginx-labels created            
[root@master01 ~/yaml]# kubectl get pod -n dev --show-labels 
NAME           READY   STATUS    RESTARTS   AGE   LABELS
nginx          1/1     Running   0          64m   <none>
nginx-labels   1/1     Running   0          32s   env=test,version=2.0

```

## Deployment

在kubernetes中，Pod是最小的控制单元，但是kubernetes很少直接控制Pod，一般都是通过Pod控制器来完成的。Pod控制器用于pod的管理，确保pod资源符合预期的状态，当pod的资源出现故障时，会尝试进行重启或重建pod。

![image-20210819104005933](kubernetes操作实战.assets\image-20210819104005933.png)

> \# 命令格式: kubectl run deployment名称 [参数] 
>
> \# --image 指定pod的镜像
>
> \# --port  指定端口
>
> \# --replicas 指定创建pod数量  #已经被弃用使用deployment
>
> \# --namespace 指定namespace

```bash
[root@master01 ~/yaml]# cat deploy-nginx.yaml #使用声明创建
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: nginx
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels: 
        run: nginx
    spec:
      containers:
      - image: nginx:latest
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
          
[root@master01 ~/yaml]# kubectl create -f deploy-nginx.yaml  #创建
deployment.apps/nginx created

[root@master01 ~/yaml]# kubectl get deployments.apps -n dev #查看
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   3/3     3            3           50s

# UP-TO-DATE：成功升级的副本数量
# AVAILABLE：可用副本的数量

[root@master01 ~/yaml]# kubectl get pod -n dev 
NAME                     READY   STATUS    RESTARTS   AGE
nginx                    1/1     Running   1          6h8m
nginx-5bdb994596-d67qg   1/1     Running   0          4m31s
nginx-5bdb994596-xkk2d   1/1     Running   0          4m31s
nginx-5bdb994596-z6r79   1/1     Running   0          4m31s
nginx-labels             1/1     Running   1          5h3m

[root@master01 ~/yaml]# kubectl get pod -n dev -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP               NODE     NOMINATED NODE   READINESS GATES
nginx                    1/1     Running   1          6h8m    10.244.196.149   node01   <none>           <none>
nginx-5bdb994596-d67qg   1/1     Running   0          5m16s   10.244.196.153   node01   <none>           <none>
nginx-5bdb994596-xkk2d   1/1     Running   0          5m16s   10.244.196.152   node01   <none>           <none>
nginx-5bdb994596-z6r79   1/1     Running   0          5m16s   10.244.196.151   node01   <none>           <none>
nginx-labels             1/1     Running   1          5h4m    10.244.196.150   node01   <none>           <none>

```

![image-20210819104057097](kubernetes操作实战.assets\image-20210819104057097.png)

```bash
[root@master01 ~/yaml]# kubectl describe deployments.apps nginx -n dev
Name:                   nginx
Namespace:              dev
CreationTimestamp:      Wed, 19 May 2021 21:11:22 +0800
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               run=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  run=nginx
  Containers:
   nginx:
    Image:        nginx:latest
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-5bdb994596 (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  18m   deployment-controller  Scaled up replica set nginx-5bdb994596 to 3
  
#删除
[root@master01 ~/yaml]# kubectl delete deployments.apps nginx -n dev
deployment.apps "nginx" deleted
[root@master01 ~/yaml]# kubectl get deployments.apps -n dev
No resources found in dev namespace.

```

## service

通过上面的学习，已经能够利用Deployment来创建一组Pod来提供具有高可用性的服务。

虽然每个Pod都会分配一个单独的Pod IP，然而却存在如下两问题：

\- Pod IP 会随着Pod的重建产生变化

\- Pod IP 仅仅是集群内可见的虚拟IP，外部无法访问

这样对于访问这个服务带来了难度。因此，kubernetes设计了Service来解决这个问题。

Service可以看作是一组同类Pod**对外的访问接口**。借助Service，应用可以方便地实现服务发现和负载均衡。

![image-20210819104135673](kubernetes操作实战.assets\image-20210819104135673.png)

### 创建集群内部service

```bash
[root@master01 ~/yaml]# kubectl get deployments.apps -n dev #确认deployment存在
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   3/3     3            3           23s

[root@master01 ~/yaml]# kubectl expose deployment nginx --name=svc-nginx1 --type=ClusterIP --port=80 --target-port=80 -n dev
service/svc-nginx1 exposed

[root@master01 ~/yaml]# kubectl get svc #默认到default
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        36h
nginx        NodePort    10.110.173.117   <none>        80:30136/TCP   30h

[root@master01 ~/yaml]# kubectl get svc -n dev
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
svc-nginx1   ClusterIP   10.107.144.224   <none>        80/TCP    31s

[root@master01 ~/yaml]# kubectl get svc svc-nginx1 -n dev -o wide
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE   SELECTOR
svc-nginx1   ClusterIP   10.107.144.224   <none>        80/TCP    72s   run=nginx

[root@master01 ~/yaml]# kubectl get svc svc-nginx1 -n dev  #加不加name都可以
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
svc-nginx1   ClusterIP   10.107.144.224   <none>        80/TCP    84s

```

\# 这里产生了一个CLUSTER-IP，这就是service的IP，在Service的生命周期中，这个地址是不会变动的

\# 可以通过这个IP访问当前service对应的POD

```bash
[root@master01 ~/yaml]# curl 10.107.144.224:80
```

### 创建集群外部可访问的service

上面创建的IP地址为ClusterIP,只能集群内部访问如果需要外部访问需要type改成NodePort

![image-20210819104301557](kubernetes操作实战.assets\image-20210819104301557.png)

```bash
[root@master01 ~/yaml]# kubectl expose deployment nginx --name=svc-nginx2 --type=NodePort --port=80 --target-port=80 -n dev
service/svc-nginx2 exposed
[root@master01 ~/yaml]# kubectl get svc -n dev
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
svc-nginx1   ClusterIP   10.107.144.224   <none>        80/TCP         21m
svc-nginx2   NodePort    10.101.192.28    <none>        80:31035/TCP   15s


删除service
[root@master01 ~/yaml]# kubectl get svc -n dev
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
svc-nginx1   ClusterIP   10.107.144.224   <none>        80/TCP         21m
svc-nginx2   NodePort    10.101.192.28    <none>        80:31035/TCP   15s
[root@master01 ~/yaml]# kubectl delete svc svc-nginx1 -n dev
service "svc-nginx1" deleted
[root@master01 ~/yaml]# kubectl get svc -n dev
NAME         TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
svc-nginx2   NodePort   10.101.192.28   <none>        80:31035/TCP   4m10s
[root@master01 ~/yaml]# kubectl get svc 
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        36h
nginx        NodePort    10.110.173.117   <none>        80:30136/TCP   30h
[root@master01 ~/yaml]# kubectl delete svc nginx 
service "nginx" deleted
[root@master01 ~/yaml]# kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   36h #这个不能删除

```

```bash
[root@master01 ~/yaml]# kubectl get svc svc-nginx2 -n dev -o yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2021-05-19T14:04:10Z"
  name: svc-nginx2
  namespace: dev
  resourceVersion: "100772"
  uid: 528936e7-1ed1-4dd6-9bfc-ae43638436aa
spec:
  clusterIP: 10.101.192.28
  clusterIPs:
  - 10.101.192.28
  externalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - nodePort: 31035
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: nginx
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}
```

```bash
[root@master01 ~/yaml]# cat svc-service.yaml 
apiVersion: v1
kind: Service
metadata:
  name: svc-nginx
  namespace: dev
spec:
  clusterIP: 10.107.144.224 #这里销毁重新创建也不会变
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: nginx
  type: ClusterIP

[root@master01 ~/yaml]# kubectl create -f svc-service.yaml 
service/svc-nginx created
[root@master01 ~/yaml]# kubectl get svc -n dev
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
svc-nginx    ClusterIP   10.107.144.224   <none>        80/TCP         60s
svc-nginx2   NodePort    10.101.192.28    <none>        80:31035/TCP   23m
```

# pod详解

![image-20210819104414801](kubernetes操作实战.assets\image-20210819104414801.png)

每个Pod中都可以包含一个或者多个容器，这些容器可以分为两类：

\- 用户程序所在的容器，数量可多可少

\- Pause容器，这是每个Pod都会有的一个根容器，它的作用有两个：

 \- 可以以它为依据，评估整个Pod的健康状态

 \- 可以在根容器上设置Ip地址，其它容器都此Ip（Pod IP），以实现Pod内部的网路通信

所以在部署的时候这个镜像必须存在，通常情况会三台一样的拉取

## pod定义

## pod资源清单

```bash
apiVersion: v1     #必选，版本号，例如v1
kind: Pod       　 #必选，资源类型，例如 Pod
metadata:       　 #必选，元数据
  name: string     #必选，Pod名称
  namespace: string  #Pod所属的命名空间,默认为"default"
  labels:       　　  #自定义标签列表
    - name: string      　          
spec:  #必选，Pod中容器的详细定义
  containers:  #必选，Pod中容器列表
  - name: string   #必选，容器名称
    image: string  #必选，容器的镜像名称
    imagePullPolicy: [ Always|Never|IfNotPresent ]  #获取镜像的策略 
    command: [string]   #容器的启动命令列表，如不指定，使用打包时使用的启动命令
    args: [string]      #容器的启动命令参数列表
    workingDir: string  #容器的工作目录
    volumeMounts:       #挂载到容器内部的存储卷配置
    - name: string      #引用pod定义的共享存储卷的名称，需用volumes[]部分定义的的卷名
      mountPath: string #存储卷在容器内mount的绝对路径，应少于512字符
      readOnly: boolean #是否为只读模式
    ports: #需要暴露的端口库号列表
    - name: string        #端口的名称
      containerPort: int  #容器需要监听的端口号
      hostPort: int       #容器所在主机需要监听的端口号，默认与Container相同
      protocol: string    #端口协议，支持TCP和UDP，默认TCP
    env:   #容器运行前需设置的环境变量列表
    - name: string  #环境变量名称
      value: string #环境变量的值
    resources: #资源限制和请求的设置
      limits:  #资源限制的设置
        cpu: string     #Cpu的限制，单位为core数，将用于docker run --cpu-shares参数
        memory: string  #内存限制，单位可以为Mib/Gib，将用于docker run --memory参数
      requests: #资源请求的设置
        cpu: string    #Cpu请求，容器启动的初始可用数量
        memory: string #内存请求,容器启动的初始可用数量
    lifecycle: #生命周期钩子
		postStart: #容器启动后立即执行此钩子,如果执行失败,会根据重启策略进行重启
		preStop: #容器终止前执行此钩子,无论结果如何,容器都会终止
    livenessProbe:  #对Pod内各容器健康检查的设置，当探测无响应几次后将自动重启该容器
      exec:       　 #对Pod容器内检查方式设置为exec方式
        command: [string]  #exec方式需要制定的命令或脚本
      httpGet:       #对Pod内个容器健康检查方法设置为HttpGet，需要制定Path、port
        path: string
        port: number
        host: string
        scheme: string
        HttpHeaders:
        - name: string
          value: string
      tcpSocket:     #对Pod内个容器健康检查方式设置为tcpSocket方式
         port: number
       initialDelaySeconds: 0       #容器启动完成后首次探测的时间，单位为秒
       timeoutSeconds: 0    　　    #对容器健康检查探测等待响应的超时时间，单位秒，默认1秒
       periodSeconds: 0     　　    #对容器监控检查的定期探测时间设置，单位秒，默认10秒一次
       successThreshold: 0
       failureThreshold: 0
       securityContext:
         privileged: false
  restartPolicy: [Always | Never | OnFailure]  #Pod的重启策略
  nodeName: <string> #设置NodeName表示将该Pod调度到指定到名称的node节点上
  nodeSelector: obeject #设置NodeSelector表示将该Pod调度到包含这个label的node上
  imagePullSecrets: #Pull镜像时使用的secret名称，以key：secretkey格式指定
  - name: string
  hostNetwork: false   #是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络
  volumes:   #在该pod上定义共享存储卷列表
  - name: string    #共享存储卷名称 （volumes类型有很多种）
    emptyDir: {}    #类型为emtyDir的存储卷，与Pod同生命周期的一个临时目录。为空值
    hostPath: string   #类型为hostPath的存储卷，表示挂载Pod所在宿主机的目录
      path: string      　    #Pod所在宿主机的目录，将被用于同期中mount的目录
    secret:      　#类型为secret的存储卷，挂载集群与定义的secret对象到容器内部
      scretname: string  
      items:     
      - key: string
        path: string
    configMap:   #类型为configMap的存储卷，挂载预定义的configMap对象到容器内部
      name: string
      items:
      - key: string
        path: string
```

### kubectl帮助

\#小提示：

\#  在这里，可通过一个命令来查看每种资源的可配置项

\#  kubectl explain 资源类型     查看某种资源可以配置的一级属性

\#  kubectl explain 资源类型.属性   查看属性的子属性

```bash
[root@master01 ~/yaml]# kubectl explain pod
KIND:     Pod
VERSION:  v1

DESCRIPTION:
     Pod is a collection of containers that can run on a host. This resource is
     created by clients and scheduled onto hosts.

FIELDS:
   apiVersion	<string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

   kind	<string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

   metadata	<Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   spec	<Object>
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

   status	<Object>
     Most recently observed status of the pod. This data may not be up to date.
     Populated by the system. Read-only. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
[root@master01 ~/yaml]# kubectl explain pod.kind
…
[root@master01 ~/yaml]# kubectl explain pod.metadata
…

```

```bash
在kubernetes中基本所有资源的一级属性都是一样的，主要包含5部分：
- apiVersion   \<string> 版本，由kubernetes内部定义，版本号必须可以用 kubectl api-versions 查询到
- kind \<string>         类型，由kubernetes内部定义，版本号必须可以用 kubectl api-resources 查询到
- metadata   \<Object> 元数据，主要是资源标识和说明，常用的有name、namespace、labels等
- spec \<Object>       描述，这是配置中最重要的一部分，里面是对各种资源配置的详细描述                
- status  \<Object>     状态信息，里面的内容不需要定义，由kubernetes自动生成


在上面的属性中，spec是接下来研究的重点，继续看下它的常见子属性:
- containers   <[]Object>   容器列表，用于定义容器的详细信息 
- nodeName \<String>      根据nodeName的值将pod调度到指定的Node节点上
- nodeSelector   <map[]>  根据NodeSelector中定义的信息选择将该Pod调度到包含这些label的Node 上
- hostNetwork  \<boolean> 是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络
- volumes  <[]Object>     存储卷，用于定义Pod上面挂在的存储信息 
- restartPolicy	\<string>     重启策略，表示Pod在遇到故障的时候的处理策略
```

## Pod配置

```bash
[root@master01 ~/yaml]# kubectl explain pod.spec.containers
KIND:     Pod
VERSION:  v1
RESOURCE: containers <[]Object>   # 数组，代表可以有多个容器
FIELDS:
   name  <string>     # 容器名称
   image <string>     # 容器需要的镜像地址
   imagePullPolicy  <string> # 镜像拉取策略  一般docker 阿里都可以
   command  <[]string> # 容器的启动命令列表，如不指定，使用打包时使用的启动命令
   args     <[]string> # 容器的启动命令需要的参数列表
   env      <[]Object> # 容器环境变量的配置
   ports    <[]Object>     # 容器需要暴露的端口号列表
   resources <Object>      # 资源限制和资源请求的设置
上面列出的只是常见的
```

```bash
[root@master01 ~/yaml]# cat pod-base.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-base
  namespace: dev
  labels:
    user: test

spec:
  containers:
  - name: nginx
    image: nginx:latest
  - name: busybox
    image: busybox:latest
[root@master01 ~/yaml]# kubectl create -f pod-base.yaml 
pod/pod-base created
[root@master01 ~/yaml]# kubectl get pods -n dev
NAME                     READY   STATUS     RESTARTS   AGE
nginx                    1/1     Running    2          19h
nginx-5bdb994596-lq5pq   1/1     Running    1          12h
nginx-5bdb994596-spppf   1/1     Running    1          12h
nginx-5bdb994596-wllzv   1/1     Running    1          12h
nginx-labels             1/1     Running    2          18h
pod-base                 1/2     NotReady   0          22s
[root@master01 ~/yaml]# kubectl get pod -n dev
NAME                     READY   STATUS             RESTARTS   AGE
nginx                    1/1     Running            2          19h
nginx-5bdb994596-lq5pq   1/1     Running            1          12h
nginx-5bdb994596-spppf   1/1     Running            1          12h
nginx-5bdb994596-wllzv   1/1     Running            1          12h
nginx-labels             1/1     Running            2          18h
pod-base                 1/2     CrashLoopBackOff   2          55s
```

从上面可以看到已经创建了2个容器一个已经准备好了，另外一个出问题了。

![image-20210819104712280](kubernetes操作实战.assets\image-20210819104712280.png)

镜像也pull了但是就是不准备就绪是因为没有启动命令导致busybox启动之后就自动关机了

在后面命令中我们继续这个例子

### 镜像拉取

```bash
[root@master01 ~/yaml]# cat pod-imagepullpolicy.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-imagepullpolicy
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.19.1
    imagePullPolicy: Always
  - name: busybox
    image: busybox:latest
[root@master01 ~/yaml]# kubectl create -f pod-imagepullpolicy.yaml 
pod/pod-imagepullpolicy created
```

拉取过程

```bash
[root@master01 ~/yaml]# kubectl describe pod pod-imagepullpolicy -n dev
…
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  92s                default-scheduler  Successfully assigned dev/pod-imagepullpolicy to node01
  Normal   Pulling    90s                kubelet            Pulling image "nginx:1.19.1"
  Normal   Pulled     61s                kubelet            Successfully pulled image "nginx:1.19.1" in 29.051333965s
  Normal   Created    60s                kubelet            Created container nginx
  Normal   Started    60s                kubelet            Started container nginx
  Normal   Pulled     56s                kubelet            Successfully pulled image "busybox:latest" in 4.027782315s
  Normal   Pulled     51s                kubelet            Successfully pulled image "busybox:latest" in 3.894751555s
  Normal   Pulled     33s                kubelet            Successfully pulled image "busybox:latest" in 3.704859024s
  Normal   Created    32s (x3 over 56s)  kubelet            Created container busybox
  Normal   Started    32s (x3 over 56s)  kubelet            Started container busybox
  Warning  BackOff    20s (x4 over 50s)  kubelet            Back-off restarting failed container
  Normal   Pulling    6s (x4 over 60s)   kubelet            Pulling image "busybox:latest"
```

> imagePullPolicy，用于设置镜像拉取策略，kubernetes支持配置三种拉取策略：
>
> \- Always：总是从远程仓库拉取镜像（一直远程下载）
>
> \- IfNotPresent：本地有则使用本地镜像，本地没有则从远程仓库拉取镜像（本地有就本地 本地没远程下载）
>
> \- Never：只使用本地镜像，从不去远程仓库拉取，本地没有就报错 （一直使用本地）
>
> 默认值说明：
>
>   如果镜像tag为具体版本号， 默认策略是：IfNotPresent
>
> ​    如果镜像tag为：latest（最终版本） ，默认策略是always

### 启动命令

```bash
[root@master01 ~/yaml]# cat pod-base.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-base
  namespace: dev
  labels:
    user: test

spec:
  containers:
  - name: nginx
    image: nginx:latest
  - name: busybox
    image: busybox:latest
command: ["/bin/sh","-c","touch /tmp/hello.txt;while true;do /bin/echo $(date +%T)>>/tmp/hello.txt;sleep 3; done;"] 
[root@master01 ~/yaml]# kubectl create -f pod-base.yaml 
pod/pod-base created
[root@master01 ~/yaml]# kubectl get pod -n dev
NAME       READY   STATUS    RESTARTS   AGE
pod-base   1/1     Running   0          17s
[root@master01 ~/yaml]# kubectl exec pod-base -n dev -it -c busybox /bin/sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
/ # tail -F /tmp/hello.txt
```

补充一个命令: kubectl exec pod名称 -n 命名空间 -it -c 容器名称 /bin/sh 在容器内部执行命令

特别说明：

  通过上面发现command已经可以完成启动命令和传递参数的功能，为什么这里还要提供一个args选项，用于传递参数呢?这其实跟docker有点关系，kubernetes中的command、args两项其实是实现覆盖Dockerfile中ENTRYPOINT的功能。

 1 如果command和args均没有写，那么用Dockerfile的配置。

 2 如果command写了，但args没有写，那么Dockerfile默认的配置会被忽略，执行输入的command

 3 如果command没写，但args写了，那么Dockerfile中配置的ENTRYPOINT的命令会被执行，使用当前args的参数

 4 如果command和args都写了，那么Dockerfile的配置被忽略，执行command并追加上args参数

### 环境变量

```bash
[root@master01 ~/yaml]# cat pod-env.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-env
  namespace: dev
spec:
  containers:
  - name: busybox
    image: busybox:latest
    command: ["/bin/sh","-c","touch /tmp/hello.txt;while true;do /bin/echo $(date +%T)>>/tmp/hello.txt; sleep; done"]
    env:
      - name: "usern"
        value: "wang"
      - name: "passwd"
        value: "123456"
[root@master01 ~/yaml]# kubectl create -f pod-env.yaml 
pod/pod-env created
[root@master01 ~/yaml]# kubectl get pod -n dev
NAME       READY   STATUS    RESTARTS   AGE
pod-base   1/1     Running   0          20m
pod-env    1/1     Running   0          10s
[root@master01 ~/yaml]# kubectl exec pod-env -n dev -c  busybox -it /bin/sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
/ # echo $usern
wang
/ # echo $passwd
123456
/ #
```

### 端口设置

```bash
[root@master01 ~/yaml]# kubectl explain pod.spec.containers.ports
KIND:     Pod
VERSION:  v1

RESOURCE: ports <[]Object>

DESCRIPTION:
     List of ports to expose from the container. Exposing a port here gives the
     system additional information about the network connections a container
     uses, but is primarily informational. Not specifying a port here DOES NOT
     prevent that port from being exposed. Any port which is listening on the
     default "0.0.0.0" address inside a container will be accessible from the
     network. Cannot be updated.

     ContainerPort represents a network port in a single container.

FIELDS:
   containerPort	<integer> -required- # 容器要监听的端口(0<x<65536)
     Number of port to expose on the pod's IP address. This must be a valid port
     number, 0 < x < 65536.

   hostIP	<string># 要将外部端口绑定到的主机IP(一般省略)
     What host IP to bind the external port to.

   hostPort	<integer># 容器要在主机上公开的端口，如果设置，主机上只能运行容器的一个副本(一般省略)
     Number of port to expose on the host. If specified, this must be a valid
     port number, 0 < x < 65536. If HostNetwork is specified, this must match
     ContainerPort. Most containers do not need this.

   name	<string> # 端口名称，如果指定，必须保证name在pod中是唯一的	
     If specified, this must be an IANA_SVC_NAME and unique within the pod. Each
     named port in a pod must have a unique name. Name for the port that can be
     referred to by services.

   protocol	<string># 端口协议。必须是UDP、TCP或SCTP。默认为“TCP”
     Protocol for port. Must be UDP, TCP, or SCTP. Defaults to "TCP".
```

```bash
[root@master01 ~/yaml]# cat pod-ports.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-ports
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - name: nginx-port
      containerPort: 80
      protocol: TCP
 
[root@master01 ~/yaml]# kubectl create -f pod-ports.yaml 
pod/pod-ports created
```

![image-20210819104957457](kubernetes操作实战.assets\image-20210819104957457.png)

### 资源配额

容器中的程序要运行，肯定是要占用一定资源的，比如cpu和内存等，如果不对某个容器的资源做限制，那么它就可能吃掉大量资源，导致其它容器无法运行。针对这种情况，kubernetes提供了对内存和cpu的资源进行配额的机制，这种机制主要通过resources选项实现，他有两个子选项：

\- limits：用于限制运行时容器的最大占用资源，当容器占用资源超过limits时会被终止，并进行重启

\- requests ：用于设置容器需要的最小资源，如果环境资源不够，容器将无法启动

可以通过上面两个选项设置资源的上下限。

```bash
[root@master01 ~/yaml]# cat pod-resources.yaml 
apiVersion: v1
kind: Pod
metadata: 
  name: pod-resources
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:latest
    resources:
      limits: 
        cpu: "2"
        memory: "200Mi"
      requests:
        cpu: "1"
        memory: "100Mi"
[root@master01 ~/yaml]# kubectl create -f pod-resources.yaml 
pod/pod-resources created
[root@master01 ~/yaml]# kubectl get pod pod-resources -n dev
NAME            READY   STATUS    RESTARTS   AGE
pod-resources   1/1     Running   0          30s
```

设置超过物理资源会导致container无法启动

## pod生命周期

我们一般将pod对象从创建至终的这段时间范围称为pod的生命周期，它主要包含下面的过程：

\- pod创建过程

\- 运行初始化容器（init container）过程

\- 运行主容器（main container）

 \- 容器启动后钩子（post start）、容器终止前钩子（pre stop）

 \- 容器的存活性探测（liveness probe）、就绪性探测（readiness probe）

\- pod终止过程

![image-20210819105149567](kubernetes操作实战.assets\image-20210819105149567.png)

在整个生命周期中，Pod会出现5种状态（相位），分别如下：

\- 挂起（Pending）：apiserver已经创建了pod资源对象，但它尚未被调度完成或者仍处于下载镜像的过程中

\- 运行中（Running）：pod已经被调度至某节点，并且所有容器都已经被kubelet创建完成

\- 成功（Succeeded）：pod中的所有容器都已经成功终止并且不会被重启

\- 失败（Failed）：所有容器都已经终止，但至少有一个容器终止失败，即容器返回了非0值的退出状态

\- 未知（Unknown）：apiserver无法正常获取到pod对象的状态信息，通常由网络通信失败所导致

### 创建和终止

pod的创建过程

1. 用户通过kubectl或其他api客户端提交需要创建的pod信息给apiServer

2. apiServer开始生成pod对象的信息，并将信息存入etcd，然后返回确认信息至客户端

3. apiServer开始反映etcd中的pod对象的变化，其它组件使用watch机制来跟踪检查apiServer上的变动

4. scheduler发现有新的pod对象要创建，开始为Pod分配主机并将结果信息更新至apiServer

5. node节点上的kubelet发现有pod调度过来，尝试调用docker启动容器，并将结果回送至apiServer

6. apiServer将接收到的pod状态信息存入etcd中

![image-20210819105236310](kubernetes操作实战.assets\image-20210819105236310.png)

pod的终止过程

1. 用户向apiServer发送删除pod对象的命令

2. apiServcer中的pod对象信息会随着时间的推移而更新，在宽限期内（默认30s），pod被视为dead
3.  将pod标记为terminating状态

4. kubelet在监控到pod对象转为terminating状态的同时启动pod关闭过程

5. 端点控制器监控到pod对象的关闭行为时将其从所有匹配到此端点的service资源的端点列表中移除

6. 如果当前pod对象定义了preStop钩子处理器，则在其标记为terminating后即会以同步的方式启动执行

7. pod对象中的容器进程收到停止信号

8. 宽限期结束后，若pod中还存在仍在运行的进程，那么pod对象会收到立即终止的信号

9. kubelet请求apiServer将此pod资源的宽限期设置为0从而完成删除操作，此时pod对于用户已不可见

#### 初始化容器

初始化容器是在pod的主容器启动之前要运行的容器，主要是做一些主容器的前置工作，它具有两大特征：

1. 初始化容器必须运行完成直至结束，若某初始化容器运行失败，那么kubernetes需要重启它直到成功完成

2. 初始化容器必须按照定义的顺序执行，当且仅当前一个成功之后，后面的一个才能运行

初始化容器有很多的应用场景，下面列出的是最常见的几个：

\- 提供主容器镜像中不具备的工具程序或自定义代码

\- 初始化容器要先于应用容器串行启动并运行完成，因此可用于延后应用容器的启动直至其依赖的条件得到满足

接下来做一个案例，模拟下面这个需求：

假设要以主容器来运行nginx，但是要求在运行nginx之前先要能够连接上mysql和redis所在服务器

  为了简化测试，事先规定好mysql`(10.244.196.179)`和redis`(10.244.196.179)`服务器的地址   这个IP地址只是测试能不能ping通

```bash
[root@master01 ~/yaml]# cat pod-initcontainer.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-initcontainer
  namespace: dev
spec:
  containers:
  - name: main-container
    image: nginx:latest
    ports:
    - name: nginx-port
      containerPort: 80
  initContainers:
    - name: test-mysql
      image: busybox:latest
      command: ['sh','-c','until ping 10.244.196.179 -c 1;do echo waiting for mysql ...;sleep 2;done;']
    - name: test-redis
      image: busybox:latest
      command: ['sh','-c','until ping 10.244.196.179 -c 1;do echo waiting for reide...;sleep 2; done;'] 
      
# 创建pod
[root@master ~]# kubectl create -f pod-initcontainer.yaml
pod/pod-initcontainer created

# 查看pod状态
# 发现pod卡在启动第一个初始化容器过程中，后面的容器不会运行
root@master ~]# kubectl describe pod  pod-initcontainer -n dev
........
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  49s   default-scheduler  Successfully assigned dev/pod-initcontainer to node1
  Normal  Pulled     48s   kubelet, node1     Container image "busybox:1.30" already present on machine
  Normal  Created    48s   kubelet, node1     Created container test-mysql
  Normal  Started    48s   kubelet, node1     Started container test-mysql
```

```bash
# 动态查看pod
[root@master ~]# kubectl get pods pod-initcontainer -n dev -w
NAME                             READY   STATUS     RESTARTS   AGE
pod-initcontainer                0/1     Init:0/2   0          15s
pod-initcontainer                0/1     Init:1/2   0          52s
pod-initcontainer                0/1     Init:1/2   0          53s
pod-initcontainer                0/1     PodInitializing   0          89s
pod-initcontainer                1/1     Running           0          90s

# 接下来新开一个shell，为当前服务器新增两个ip，观察pod的变化
[root@master ~]# ifconfig ens33:1 192.168.109.201 netmask 255.255.255.0 up
[root@master ~]# ifconfig ens33:2 192.168.109.202 netmask 255.255.255.0 up

```

#### 钩子函数

 

钩子函数能够感知自身生命周期中的事件，并在相应的时刻到来时运行用户指定的程序代码。

kubernetes在主容器的启动之后和停止之前提供了两个钩子函数：

\- post start：容器创建之后执行，如果失败了会重启容器

\- pre stop ：容器终止之前执行，执行完成之后容器将成功终止，在其完成之前会阻塞删除容器的操作

钩子处理器支持使用下面三种方式定义动作：

\- Exec命令：在容器内执行一次命令

```bash
……
  lifecycle:
    postStart: 
      exec:
        command:
        - cat
        - /tmp/healthy
……
```

-CPSocket：在当前容器尝试访问指定的socket

```bash
……      
  lifecycle:
    postStart:
      tcpSocket:
        port: 8080
……
```

HTTPGet：在当前容器中向某url发起http请求

```bash
……
  lifecycle:
    postStart:
      httpGet:
        path: / #URI地址
        port: 80 #端口号
        host: 192.168.109.100 #主机地址
        scheme: HTTP #支持的协议，http或者https
……
```

接下来，以exec方式为例，演示下钩子函数的使用，创建pod-hook-exec.yaml文件，内容如下：

```bash
[root@master01 ~/yaml]# cat pod-hook-exec.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-hook-exec
  namespace: dev
spec:
  containers:
  - name: main-container
    image: nginx:latest
    ports:
    - name: nginx-port
      containerPort: 80
    lifecycle:
      postStart:
        exec:  # 在容器启动的时候执行一个命令，修改掉nginx的默认首页内容
          command: ['/bin/sh','-c','echo postStart...>/usr/share/nginx/html/index.html']
      preStop:
        exec: # 在容器停止之前停止nginx服务
          command: ['/usr/sbin/nginx','-s','quit']
[root@master01 ~/yaml]# kubectl create -f pod-hook-exec.yaml 
pod/pod-hook-exec created
[root@master01 ~/yaml]# kubectl get pod -n dev -o wide
NAME                READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
pod-base            1/1     Running   0          62m   10.244.196.179   node01   <none>           <none>
pod-hook-exec       1/1     Running   0          36s   10.244.196.181   node01   <none>           <none>
pod-initcontainer   1/1     Running   0          49m   10.244.196.180   node01   <none>           <none>
[root@master01 ~/yaml]# curl 10.244.196.181
postStart...
```

#### 容器探测

容器探测用于检测容器中的应用实例是否正常工作，是保障业务可用性的一种传统机制。如果经过探测，实例的状态不符合预期，那么kubernetes就会把该问题实例" 摘除 "，不承担业务流量。kubernetes提供了两种探针来实现容器探测，分别是：

\- liveness probes：存活性探针，用于检测应用实例当前是否处于正常运行状态，如果不是，k8s会重启容器

\- readiness probes：就绪性探针，用于检测应用实例当前是否可以接收请求，如果不能，k8s不会转发流量

\> livenessProbe 决定是否重启容器，readinessProbe 决定是否将请求转发给容器。

上面两种探针目前均支持三种探测方式：

\- Exec命令：在容器内执行一次命令，如果命令执行的退出码为0，则认为程序正常，否则不正常

```bash
……
  livenessProbe:
    exec:
      command:
      - cat
      - /tmp/healthy
……
```

TCPSocket：将会尝试访问一个用户容器的端口，如果能够建立这条连接，则认为程序正常，否则不正常

```bash
……      
  livenessProbe:
    tcpSocket:
      port: 8080
……
```

HTTPGet：调用容器内Web应用的URL，如果返回的状态码在200和399之间，则认为程序正常，否则不正常

```bash
……
  livenessProbe:
    httpGet:
      path: / #URI地址
      port: 80 #端口号
      host: 127.0.0.1 #主机地址
      scheme: HTTP #支持的协议，http或者https
……
```

下面以liveness probes为例，做几个演示：

方式1：exec

```bash
[root@master01 ~/yaml]# cat pod-liveness-exec.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-liveness-exec
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - name: nginx-ports
      containerPort: 80
    livenessProbe:
      exec:
        command: ["/bin/cat","/tmp/hello.txt"]
[root@master01 ~/yaml]# kubectl create -f pod-liveness-exec.yaml 
pod/pod-liveness-exec created
[root@master01 ~/yaml]# kubectl describe pods pod-liveness-exec -n dev
…
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  30s                default-scheduler  Successfully assigned dev/pod-liveness-exec to node01
  Normal   Pulling    27s                kubelet            Pulling image "nginx:latest"
  Normal   Pulled     23s                kubelet            Successfully pulled image "nginx:latest" in 4.677305533s
  Normal   Created    23s                kubelet            Created container nginx
  Normal   Started    22s                kubelet            Started container nginx
  Warning  Unhealthy  10s (x2 over 19s)  kubelet            Liveness probe failed: /bin/cat: /tmp/hello.txt: No such file or directory
```

\# 观察上面的信息就会发现nginx容器启动之后就进行了健康检查

\# 检查失败之后，容器被kill掉，然后尝试进行重启（这是重启策略的作用，后面讲解）

\# 稍等一会之后，再观察pod信息，就可以看到RESTARTS不再是0，而是一直增长

```bash
[root@master01 ~/yaml]# kubectl get pod -n dev
NAME                READY   STATUS    RESTARTS   AGE
pod-liveness-exec   1/1     Running   4          2m23s
[root@master01 ~/yaml]# kubectl get pod -n dev
NAME                READY   STATUS             RESTARTS   AGE
pod-liveness-exec   0/1     CrashLoopBackOff   4          2m59s
[root@master01 ~/yaml]# kubectl get pod -n dev
NAME                READY   STATUS             RESTARTS   AGE
pod-liveness-exec   0/1     CrashLoopBackOff   4          3m11s
[root@master01 ~/yaml]# kubectl get pod -n dev
NAME                READY   STATUS             RESTARTS   AGE
pod-liveness-exec   0/1     CrashLoopBackOff   5          4m1s
```

\# 当然接下来，可以修改成一个存在的文件，比如/tmp/hello.txt，再试，结果就正常了......

方式2 port

创建pod-liveness-tcpsocket.yaml

```bash
apiVersion: v1
kind: Pod
metadata:
  name: pod-liveness-tcpsocket
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports: 
    - name: nginx-port
      containerPort: 80
    livenessProbe:
      tcpSocket:
        port: 8080 # 尝试访问8080端口
[root@master ~]# kubectl create -f pod-liveness-tcpsocket.yaml
[root@master ~]# kubectl describe pods pod-liveness-tcpsocket -n dev
…
Warning  Unhealthy  <invalid> (x2 over <invalid>)  kubelet, node2     Liveness probe failed: dial tcp 10.244.2.44:8080: connect: connection refused
# 观察上面的信息，发现尝试访问8080端口,但是失败了
# 稍等一会之后，再观察pod信息，就可以看到RESTARTS不再是0，而是一直增长
[root@master ~]# kubectl get pods pod-liveness-tcpsocket  -n dev
NAME                     READY   STATUS             RESTARTS   AGE
pod-liveness-tcpsocket   0/1     CrashLoopBackOff   2          3m19s
# 当然接下来，可以修改成一个可以访问的端口，比如80，再试，结果就正常了......
```

方式3：httpget

创建pod-liveness-httpget.yaml

```bash
apiVersion: v1
kind: Pod
metadata:
  name: pod-liveness-httpget
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - name: nginx-port
      containerPort: 80
    livenessProbe:
      httpGet:  # 其实就是访问http://127.0.0.1:80/hello  
        scheme: HTTP #支持的协议，http或者https
        port: 80 #端口号
        path: /hello #URI地址
[root@master ~]# kubectl create -f pod-liveness-httpget.yaml
# 查看Pod详情
[root@master ~]# kubectl describe pod pod-liveness-httpget -n dev
…
Warning  Unhealthy  6s (x6 over 56s)  kubelet, node1     Liveness probe failed: HTTP probe failed with statuscode: 404
  Normal   Killing    6s (x2 over 36s)  kubelet, node1     Container nginx failed liveness probe, will be restarted
# 观察上面信息，尝试访问路径，但是未找到,出现404错误
# 稍等一会之后，再观察pod信息，就可以看到RESTARTS不再是0，而是一直增长
[root@master ~]# kubectl get pod pod-liveness-httpget -n dev
NAME                   READY   STATUS    RESTARTS   AGE
pod-liveness-httpget   1/1     Running   5          3m17s
 过一会儿running就变成其他的了
# 当然接下来，可以修改成一个可以访问的路径path，比如/，再试，结果就正常了......
```

至此，已经使用liveness Probe演示了三种探测方式，但是查看livenessProbe的子属性，会发现除了这三种方式，还有一些其他的配置，在这里一并解释下：

```bash
[root@master ~]# kubectl explain pod.spec.containers.livenessProbe
FIELDS:
   exec <Object>  
   tcpSocket    <Object>
   httpGet      <Object>
   initialDelaySeconds  <integer>  # 容器启动后等待多少秒执行第一次探测
   timeoutSeconds       <integer>  # 探测超时时间。默认1秒，最小1秒
   periodSeconds        <integer>  # 执行探测的频率。默认是10秒，最小1秒
   failureThreshold     <integer>  # 连续探测失败多少次才被认定为失败。默认是3。最小值是1
   successThreshold     <integer>  # 连续探测成功多少次才被认定为成功。默认是1
[root@master ~]# more pod-liveness-httpget.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-liveness-httpget
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - name: nginx-port
      containerPort: 80
    livenessProbe:
      httpGet:
        scheme: HTTP
        port: 80 
        path: /
      initialDelaySeconds: 30 # 容器启动后30s开始探测
      timeoutSeconds: 5 # 探测超时时间为5s
```

重启策略

在上一节中，一旦容器探测出现了问题，kubernetes就会对容器所在的Pod进行重启，其实这是由pod的重启策略决定的，pod的重启策略有 3 种，分别如下：

\- Always ：容器失效时，自动重启该容器，这也是默认值。

\- OnFailure ： 容器终止运行且退出码不为0时重启

\- Never ： 不论状态为何，都不重启该容器

  重启策略适用于pod对象中的所有容器，首次需要重启的容器，将在其需要时立即进行重启，随后再次需要重启的操作将由kubelet延迟一段时间后进行，且反复的重启操作的延迟时长以此为10s、20s、40s、80s、160s和300s，300s是最大延迟时长。

创建pod-restartpolicy.yaml：

```bash
apiVersion: v1
kind: Pod
metadata:
  name: pod-restartpolicy
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - name: nginx-port
      containerPort: 80
    livenessProbe:
      httpGet:
        scheme: HTTP
        port: 80
        path: /hello
  restartPolicy: Never # 设置重启策略为Never
# 创建Pod
[root@master ~]# kubectl create -f pod-restartpolicy.yaml
pod/pod-restartpolicy created
# 查看Pod详情，发现nginx容器失败
[root@master ~]# kubectl  describe pods pod-restartpolicy  -n dev
......
  Warning  Unhealthy  15s (x3 over 35s)  kubelet, node1     Liveness probe failed: HTTP probe failed with statuscode: 404
  Normal   Killing    15s                kubelet, node1     Container nginx failed liveness probe
  
# 多等一会，再观察pod的重启次数，发现一直是0，并未重启   
[root@master ~]# kubectl  get pods pod-restartpolicy -n dev
NAME                   READY   STATUS    RESTARTS   AGE
pod-restartpolicy      0/1     Running   0          5min42s
```

## pod调度

在默认情况下，一个Pod在哪个Node节点上运行，是由Scheduler组件采用相应的算法计算出来的，这个过程是不受人工控制的。但是在实际使用中，这并不满足的需求，因为很多情况下，我们想控制某些Pod到达某些节点上，那么应该怎么做呢？这就要求了解kubernetes对Pod的调度规则，kubernetes提供了四大类调度方式：

\- 自动调度：运行在哪个节点上完全由Scheduler经过一系列的算法计算得出

\- 定向调度：NodeName、NodeSelector

\- 亲和性调度：NodeAffinity、PodAffinity、PodAntiAffinity

\- 污点（容忍）调度：Taints、Toleration

### 定向调度

  定向调度，指的是利用在pod上声明nodeName或者nodeSelector，以此将Pod调度到期望的node节点上。注意，这里的调度是强制的，这就意味着即使要调度的目标Node不存在，也会向上面进行调度，只不过pod运行失败而已。

**NodeName**

  NodeName用于强制约束将Pod调度到指定的Name的Node节点上。这种方式，其实是直接跳过Scheduler的调度逻辑，直接将Pod调度到指定名称的节点。

接下来，实验一下：创建一个pod-nodename.yaml文件

```bash
[root@master01 ~/yaml]# cat pod-nodename.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodename
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:latest
  nodeName: master02

[root@master01 ~/yaml]# kubectl create -f pod-nodename.yaml 
pod/pod-nodename created
[root@master01 ~/yaml]# kubectl get pods -n dev -o wide
NAME           READY   STATUS    RESTARTS   AGE     IP              NODE       NOMINATE
pod-nodename   1/1     Running   0          2m45s   10.244.59.203   master02   <none>  
 # 接下来，删除pod，修改nodeName的值为node3（并没有node3节点）
[root@master ~]# kubectl delete -f pod-nodename.yaml
pod "pod-nodename" deleted
[root@master ~]# vim pod-nodename.yaml
[root@master ~]# kubectl create -f pod-nodename.yaml
pod/pod-nodename created

#再次查看，发现已经向Node3节点调度，但是由于不存在node3节点，所以pod无法正常运行
[root@master ~]# kubectl get pods pod-nodename -n dev -o wide
NAME           READY   STATUS    RESTARTS   AGE   IP       NODE    ......
pod-nodename   0/1     Pending   0          6s    <none>   node3   ......  
```

**NodeSelector**

  NodeSelector用于将pod调度到添加了指定标签的node节点上。它是通过kubernetes的label-selector机制实现的，也就是说，在pod创建之前，会由scheduler使用MatchNodeSelector调度策略进行label匹配，找出目标node，然后将pod调度到目标节点，该匹配规则是强制约束。

接下来，实验一下：

1 首先分别为node节点添加标签

```bash
[root@master ~]# kubectl label nodes node1 nodeenv=pro
node/node1 labeled
[root@master ~]# kubectl label nodes node2 nodeenv=test
node/node2 labeled
```

2 创建一个pod-nodeselector.yaml文件，并使用它创建Pod

```bash
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodeselector
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:latest
  nodeSelector: 
    nodeenv: pro # 指定调度到具有nodeenv=pro标签的节点上
#创建Pod
[root@master ~]# kubectl create -f pod-nodeselector.yaml
pod/pod-nodeselector created

#查看Pod调度到NODE属性，确实是调度到了node1节点上
[root@master ~]# kubectl get pods pod-nodeselector -n dev -o wide
NAME               READY   STATUS    RESTARTS   AGE     IP          NODE    ......
pod-nodeselector   1/1     Running   0          47s   10.244.1.87   node1   ......

# 接下来，删除pod，修改nodeSelector的值为nodeenv: xxxx（不存在打有此标签的节点）
[root@master ~]# kubectl delete -f pod-nodeselector.yaml
pod "pod-nodeselector" deleted
[root@master ~]# vim pod-nodeselector.yaml
[root@master ~]# kubectl create -f pod-nodeselector.yaml
pod/pod-nodeselector created

#再次查看，发现pod无法正常运行,Node的值为none
[root@master ~]# kubectl get pods -n dev -o wide
NAME               READY   STATUS    RESTARTS   AGE     IP       NODE    
pod-nodeselector   0/1     Pending   0          2m20s   <none>   <none>

# 查看详情,发现node selector匹配失败的提示
[root@master ~]# kubectl describe pods pod-nodeselector -n dev
.......
Events:
  Type     Reason            Age        From               Message
  ----     ------            ----       ----               -------
  Warning  FailedScheduling  <unknown>  default-scheduler  0/3 nodes are available: 3 node(s) didn't match node selector.
  Warning  FailedScheduling  <unknown>  default-scheduler  0/3 nodes are available: 3 node(s) didn't match node selector.
```

### 亲和性调度

  上面介绍了两种定向调度的方式，使用起来非常方便，但是也有一定的问题，那就是如果没有满足条件的Node，那么Pod将不会被运行，即使在集群中还有可用Node列表也不行，这就限制了它的使用场景。

  基于上面的问题，kubernetes还提供了一种亲和性调度（Affinity）。它在NodeSelector的基础之上的进行了扩展，可以通过配置的形式，实现优先选择满足条件的Node进行调度，如果没有，也可以调度到不满足条件的节点上，使调度更加灵活。

Affinity主要分为三类：

\- nodeAffinity(node亲和性）: 以node为目标，解决pod可以调度到哪些node的问题

\- podAffinity(pod亲和性) : 以pod为目标，解决pod可以和哪些已存在的pod部署在同一个拓扑域中的问题

\- podAntiAffinity(pod反亲和性) : 以pod为目标，解决pod不能和哪些已存在pod部署在同一个拓扑域中的问题

\> 关于亲和性(反亲和性)使用场景的说明：

\> **亲和性**：如果两个应用频繁交互，那就有必要利用亲和性让两个应用的尽可能的靠近，这样可以减少因网络通信而带来的性能损耗。

\>**反亲和性**：当应用的采用多副本部署时，有必要采用反亲和性让各个应用实例打散分布在各个node上，这样可以提高服务的高可用性。

#### NodeAffinity

首先来看一下`NodeAffinity`的可配置项：

```bash
pod.spec.affinity.nodeAffinity
  requiredDuringSchedulingIgnoredDuringExecution  Node节点必须满足指定的所有规则才可以，相当于硬限制
    nodeSelectorTerms  节点选择列表
      matchFields   按节点字段列出的节点选择器要求列表
      matchExpressions   按节点标签列出的节点选择器要求列表(推荐)
        key    键
        values 值
        operator 关系符 支持Exists, DoesNotExist, In, NotIn, Gt, Lt
  preferredDuringSchedulingIgnoredDuringExecution 优先调度到满足指定的规则的Node，相当于软限制 (倾向)
    preference   一个节点选择器项，与相应的权重相关联
      matchFields   按节点字段列出的节点选择器要求列表
      matchExpressions   按节点标签列出的节点选择器要求列表(推荐)
        key    键
        values 值
        operator 关系符 支持In, NotIn, Exists, DoesNotExist, Gt, Lt
	weight 倾向权重，在范围1-100。
```

```bash
关系符的使用说明:
- matchExpressions:
  - key: nodeenv              # 匹配存在标签的key为nodeenv的节点
    operator: Exists
  - key: nodeenv              # 匹配标签的key为nodeenv,且value是"xxx"或"yyy"的节点
    operator: In
    values: ["xxx","yyy"]
  - key: nodeenv              # 匹配标签的key为nodeenv,且value大于"xxx"的节点
    operator: Gt
    values: "xxx"
```

接下来首先演示一下`requiredDuringSchedulingIgnoredDuringExecution` ,

创建pod-nodeaffinity-required.yam

```bash
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodeaffinity-required
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
  affinity:  #亲和性设置
    nodeAffinity: #设置node亲和性
      requiredDuringSchedulingIgnoredDuringExecution: # 硬限制
        nodeSelectorTerms:
        - matchExpressions: # 匹配env的值在["xxx","yyy"]中的标签
          - key: nodeenv
            operator: In
            values: ["xxx","yyy"]
# 创建pod
[root@master ~]# kubectl create -f pod-nodeaffinity-required.yaml
pod/pod-nodeaffinity-required created

# 查看pod状态 （运行失败）
[root@master ~]# kubectl get pods pod-nodeaffinity-required -n dev -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP       NODE    ...... 
pod-nodeaffinity-required   0/1     Pending   0          16s   <none>   <none>  ......

# 查看Pod的详情
# 发现调度失败，提示node选择失败
[root@master ~]# kubectl describe pod pod-nodeaffinity-required -n dev
......
  Warning  FailedScheduling  <unknown>  default-scheduler  0/3 nodes are available: 3 node(s) didn't match node selector.
  Warning  FailedScheduling  <unknown>  default-scheduler  0/3 nodes are available: 3 node(s) didn't match node selector.

#接下来，停止pod
[root@master ~]# kubectl delete -f pod-nodeaffinity-required.yaml
pod "pod-nodeaffinity-required" deleted

# 修改文件，将values: ["xxx","yyy"]------> ["pro","yyy"]
[root@master ~]# vim pod-nodeaffinity-required.yaml

# 再次启动
[root@master ~]# kubectl create -f pod-nodeaffinity-required.yaml
pod/pod-nodeaffinity-required created

# 此时查看，发现调度成功，已经将pod调度到了node1上
[root@master ~]# kubectl get pods pod-nodeaffinity-required -n dev -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP            NODE  ...... 
pod-nodeaffinity-required   1/1     Running   0          11s   10.244.1.89   node1 ......
```

上面注意。Master默认是有污点，不能迁移。

 

接下来再演示一下`requiredDuringSchedulingIgnoredDuringExecution` ,

创建pod-nodeaffinity-preferred.yaml

```bash
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodeaffinity-preferred
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
  affinity:  #亲和性设置
    nodeAffinity: #设置node亲和性
      preferredDuringSchedulingIgnoredDuringExecution: # 软限制
      - weight: 1
        preference:
          matchExpressions: # 匹配env的值在["xxx","yyy"]中的标签(当前环境没有)
          - key: nodeenv
            operator: In
            values: ["xxx","yyy"]
# 创建pod
[root@master ~]# kubectl create -f pod-nodeaffinity-preferred.yaml
pod/pod-nodeaffinity-preferred created

# 查看pod状态 （运行成功）
[root@master ~]# kubectl get pod pod-nodeaffinity-preferred -n dev
NAME                         READY   STATUS    RESTARTS   AGE
pod-nodeaffinity-preferred   1/1     Running   0          40s
```

NodeAffinity规则设置的注意事项：

  1 如果同时定义了nodeSelector和nodeAffinity，那么必须两个条件都得到满足，Pod才能运行在指定的Node上

  2 如果nodeAffinity指定了多个nodeSelectorTerms，那么只需要其中一个能够匹配成功即可

  3 如果一个nodeSelectorTerms中有多个matchExpressions ，则一个节点必须满足所有的才能匹配成功

  4 如果一个pod所在的Node在Pod运行期间其标签发生了改变，不再符合该Pod的节点亲和性需求，则系统将忽略此变化

#### **PodAffinity**

PodAffinity主要实现以运行的Pod为参照，实现让新创建的Pod跟参照pod在一个区域的功能。

首先来看一下`PodAffinity`的可配置项：

```bash
pod.spec.affinity.podAffinity
  requiredDuringSchedulingIgnoredDuringExecution  硬限制
    namespaces       指定参照pod的namespace
    topologyKey      指定调度作用域
    labelSelector    标签选择器
      matchExpressions  按节点标签列出的节点选择器要求列表(推荐)
        key    键
        values 值
        operator 关系符 支持In, NotIn, Exists, DoesNotExist.
      matchLabels    指多个matchExpressions映射的内容
  preferredDuringSchedulingIgnoredDuringExecution 软限制
    podAffinityTerm  选项
      namespaces      
      topologyKey
      labelSelector
        matchExpressions  
          key    键
          values 值
          operator
        matchLabels 
    weight 倾向权重，在范围1-100
```

topologyKey用于指定调度时作用域,例如:

  如果指定为kubernetes.io/hostname，那就是以Node节点为区分范围

​    如果指定为beta.kubernetes.io/os,则以Node节点的操作系统类型来区分

接下来，演示下`requiredDuringSchedulingIgnoredDuringExecution`,

1）首先创建一个参照Pod，pod-podaffinity-target.yaml：

```bash
apiVersion: v1
kind: Pod
metadata:
  name: pod-podaffinity-target
  namespace: dev
  labels:
    podenv: pro #设置标签
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
  nodeName: node1 # 将目标pod名确指定到node1上
```

```bash
# 启动目标pod
[root@master ~]# kubectl create -f pod-podaffinity-target.yaml
pod/pod-podaffinity-target created

# 查看pod状况
[root@master ~]# kubectl get pods  pod-podaffinity-target -n dev
NAME                     READY   STATUS    RESTARTS   AGE
pod-podaffinity-target   1/1     Running   0          4s

创建pod-podaffinity-required.yaml，内容如下
apiVersion: v1
kind: Pod
metadata:
  name: pod-podaffinity-required
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
  affinity:  #亲和性设置
    podAffinity: #设置pod亲和性
      requiredDuringSchedulingIgnoredDuringExecution: # 硬限制
      - labelSelector:
          matchExpressions: # 匹配env的值在["xxx","yyy"]中的标签
          - key: podenv
            operator: In
            values: ["xxx","yyy"]
        topologyKey: kubernetes.io/hostname
```

上面配置表达的意思是：新Pod必须要与拥有标签nodeenv=xxx或者nodeenv=yyy的pod在同一Node上，显然现在没有这样pod，接下来，运行测试一下。

```bash
# 启动pod
[root@master ~]# kubectl create -f pod-podaffinity-required.yaml
pod/pod-podaffinity-required created

# 查看pod状态，发现未运行
[root@master ~]# kubectl get pods pod-podaffinity-required -n dev
NAME                       READY   STATUS    RESTARTS   AGE
pod-podaffinity-required   0/1     Pending   0          9s

# 查看详细信息
[root@master ~]# kubectl describe pods pod-podaffinity-required  -n dev
......
Events:
  Type     Reason            Age        From               Message
  ----     ------            ----       ----               -------
  Warning  FailedScheduling  <unknown>  default-scheduler  0/3 nodes are available: 2 node(s) didn't match pod affinity rules, 1 node(s) had taints that the pod didn't tolerate.

# 接下来修改  values: ["xxx","yyy"]----->values:["pro","yyy"]
# 意思是：新Pod必须要与拥有标签nodeenv=xxx或者nodeenv=yyy的pod在同一Node上
[root@master ~]# vim pod-podaffinity-required.yaml

# 然后重新创建pod，查看效果
[root@master ~]# kubectl delete -f  pod-podaffinity-required.yaml
pod "pod-podaffinity-required" deleted
[root@master ~]# kubectl create -f pod-podaffinity-required.yaml
pod/pod-podaffinity-required created

# 发现此时Pod运行正常
[root@master ~]# kubectl get pods pod-podaffinity-required -n dev
NAME                       READY   STATUS    RESTARTS   AGE   LABELS
pod-podaffinity-required   1/1     Running   0          6s    <none>
```

关于`PodAffinity`的 `preferredDuringSchedulingIgnoredDuringExecution`，这里不再演示。

#### **PodAntiAffinity**

PodAntiAffinity主要实现以运行的Pod为参照，让新创建的Pod跟参照pod不在一个区域中的功能。

它的配置方式和选项跟PodAffinty是一样的，这里不再做详细解释，直接做一个测试案例。

1）继续使用上个案例中目标pod

```bash
[root@master ~]# kubectl get pods -n dev -o wide --show-labels
NAME                     READY   STATUS    RESTARTS   AGE     IP            NODE    LABELS
pod-podaffinity-required 1/1     Running   0          3m29s   10.244.1.38   node1   <none>     
pod-podaffinity-target   1/1     Running   0          9m25s   10.244.1.37   node1   podenv=pro
2）创建pod-podantiaffinity-required.yaml，内容如下：
apiVersion: v1
kind: Pod
metadata:
  name: pod-podantiaffinity-required
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
  affinity:  #亲和性设置
    podAntiAffinity: #设置pod亲和性
      requiredDuringSchedulingIgnoredDuringExecution: # 硬限制
      - labelSelector:
          matchExpressions: # 匹配podenv的值在["pro"]中的标签
          - key: podenv
            operator: In
            values: ["pro"]
        topologyKey: kubernetes.io/hostname
```

上面配置表达的意思是：新Pod必须要与拥有标签nodeenv=pro的pod不在同一Node上，运行测试一下。

```bash
# 创建pod
[root@master ~]# kubectl create -f pod-podantiaffinity-required.yaml
pod/pod-podantiaffinity-required created

# 查看pod
# 发现调度到了node2上
[root@master ~]# kubectl get pods pod-podantiaffinity-required -n dev -o wide
NAME                           READY   STATUS    RESTARTS   AGE   IP            NODE   .. 
pod-podantiaffinity-required   1/1     Running   0          30s   10.244.1.96   node2  ..
```

## 污点和容忍

 

### **污点（Taints**）

  前面的调度方式都是站在Pod的角度上，通过在Pod上添加属性，来确定Pod是否要调度到指定的Node上，其实我们也可以站在Node的角度上，通过在Node上添加**污点**属性，来决定是否允许Pod调度过来。

  Node被设置上污点之后就和Pod之间存在了一种相斥的关系，进而拒绝Pod调度进来，甚至可以将已经存在的Pod驱逐出去。

污点的格式为：`key=value:effect`, key和value是污点的标签，effect描述污点的作用，支持如下三个选项：

\- PreferNoSchedule：kubernetes将尽量避免把Pod调度到具有该污点的Node上，除非没有其他节点可调度

\- NoSchedule：kubernetes将不会把Pod调度到具有该污点的Node上，但不会影响当前Node上已存在的Pod

\- NoExecute：kubernetes将不会把Pod调度到具有该污点的Node上，同时也会将Node上已存在的Pod驱离

![image-20210819110714437](kubernetes操作实战.assets\image-20210819110714437.png)

使用kubectl设置和去除污点的命令示例如下：

```bash
# 设置污点
kubectl taint nodes node1 key=value:effect

# 去除污点
kubectl taint nodes node1 key:effect-

# 去除所有污点
kubectl taint nodes node1 key-
```

接下来，演示下污点的效果：

```bash
1. 准备节点node1（为了演示效果更加明显，暂时停止node2节点）
2. 为node1节点设置一个污点: `tag=heima:PreferNoSchedule`；然后创建pod1( pod1 可以 )
3. 修改为node1节点设置一个污点: `tag=heima:NoSchedule`；然后创建pod2( pod1 正常  pod2 失败 )
4. 修改为node1节点设置一个污点: `tag=heima:NoExecute`；然后创建pod3 ( 3个pod都失败 )
```

```bash
# 为node1设置污点(PreferNoSchedule)
[root@master ~]# kubectl taint nodes node1 tag=heima:PreferNoSchedule

# 创建pod1
[root@master ~]# kubectl run taint1 --image=nginx:1.17.1 -n dev
[root@master ~]# kubectl get pods -n dev -o wide
NAME                      READY   STATUS    RESTARTS   AGE     IP           NODE   
taint1-7665f7fd85-574h4   1/1     Running   0          2m24s   10.244.1.59   node1    

# 为node1设置污点(取消PreferNoSchedule，设置NoSchedule)
[root@master ~]# kubectl taint nodes node1 tag:PreferNoSchedule-
[root@master ~]# kubectl taint nodes node1 tag=heima:NoSchedule

# 创建pod2
[root@master ~]# kubectl run taint2 --image=nginx:1.17.1 -n dev
[root@master ~]# kubectl get pods taint2 -n dev -o wide
NAME                      READY   STATUS    RESTARTS   AGE     IP            NODE
taint1-7665f7fd85-574h4   1/1     Running   0          2m24s   10.244.1.59   node1 
taint2-544694789-6zmlf    0/1     Pending   0          21s     <none>        <none>   

# 为node1设置污点(取消NoSchedule，设置NoExecute)
[root@master ~]# kubectl taint nodes node1 tag:NoSchedule-
[root@master ~]# kubectl taint nodes node1 tag=heima:NoExecute

# 创建pod3
[root@master ~]# kubectl run taint3 --image=nginx:1.17.1 -n dev
[root@master ~]# kubectl get pods -n dev -o wide
NAME                      READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED 
taint1-7665f7fd85-htkmp   0/1     Pending   0          35s   <none>   <none>   <none>    
taint2-544694789-bn7wb    0/1     Pending   0          35s   <none>   <none>   <none>     
taint3-6d78dbd749-tktkq   0/1     Pending   0          6s    <none>   <none>   <none>     
```

> 小提示：
>
>   使用kubeadm搭建的集群，默认就会给master节点添加一个污点标记,所以pod就不会调度到master节点上.

### **容忍（Toleration**）

  上面介绍了污点的作用，我们可以在node上添加污点用于拒绝pod调度上来，但是如果就是想将一个pod调度到一个有污点的node上去，这时候应该怎么做呢？这就要使用到**容忍**。

![image-20210819110845942](kubernetes操作实战.assets\image-20210819110845942.png)

\> 污点就是拒绝，容忍就是忽略，Node通过污点拒绝pod调度上去，Pod通过容忍忽略拒绝

 

下面先通过一个案例看下效果：

 

1. 上一小节，已经在node1节点上打上了`NoExecute`的污点，此时pod是调度不上去的

2. 本小节，可以通过给pod添加容忍，然后将其调度上去

创建pod-toleration.yaml,内容如下

```bash
apiVersion: v1
kind: Pod
metadata:
  name: pod-toleration
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
  tolerations:      # 添加容忍
  - key: "tag"        # 要容忍的污点的key
    operator: "Equal" # 操作符
    value: "heima"    # 容忍的污点的value
    effect: "NoExecute"   # 添加容忍的规则，这里必须和标记的污点规则相同
```

```bash
# 添加容忍之前的pod
[root@master ~]# kubectl get pods -n dev -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED 
pod-toleration   0/1     Pending   0          3s    <none>   <none>   <none>           

# 添加容忍之后的pod
[root@master ~]# kubectl get pods -n dev -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP            NODE    NOMINATED
pod-toleration   1/1     Running   0          3s    10.244.1.62   node1   <none>   
```

下面看一下容忍的详细配置:

```bash
[root@master ~]# kubectl explain pod.spec.tolerations
......
FIELDS:
   key       # 对应着要容忍的污点的键，空意味着匹配所有的键
   value     # 对应着要容忍的污点的值
   operator  # key-value的运算符，支持Equal和Exists（默认）
   effect    # 对应污点的effect，空意味着匹配所有影响
   tolerationSeconds   # 容忍时间, 当effect为NoExecute时生效，表示pod在Node上的停留时间
```

# pod控制器

Pod是kubernetes的最小管理单元，在kubernetes中，按照pod的创建方式可以将其分为两类：

\- 自主式pod：kubernetes直接创建出来的Pod，这种pod删除后就没有了，也不会重建

\- 控制器创建的pod：kubernetes通过控制器创建的pod，这种pod删除了之后还会自动重建    

> 什么是Pod控制器 

> Pod控制器是管理pod的中间层，使用Pod控制器之后，只需要告诉Pod控制器，想要多少个什么样的Pod就可以了，它会创建出满足条件的Pod并确保每一个Pod资源处于用户期望的目标状态。如果Pod资源在运行中出现故障，它会基于指定策略重新编排Pod。

在kubernetes中，有很多类型的pod控制器，每种都有自己的适合的场景，常见的有下面这些：

> - ReplicationController**：比较原始的pod**控制器，已经被废弃，由ReplicaSet替代
>
> - ReplicaSet：保证副本数量一直维持在期望值，并支持pod数量扩缩容，镜像版本升级
>
> - Deployment：通过控制ReplicaSet来控制Pod，并支持滚动升级、回退版本
>
> - Horizontal Pod Autoscaler：可以根据集群负载自动水平调整Pod的数量，实现削峰填谷
>
> - DaemonSet：在集群中的指定Node上运行且仅运行一个副本，一般用于守护进程类的任务
>
> - Job：它创建出来的pod只要完成任务就立即退出，不需要重启或重建，用于执行一次性任务
>
> - Cronjob：它创建的Pod负责周期性任务控制，不需要持续后台运行
>
> - StatefulSet：管理有状态应用

## ReplicaSet(RS)

  ReplicaSet的主要作用是**保证一定数量的****pod****正常运行**，它会持续监听这些Pod的运行状态，一旦Pod发生故障，就会重启或重建。同时它还支持对pod数量的扩缩容和镜像版本的升降级。

![image-20210819111321195](kubernetes操作实战.assets\image-20210819111321195.png)

ReplicaSet的资源清单文件：

```bash
apiVersion: apps/v1 # 版本号
kind: ReplicaSet # 类型       
metadata: # 元数据
  name: # rs名称 
  namespace: # 所属命名空间 
  labels: #标签
    controller: rs
spec: # 详情描述
  replicas: 3 # 副本数量
  selector: # 选择器，通过它指定该控制器管理哪些pod
    matchLabels:      # Labels匹配规则
      app: nginx-pod
    matchExpressions: # Expressions匹配规则
      - {key: app, operator: In, values: [nginx-pod]}
  template: # 模板，当副本数量不足时，会根据下面的模板创建pod副本
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.1
        ports:
        - containerPort: 80
```

> 在这里面，需要新了解的配置项就是`spec`下面几个选项：
>
> \- replicas：指定副本数量，其实就是当前rs创建出来的pod的数量，默认为1
>
> \- selector：选择器，它的作用是建立pod控制器和pod之间的关联关系，采用的Label Selector机制
>
>    **在pod**模板上定义label，在控制器上定义选择器，就可以表明当前控制器能管理哪些pod了
>
> \- template：模板，就是当前控制器创建pod所使用的模板板，里面其实就是前一章学过的pod的定义

### 创建

创建pc-replicaset.yaml文件，内容如下：

```bash
[root@master01 ~/yaml]# cat pod-replicaset.yaml 
apiVersion: apps/v1
kind: ReplicaSet   
metadata:
  name: pc-replicaset
  namespace: dev
spec:
  replicas: 3
  selector: 
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - name: nginx
        image: nginx:latest
[root@master01 ~/yaml]# kubectl create -f pod-replicaset.yaml 
replicaset.apps/pc-replicaset created
# 查看rs
# DESIRED:期望副本数量  
# CURRENT:当前副本数量  
# READY:已经准备好提供服务的副本数量
[root@master01 ~/yaml]# kubectl get rs pc-replicaset -n dev -o wide
NAME            DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES         SELECTOR
pc-replicaset   3         3         3       79s   nginx        nginx:latest   app=nginx-pod
[root@master01 ~/yaml]# kubectl get replicasets.apps -n dev #跟上面一样
NAME            DESIRED   CURRENT   READY   AGE
pc-replicaset   3         3         3       3m52s
[root@master01 ~/yaml]# kubectl get pod -n dev
NAME                         READY   STATUS    RESTARTS   AGE
pc-replicaset-ls9hm          1/1     Running   0          2m40s
pc-replicaset-prcqr          1/1     Running   0          2m40s
pc-replicaset-xmh5h          1/1     Running   0          2m40s
pod-nodeaffinity-preferred   1/1     Running   1          44h
pod-nodename                 1/1     Running   1          45h
# 查看当前控制器创建出来的pod
# 这里发现控制器创建出来的pod的名称是在控制器名称后面拼接了-xxxxx随机码
```

### 扩缩容

```bash
[root@master01 ~/yaml]# kubectl edit replicasets.apps pc-replicaset -n dev
replicaset.apps/pc-replicaset edited
[root@master ~]# kubectl edit rs pc-replicaset -n dev #上一条的简写
[root@master01 ~/yaml]# kubectl get pod -n dev
NAME                  READY   STATUS    RESTARTS   AGE
pc-replicaset-bsxhr   1/1     Running   0          40s
pc-replicaset-ls9hm   1/1     Running   0          8m54s
pc-replicaset-prcqr   1/1     Running   0          8m54s
pc-replicaset-psxdf   1/1     Running   0          40s
pc-replicaset-v5vtk   1/1     Running   0          40s
pc-replicaset-xmh5h   1/1     Running   0          8m54s
#当然也可以直接使用命令实现
#使用scale命令实现扩缩容， 后面--replicas=n直接指定目标数量即可
[root@master01 ~/yaml]# kubectl scale replicaset pc-replicaset --replicas=2 -n dev
replicaset.apps/pc-replicaset scaled
[root@master01 ~/yaml]# kubectl get pod -n dev
NAME                  READY   STATUS        RESTARTS   AGE
pc-replicaset-ls9hm   1/1     Running       0          11m
pc-replicaset-prcqr   0/1     Terminating   0          11m
pc-replicaset-psxdf   0/1     Terminating   0          3m34s
pc-replicaset-v5vtk   0/1     Terminating   0          3m34s
pc-replicaset-xmh5h   1/1     Running       0          11m
#上面四个删除的只看到3个，速度慢了点没有看全。
```

### 镜像升级

```bash
[root@master01 ~/yaml]# kubectl edit replicasets.apps pc-replicaset -n dev
replicaset.apps/pc-replicaset edited
#image改成1.19.0
[root@master01 ~/yaml]# kubectl get pod -n dev -o wide
NAME                  READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
pc-replicaset-ls9hm   1/1     Running   0          47m   10.244.196.129   node01   <none>           <none>
pc-replicaset-xmh5h   1/1     Running   0          47m   10.244.196.130   node01   <none>           <none>
[root@master01 ~/yaml]# kubectl get replicasets.apps -n dev -o wide
NAME            DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES         SELECTOR
pc-replicaset   2         2         2       48m   nginx        nginx:1.19.0   app=nginx-pod
# 同样的道理，也可以使用命令完成这个工作
# kubectl set image rs rs名称 容器=镜像版本 -n namespace
[root@master01 ~/yaml]# kubectl set image rs pc-replicaset nginx=nginx:1.19.1 -n dev
replicaset.apps/pc-replicaset image updated
#replicaset=rs
[root@master01 ~/yaml]# kubectl set image replicaset pc-replicaset nginx=nginx:1.19.0 -n dev
replicaset.apps/pc-replicaset image updated
[root@master01 ~/yaml]# kubectl get replicasets.apps -n dev -o wide
NAME            DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES         SELECTOR
pc-replicaset   2         2         2       54m   nginx        nginx:1.19.0   app=nginx-pod
[root@master01 ~/yaml]# kubectl set image replicaset pc-replicaset nginx=nginx:1.19.1 -n dev
replicaset.apps/pc-replicaset image updated
[root@master01 ~/yaml]# kubectl get replicasets.apps -n dev -o wide
NAME            DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES         SELECTOR
pc-replicaset   2         2         2       55m   nginx        nginx:1.19.1   app=nginx-pod
[root@master01 ~/yaml]#
```

### 删除replicaset 

```bash
[root@master01 ~/yaml]# kubectl delete replicaset pc-replicaset -n dev 
#删除replicaset 和pod 在kubernetes删除RS前，会将RS的replicasclear调整为0，等待所有的Pod被删除后，在执行RS对象的删除
replicaset.apps "pc-replicaset" deleted
[root@master01 ~/yaml]# kubectl delete replicaset pc-replicaset -n dev --cascade=false #不推荐，只删除replicaset pod保留
[root@master01 ~/yaml]# kubectl get replicaset -n dev
No resources found in dev namespace.
#也可以使用yaml直接删除(推荐) 
[root@master ~]# kubectl delete -f pc-replicaset.yaml
```

## deployment(deploy)

为了更好的解决服务编排的问题，kubernetes在V1.2版本开始，引入了Deployment控制器。值得一提的是，这种控制器并不直接管理pod，而是通过管理ReplicaSet来简介管理Pod，即：Deployment管理ReplicaSet，ReplicaSet管理Pod。所以Deployment比ReplicaSet功能更加强大。

![image-20210819112046799](kubernetes操作实战.assets\image-20210819112046799.png)

Deployment主要功能有下面几个：

Ø 支持ReplicaSet的所有功能

Ø 支持发布的停止、继续

Ø 支持滚动升级和回滚版本

Deployment的资源清单文件：

```bash
apiVersion: apps/v1 # 版本号
kind: Deployment # 类型       
metadata: # 元数据
  name: # rs名称 
  namespace: # 所属命名空间 
  labels: #标签
    controller: deploy
spec: # 详情描述
  replicas: 3 # 副本数量
  revisionHistoryLimit: 3 # 保留历史版本
  paused: false # 暂停部署，默认是false
  progressDeadlineSeconds: 600 # 部署超时时间（s），默认是600
  strategy: # 策略
    type: RollingUpdate # 滚动更新策略
    rollingUpdate: # 滚动更新
      maxSurge: 30% # 最大额外可以存在的副本数，可以为百分比，也可以为整数
      maxUnavailable: 30% # 最大不可用状态的 Pod 的最大值，可以为百分比，也可以为整数
  selector: # 选择器，通过它指定该控制器管理哪些pod
    matchLabels:      # Labels匹配规则
      app: nginx-pod
    matchExpressions: # Expressions匹配规则
      - {key: app, operator: In, values: [nginx-pod]}
  template: # 模板，当副本数量不足时，会根据下面的模板创建pod副本
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.1
        ports:
        - containerPort: 80
```

### 创建

创建pc-deployment.yaml，内容如下：

```bash
apiVersion: apps/v1
kind: Deployment      
metadata:
  name: pc-deployment
  namespace: dev
spec: 
  replicas: 3
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.1
```

```bash
[root@master01 ~/yaml]# kubectl create -f pod-deployment.yaml --record=true
deployment.apps/pc-deployment created
#将kubectl的 --record 的 flag 设置为 true可以在 annotation 中记录当前命令创建或者升级了该资源。这在未来会很有用，例如，查看在每个 Deployment revision 中执行了哪些命令。
[root@master01 ~/yaml]# kubectl get deployments.apps -n dev
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
pc-deployment   3/3     3            3           3m37s
# 查看deployment
# UP-TO-DATE 最新版本的pod的数量
# AVAILABLE  当前可用的pod的数量
[root@master01 ~/yaml]# kubectl get replicasets.apps  -n dev
NAME                       DESIRED   CURRENT   READY   AGE
pc-deployment-7958587476   3         3         3       7m10s
[root@master01 ~/yaml]# kubectl get pod -n dev
NAME                             READY   STATUS    RESTARTS   AGE
pc-deployment-7958587476-l9l9t   1/1     Running   0          7m46s
pc-deployment-7958587476-xfvxg   1/1     Running   0          7m46s
pc-deployment-7958587476-xsjmf   1/1     Running   0          7m46s
```

### 扩缩容

```bash
[root@master01 ~/yaml]# kubectl scale deployment pc-deployment --replicas=5 -n dev
deployment.apps/pc-deployment scaled
[root@master01 ~/yaml]# kubectl get deployments.apps pc-deployment -n dev
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
pc-deployment   5/5     5            5           10m
[root@master01 ~/yaml]# kubectl get pods -n dev
NAME                             READY   STATUS    RESTARTS   AGE
pc-deployment-7958587476-7dskk   1/1     Running   0          45s
pc-deployment-7958587476-l9l9t   1/1     Running   0          10m
pc-deployment-7958587476-s25mg   1/1     Running   0          45s
pc-deployment-7958587476-xfvxg   1/1     Running   0          10m
pc-deployment-7958587476-xsjmf   1/1     Running   0          10m
[root@master01 ~/yaml]# kubectl edit deployments.apps pc-deployment -n dev 
#修改replicas=2
[root@master01 ~/yaml]# kubectl get pods -n dev
NAME                             READY   STATUS    RESTARTS   AGE
pc-deployment-7958587476-l9l9t   1/1     Running   0          15m
pc-deployment-7958587476-xfvxg   1/1     Running   0          15m
```

### 镜像更新

deployment支持两种更新策略:`重建更新`和`滚动更新`,可以通过`strategy`指定策略类型,支持两个属性:

> strategy：指定新的Pod替换旧的Pod的策略， 支持两个属性：
>
>  type：指定策略类型，支持两种策略
>
>   Recreate：在创建出新的Pod之前会先杀掉所有已存在的Pod
>
>   RollingUpdate：滚动更新，就是杀死一部分，就启动一部分，在更新过程中，存在两个版本Pod
>
>  rollingUpdate：当type为RollingUpdate时生效，用于为RollingUpdate设置参数，支持两个属性：
>
>   maxUnavailable：用来指定在升级过程中不可用Pod的最大数量，默认为25%。
>
>   maxSurge： 用来指定在升级过程中可以超过期望的Pod的最大数量，默认为25%。

#### **重建更新**

编辑pc-deployment.yaml,在spec节点下添加更新策略

```bash
spec:
  strategy: # 策略
    type: Recreate # 重建更新
```

```bash
[root@master01 ~/yaml]# kubectl set image deployment pc-deployment nginx=nginx:1.19.2 -n dev
deployment.apps/pc-deployment image updated
[root@master01 ~/yaml]# kubectl get pods -n dev -w
NAME                             READY   STATUS              RESTARTS   AGE
pc-deployment-7574d687c-stfxb    0/1     ContainerCreating   0          57s
pc-deployment-7958587476-l9l9t   1/1     Running             0          62m
pc-deployment-7958587476-xfvxg   1/1     Running             0          62m
pc-deployment-7574d687c-stfxb    1/1     Running             0          62s
pc-deployment-7958587476-xfvxg   1/1     Terminating         0          62m
pc-deployment-7574d687c-4rjhj    0/1     Pending             0          0s
pc-deployment-7574d687c-4rjhj    0/1     Pending             0          0s
pc-deployment-7574d687c-4rjhj    0/1     ContainerCreating   0          0s
pc-deployment-7958587476-xfvxg   0/1     Terminating         0          62m
pc-deployment-7574d687c-4rjhj    0/1     ContainerCreating   0          2s
pc-deployment-7574d687c-4rjhj    1/1     Running             0          2s
pc-deployment-7958587476-l9l9t   1/1     Terminating         0          62m
pc-deployment-7958587476-l9l9t   0/1     Terminating         0          62m
pc-deployment-7958587476-l9l9t   0/1     Terminating         0          62m
pc-deployment-7958587476-l9l9t   0/1     Terminating         0          62m
pc-deployment-7958587476-xfvxg   0/1     Terminating         0          62m
pc-deployment-7958587476-xfvxg   0/1     Terminating         0          62m
[root@master01 ~/yaml]# kubectl get pod -n dev
NAME                            READY   STATUS    RESTARTS   AGE
pc-deployment-7574d687c-4rjhj   1/1     Running   0          4m22s
pc-deployment-7574d687c-stfxb   1/1     Running   0          5m24s
```

从上面监控过程可以看到先删除旧的再创建新的

#### **滚动更新**

```bash
[root@master01 ~/yaml]# cat pod-deployment.yaml 
apiVersion: apps/v1
kind: Deployment      
metadata:
  name: pc-deployment
  namespace: dev
spec: 
  replicas: 5
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.1
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
```

```bash
[root@master01 ~/yaml]# kubectl set image deployment pc-deployment nginx=nginx:1.19.1 -n dev
deployment.apps/pc-deployment image updated
[root@master01 ~/yaml]# kubectl get pods -n dev -w
NAME                            READY   STATUS    RESTARTS   AGE
pc-deployment-7574d687c-4rjhj   1/1     Running   0          35m
pc-deployment-7574d687c-cdrbw   1/1     Running   0          31s
pc-deployment-7574d687c-mwwrv   1/1     Running   0          31s
pc-deployment-7574d687c-n9l5p   1/1     Running   0          31s
pc-deployment-7574d687c-stfxb   1/1     Running   0          36m
pc-deployment-7958587476-hcl4p   0/1     Pending   0          0s
pc-deployment-7958587476-qc8pb   0/1     Pending   0          0s
pc-deployment-7958587476-hcl4p   0/1     ContainerCreating   0          0s
pc-deployment-7574d687c-cdrbw    1/1     Terminating         0          111s
pc-deployment-7958587476-qc8pb   0/1     ContainerCreating   0          0s
pc-deployment-7958587476-kzgw7   0/1     Pending             0          0s
pc-deployment-7958587476-kzgw7   0/1     ContainerCreating   0          1s
pc-deployment-7958587476-hcl4p   0/1     ContainerCreating   0          4s
pc-deployment-7958587476-qc8pb   0/1     ContainerCreating   0          5s
pc-deployment-7574d687c-cdrbw    0/1     Terminating         0          116s
pc-deployment-7958587476-kzgw7   0/1     ContainerCreating   0          5s
pc-deployment-7958587476-hcl4p   1/1     Running             0          6s
pc-deployment-7574d687c-mwwrv    1/1     Terminating         0          117s
pc-deployment-7958587476-tbss9   0/1     Pending             0          0s
pc-deployment-7958587476-tbss9   0/1     Pending             0          0s
pc-deployment-7574d687c-cdrbw    0/1     Terminating         0          117s
pc-deployment-7574d687c-cdrbw    0/1     Terminating         0          117s
pc-deployment-7958587476-tbss9   0/1     ContainerCreating   0          1s
pc-deployment-7958587476-qc8pb   1/1     Running             0          7s
pc-deployment-7574d687c-n9l5p    1/1     Terminating         0          118s
pc-deployment-7958587476-5v7ps   0/1     Pending             0          1s
pc-deployment-7958587476-5v7ps   0/1     Pending             0          1s
pc-deployment-7958587476-5v7ps   0/1     ContainerCreating   0          1s
pc-deployment-7958587476-kzgw7   1/1     Running             0          8s
pc-deployment-7574d687c-4rjhj    1/1     Terminating         0          36m
pc-deployment-7958587476-tbss9   0/1     ContainerCreating   0          4s
pc-deployment-7574d687c-n9l5p    0/1     Terminating         0          2m1s
pc-deployment-7574d687c-mwwrv    0/1     Terminating         0          2m2s
pc-deployment-7958587476-5v7ps   0/1     ContainerCreating   0          4s
pc-deployment-7574d687c-4rjhj    0/1     Terminating         0          36m
pc-deployment-7958587476-tbss9   1/1     Running             0          6s
pc-deployment-7958587476-5v7ps   1/1     Running             0          5s
pc-deployment-7574d687c-stfxb    1/1     Terminating         0          38m
pc-deployment-7574d687c-n9l5p    0/1     Terminating         0          2m3s
pc-deployment-7574d687c-n9l5p    0/1     Terminating         0          2m3s
pc-deployment-7574d687c-stfxb    0/1     Terminating         0          38m
pc-deployment-7574d687c-stfxb    0/1     Terminating         0          38m
pc-deployment-7574d687c-stfxb    0/1     Terminating         0          38m
pc-deployment-7574d687c-4rjhj    0/1     Terminating         0          37m
pc-deployment-7574d687c-4rjhj    0/1     Terminating         0          37m
pc-deployment-7574d687c-mwwrv    0/1     Terminating         0          2m13s
pc-deployment-7574d687c-mwwrv    0/1     Terminating         0          2m13s
[root@master01 ~/yaml]# kubectl get pod -n dev
NAME                             READY   STATUS    RESTARTS   AGE
pc-deployment-7958587476-5v7ps   1/1     Running   0          76s
pc-deployment-7958587476-hcl4p   1/1     Running   0          83s
pc-deployment-7958587476-kzgw7   1/1     Running   0          83s
pc-deployment-7958587476-qc8pb   1/1     Running   0          83s
pc-deployment-7958587476-tbss9   1/1     Running   0          77s
```

\# 至此，新版本的pod创建完毕，就版本的pod销毁完毕

\# 中间过程是滚动进行的，也就是边销毁边创建

从上面过程来看是先创建起来再删除是按照设计的百分比进行作业的。

![image-20210819112436831](kubernetes操作实战.assets\image-20210819112436831.png)

\# 查看rs,发现原来的rs的依旧存在，只是pod数量变为了0，而后又新产生了一个rs，pod数量为5

\# 其实这就是deployment能够进行版本回退的奥妙所在，后面会详细解释

```bash
[root@master01 ~/yaml]# kubectl get replicasets.apps -n dev
NAME                       DESIRED   CURRENT   READY   AGE
pc-deployment-7574d687c    0         0         0       77m
pc-deployment-7958587476   5         5         5       138m
```

### 版本回退

deployment支持版本升级过程中的暂停、继续功能以及版本回退等诸多功能，下面具体来看.

kubectl rollout： 版本升级相关功能，支持下面的选项：

> Ø status    显示当前升级状态
>
> Ø history   显示 升级历史记录
>
> Ø pause    暂停版本升级过程
>
> Ø resume  继续已经暂停的版本升级过程
>
> Ø restart   重启版本升级过程
>
> Ø undo    回滚到上一级版本（可以使用--to-revision回滚到指定版本）

```bash
[root@master01 ~/yaml]# kubectl rollout status deployment pc-deployment -n dev
deployment "pc-deployment" successfully rolled out
#查看当前版本升级状态
[root@master01 ~/yaml]# kubectl rollout history deployment pc-deployment -n dev
deployment.apps/pc-deployment 
REVISION  CHANGE-CAUSE
2         kubectl create --filename=pod-deployment.yaml --record=true
3         kubectl create --filename=pod-deployment.yaml --record=true
这里是升级过一次，1被3替换了。下次回退之后显示就是3,4了
# 版本回滚
# 这里直接使用--to-revision=1回滚到了1版本， 如果省略这个选项，就是回退到上个版本，就是2版本
[root@master01 ~/yaml]# kubectl rollout undo deployment pc-deployment --to-revision=2 -n dev #回退到第二个版本
deployment.apps/pc-deployment rolled back
[root@master01 ~/yaml]# kubectl get deployments.apps -n dev -o wide
NAME            READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES         SELECTOR
pc-deployment   5/5     5            5           5h25m   nginx        nginx:1.19.2   app=nginx-pod
[root@master01 ~/yaml]# kubectl rollout history deployment -n dev
deployment.apps/pc-deployment 
REVISION  CHANGE-CAUSE
3         kubectl create --filename=pod-deployment.yaml --record=true
4         kubectl create --filename=pod-deployment.yaml --record=true
#当服务器上很多容器时候指定容器名查询
[root@master01 ~/yaml]# kubectl rollout history deployment pc-deployment -n dev
deployment.apps/pc-deployment 
REVISION  CHANGE-CAUSE
3         kubectl create --filename=pod-deployment.yaml --record=true
4         kubectl create --filename=pod-deployment.yaml --record=true
[root@master01 ~/yaml]# kubectl rollout undo deployment pc-deployment -n dev --to-revision=3
deployment.apps/pc-deployment rolled back
[root@master01 ~/yaml]# kubectl get deployments.apps pc-deployment -n dev -o wide
NAME            READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES         SELECTOR
pc-deployment   5/5     5            5           5h29m   nginx        nginx:1.19.1   app=nginx-pod
[root@master01 ~/yaml]# kubectl get replicasets.apps -n dev
NAME                       DESIRED   CURRENT   READY   AGE
pc-deployment-7574d687c    0         0         0       4h33m
pc-deployment-7958587476   5         5         5       5h35m
#查看replicaset 第一个rs版本没有运行，第二个再运行
#deployment 实现版本回退就是因为历史记录rs来实现的
```

### 金丝雀发布

  Deployment控制器支持控制更新过程中的控制，如“暂停(pause)”或“继续(resume)”更新操作。

  比如有一批新的Pod资源创建完成后立即暂停更新过程，此时，仅存在一部分新版本的应用，主体部分还是旧的版本。然后，再筛选一小部分的用户请求路由到新版本的Pod应用，继续观察能否稳定地按期望的方式运行。确定没问题之后再继续完成余下的Pod资源滚动更新，否则立即回滚更新操作。这就是所谓的金丝雀发布。

```bash
#更新deployment的版本，并配置暂停deployment
[root@master01 ~/yaml]# kubectl set image deployment pc-deployment nginx=nginx:1.19.0 -n dev && kubectl rollout pause deployment pc-deployment -n dev
deployment.apps/pc-deployment image updated
deployment.apps/pc-deployment paused
#查看更新状态
[root@master01 ~/yaml]# kubectl rollout status deployment  pc-deployment -n dev
Waiting for deployment "pc-deployment" rollout to finish: 2 out of 5 new replicas have been updated...  #pause
Waiting for deployment spec update to be observed...
Waiting for deployment spec update to be observed... 
Waiting for deployment "pc-deployment" rollout to finish: 2 out of 5 new replicas have been updated...
Waiting for deployment "pc-deployment" rollout to finish: 2 out of 5 new replicas have been updated...
Waiting for deployment "pc-deployment" rollout to finish: 2 old replicas are pending termination...
Waiting for deployment "pc-deployment" rollout to finish: 2 old replicas are pending termination...
Waiting for deployment "pc-deployment" rollout to finish: 2 old replicas are pending termination...
Waiting for deployment "pc-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "pc-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "pc-deployment" rollout to finish: 1 old replicas are pending termination...
deployment "pc-deployment" successfully rolled out
## 监控更新的过程，可以看到已经新增了2个资源，但是并未按照预期的状态去删除一个旧的资源，就是因为使用了pause暂停命令
[root@master01 ~/yaml]# kubectl get replicasets.apps -n dev -o wide
NAME                       DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES         SELECTOR
pc-deployment-7574d687c    0         0         0       4h39m   nginx        nginx:1.19.2   app=nginx-pod,pod-template-hash=7574d687c
pc-deployment-76b47d6567   2         2         2       67s     nginx        nginx:1.19.0   app=nginx-pod,pod-template-hash=76b47d6567
pc-deployment-7958587476   5         5         5       5h41m   nginx        nginx:1.19.1   app=nginx-pod,pod-template-hash=7958587476
[root@master01 ~/yaml]# kubectl get pod -n dev
NAME                             READY   STATUS    RESTARTS   AGE
pc-deployment-76b47d6567-fm8xd   1/1     Running   0          88s
pc-deployment-76b47d6567-r6g49   1/1     Running   0          88s
pc-deployment-7958587476-27fv7   1/1     Running   0          13m
pc-deployment-7958587476-2tlf8   1/1     Running   0          87s
pc-deployment-7958587476-42b8f   1/1     Running   0          13m
pc-deployment-7958587476-67lkt   1/1     Running   0          13m
pc-deployment-7958587476-t8lcq   1/1     Running   0          13m
#这里确保更新的pod没有问题就可以接着发布
[root@master01 ~/yaml]# kubectl rollout resume deployment pc-deployment -n dev
deployment.apps/pc-deployment resumed
#查看更新后的情况
[root@master01 ~/yaml]# kubectl get replicasets.apps -n dev -o wide
NAME                       DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES         SELECTOR
pc-deployment-7574d687c    0         0         0       4h42m   nginx        nginx:1.19.2   app=nginx-pod,pod-template-hash=7574d687c
pc-deployment-76b47d6567   5         5         5       3m23s   nginx        nginx:1.19.0   app=nginx-pod,pod-template-hash=76b47d6567
pc-deployment-7958587476   0         0         0       5h43m   nginx        nginx:1.19.1   app=nginx-pod,pod-template-hash=7958587476
[root@master01 ~/yaml]# kubectl get pod -n dev
NAME                             READY   STATUS    RESTARTS   AGE
pc-deployment-76b47d6567-cq6f2   1/1     Running   0          79s
pc-deployment-76b47d6567-fm8xd   1/1     Running   0          3m55s
pc-deployment-76b47d6567-r6g49   1/1     Running   0          3m55s
pc-deployment-76b47d6567-tknt9   1/1     Running   0          79s
pc-deployment-76b47d6567-z6pth   1/1     Running   0          79s

```

### 删除deployment

```bash
[root@master01 ~/yaml]# kubectl delete -f pod-deployment.yaml 
deployment.apps "pc-deployment" deleted
[root@master01 ~/yaml]# kubectl get pod -n dev   #看到正在删除中
NAME                             READY   STATUS        RESTARTS   AGE
pc-deployment-76b47d6567-cq6f2   0/1     Terminating   0          10m
pc-deployment-76b47d6567-fm8xd   0/1     Terminating   0          12m
pc-deployment-76b47d6567-r6g49   0/1     Terminating   0          12m
```

## horizontal pod autoscaler(HPA)

  在前面的学习中，我们已经可以实现通过手工执行`kubectl scale`命令实现Pod扩容或缩容，但是这显然不符合Kubernetes的定位目标--自动化、智能化。 Kubernetes期望可以实现通过监测Pod的使用情况，实现pod数量的自动调整，于是就产生了Horizontal Pod Autoscaler（HPA）这种控制器。

  HPA可以获取每个Pod利用率，然后和HPA中定义的指标进行对比，同时计算出需要伸缩的具体值，最后实现Pod的数量的调整。其实HPA与之前的Deployment一样，也属于一种Kubernetes资源对象，它通过追踪分析RC控制的所有目标Pod的负载变化情况，来确定是否需要针对性地调整目标Pod的副本数，这是HPA的实现原理。

![image-20210819112718725](kubernetes操作实战.assets\image-20210819112718725.png)

接下来，我们来做一个实验

### metrics-server

1 安装metrics-server

metrics-server可以用来收集集群中的资源使用情况

https://github.com/kubernetes-sigs/metrics-server/releases

官网给出的方法是直接运行安装

> kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.4.4/components.yaml

但是国内不能连接k8s.gcr.io怎么办呢？

改成docker hub  --hub.docker.com

wget https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.4.4/components.yaml

修改文件

![image-20210819112821291](kubernetes操作实战.assets\image-20210819112821291.png)

添加args参数： - --kubelet-insecure-tls

​        image: bitnami/metrics-server:0.4.4 

```bash
[root@master01 ~/yaml]# kubectl create -f components.yaml 
serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
service/metrics-server created
deployment.apps/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
[root@master01 ~/yaml]# kubectl get pods -n kube-system 
NAME                                     READY   STATUS    RESTARTS   AGE
calico-kube-controllers-8db96c76-9bvfh   1/1     Running   11         5d13h
calico-node-2xkr7                        1/1     Running   11         5d13h
calico-node-77dpc                        1/1     Running   11         5d13h
calico-node-nzpl6                        1/1     Running   26         5d13h
coredns-558bd4d5db-dkddw                 1/1     Running   10         5d14h
coredns-558bd4d5db-pt6sm                 1/1     Running   10         5d14h
kube-apiserver-master01                  1/1     Running   11         5d14h
kube-apiserver-master02                  1/1     Running   11         5d13h
kube-controller-manager-master01         1/1     Running   11         5d14h
kube-controller-manager-master02         1/1     Running   11         5d13h
kube-proxy-jjjfj                         1/1     Running   10         5d13h
kube-proxy-m7wn7                         1/1     Running   10         5d14h
kube-proxy-vvs7v                         1/1     Running   9          5d13h
kube-scheduler-master01                  1/1     Running   11         5d14h
kube-scheduler-master02                  1/1     Running   10         5d13h
metrics-server-864f7b9584-cjb5l          1/1     Running   0          4m22s
[root@master01 ~/yaml]# kubectl top node
W0523 23:43:21.339255  102486 top_node.go:119] Using json format to get metrics. Next release will switch to protocol-buffers, switch early by passing --use-protocol-buffers flag
NAME       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
master01   319m         31%    1253Mi          72%       
master02   355m         35%    1240Mi          72%       
node01     214m         21%    899Mi           52%       
[root@master01 ~/yaml]# kubectl top pod -n kube-system 
W0523 23:44:07.209158  103229 top_pod.go:140] Using json format to get metrics. Next release will switch to protocol-buffers, switch early by passing --use-protocol-buffers flag
NAME                                     CPU(cores)   MEMORY(bytes)   
calico-kube-controllers-8db96c76-9bvfh   2m           16Mi            
calico-node-2xkr7                        34m          54Mi            
calico-node-77dpc                        36m          59Mi            
calico-node-nzpl6                        41m          68Mi            
coredns-558bd4d5db-dkddw                 5m           14Mi            
coredns-558bd4d5db-pt6sm                 4m           14Mi            
kube-apiserver-master01                  75m          491Mi           
kube-apiserver-master02                  76m          510Mi           
kube-controller-manager-master01         4m           31Mi            
kube-controller-manager-master02         20m          72Mi            
kube-proxy-jjjfj                         1m           27Mi            
kube-proxy-m7wn7                         1m           31Mi            
kube-proxy-vvs7v                         1m           32Mi            
kube-scheduler-master01                  3m           36Mi            
kube-scheduler-master02                  3m           27Mi            
metrics-server-864f7b9584-cjb5l          7m           13Mi            
```

到此metrics-server安装完成。

2 准备deployment和servie

```bash
[root@master01 ~/yaml]# cat deploy-nginx.yaml 
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: nginx
  namespace: dev
spec:
  replicas: 1
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels: 
        run: nginx
    spec:
      containers:
      - image: nginx:latest
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
        resources:
          requests:
            cpu: "100Mi"     #命令方式已被淘汰
```

上面因为机器CPU资源不足用下面的命令创建一个nginx测试

```bash
[root@master01 ~/yaml]# kubectl create deployment nginx --image=nginx:latest -n dev
deployment.apps/nginx created
[root@master01 ~/yaml]# kubectl get deployments.apps,pod -n dev
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           29s

NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-55649fd747-5dwjb   1/1     Running   0          28s
[root@master01 ~/yaml]# kubectl expose deployment nginx --type=NodePort --port=80 -n dev
service/nginx exposed
[root@master01 ~/yaml]# kubectl get deployments.apps,pod,svc -n dev
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           3m5s

NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-55649fd747-5dwjb   1/1     Running   0          3m4s

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/nginx        NodePort    10.111.5.90      <none>        80:31343/TCP   23s
```

**部署HPA**

创建pc-hpa.yaml

```bash
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: pc-hpa
  namespace: dev
spec:
  minReplicas: 1  #最小pod数量
  maxReplicas: 10 #最大pod数量
  targetCPUUtilizationPercentage: 3 # CPU使用率指标
  scaleTargetRef:   # 指定要控制的nginx信息
    apiVersion: apps/v1
    kind: Deployment  
    name: nginx  
```

```bash
为了测试干掉污点
[root@master01 ~/yaml]# kubectl taint nodes master02 node-role.kubernetes.io/master-
[root@master01 ~/yaml]# kubectl taint nodes master01 node-role.kubernetes.io/master-

[root@master01 ~/yaml]# kubectl create -f pc-hpa.yaml 
horizontalpodautoscaler.autoscaling/pc-hpa created
[root@master01 ~/yaml]# kubectl get ho
horizontalpodautoscalers.autoscaling  hostendpoints.crd.projectcalico.org   
[root@master01 ~/yaml]# kubectl get horizontalpodautoscalers.autoscaling -n dev
NAME     REFERENCE          TARGETS        MINPODS   MAXPODS   REPLICAS   AGE
pc-hpa   Deployment/nginx   <unknown>/3%   1         10        1          27s
[root@master01 ~/yaml]# kubectl get hpa -n dev #简写
NAME     REFERENCE          TARGETS        MINPODS   MAXPODS   REPLICAS   AGE
pc-hpa   Deployment/nginx   <unknown>/3%   1         10        1          43s
```

测试

使用压测工具对service地址`192.168.1.26:31343`进行压测，然后通过控制台查看hpa和pod的变化

hpa变化

```bash
[root@master ~]# kubectl get hpa -n dev -w
NAME     REFERENCE          TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
pc-hpa   Deployment/nginx   0%/3%     1         10        1          4m11s
pc-hpa   Deployment/nginx   0%/3%     1         10        1          5m19s
pc-hpa   Deployment/nginx   22%/3%    1         10        1          6m50s
pc-hpa   Deployment/nginx   22%/3%    1         10        4          7m5s
pc-hpa   Deployment/nginx   22%/3%    1         10        8          7m21s
pc-hpa   Deployment/nginx   6%/3%     1         10        8          7m51s
pc-hpa   Deployment/nginx   0%/3%     1         10        8          9m6s
pc-hpa   Deployment/nginx   0%/3%     1         10        8          13m
pc-hpa   Deployment/nginx   0%/3%     1         10        1          14m
```

deployment变化

```bash
[root@master ~]# kubectl get deployment -n dev -w
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           11m
nginx   1/4     1            1           13m
nginx   1/4     1            1           13m
nginx   1/4     1            1           13m
nginx   1/4     4            1           13m
nginx   1/8     4            1           14m
nginx   1/8     4            1           14m
nginx   1/8     4            1           14m
nginx   1/8     8            1           14m
nginx   2/8     8            2           14m
nginx   3/8     8            3           14m
nginx   4/8     8            4           14m
nginx   5/8     8            5           14m
nginx   6/8     8            6           14m
nginx   7/8     8            7           14m
nginx   8/8     8            8           15m
nginx   8/1     8            8           20m
nginx   8/1     8            8           20m
nginx   1/1     1            1           20m
```

pod变化

```bash
[root@master ~]# kubectl get pods -n dev -w
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7df9756ccc-bh8dr   1/1     Running   0          11m
nginx-7df9756ccc-cpgrv   0/1     Pending   0          0s
nginx-7df9756ccc-8zhwk   0/1     Pending   0          0s
nginx-7df9756ccc-rr9bn   0/1     Pending   0          0s
nginx-7df9756ccc-cpgrv   0/1     ContainerCreating   0          0s
nginx-7df9756ccc-8zhwk   0/1     ContainerCreating   0          0s
nginx-7df9756ccc-rr9bn   0/1     ContainerCreating   0          0s
nginx-7df9756ccc-m9gsj   0/1     Pending             0          0s
nginx-7df9756ccc-g56qb   0/1     Pending             0          0s
nginx-7df9756ccc-sl9c6   0/1     Pending             0          0s
nginx-7df9756ccc-fgst7   0/1     Pending             0          0s
nginx-7df9756ccc-g56qb   0/1     ContainerCreating   0          0s
nginx-7df9756ccc-m9gsj   0/1     ContainerCreating   0          0s
nginx-7df9756ccc-sl9c6   0/1     ContainerCreating   0          0s
nginx-7df9756ccc-fgst7   0/1     ContainerCreating   0          0s
nginx-7df9756ccc-8zhwk   1/1     Running             0          19s
nginx-7df9756ccc-rr9bn   1/1     Running             0          30s
nginx-7df9756ccc-m9gsj   1/1     Running             0          21s
nginx-7df9756ccc-cpgrv   1/1     Running             0          47s
nginx-7df9756ccc-sl9c6   1/1     Running             0          33s
nginx-7df9756ccc-g56qb   1/1     Running             0          48s
nginx-7df9756ccc-fgst7   1/1     Running             0          66s
nginx-7df9756ccc-fgst7   1/1     Terminating         0          6m50s
nginx-7df9756ccc-8zhwk   1/1     Terminating         0          7m5s
nginx-7df9756ccc-cpgrv   1/1     Terminating         0          7m5s
nginx-7df9756ccc-g56qb   1/1     Terminating         0          6m50s
nginx-7df9756ccc-rr9bn   1/1     Terminating         0          7m5s
nginx-7df9756ccc-m9gsj   1/1     Terminating         0          6m50s
nginx-7df9756ccc-sl9c6   1/1     Terminating         0          6m50s
```

## DaemonSet(DS)

  DaemonSet类型的控制器可以保证在集群中的每一台（或指定）节点上都运行一个副本。一般适用于日志收集、节点监控等场景。也就是说，如果一个Pod提供的功能是节点级别的（每个节点都需要且只需要一个），那么这类Pod就适合使用DaemonSet类型的控制器创建。

![image-20210819113222812](kubernetes操作实战.assets\image-20210819113222812.png)

DaemonSet控制器的特点：

> Ø 每当向集群中添加一个节点时，指定的 Pod 副本也将添加到该节点上
>
> Ø 当节点从集群中移除时，Pod 也就被垃圾回收了

下面先来看下DaemonSet的资源清单文件

```bash
apiVersion: apps/v1 # 版本号
kind: DaemonSet # 类型       
metadata: # 元数据
  name: # rs名称 
  namespace: # 所属命名空间 
  labels: #标签
    controller: daemonset
spec: # 详情描述
  revisionHistoryLimit: 3 # 保留历史版本
  updateStrategy: # 更新策略
    type: RollingUpdate # 滚动更新策略
    rollingUpdate: # 滚动更新
      maxUnavailable: 1 # 最大不可用状态的 Pod 的最大值，可以为百分比，也可以为整数
  selector: # 选择器，通过它指定该控制器管理哪些pod
    matchLabels:      # Labels匹配规则
      app: nginx-pod
    matchExpressions: # Expressions匹配规则
      - {key: app, operator: In, values: [nginx-pod]}
  template: # 模板，当副本数量不足时，会根据下面的模板创建pod副本
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.1
        ports:
        - containerPort: 80
```

创建pc-daemonset.yaml，内容如下：

```bash
[root@master01 ~/yaml]# kubectl create -f daemonset.yaml 
daemonset.apps/pc-daemonset created
[root@master01 ~/yaml]# cat daemonset.yaml 
apiVersion: apps/v1
kind: DaemonSet      
metadata:
  name: pc-daemonset
  namespace: dev
spec: 
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - name: nginx
        image: nginx:latest
[root@master01 ~/yaml]# kubectl get daemonsets.apps -n dev
NAME           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
pc-daemonset   3         3         3       3            3           <none>          3m33s
[root@master01 ~/yaml]# kubectl get daemonsets.apps -n dev -o wide  #daemonsets=ds
NAME           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE     CONTAINERS   IMAGES         SELECTOR
pc-daemonset   3         3         3       3            3           <none>          3m42s   nginx        nginx:latest   app=nginx-pod
[root@master01 ~/yaml]# kubectl get pod -n dev -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP               NODE       NOMINATED NODE   READINESS GATES
pc-daemonset-bqgnv       1/1     Running   0          4m16s   10.244.196.145   node01     <none>           <none>
pc-daemonset-hbw5r       1/1     Running   0          4m16s   10.244.59.210    master02   <none>           <none>
pc-daemonset-k7d5r       1/1     Running   0          4m16s   10.244.241.97    master01   <none>           <none>
```

\# 查看pod,发现在每个Node上都运行一个pod

```bash
[root@master01 ~/yaml]# kubectl delete -f daemonset.yaml  #删除
daemonset.apps "pc-daemonset" deleted
```

## job

Job，主要用于负责**批量处理****(****一次要处理指定数量任务)**短暂的**一次性****(****每个任务仅运行一次就结束)****任务**。Job特点如下：

\- 当Job创建的pod执行成功结束时，Job将记录成功结束的pod数量

\- 当成功结束的pod达到指定的数量时，Job将完成执行

![image-20210819113424423](kubernetes操作实战.assets\image-20210819113424423.png)



Job的资源清单文件：

```bash
apiVersion: batch/v1 # 版本号
kind: Job # 类型       
metadata: # 元数据
  name: # rs名称 
  namespace: # 所属命名空间 
  labels: #标签
    controller: job
spec: # 详情描述
  completions: 1 # 指定job需要成功运行Pods的次数。默认值: 1
  parallelism: 1 # 指定job在任一时刻应该并发运行Pods的数量。默认值: 1
  activeDeadlineSeconds: 30 # 指定job可运行的时间期限，超过时间还未结束，系统将会尝试进行终止。
  backoffLimit: 6 # 指定job失败后进行重试的次数。默认是6
  manualSelector: true # 是否可以使用selector选择器选择pod，默认是false
  selector: # 选择器，通过它指定该控制器管理哪些pod
    matchLabels:      # Labels匹配规则
      app: counter-pod
    matchExpressions: # Expressions匹配规则
      - {key: app, operator: In, values: [counter-pod]}
  template: # 模板，当副本数量不足时，会根据下面的模板创建pod副本
    metadata:
      labels:
        app: counter-pod
    spec:
      restartPolicy: Never # 重启策略只能设置为Never或者OnFailure
      containers:
      - name: counter
        image: busybox:1.30
        command: ["bin/sh","-c","for i in 9 8 7 6 5 4 3 2 1; do echo $i;sleep 2;done"]
```

关于重启策略设置的说明：

  如果指定为OnFailure，则job会在pod出现故障时重启容器，而不是创建pod，failed次数不变

  如果指定为Never，则job会在pod出现故障时创建新的pod，并且故障pod不会消失，也不会重启，failed次数加1

  如果指定为Always的话，就意味着一直重启，意味着job任务会重复去执行了，当然不对，所以不能设置为Always

创建pc-job.yaml，内容如下：

```bash
apiVersion: batch/v1
kind: Job      
metadata:
  name: pc-job
  namespace: dev
spec:
  manualSelector: true
  selector:
    matchLabels:
      app: counter-pod
  template:
    metadata:
      labels:
        app: counter-pod
    spec:
      restartPolicy: Never
      containers:
      - name: counter
        image: busybox:1.30
        command: ["bin/sh","-c","for i in 9 8 7 6 5 4 3 2 1; do echo $i;sleep 3;done"]
```

```bash
[root@master01 ~/yaml]# kubectl create -f pc-job.yaml 
job.batch/pc-job created
[root@master01 ~/yaml]# kubectl get jobs.batch -n dev -o wide
NAME     COMPLETIONS   DURATION   AGE   CONTAINERS   IMAGES         SELECTOR
pc-job   1/1           41s        46s   counter      busybox:1.30   app=counter-pod
[root@master01 ~/yaml]# kubectl get pod -n dev -w  #没有看到running 速度慢了点
NAME           READY   STATUS      RESTARTS   AGE
pc-job-2zpwc   0/1     Completed   0          94s
# 通过观察pod状态可以看到，pod在运行完毕任务后，就会变成Completed状态
```

\# 接下来，调整下pod运行的总数量和并行数量 即：在spec下设置下面两个选项

\# completions: 6 # 指定job需要成功运行Pods的次数为6

\# parallelism: 3 # 指定job并发运行Pods的数量为3

\# 然后重新运行job，观察效果，此时会发现，job会每次运行3个pod，总共执行了6个pod

```bash
[root@master ~]# kubectl get pods -n dev -w  #借用
NAME           READY   STATUS    RESTARTS   AGE
pc-job-684ft   1/1     Running   0          5s
pc-job-jhj49   1/1     Running   0          5s
pc-job-pfcvh   1/1     Running   0          5s
pc-job-684ft   0/1     Completed   0          11s
pc-job-v7rhr   0/1     Pending     0          0s
pc-job-v7rhr   0/1     Pending     0          0s
pc-job-v7rhr   0/1     ContainerCreating   0          0s
pc-job-jhj49   0/1     Completed           0          11s
pc-job-fhwf7   0/1     Pending             0          0s
pc-job-fhwf7   0/1     Pending             0          0s
pc-job-pfcvh   0/1     Completed           0          11s
pc-job-5vg2j   0/1     Pending             0          0s
pc-job-fhwf7   0/1     ContainerCreating   0          0s
pc-job-5vg2j   0/1     Pending             0          0s
pc-job-5vg2j   0/1     ContainerCreating   0          0s
pc-job-fhwf7   1/1     Running             0          2s
pc-job-v7rhr   1/1     Running             0          2s
pc-job-5vg2j   1/1     Running             0          3s
pc-job-fhwf7   0/1     Completed           0          12s
pc-job-v7rhr   0/1     Completed           0          12s
pc-job-5vg2j   0/1     Completed           0          12s
[root@master01 ~/yaml]# kubectl delete -f pc-job.yaml #删除
job.batch "pc-job" deleted
```

## cronjob(CJ)

  CronJob控制器以Job控制器资源为其管控对象，并借助它管理pod资源对象，Job控制器定义的作业任务在其控制器资源创建之后便会立即执行，但CronJob可以以类似于Linux操作系统的周期性任务作业计划的方式控制其运行**时间点**及**重复运行**的方式。也就是说，**CronJob*可以在特定的时间点(****反复的)去运行job任务。

![image-20210819113626958](kubernetes操作实战.assets\image-20210819113626958.png)



CronJob的资源清单文件：

```bash
apiVersion: batch/v1beta1 # 版本号
kind: CronJob # 类型       
metadata: # 元数据
  name: # rs名称 
  namespace: # 所属命名空间 
  labels: #标签
    controller: cronjob
spec: # 详情描述
  schedule: # cron格式的作业调度运行时间点,用于控制任务在什么时间执行
  concurrencyPolicy: # 并发执行策略，用于定义前一次作业运行尚未完成时是否以及如何运行后一次的作业
  failedJobHistoryLimit: # 为失败的任务执行保留的历史记录数，默认为1
  successfulJobHistoryLimit: # 为成功的任务执行保留的历史记录数，默认为3
  startingDeadlineSeconds: # 启动作业错误的超时时长
  jobTemplate: # job控制器模板，用于为cronjob控制器生成job对象;下面其实就是job的定义
    metadata:
    spec:
      completions: 1
      parallelism: 1
      activeDeadlineSeconds: 30
      backoffLimit: 6
      manualSelector: true
      selector:
        matchLabels:
          app: counter-pod
        matchExpressions: 规则
          - {key: app, operator: In, values: [counter-pod]}
      template:
        metadata:
          labels:
            app: counter-pod
        spec:
          restartPolicy: Never 
          containers:
          - name: counter
            image: busybox:1.30
            command: ["bin/sh","-c","for i in 9 8 7 6 5 4 3 2 1; do echo $i;sleep 20;done"]
```

> 需要重点解释的几个选项：
>
> schedule: cron表达式，用于指定任务的执行时间
>
> ​    */1  *   *  *   *
>
> ​    <分钟> <小时> <日> <月份> <星期>
>
>  
>
>   分钟 值从 0 到 59.
>
>   小时 值从 0 到 23.
>
>   日 值从 1 到 31.
>
>   月 值从 1 到 12.
>
>   星期 值从 0 到 6, 0 代表星期日
>
>   多个时间可以用逗号隔开； 范围可以用连字符给出；*可以作为通配符； /表示每...
>
> concurrencyPolicy:
>
> ​    Allow:  允许Jobs并发运行(默认)
>
> ​    Forbid: 禁止并发运行，如果上一次运行尚未完成，则跳过下一次运行
>
> ​    Replace: 替换，取消当前正在运行的作业并用新作业替换它

创建pc-cronjob.yaml，内容如下：

```bash
apiVersion: batch/v1
kind: CronJob
metadata:
  name: pc-cronjob
  namespace: dev
  labels:
    controller: cronjob
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    metadata:
    spec:
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: counter
            image: busybox:1.30
            command: ["bin/sh","-c","for i in 9 8 7 6 5 4 3 2 1; do echo $i;sleep 3;done"]
```

```bash
[root@master01 ~/yaml]# kubectl create -f pc-cronjob.yaml 
Warning: batch/v1beta1 CronJob is deprecated in v1.21+, unavailable in v1.25+; use batch/v1 CronJob
cronjob.batch/pc-cronjob created
[root@master01 ~/yaml]# kubectl get c
certificatesigningrequests.certificates.k8s.io  controllerrevisions.apps
clusterinformations.crd.projectcalico.org       cronjobs.batch
clusterrolebindings.rbac.authorization.k8s.io   csidrivers.storage.k8s.io
clusterroles.rbac.authorization.k8s.io          csinodes.storage.k8s.io
componentstatuses                               csistoragecapacities.storage.k8s.io
configmaps                                      customresourcedefinitions.apiextensions.k8s.io
[root@master01 ~/yaml]# kubectl get cronjobs.batch -n dev
NAME         SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
pc-cronjob   */1 * * * *   False     0        52s             60s
[root@master01 ~/yaml]# kubectl get jobs.batch -n dev
NAME                  COMPLETIONS   DURATION   AGE
pc-cronjob-27030591   1/1           29s        69s
pc-cronjob-27030592   0/1           9s         9s
[root@master01 ~/yaml]# kubectl get pods -n dev
NAME                        READY   STATUS      RESTARTS   AGE
pc-cronjob-27030591-ccswg   0/1     Completed   0          90s
pc-cronjob-27030592-57fwg   0/1     Completed   0          30s
[root@master01 ~/yaml]# kubectl delete -f pc-cronjob.yaml 
Warning: batch/v1beta1 CronJob is deprecated in v1.21+, unavailable in v1.25+; use batch/v1 CronJob
cronjob.batch "pc-cronjob" deleted
```

# service

本章节主要介绍kubernetes的流量负载组件：Service和Ingress。

在kubernetes中，pod是应用程序的载体，我们可以通过pod的ip来访问应用程序，但是pod的ip地址不是固定的，这也就意味着不方便直接采用pod的ip对服务进行访问。

  为了解决这个问题，kubernetes提供了Service资源，Service会对提供同一个服务的多个pod进行聚合，并且提供一个统一的入口地址。通过访问Service的入口地址就能访问到后面的pod服务。

![image-20210819113806300](kubernetes操作实战.assets\image-20210819113806300.png)

Service在很多情况下只是一个概念，真正起作用的其实是kube-proxy服务进程，每个Node节点上都运行着一个kube-proxy服务进程。当创建Service的时候会通过api-server向etcd写入创建的service的信息，而kube-proxy会基于监听的机制发现这种Service的变动，然后**它会将最新的**Service***信息转换成对应的访问规则**。

![image-20210819113827271](kubernetes操作实战.assets\image-20210819113827271.png)

```bash
[root@master01 ~/yaml]# yum install ipvsadm

# 10.97.97.97:80 是service提供的访问入口
# 当访问这个入口的时候，可以发现后面有三个pod的服务在等待调用，
# kube-proxy会基于rr（轮询）的策略，将请求分发到其中一个pod上去
# 这个规则会同时在集群内的所有节点上都生成，所以在任何一个节点上访问都可以。
[root@node1 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.97.97.97:80 rr
  -> 10.244.1.39:80               Masq    1      0          0
  -> 10.244.1.40:80               Masq    1      0          0
  -> 10.244.2.33:80               Masq    1      0          0

```

## kube-proxy目前支持三种工作模式

### userspace 模式

​    userspace模式下，kube-proxy会为每一个Service创建一个监听端口，发向Cluster IP的请求被Iptables规则重定向到kube-proxy监听的端口上，kube-proxy根据LB算法选择一个提供服务的Pod并和其建立链接，以将请求转发到Pod上。
   该模式下，kube-proxy充当了一个四层负责均衡器的角色。由于kube-proxy运行在userspace中，在进行转发处理时会增加内核和用户空间之间的数据拷贝，虽然比较稳定，但是效率比较低。

![image-20210819113940522](kubernetes操作实战.assets\image-20210819113940522.png)

### **iptables** **模式**

  iptables模式下，kube-proxy为service后端的每个Pod创建对应的iptables规则，直接将发向Cluster IP的请求重定向到一个Pod IP。

  该模式下kube-proxy不承担四层负责均衡器的角色，只负责创建iptables规则。该模式的优点是较userspace模式效率更高，但不能提供灵活的LB策略，当后端Pod不可用时也无法进行重试。

![image-20210819114004821](kubernetes操作实战.assets\image-20210819114004821.png)

### **ipvs** **模式**

  ipvs模式和iptables类似，kube-proxy监控Pod的变化并创建相应的ipvs规则。ipvs相对iptables转发效率更高。除此以外，ipvs支持更多的LB算法。

![image-20210819114034368](kubernetes操作实战.assets\image-20210819114034368.png)

\# 此模式必须安装ipvs内核模块，否则会降级为iptables

\# 开启ipvs

![image-20210819114054960](kubernetes操作实战.assets\image-20210819114054960.png)

```bash
[root@master01 ~/yaml]# kubectl edit configmaps kube-proxy -n kube-system
Edit cancelled, no changes made.
[root@master01 ~/yaml]# kubectl edit cm kube-proxy -n kube-system
Edit cancelled, no changes made.
[root@master ~]# kubectl delete pod -l k8s-app=kube-proxy -n kube-system
[root@node1 ~]# ipvsadm -Ln
```

![image-20210819114117648](kubernetes操作实战.assets\image-20210819114117648.png)

### service 类型

Service的资源清单文件：

```bash
kind: Service  # 资源类型
apiVersion: v1  # 资源版本
metadata: # 元数据
  name: service # 资源名称
  namespace: dev # 命名空间
spec: # 描述
  selector: # 标签选择器，用于确定当前service代理哪些pod
    app: nginx
  type: # Service类型，指定service的访问方式
  clusterIP:  # 虚拟服务的ip地址
  sessionAffinity: # session亲和性，支持ClientIP、None两个选项
  ports: # 端口信息
    - protocol: TCP 
      port: 3017  # service端口
      targetPort: 5003 # pod端口
      nodePort: 31122 # 主机端口
```

> Ø ClusterIP：默认值，它是Kubernetes系统自动分配的虚拟IP，只能在集群内部访问
>
> Ø NodePort：将Service通过指定的Node上的端口暴露给外部，通过此方法，就可以在集群外部访问服务
>
> Ø LoadBalancer：使用外接负载均衡器完成到服务的负载分发，注意此模式需要外部云环境支持
>
> Ø ExternalName： 把集群外部的服务引入集群内部，直接使用

环境准备

在使用service之前，首先利用Deployment创建出3个pod，注意要为pod设置`app=nginx-pod`的标签

创建deployment.yaml，内容如下：

```bash
[root@master01 ~/yaml]# cat deployment.yaml 
apiVersion: apps/v1
kind: Deployment      
metadata:
  name: pc-deployment
  namespace: dev
spec: 
  replicas: 3
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80

[root@master01 ~/yaml]# kubectl create -f deployment.yaml 
deployment.apps/pc-deployment created
[root@master01 ~/yaml]# kubectl get deployments.apps,pod -n dev
NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/pc-deployment   3/3     3            3           16s

NAME                                 READY   STATUS    RESTARTS   AGE
pod/pc-deployment-8576748ffb-26fp5   1/1     Running   0          16s
pod/pc-deployment-8576748ffb-bghf9   1/1     Running   0          16s
pod/pc-deployment-8576748ffb-ll42m   1/1     Running   0          16s
[root@master01 ~/yaml]# kubectl get deployments.apps,pod -n dev -o wide
NAME                            READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES         SELECTOR
deployment.apps/pc-deployment   3/3     3            3           25s   nginx        nginx:latest   app=nginx-pod

NAME                                 READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
pod/pc-deployment-8576748ffb-26fp5   1/1     Running   0          25s   10.244.196.149   node01   <none>           <none>
pod/pc-deployment-8576748ffb-bghf9   1/1     Running   0          25s   10.244.196.150   node01   <none>           <none>
pod/pc-deployment-8576748ffb-ll42m   1/1     Running   0          25s   10.244.196.166   node01   <none>           <none>
```

\# 为了方便后面的测试，修改下三台nginx的index.html页面（三台修改的IP地址不一致）

```bash
# kubectl exec -it pc-deployment-8576748ffb-26fp5 -n dev /bin/sh
# echo "10.244.196.149" > /usr/share/nginx/html/index.html
[root@master01 ~/yaml]# kubectl exec -it pc-deployment-8576748ffb-26fp5 -n dev /bin/sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
# echo "10.244.196.149" > /usr/share/nginx/html/index.html
# exit
[root@master01 ~/yaml]# kubectl exec -it pc-deployment-8576748ffb-bghf9 -n dev /bin/sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
# echo "10.244.196.150" > /usr/share/nginx/html/index.html
# exit
[root@master01 ~/yaml]# kubectl exec -it pc-deployment-8576748ffb-ll42m -n dev /bin/sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
# echo "10.244.196.166" > /usr/share/nginx/html/index.html
# exit
[root@master01 ~/yaml]# for i in `kubectl get pod -n dev -o wide|awk 'NR==2,NR==4 {print $6}'`;do curl $i;echo "-------------------------";done
10.244.196.149
-------------------------
10.244.196.150
-------------------------
10.244.196.166
-------------------------
```

这里想用shell 偷懒没想到echo 命令执行成功了就是内容不变（后面再探索）

#### ClusterIP类型的Service

创建service-clusterip.yaml文件

```bash
[root@master01 ~/yaml]# kubectl create -f service-clusterip.yaml 
service/service-clusterip created
[root@master01 ~/yaml]# kubectl get service -n dev -o wide
NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE   SELECTOR
service-clusterip   ClusterIP   10.103.70.180   <none>        80/TCP    19s   app=nginx-pod
[root@master01 ~/yaml]# cat service-clusterip.yaml 
apiVersion: v1
kind: Service
metadata:
  name: service-clusterip
  namespace: dev
spec:
  selector:
    app: nginx-pod
  clusterIP:  # service的ip地址，如果不写，默认会生成一个
  type: ClusterIP
  ports:
  - port: 80  # Service端口       
    targetPort: 80 # pod端口
[root@master01 ~/yaml]# kubectl describe service service-clusterip -n dev
Name:              service-clusterip
Namespace:         dev
Labels:            <none>
Annotations:       <none>
Selector:          app=nginx-pod
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.103.70.180
IPs:               10.103.70.180
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.196.149:80,10.244.196.150:80,10.244.196.166:80
Session Affinity:  None
Events:            <none>
# 查看service的详细信息
# 在这里有一个Endpoints列表，里面就是当前service可以负载到的服务入口
[root@master01 ~/yaml]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.103.70.180:80 rr
  -> 10.244.196.149:80            Masq    1      0          0         
  -> 10.244.196.150:80            Masq    1      0          0         
  -> 10.244.196.166:80            Masq    1      0          0      
[root@master01 ~/yaml]# curl 10.103.70.180
10.244.196.166
[root@master01 ~/yaml]# curl 10.103.70.180
10.244.196.150
[root@master01 ~/yaml]# curl 10.103.70.180
10.244.196.149
[root@master01 ~/yaml]# curl 10.103.70.180
10.244.196.166
[root@master01 ~/yaml]# curl 10.103.70.180
10.244.196.150
[root@master01 ~/yaml]# curl 10.103.70.180
10.244.196.149
```

**Endpoint**

  Endpoint是kubernetes中的一个资源对象，存储在etcd中，用来记录一个service对应的所有pod的访问地址，它是根据service配置文件中selector描述产生的。

  一个Service由一组Pod组成，这些Pod通过Endpoints暴露出来，**Endpoints**是实现实际服务的端点集合。换句话说，service和pod之间的联系是通过endpoints实现的。

![image-20210819123514749](kubernetes操作实战.assets\image-20210819123514749.png)

```bash
[root@master01 ~/yaml]# kubectl get endpoints -n dev -o wide
```

**负载分发策略**

对Service的访问被分发到了后端的Pod上去，目前kubernetes提供了两种负载分发策略：

Ø 如果不定义，默认使用kube-proxy的策略，比如随机、轮询

Ø 基于客户端地址的会话保持模式，即来自同一个客户端发起的所有请求都会转发到固定的一个Pod上

 此模式可以使在spec中添加`sessionAffinity:ClientIP`选项

```bash
[root@master01 ~/yaml]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.103.70.180:80 rr
  -> 10.244.196.149:80            Masq    1      0          0         
  -> 10.244.196.150:80            Masq    1      0          0         
  -> 10.244.196.166:80            Masq    1      0          0      
[root@master01 ~/yaml]# curl 10.103.70.180
10.244.196.166
[root@master01 ~/yaml]# curl 10.103.70.180
10.244.196.150
[root@master01 ~/yaml]# curl 10.103.70.180
10.244.196.149
[root@master01 ~/yaml]# curl 10.103.70.180
10.244.196.166
[root@master01 ~/yaml]# curl 10.103.70.180
10.244.196.150
[root@master01 ~/yaml]# curl 10.103.70.180
10.244.196.149
# 添加分发策略----sessionAffinity:ClientIP
[root@master01 ~/yaml]# curl 10.103.70.180
10.244.196.149
[root@master01 ~/yaml]# curl 10.103.70.180
10.244.196.149
[root@master01 ~/yaml]# curl 10.103.70.180
10.244.196.149
#删除
[root@master01 ~/yaml]# kubectl delete -f service-clusterip.yaml 
service "service-clusterip" deleted
```

#### HeadLiness类型的Service

  在某些场景中，开发人员可能不想使用Service提供的负载均衡功能，而希望自己来控制负载均衡策略，针对这种情况，kubernetes提供了HeadLiness Service，这类Service不会分配Cluster IP，如果想要访问service，只能通过service的域名进行查询。

创建service-headliness.yaml

```bash
apiVersion: v1
kind: Service
metadata:
  name: service-headliness
  namespace: dev
spec:
  selector:
    app: nginx-pod
  clusterIP: None # 将clusterIP设置为None，即可创建headliness Service
  type: ClusterIP
  ports:
  - port: 80    
    targetPort: 80
[root@master01 ~/yaml]# kubectl create -f service-headliness.yaml 
service/service-headliness created
[root@master01 ~/yaml]# kubectl get service service-headliness -n dev -o wide
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   SELECTOR
service-headliness   ClusterIP   None         <none>        80/TCP    32s   app=nginx-pod
[root@master01 ~/yaml]# kubectl describe service service-headliness -n dev
Name:              service-headliness
Namespace:         dev
Labels:            <none>
Annotations:       <none>
Selector:          app=nginx-pod
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                None
IPs:               None
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.196.149:80,10.244.196.150:80,10.244.196.166:80
Session Affinity:  None
Events:            <none>
[root@master01 ~/yaml]# kubectl exec -it pc-deployment-8576748ffb-26fp5 -n dev -- cat /etc/resolv.conf
nameserver 10.96.0.10
search dev.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
[root@master01 ~/yaml]# yum provides dig
[root@master01 ~/yaml]# yum install -y bind-utils
[root@master01 ~/yaml]# dig @10.96.0.10 service-headliness.dev.svc.cluster.local

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.5 <<>> @10.96.0.10 service-headliness.dev.svc.cluster.local
; (1 server found)
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 65495
;; flags: qr aa rd; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;service-headliness.dev.svc.cluster.local. IN A

;; ANSWER SECTION:
service-headliness.dev.svc.cluster.local. 30 IN	A 10.244.196.150
service-headliness.dev.svc.cluster.local. 30 IN	A 10.244.196.149
service-headliness.dev.svc.cluster.local. 30 IN	A 10.244.196.166

;; Query time: 126 msec
;; SERVER: 10.96.0.10#53(10.96.0.10)
;; WHEN: Mon May 24 21:40:24 CST 2021
;; MSG SIZE  rcvd: 237
```

#### NodePort类型的Service

  在之前的样例中，创建的Service的ip地址只有集群内部才可以访问，如果希望将Service暴露给集群外部使用，那么就要使用到另外一种类型的Service，称为NodePort类型。NodePort的工作原理其实就是**将**service**的端口映射到Node**的一个端口上**，然后就可以通过`NodeIp:NodePort`来访问service了。

![image-20210819123738453](kubernetes操作实战.assets\image-20210819123738453.png)

创建service-nodeport.yaml

```bash
apiVersion: v1
kind: Service
metadata:
  name: service-nodeport
  namespace: dev
spec:
  selector:
    app: nginx-pod
  type: NodePort # service类型
  ports:
  - port: 80
    nodePort: 30002 # 指定绑定的node的端口(默认的取值范围是：30000-32767), 如果不指定，会默认分配
    targetPort: 80
# 创建service
[root@master ~]# kubectl create -f service-nodeport.yaml
service/service-nodeport created

# 查看service
[root@master ~]# kubectl get svc -n dev -o wide
NAME               TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)       SELECTOR
service-nodeport   NodePort   10.105.64.191   <none>        80:30002/TCP  app=nginx-pod

# 接下来可以通过电脑主机的浏览器去访问集群中任意一个nodeip的30002端口，即可访问到pod
```

#### LoadBalancer类型的Service

  LoadBalancer和NodePort很相似，目的都是向外部暴露一个端口，区别在于LoadBalancer会在集群的外部再来做一个负载均衡设备，而这个设备需要外部环境支持的，外部服务发送到这个设备上的请求，会被设备负载之后转发到集群中。

![image-20210819123824661](kubernetes操作实战.assets\image-20210819123824661.png)

需要vip地址。这个例子刚好就是这个环境VIP192.168.1.26

#### ExternalName类型的Service

   ExternalName类型的Service用于引入集群外部的服务，它通过`externalName`属性指定外部一个服务的地址，然后在集群内部访问此service就可以访问到外部的服务了。

![image-20210819123903511](kubernetes操作实战.assets\image-20210819123903511.png)

```bash
apiVersion: v1
kind: Service
metadata:
  name: service-externalname
  namespace: dev
spec:
  type: ExternalName # service类型
  externalName: www.baidu.com  #改成ip地址也可以
```

```bash
# 创建service
[root@master ~]# kubectl  create -f service-externalname.yaml
service/service-externalname created

# 域名解析
[root@master ~]# dig @10.96.0.10 service-externalname.dev.svc.cluster.local
service-externalname.dev.svc.cluster.local. 30 IN CNAME www.baidu.com.
www.baidu.com.          30      IN      CNAME   www.a.shifen.com.
www.a.shifen.com.       30      IN      A       39.156.66.18
www.a.shifen.com.       30      IN      A       39.156.66.14
```

# ingress 

在前面课程中已经提到，Service对集群之外暴露服务的主要方式有两种：NotePort和LoadBalancer，但是这两种方式，都有一定的缺点：

NodePort方式的缺点是会占用很多集群机器的端口，那么当集群服务变多的时候，这个缺点就愈发明显

LB方式的缺点是每个service需要一个LB，浪费、麻烦，并且需要kubernetes之外设备的支持

基于这种现状，kubernetes提供了Ingress资源对象，Ingress只需要一个NodePort或者一个LB就可以满足暴露多个Service的需求。工作机制大致如下图表示：

![image-20210819124032514](kubernetes操作实战.assets\image-20210819124032514.png)

实际上，Ingress相当于一个7层的负载均衡器，是kubernetes对反向代理的一个抽象，它的工作原理类似于Nginx，可以理解成在**Ingress**里建立诸多映射规则，Ingress Controller通过监听这些配置规则并转化成Nginx的反向代理配置 , **然后对外部提供服务。**在这里有两个核心概念：

> Ø ingress：kubernetes中的一个对象，作用是定义请求如何转发到service的规则
>
> Ø ingress controller：具体实现反向代理及负载均衡的程序，对ingress定义的规则进行解析，根据配置的规则来实现请求转发，实现方式有很多，比如Nginx, Contour, Haproxy等等

Ingress（以Nginx为例）的工作原理如下：

> Ø 用户编写Ingress规则，说明哪个域名对应kubernetes集群中的哪个Service
>
> Ø Ingress控制器动态感知Ingress服务规则的变化，然后生成一段对应的Nginx反向代理配置
>
> Ø Ingress控制器会将生成的Nginx配置写入到一个运行着的Nginx服务中，并动态更新
>
> Ø 到此为止，其实真正在工作的就是一个Nginx了，内部配置了用户定义的请求转发规则

![image-20210819124210995](kubernetes操作实战.assets\image-20210819124210995.png)

## ingress 实践

### 环境准备

ingress-nginx/controller:v0.46.0

官网： bare-metal 方式

https://kubernetes.github.io/ingress-nginx/deploy/

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.46.0/deploy/static/provider/baremetal/deploy.yaml

https://github.com/kubernetes/ingress-nginx/blob/master/docs/deploy/baremetal.md

![image-20210819125215496](kubernetes操作实战.assets\image-20210819125215496.png)

\#修改image源

image: willdockerhub/ingress-nginx-controller:v0.46.0

https://hub.docker.com/r/willdockerhub/ingress-nginx-controller/tags?page=1&ordering=last_updated

```bash
[root@master01 ~/yaml]# kubectl create -f deploy.yaml 
[root@master01 ~/yaml]# kubectl get pod -n ingress-nginx 
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-l9qmm        0/1     Completed   0          2m53s
ingress-nginx-admission-patch-stk2s         0/1     Completed   0          2m53s
ingress-nginx-controller-79bd6c999d-zptnn   1/1     Running     0          2m53s
[root@master01 ~/yaml]# kubectl get svc -n ingress-nginx 
NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.111.57.126   <none>        80:32509/TCP,443:32251/TCP   3m59s
ingress-nginx-controller-admission   ClusterIP   10.105.71.90    <none>        443/TCP                      4m
上面ingress-nginx-admission-create  ingress-nginx-admission-patch 
也不知道是干嘛的，不能启动起来
```

```bash
[root@master01 ~/yaml]# kubectl exec -it ingress-nginx-controller-79bd6c999d-zptnn  -n ingress-nginx -- /nginx-ingress-controller --version
-------------------------------------------------------------------------------
NGINX Ingress controller
  Release:       v0.46.0
  Build:         6348dde672588d5495f70ec77257c230dc8da134
  Repository:    https://github.com/kubernetes/ingress-nginx
  nginx version: nginx/1.19.6
```

### 准备service和pod

![image-20210819125359445](kubernetes操作实战.assets\image-20210819125359445.png)

创建tomcat-nginx.yaml

```bash
[root@master01 ~/yaml]# cat tomcat-nginx.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deployment
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      app: tomcat-pod
  template:
    metadata:
      labels:
        app: tomcat-pod
    spec:
      containers:
      - name: tomcat
        image: tomcat:8.5-jre10-slim
        ports:
        - containerPort: 8080

---

apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: dev
spec:
  selector:
    app: nginx-pod
  clusterIP: None
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 80

---

apiVersion: v1
kind: Service
metadata:
  name: tomcat-service
  namespace: dev
spec:
  selector:
    app: tomcat-pod
  clusterIP: None
  type: ClusterIP
  ports:
  - port: 8080
    targetPort: 8080
```

```bash
[root@master01 ~/yaml]# kubectl create -f tomcat-nginx.yaml 
deployment.apps/nginx-deployment created
deployment.apps/tomcat-deployment created
service/nginx-service created
service/tomcat-service created
[root@master01 ~/yaml]# kubectl get service -n dev
NAME             TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
nginx-service    ClusterIP   None         <none>        80/TCP     16s
tomcat-service   ClusterIP   None         <none>        8080/TCP   16s
```

### http代理

```bash
[root@master01 ~/yaml]# cat http-ingress.yaml 
apiVersion: extensions/v1
kind: Ingress
metadata:
  name: ingress-http
  namespace: dev
spec:
  rules:
  - host: nginx.itheima.com
    http:
      paths:
      - path: /
        backend:
          serviceName: nginx-service
          servicePort: 80
  - host: tomcat.itheima.com
    http:
      paths:
      - path: /
        backend:
          serviceName: tomcat-service
          servicePort: 8080
```

```bash
[root@master01 ~/yaml]# kubectl create -f http-ingress.yaml 
ingress.extensions/ingress-http created
[root@master01 ~/yaml]# kubectl get ingress ingress-http -n dev
NAME           CLASS    HOSTS                                  ADDRESS        PORTS   AGE
ingress-http   <none>   nginx.itheima.com,tomcat.itheima.com   192.168.1.22   80      46m
[root@master01 ~/yaml]# kubectl describe ingress ingress-http -n dev
Name:             ingress-http
Namespace:        dev
Address:          192.168.1.22
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host                Path  Backends
  ----                ----  --------
  nginx.latest.com    
                      /   nginx-service:80 (10.244.196.159:80,10.244.196.160:80,10.244.196.168:80 + 3 more...)
  tomcat.itheima.com  
                      /   tomcat-service:8080 (10.244.196.165:8080,10.244.59.219:8080,10.244.59.220:8080)
Annotations:          <none>
Events:
  Type    Reason  Age                   From                      Message
  ----    ------  ----                  ----                      -------
  Normal  Sync    116s (x2 over 2m22s)  nginx-ingress-controller  Scheduled for sync
```

windows主机上面修改这个

192.168.1.22 nginx.itheima.com tomcat.itheima.com  域名错误是不能访问的。

![image-20210819125539683](kubernetes操作实战.assets\image-20210819125539683.png)

![image-20210819125550660](kubernetes操作实战.assets\image-20210819125550660.png)

![image-20210819125559999](kubernetes操作实战.assets\image-20210819125559999.png)

上面集群地址

https://github.com/kubernetes/ingress-nginx/blob/master/docs/deploy/baremetal.md 这可以查询一些关于其他service信息。

### https代理

```bash
# 生成证书
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/C=CN/ST=BJ/L=BJ/O=nginx/CN=itheima.com"

# 创建密钥
kubectl create secret tls tls-secret --key tls.key --cert tls.crt

[root@master01 ~/yaml]# openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/C=CN/ST=BJ/L=BJ/O=nginx/CN=itheima.com"
Generating a 2048 bit RSA private key
...........+++
..............................+++
writing new private key to 'tls.key'
-----
[root@master01 ~/yaml]# kubectl create secret tls tls-secret --key tls.key --cert tls.crt
secret/tls-secret created
[root@master01 ~/yaml]# kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-vgdmv   kubernetes.io/service-account-token   3      7d11h
tls-secret            kubernetes.io/tls                     2      3m33s
```

创建ingress-https.yaml

```bash
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-https
  namespace: dev
spec:
  tls:
    - hosts:
      - nginx.itheima.com
      - tomcat.itheima.com
      secretName: tls-secret # 指定秘钥
  rules:
  - host: nginx.itheima.com
    http:
      paths:
      - path: /
        backend:
          serviceName: nginx-service
          servicePort: 80
  - host: tomcat.itheima.com
    http:
      paths:
      - path: /
        backend:
          serviceName: tomcat-service
          servicePort: 8080
```

```bash
[root@master01 ~/yaml]# kubectl create -f ingress-https.yaml
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
Error from server (BadRequest): error when creating "ingress-https.yaml": admission webhook "validate.nginx.ingress.kubernetes.io" denied the request: host "nginx.itheima.com" and path "/" is already defined in ingress dev/ingress-http
#因为跟上面的 域名冲突导致
[root@master01 ~/yaml]# kubectl delete -f ingress-http.yaml
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
ingress.extensions "ingress-http" deleted
[root@master01 ~/yaml]# kubectl create -f ingress-https.yaml
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
ingress.extensions/ingress-https created
[root@master01 ~/yaml]# kubectl get ingress ingress-https -n dev
NAME            CLASS    HOSTS                                  ADDRESS   PORTS     AGE
ingress-https   <none>   nginx.itheima.com,tomcat.itheima.com             80, 443   18s
[root@master01 ~/yaml]# kubectl describe ingress ingress-https -n dev
Name:             ingress-https
Namespace:        dev
Address:          192.168.1.22
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
TLS:
  tls-secret terminates nginx.itheima.com,tomcat.itheima.com
Rules:
  Host                Path  Backends
  ----                ----  --------
  nginx.itheima.com
                      /   nginx-service:80 (10.244.196.183:80,10.244.241.111:80,10.244.241.112:80)
  tomcat.itheima.com
                      /   tomcat-service:8080 (10.244.196.191:8080,10.244.59.225:8080,10.244.59.226:8080)
Annotations:          <none>
Events:
  Type    Reason  Age               From                      Message
  ----    ------  ----              ----                      -------
  Normal  Sync    6s (x2 over 41s)  nginx-ingress-controller  Scheduled for sync

[root@master01 ~/yaml]# kubectl get service -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.109.52.35   <none>        80:32655/TCP,443:32284/TCP   6h24m
ingress-nginx-controller-admission   ClusterIP   10.99.201.75   <none>        443/TCP                      6h24m
```

![image-20210819125736961](kubernetes操作实战.assets\image-20210819125736961.png)

![image-20210819125747481](kubernetes操作实战.assets\image-20210819125747481.png)

OK 到这里应该算是完成了 

证书服务后期再调整

ingress-nginx-admission-create-txg42   

ingress-nginx-admission-patch-6mh4c 

这两也不知道是干嘛要在yaml 里面

LB方式需要设置 VIP192.168.1.26才行，但是外面的LB是提供给APISERVER使用的。

为了节约虚拟机性能

```bash
[root@master01 ~/yaml]# kubectl delete -f ingress-https.yaml
[root@master01 ~/yaml]# kubectl delete -f tomcat-nginx.yaml
[root@master01 ~/yaml]# kubectl delete -f deploy.yaml
```

# 数据存储

  在前面已经提到，容器的生命周期可能很短，会被频繁地创建和销毁。那么容器在销毁时，保存在容器中的数据也会被清除。这种结果对用户来说，在某些情况下是不乐意看到的。为了持久化保存容器的数据，kubernetes引入了Volume的概念。

  Volume是Pod中能够被多个容器访问的共享目录，它被定义在Pod上，然后被一个Pod里的多个容器挂载到具体的文件目录下，kubernetes通过Volume实现同一个Pod中不同容器之间的数据共享以及数据的持久化存储。Volume的生命容器不与Pod中单个容器的生命周期相关，当容器终止或者重启时，Volume中的数据也不会丢失。

kubernetes的Volume支持多种类型，比较常见的有下面几个：

Ø 简单存储：EmptyDir、HostPath、NFS

Ø 高级存储：PV、PVC

Ø 配置存储：ConfigMap、Secret



## 基本存储

### EmptyDir

  EmptyDir是最基础的Volume类型，一个EmptyDir就是Host上的一个空目录。

  EmptyDir是在Pod被分配到Node时创建的，它的初始内容为空，并且无须指定宿主机上对应的目录文件，因为kubernetes会自动分配一个目录，当Pod销毁时， EmptyDir中的数据也会被永久删除。 EmptyDir用途如下：

Ø 临时空间，例如用于某些应用程序运行时所需的临时目录，且无须永久保留

Ø 一个容器需要从另一个容器中获取数据的目录（多容器共享目录）

接下来，通过一个容器之间文件共享的案例来使用一下EmptyDir。

  在一个Pod中准备两个容器nginx和busybox，然后声明一个Volume分别挂在到两个容器的目录中，然后nginx容器负责向Volume中写日志，busybox中通过命令将日志内容读到控制台。不需要用户创建，结束后这个目录自动销毁

![image-20210819125955714](kubernetes操作实战.assets\image-20210819125955714.png)

创建一个volume-emptydir.yaml

```bash
[root@master01 ~/yaml]# cat volume-emptydir.yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-emptydir
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:  # 将logs-volume挂在到nginx容器中，对应的目录为 /var/log/nginx
    - name: logs-volume
      mountPath: /var/log/nginx
  - name: busybox
    image: busybox:1.30
    command: ["/bin/sh","-c","tail -f /logs/access.log"] # 初始命令，动态读取指定文件中内容
    volumeMounts:  # 将logs-volume 挂在到busybox容器中，对应的目录为 /logs
    - name: logs-volume
      mountPath: /logs
  volumes: # 声明volume， name为logs-volume，类型为emptyDir
  - name: logs-volume
    emptyDir: {}
[root@master01 ~/yaml]# kubectl create -f volume-emptydir.yaml
pod/volume-emptydir created
[root@master01 ~/yaml]# kubectl get pod -n dev
NAME              READY   STATUS    RESTARTS   AGE
volume-emptydir   2/2     Running   0          7s
[root@master01 ~/yaml]# kubectl get pod -n dev -o wide
NAME              READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
volume-emptydir   2/2     Running   0          28s   10.244.196.177   node01   <none>           <none>
[root@master01 ~/yaml]# curl 10.244.196.177
[root@master01 ~/yaml]# kubectl logs -f volume-emptydir -n dev -c busybox
10.244.241.64 - - [25/May/2021:13:11:37 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"

```

### HostPath

 

  上节课提到，EmptyDir中数据不会被持久化，它会随着Pod的结束而销毁，如果想简单的将数据持久化到主机中，可以选择HostPath。

  HostPath就是将Node主机中一个实际目录挂在到Pod中，以供容器使用，这样的设计就可以保证Pod销毁了，但是数据依据可以存在于Node主机上。

![image-20210819130048045](kubernetes操作实战.assets\image-20210819130048045.png)

创建一个volume-hostpath.yaml：

```bash
apiVersion: v1
kind: Pod
metadata:
  name: volume-hostpath
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
    ports:
    - containerPort: 80
    volumeMounts:
    - name: logs-volume
      mountPath: /var/log/nginx
  - name: busybox
    image: busybox:1.30
    command: ["/bin/sh","-c","tail -f /logs/access.log"]
    volumeMounts:
    - name: logs-volume
      mountPath: /logs
  volumes:
  - name: logs-volume
    hostPath: 
      path: /root/logs
      type: DirectoryOrCreate  # 目录存在就使用，不存在就先创建后使用
```

> 关于type的值的一点说明：
>
> ​    DirectoryOrCreate 目录存在就使用，不存在就先创建后使用
>
> ​    Directory 目录必须存在
>
> ​    FileOrCreate 文件存在就使用，不存在就先创建后使用
>
> ​    File 文件必须存在 
>
>   Socket  unix套接字必须存在
>
> ​    CharDevice  字符设备必须存在
>
> ​    BlockDevice 块设备必须存在

```bash
[root@master01 ~/yaml]# cat volume-hostpath.yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-hostpath
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:
    - name: logs-volume
      mountPath: /var/log/nginx
  - name: busybox
    image: busybox:1.30
    command: ["/bin/sh","-c","tail -f /logs/access.log"]
    volumeMounts:
    - name: logs-volume
      mountPath: /logs
  volumes:
  - name: logs-volume
    hostPath:
      path: /root/logs
      type: DirectoryOrCreate  # 目录存在就使用，不存在就先创建后使用
      
[root@master01 ~/yaml]# kubectl create -f volume-hostpath.yaml
pod/volume-hostpath created
[root@master01 ~/yaml]# kubectl get pod -n dev
NAME              READY   STATUS    RESTARTS   AGE
volume-emptydir   2/2     Running   0          10m
volume-hostpath   2/2     Running   0          33s
[root@master01 ~/yaml]# kubectl get pod -n dev -o wide
NAME              READY   STATUS    RESTARTS   AGE   IP               NODE       NOMINATED NODE   READINESS GATES
volume-emptydir   2/2     Running   0          11m   10.244.196.177   node01     <none>           <none>
volume-hostpath   2/2     Running   0          75s   10.244.241.113   master01   <none>           <none>
[root@master01 ~/yaml]# curl -I 10.244.241.113
HTTP/1.1 200 OK
Server: nginx/1.19.10
Date: Tue, 25 May 2021 13:22:54 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 13 Apr 2021 15:13:59 GMT
Connection: keep-alive
ETag: "6075b537-264"
Accept-Ranges: bytes
注意: 下面的操作需要到Pod所在的节点运行（案例中是master01）
[root@master01 ~/yaml]# ls ~/logs/
access.log  error.log
# 同样的道理，如果在此目录下创建一个文件，到容器中也是可以看到的
```

### nfs

   HostPath可以解决数据持久化的问题，但是一旦Node节点故障了，Pod如果转移到了别的节点，又会出现问题了，此时需要准备单独的网络存储系统，比较常用的用NFS、CIFS。

  NFS是一个网络文件存储系统，可以搭建一台NFS服务器，然后将Pod中的存储直接连接到NFS系统上，这样的话，无论Pod在节点上怎么转移，只要Node跟NFS的对接没问题，数据就可以成功访问。

![image-20210819130204970](kubernetes操作实战.assets\image-20210819130204970.png)

1）首先要准备nfs的服务器，这里为了简单，直接是master节点做nfs服务器

```bash
# 在master上安装nfs服务
[root@master ~]# yum install nfs-utils -y
# 准备一个共享目录
[root@master ~]# mkdir /root/data/nfs -pv
# 将共享目录以读写权限暴露给192.168.109.0/24网段中的所有主机
[root@master ~]# vim /etc/exports
[root@master ~]# more /etc/exports
/root/data/nfs     192.168.109.0/24(rw,no_root_squash)
# 启动nfs服务
[root@master ~]# systemctl start nfs
```

2）接下来，要在的每个node节点上都安装下nfs，这样的目的是为了node节点可以驱动nfs设备

```bash
# 在node上安装nfs服务，注意不需要启动
[root@master ~]# yum install nfs-utils -y
```

3）接下来，就可以编写pod的配置文件了，创建volume-nfs.yaml

```bash
apiVersion: v1
kind: Pod
metadata:
  name: volume-nfs
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
    ports:
    - containerPort: 80
    volumeMounts:
    - name: logs-volume
      mountPath: /var/log/nginx
  - name: busybox
    image: busybox:1.30
    command: ["/bin/sh","-c","tail -f /logs/access.log"] 
    volumeMounts:
    - name: logs-volume
      mountPath: /logs
  volumes:
  - name: logs-volume
    nfs:
      server: 192.168.109.100  #nfs服务器地址
      path: /root/data/nfs #共享文件路径
```

4）最后，运行下pod，观察结果

```bash
# 创建pod
[root@master ~]# kubectl create -f volume-nfs.yaml
pod/volume-nfs created

# 查看pod
[root@master ~]# kubectl get pods volume-nfs -n dev
NAME                  READY   STATUS    RESTARTS   AGE
volume-nfs        2/2     Running   0          2m9s

# 查看nfs服务器上的共享目录，发现已经有文件了
[root@master ~]# ls /root/data/
access.log  error.log
```

## 高级存储

  前面已经学习了使用NFS提供存储，此时就要求用户会搭建NFS系统，并且会在yaml配置nfs。由于kubernetes支持的存储系统有很多，要求客户全都掌握，显然不现实。为了能够屏蔽底层存储实现的细节，方便用户使用， kubernetes引入PV和PVC两种资源对象。

  PV（Persistent Volume）是持久化卷的意思，是对底层的共享存储的一种抽象。一般情况下PV由kubernetes管理员进行创建和配置，它与底层具体的共享存储技术有关，并通过插件完成与共享存储的对接。

  PVC（Persistent Volume Claim）是持久卷声明的意思，是用户对于存储需求的一种声明。换句话说，PVC其实就是用户向kubernetes系统发出的一种资源需求申请.

![image-20210819130345750](kubernetes操作实战.assets\image-20210819130345750.png)

使用了PV和PVC之后，工作可以得到进一步的细分：

Ø 存储：存储工程师维护

Ø PV： kubernetes管理员维护

Ø PVC：kubernetes用户维护

### PV

PV是存储资源的抽象，下面是资源清单文件:

```bash
apiVersion: v1  
kind: PersistentVolume
metadata:
  name: pv2
spec:
  nfs: # 存储类型，与底层真正存储对应
  capacity:  # 存储能力，目前只支持存储空间的设置
    storage: 2Gi
  accessModes:  # 访问模式
  storageClassName: # 存储类别
  persistentVolumeReclaimPolicy: # 回收策略
```

#### PV 的关键配置参数说明：

Ø 存储类型

 底层实际存储的类型，kubernetes支持多种存储类型，每种存储类型的配置都有所差异

Ø 存储能力（capacity）

  目前只支持存储空间的设置( storage=1Gi )，不过未来可能会加入IOPS、吞吐量等指标的配置

Ø 访问模式（accessModes）

 用于描述用户应用对存储资源的访问权限，访问权限包括下面几种方式：

² ReadWriteOnce（RWO）：读写权限，但是只能被单个节点挂载

² ReadOnlyMany（ROX）： 只读权限，可以被多个节点挂载

² ReadWriteMany（RWX）：读写权限，可以被多个节点挂载

 需要注意的是，底层不同的存储类型可能支持的访问模式不同

Ø 回收策略（persistentVolumeReclaimPolicy）

 当PV不再被使用了之后，对其的处理方式。目前支持三种策略：

² Retain （保留） 保留数据，需要管理员手工清理数据

² Recycle（回收） 清除 PV 中的数据，效果相当于执行 rm -rf /thevolume/*

² Delete （删除） 与 PV 相连的后端存储完成 volume 的删除操作，当然这常见于云服务商的存储服务

 需要注意的是，底层不同的存储类型可能支持的回收策略不同

Ø 存储类别

 PV可以通过storageClassName参数指定一个存储类别

² 具有特定类别的PV只能与请求了该类别的PVC进行绑定

² 未设定类别的PV则只能与不请求任何类别的PVC进行绑定

Ø 状态（status）

² 一个 PV 的生命周期中，可能会处于4中不同的阶段：

² - Available（可用）：   表示可用状态，还未被任何 PVC 绑定

² - Bound（已绑定）：   表示 PV 已经被 PVC 绑定

² Released（已释放）： 表示 PVC 被删除，但是资源还未被集群重新声明

² - Failed（失败）：     表示该 PV 的自动回收失败

#### 实验

使用NFS作为存储，来演示PV的使用，创建3个PV，对应NFS中的3个暴露的路径。

```bash
# 创建目录
[root@master ~]# mkdir /root/data/{pv1,pv2,pv3} -pv

# 暴露服务
[root@master ~]# more /etc/exports
/root/data/pv1     192.168.109.0/24(rw,no_root_squash)
/root/data/pv2     192.168.109.0/24(rw,no_root_squash)
/root/data/pv3     192.168.109.0/24(rw,no_root_squash)

# 重启服务
[root@master ~]#  systemctl restart nfs
```

创建pv.yaml

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name:  pv1
spec:
  capacity: 
    storage: 1Gi
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /root/data/pv1
    server: 192.168.109.100

---

apiVersion: v1
kind: PersistentVolume
metadata:
  name:  pv2
spec:
  capacity: 
    storage: 2Gi
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /root/data/pv2
    server: 192.168.109.100
    
---

apiVersion: v1
kind: PersistentVolume
metadata:
  name:  pv3
spec:
  capacity: 
    storage: 3Gi
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /root/data/pv3
    server: 192.168.109.100
```

```bash
# 创建 pv
[root@master ~]# kubectl create -f pv.yaml
persistentvolume/pv1 created
persistentvolume/pv2 created
persistentvolume/pv3 created

# 查看pv
[root@master ~]# kubectl get pv -o wide
NAME   CAPACITY   ACCESS MODES  RECLAIM POLICY  STATUS      AGE   VOLUMEMODE
pv1    1Gi        RWX            Retain        Available    10s   Filesystem
pv2    2Gi        RWX            Retain        Available    10s   Filesystem
pv3    3Gi        RWX            Retain        Available    9s    Filesystem
```

### PVC

PVC是资源的申请，用来声明对存储空间、访问模式、存储类别需求信息。下面是资源清单文件:

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc
  namespace: dev
spec:
  accessModes: # 访问模式
  selector: # 采用标签对PV选择
  storageClassName: # 存储类别
  resources: # 请求空间
    requests:
      storage: 5Gi
```

PVC 的关键配置参数说明：

Ø 访问模式（accessModes）

用于描述用户应用对存储资源的访问权限

Ø 选择条件（selector）

通过Label Selector的设置，可使PVC对于系统中己存在的PV进行筛选

Ø 存储类别（storageClassName）

PVC在定义时可以设定需要的后端存储的类别，只有设置了该class的pv才能被系统选出

Ø 资源请求（Resources ）

描述对存储资源的请求

#### 实验

1) 创建pvc.yaml，申请pv

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
  namespace: dev
spec:
  accessModes: 
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
      
---

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc2
  namespace: dev
spec:
  accessModes: 
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
     
---

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc3
  namespace: dev
spec:
  accessModes: 
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```

```bash
# 创建pvc
[root@master ~]# kubectl create -f pvc.yaml
persistentvolumeclaim/pvc1 created
persistentvolumeclaim/pvc2 created
persistentvolumeclaim/pvc3 created

# 查看pvc
[root@master ~]# kubectl get pvc  -n dev -o wide
NAME   STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE   VOLUMEMODE
pvc1   Bound    pv1      1Gi        RWX                           15s   Filesystem
pvc2   Bound    pv2      2Gi        RWX                           15s   Filesystem
pvc3   Bound    pv3      3Gi        RWX                           15s   Filesystem

# 查看pv
[root@master ~]# kubectl get pv -o wide
NAME  CAPACITY ACCESS MODES  RECLAIM POLICY  STATUS    CLAIM       AGE     VOLUMEMODE
pv1    1Gi        RWx        Retain          Bound    dev/pvc1    3h37m    Filesystem
pv2    2Gi        RWX        Retain          Bound    dev/pvc2    3h37m    Filesystem
pv3    3Gi        RWX        Retain          Bound    dev/pvc3    3h37m    Filesystem   
```

2) 创建pods.yaml, 使用pv

```bash
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  namespace: dev
spec:
  containers:
  - name: busybox
    image: busybox:1.30
    command: ["/bin/sh","-c","while true;do echo pod1 >> /root/out.txt; sleep 10; done;"]
    volumeMounts:
    - name: volume
      mountPath: /root/
  volumes:
    - name: volume
      persistentVolumeClaim:
        claimName: pvc1
        readOnly: false
---
apiVersion: v1
kind: Pod
metadata:
  name: pod2
  namespace: dev
spec:
  containers:
  - name: busybox
    image: busybox:1.30
    command: ["/bin/sh","-c","while true;do echo pod2 >> /root/out.txt; sleep 10; done;"]
    volumeMounts:
    - name: volume
      mountPath: /root/
  volumes:
    - name: volume
      persistentVolumeClaim:
        claimName: pvc2
        readOnly: false   
```

```bash
# 创建pod
[root@master ~]# kubectl create -f pods.yaml
pod/pod1 created
pod/pod2 created

# 查看pod
[root@master ~]# kubectl get pods -n dev -o wide
NAME   READY   STATUS    RESTARTS   AGE   IP            NODE   
pod1   1/1     Running   0          14s   10.244.1.69   node1   
pod2   1/1     Running   0          14s   10.244.1.70   node1  

# 查看pvc
[root@master ~]# kubectl get pvc -n dev -o wide
NAME   STATUS   VOLUME   CAPACITY   ACCESS MODES      AGE   VOLUMEMODE
pvc1   Bound    pv1      1Gi        RWX               94m   Filesystem
pvc2   Bound    pv2      2Gi        RWX               94m   Filesystem
pvc3   Bound    pv3      3Gi        RWX               94m   Filesystem

# 查看pv
[root@master ~]# kubectl get pv -n dev -o wide
NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM       AGE     VOLUMEMODE
pv1    1Gi        RWX            Retain           Bound    dev/pvc1    5h11m   Filesystem
pv2    2Gi        RWX            Retain           Bound    dev/pvc2    5h11m   Filesystem
pv3    3Gi        RWX            Retain           Bound    dev/pvc3    5h11m   Filesystem

# 查看nfs中的文件存储
[root@master ~]# more /root/data/pv1/out.txt
node1
node1
[root@master ~]# more /root/data/pv2/out.txt
node2
node2
```

#### 生命周期

PVC和PV是一一对应的，PV和PVC之间的相互作用遵循以下生命周期：

Ø 资源供应：管理员手动创建底层存储和PV

Ø 资源绑定：用户创建PVC，kubernetes负责根据PVC的声明去寻找PV，并绑定

 在用户定义好PVC之后，系统将根据PVC对存储资源的请求在已存在的PV中选择一个满足条件的

² 一旦找到，就将该PV与用户定义的PVC进行绑定，用户的应用就可以使用这个PVC了

² 如果找不到，PVC则会无限期处于Pending状态，直到等到系统管理员创建了一个符合其要求的PV

 PV一旦绑定到某个PVC上，就会被这个PVC独占，不能再与其他PVC进行绑定了

Ø 资源使用：用户可在pod中像volume一样使用pvc

 Pod使用Volume的定义，将PVC挂载到容器内的某个路径进行使用。

Ø 资源释放：用户删除pvc来释放pv

 当存储资源使用完毕后，用户可以删除PVC，与该PVC绑定的PV将会被标记为“已释放”，但还不能立刻与其他PVC进行绑定。通过之前PVC写入的数据可能还被留在存储设备上，只有在清除之后该PV才能再次使用。

Ø 资源回收：kubernetes根据pv设置的回收策略进行资源的回收

 对于PV，管理员可以设定回收策略，用于设置与之绑定的PVC释放资源之后如何处理遗留数据的问题。只有PV的存储空间完成回收，才能供新的PVC绑定和使用

![image-20210819130835613](kubernetes操作实战.assets\image-20210819130835613.png)

## 配置存储

### configmap

ConfigMap是一种比较特殊的存储卷，它的主要作用是用来存储配置信息的。

创建configmap.yaml，内容如下：

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: configmap
  namespace: dev
data:
  info: |
    username:admin
    password:123456
```

接下来，使用此配置文件创建configmap

```bash
# 创建configmap
[root@master ~]# kubectl create -f configmap.yaml
configmap/configmap created

# 查看configmap详情
[root@master ~]# kubectl describe cm configmap -n dev
Name:         configmap
Namespace:    dev
Labels:       <none>
Annotations:  <none>

Data
====
info:
----
username:admin
password:123456

Events:  <none>
```

接下来创建一个pod-configmap.yaml，将上面创建的configmap挂载进去

```bash
apiVersion: v1
kind: Pod
metadata:
  name: pod-configmap
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
    volumeMounts: # 将configmap挂载到目录
    - name: config
      mountPath: /configmap/config
  volumes: # 引用configmap
  - name: config
    configMap:
      name: configmap
```

```bash
# 创建pod
[root@master ~]# kubectl create -f pod-configmap.yaml
pod/pod-configmap created

# 查看pod
[root@master ~]# kubectl get pod pod-configmap -n dev
NAME            READY   STATUS    RESTARTS   AGE
pod-configmap   1/1     Running   0          6s

#进入容器
[root@master ~]# kubectl exec -it pod-configmap -n dev /bin/sh
# cd /configmap/config/
# ls
info
# more info
username:admin
password:123456

# 可以看到映射已经成功，每个configmap都映射成了一个目录
# key--->文件     value---->文件中的内容
# 此时如果更新configmap的内容, 容器中的值也会动态更新
```

### secret

 在kubernetes中，还存在一种和ConfigMap非常类似的对象，称为Secret对象。它主要用于存储敏感信息，例如密码、秘钥、证书等等。

1) 首先使用base64对数据进行编码

```bash
[root@master ~]# echo -n 'admin' | base64 #准备username
YWRtaW4=
[root@master ~]# echo -n '123456' | base64 #准备password
MTIzNDU2
```

2) 接下来编写secret.yaml，并创建Secret

```bash
apiVersion: v1
kind: Secret
metadata:
  name: secret
  namespace: dev
type: Opaque
data:
  username: YWRtaW4=
  password: MTIzNDU2
```

```bash
# 创建secret
[root@master ~]# kubectl create -f secret.yaml
secret/secret created

# 查看secret详情
[root@master ~]# kubectl describe secret secret -n dev
Name:         secret
Namespace:    dev
Labels:       <none>
Annotations:  <none>
Type:  Opaque
Data
====
password:  6 bytes
username:  5 bytes
```

3) 创建pod-secret.yaml，将上面创建的secret挂载进去：

```bash
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
    volumeMounts: # 将secret挂载到目录
    - name: config
      mountPath: /secret/config
  volumes:
  - name: config
    secret:
      secretName: secret
```

```bash
# 创建pod
[root@master ~]# kubectl create -f pod-secret.yaml
pod/pod-secret created

# 查看pod
[root@master ~]# kubectl get pod pod-secret -n dev
NAME            READY   STATUS    RESTARTS   AGE
pod-secret      1/1     Running   0          2m28s

# 进入容器，查看secret信息，发现已经自动解码了
[root@master ~]# kubectl exec -it pod-secret /bin/sh -n dev
/ # ls /secret/config/
password  username
/ # more /secret/config/username
admin
/ # more /secret/config/password
123456
```

至此，已经实现了利用secret实现了信息的编码.

# 安全认证

## 访问控制概述

  Kubernetes作为一个分布式集群的管理工具，保证集群的安全性是其一个重要的任务。所谓的安全性其实就是保证对Kubernetes的各种客户端进行认证和鉴权操作。

**客户端**

在Kubernetes集群中，客户端通常有两类：

User Account：一般是独立于kubernetes之外的其他服务管理的用户账号。

Service Account：kubernetes管理的账号，用于为Pod中的服务进程在访问Kubernetes时提供身份标识。

![image-20210819131254567](kubernetes操作实战.assets\image-20210819131254567.png)

**认证、授权与准入控制** 

ApiServer是访问及管理资源对象的唯一入口。任何一个请求访问ApiServer，都要经过下面三个流程：

Authentication（认证）：身份鉴别，只有正确的账号才能够通过认证

Authorization（授权）： 判断用户是否有权限对访问的资源执行特定的动作

Admission Control（准入控制）：用于补充授权机制以实现更加精细的访问控制功能。

![image-20210819131345328](kubernetes操作实战.assets\image-20210819131345328.png)

## 认证管理

Kubernetes集群安全的最关键点在于如何识别并认证客户端身份，它提供了3种客户端身份认证方式：

Ø HTTP Base认证：通过用户名+密码的方式认证

这种认证方式是把“用户名:密码”用BASE64算法进行编码后的字符串放在HTTP请求中的Header Authorization域里发送给服务端。服务端收到后进行解码，获取用户名及密码，然后进行用户身份认证的过程。

Ø HTTP Token认证：通过一个Token来识别合法用户

这种认证方式是用一个很长的难以被模仿的字符串--Token来表明客户身份的一种方式。每个Token对应一个用户名，当客户端发起API调用请求时，需要在HTTP Header里放入Token，API Server接到Token后会跟服务器中保存的token进行比对，然后进行用户身份认证的过程。

Ø HTTPS证书认证：基于CA根证书签名的双向数字证书认证方式

 这种认证方式是安全性最高的一种方式，但是同时也是操作起来最麻烦的一种方式。

![image-20210819131415029](kubernetes操作实战.assets\image-20210819131415029.png)

HTTPS认证大体分为3个过程：

1. 证书申请和下发

HTTPS通信双方的服务器向CA机构申请证书，CA机构下发根证书、服务端证书及私钥给申请者

2. 客户端和服务端的双向认证

 1> 客户端向服务器端发起请求，服务端下发自己的证书给客户端，

   客户端接收到证书后，通过私钥解密证书，在证书中获得服务端的公钥，

   客户端利用服务器端的公钥认证证书中的信息，如果一致，则认可这个服务器

 2> 客户端发送自己的证书给服务器端，服务端接收到证书后，通过私钥解密证书，

   在证书中获得客户端的公钥，并用该公钥认证证书信息，确认客户端是否合法

3. 服务器端和客户端进行通信

 服务器端和客户端协商好加密方案后，客户端会产生一个随机的秘钥并加密，然后发送到服务器端。

 服务器端接收这个秘钥后，双方接下来通信的所有内容都通过该随机秘钥加密

注意: Kubernetes允许同时配置多种认证方式，只要其中任意一个方式认证通过即可

## 授权管理

   授权发生在认证成功之后，通过认证就可以知道请求用户是谁， 然后Kubernetes会根据事先定义的授权策略来决定用户是否有权限访问，这个过程就称为授权。

   每个发送到ApiServer的请求都带上了用户和资源的信息：比如发送请求的用户、请求的路径、请求的动作等，授权就是根据这些信息和授权策略进行比较，如果符合策略，则认为授权通过，否则会返回错误。

API Server目前支持以下几种授权策略：

l AlwaysDeny：表示拒绝所有请求，一般用于测试

l AlwaysAllow：允许接收所有请求，相当于集群不需要授权流程（Kubernetes默认的策略）

l ABAC：基于属性的访问控制，表示使用用户配置的授权规则对用户请求进行匹配和控制

l Webhook：通过调用外部REST服务对用户进行授权

l Node：是一种专用模式，用于对kubelet发出的请求进行访问控制

l RBAC：基于角色的访问控制（kubeadm安装方式下的默认选项）

RBAC(Role-Based Access Control) 基于角色的访问控制，主要是在描述一件事情：**给哪些对象授予了哪些权限**

其中涉及到了下面几个概念：

l 对象：User、Groups、ServiceAccount

l 角色：代表着一组定义在资源上的可操作动作(权限)的集合

l 绑定：将定义好的角色跟用户绑定在一起

![image-20210819131504351](kubernetes操作实战.assets\image-20210819131504351.png)

RBAC引入了4个顶级资源对象：

l Role、ClusterRole：角色，用于指定一组权限

l RoleBinding、ClusterRoleBinding：角色绑定，用于将角色（权限）赋予给对象

**Role**、ClusterRole

一个角色就是一组权限的集合，这里的权限都是许可形式的（白名单）。

```bash
# Role只能对命名空间内的资源进行授权，需要指定nameapce
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  namespace: dev
  name: authorization-role
rules:
- apiGroups: [""]  # 支持的API组列表,"" 空字符串，表示核心API群
  resources: ["pods"] # 支持的资源对象列表
  verbs: ["get", "watch", "list"] # 允许的对资源对象的操作方法列表

# ClusterRole可以对集群范围内资源、跨namespaces的范围资源、非资源类型进行授权
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
 name: authorization-clusterrole
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

需要详细说明的是，rules中的参数：

Ø apiGroups: 支持的API组列表

"","apps", "autoscaling", "batch"

Ø resources：支持的资源对象列表

"services", "endpoints", "pods","secrets","configmaps","crontabs","deployments","jobs",

"nodes","rolebindings","clusterroles","daemonsets","replicasets","statefulsets",

"horizontalpodautoscalers","replicationcontrollers","cronjobs"

Ø verbs：对资源对象的操作方法列表

"get", "list", "watch", "create", "update", "patch", "delete", "exec"



**RoleBinding、ClusterRoleBinding**

角色绑定用来把一个角色绑定到一个目标对象上，绑定目标可以是User、Group或者ServiceAccount。

```bash
# RoleBinding可以将同一namespace中的subject绑定到某个Role下，则此subject即具有该Role定义的权限
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: authorization-role-binding
  namespace: dev
subjects:
- kind: User
  name: heima
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: authorization-role
  apiGroup: rbac.authorization.k8s.io

# ClusterRoleBinding在整个集群级别和所有namespaces将特定的subject与ClusterRole绑定，授予权限
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
 name: authorization-clusterrole-binding
subjects:
- kind: User
  name: heima
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: authorization-clusterrole
  apiGroup: rbac.authorization.k8s.io

```

**RoleBinding引用ClusterRole进行授权**

RoleBinding可以引用ClusterRole，对属于同一命名空间内ClusterRole定义的资源主体进行授权。

  一种很常用的做法就是，集群管理员为集群范围预定义好一组角色（ClusterRole），然后在多个命名空间中重复使用这些ClusterRole。这样可以大幅提高授权管理工作效率，也使得各个命名空间下的基础性授权规则与使用体验保持一致。

```bash
# 虽然authorization-clusterrole是一个集群角色，但是因为使用了RoleBinding
# 所以heima只能读取dev命名空间中的资源
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: authorization-role-binding-ns
  namespace: dev
subjects:
- kind: User
  name: heima
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: authorization-clusterrole
  apiGroup: rbac.authorization.k8s.io
```

### **实战：创建一个只能管理dev空间下Pods资源的账号**

1) 创建账号

```bash
# 1) 创建证书
[root@master pki]# cd /etc/kubernetes/pki/
[root@master pki]# (umask 077;openssl genrsa -out devman.key 2048)

# 2) 用apiserver的证书去签署
# 2-1) 签名申请，申请的用户是devman,组是devgroup
[root@master pki]# openssl req -new -key devman.key -out devman.csr -subj "/CN=devman/O=devgroup"     
# 2-2) 签署证书
[root@master pki]# openssl x509 -req -in devman.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out devman.crt -days 3650

# 3) 设置集群、用户、上下文信息
[root@master pki]# kubectl config set-cluster kubernetes --embed-certs=true --certificate-authority=/etc/kubernetes/pki/ca.crt --server=https://192.168.109.100:6443

[root@master pki]# kubectl config set-credentials devman --embed-certs=true --client-certificate=/etc/kubernetes/pki/devman.crt --client-key=/etc/kubernetes/pki/devman.key

[root@master pki]# kubectl config set-context devman@kubernetes --cluster=kubernetes --user=devman

# 切换账户到devman
[root@master pki]# kubectl config use-context devman@kubernetes
Switched to context "devman@kubernetes".

# 查看dev下pod，发现没有权限
[root@master pki]# kubectl get pods -n dev
Error from server (Forbidden): pods is forbidden: User "devman" cannot list resource "pods" in API group "" in the namespace "dev"

# 切换到admin账户
[root@master pki]# kubectl config use-context kubernetes-admin@kubernetes
Switched to context "kubernetes-admin@kubernetes".
```

2.创建Role和RoleBinding，为devman用户授权

```bash
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  namespace: dev
  name: dev-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
  
---

kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: authorization-role-binding
  namespace: dev
subjects:
- kind: User
  name: devman
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: dev-role
  apiGroup: rbac.authorization.k8s.io
  
  
[root@master pki]# kubectl create -f dev-role.yaml
role.rbac.authorization.k8s.io/dev-role created
rolebinding.rbac.authorization.k8s.io/authorization-role-binding created

```

3) 切换账户，再次验证

```bash
# 切换账户到devman
[root@master pki]# kubectl config use-context devman@kubernetes
Switched to context "devman@kubernetes".

# 再次查看
[root@master pki]# kubectl get pods -n dev
NAME                                 READY   STATUS             RESTARTS   AGE
nginx-deployment-66cb59b984-8wp2k    1/1     Running            0          4d1h
nginx-deployment-66cb59b984-dc46j    1/1     Running            0          4d1h
nginx-deployment-66cb59b984-thfck    1/1     Running            0          4d1h

# 为了不影响后面的学习,切回admin账户
[root@master pki]# kubectl config use-context kubernetes-admin@kubernetes
Switched to context "kubernetes-admin@kubernetes".
```

## 准入控制

 

通过了前面的认证和授权之后，还需要经过准入控制处理通过之后，apiserver才会处理这个请求。

准入控制是一个可配置的控制器列表，可以通过在Api-Server上通过命令行设置选择执行哪些准入控制器：

```bash
--admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,
DefaultStorageClass,ResourceQuota,DefaultTolerationSeconds
```

只有当所有的准入控制器都检查通过之后，apiserver才执行该请求，否则返回拒绝。

当前可配置的Admission Control准入控制如下：

l AlwaysAdmit：允许所有请求

l AlwaysDeny：禁止所有请求，一般用于测试

l AlwaysPullImages：在启动容器之前总去下载镜像

l DenyExecOnPrivileged：它会拦截所有想在Privileged Container上执行命令的请求

l ImagePolicyWebhook：这个插件将允许后端的一个Webhook程序来完成admission controller的功能。

l Service Account：实现ServiceAccount实现了自动化

l SecurityContextDeny：这个插件将使用SecurityContext的Pod中的定义全部失效

l ResourceQuota：用于资源配额管理目的，观察所有请求，确保在namespace上的配额不会超标

l LimitRanger：用于资源限制管理，作用于namespace上，确保对Pod进行资源限制

l InitialResources：为未设置资源请求与限制的Pod，根据其镜像的历史资源的使用情况进行设置

l NamespaceLifecycle：如果尝试在一个不存在的namespace中创建资源对象，则该创建请求将被拒绝。当删除一个namespace时，系统将会删除该namespace中所有对象。

l DefaultStorageClass：为了实现共享存储的动态供应，为未指定StorageClass或PV的PVC尝试匹配默认的StorageClass，尽可能减少用户在申请PVC时所需了解的后端存储细节

l DefaultTolerationSeconds：这个插件为那些没有设置forgiveness tolerations并具有notready:NoExecute和unreachable:NoExecute两种taints的Pod设置默认的“容忍”时间，为5min

l PodSecurityPolicy：这个插件用于在创建或修改Pod时决定是否根据Pod的security context和可用的PodSecurityPolicy对Pod的安全策略进行控制

https://kubernetes.io/zh/docs/setup/best-practices/certificates/





# 如何去寻找需要的image

https://github.com/xyz349925756/kubernetes  推荐使用第二种一键脚本拉取镜像



~~相信一路走来寻找image是最大的问题了国内到处是不可描述的原因导致国外高端技术被封锁~~

~~docker :https://hub.docker.com/ 登录进去搜索相关的镜像，一般都会以其他名字存在注意看tag信息和当前版本信息，然后回到yaml修改成自己想要的image~~ 

~~然后再看下官网指导手册。~~

~~常用镜像仓库~~

~~DockerHub镜像仓库:~~

~~https://hub.docker.com/~~

~~google镜像仓库：~~

~~https://gcr.io/google-containers/~~

~~https://gcr.io/kubernetes-helm/~~

~~https://gcr.io/google-containers/pause~~

~~quay.io镜像仓库：~~

~~https://quay.io/repository/~~

~~elastic镜像仓库：~~

~~https://www.docker.elastic.co/~~

~~RedHat镜像仓库：~~

~~https://access.redhat.com/containers~~

~~阿里云镜像仓库：~~

~~https://cr.console.aliyun.com~~

~~华为云镜像仓库：~~

~~https://console.huaweicloud.com/swr~~

~~腾讯云镜像仓库：~~

~~https://console.cloud.tencent.com/tcr~~

~~常用镜像同步神器~~

~~阿里云image-syncer：https://github.com/AliyunContainerService/image-syncer~~

~~腾讯云image-transfer：https://github.com/tkestack/image-transfer~~

~~skopeo-sync：https://github.com/containers/skopeo/blob/master/docs/skopeo-sync.1.md~~



可结合公网CICD工具TravisCI、githubAction、gitlabCI等，将gcr.io镜像定期同步到个人dockerhub账号下或国内镜像仓库中。示例参考：https://github.com/willzhang/image-sync