# k8s第三讲

## 数据存储

承接蕾姆学姐的上一节，我们已经可以使用k8s来创建pod并暴露服务了，但是容器的生命周期可能很短，会被频繁地创建和销毁。那么容器在销毁时，保存在容器中的数据也会被清除。如果生产环境也这样搞，肯定是不行的。所以为了持久化保存容器的数据，kubernetes引入了Volume的概念。

kubernetes的Volume支持多种类型，比较常见的有下面几个：

- 简单存储：EmptyDir、HostPath、NFS
- 高级存储：PV、PVC
- 配置存储：ConfigMap、Secret 

## 基本存储类型

### EmptyDir

EmptyDir是最基础的Volume类型，一个EmptyDir就是Host上的一个空目录。

EmptyDir是在Pod被分配到Node时创建的，它的初始内容为空，并且无须指定宿主机上对应的目录文件，因为kubernetes会自动分配一个目录，当Pod销毁时， EmptyDir中的数据也会被永久删除。 EmptyDir用途如下：

- 临时空间，例如用于某些应用程序运行时所需的临时目录，且无须永久保留
- 一个容器需要从另一个容器中获取数据的目录（多容器共享目录）

接下来，通过一个容器之间文件共享的案例来使用一下EmptyDir。

例子：

在一个Pod中准备两个容器nginx和busybox，然后声明一个Volume分别挂在到两个容器的目录中，然后nginx容器负责向Volume中写日志，busybox中通过命令将日志内容读到stdout。

![image-20250309164310551](https://gitee.com/vkermt/blogimage/raw/master/images/202503091643591.png)

创建一个volume-emptydir.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-emptydir
  namespace: dev
spec:
  containers:
  - name: nginx
    image: harbor.lekai.cn:8443/le/myapp:v1
    ports:
    - containerPort: 80
    volumeMounts:  # 将logs-volume挂在到nginx容器中，对应的目录为 /var/log/nginx
    - name: logs-volume
      mountPath: /var/log/nginx
  - name: busybox
    image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/busybox:1.30
    command: ["/bin/sh","-c","tail -f /logs/access.log"] # 初始命令，动态读取指定文件中内容
    volumeMounts:  # 将logs-volume 挂在到busybox容器中，对应的目录为 /logs
    - name: logs-volume
      mountPath: /logs
  volumes: # 声明volume， name为logs-volume，类型为emptyDir
  - name: logs-volume
    emptyDir: {}
```

```shell
root@master:~/test/emptydir# kubectl create ns dev
namespace/dev created
root@master:~/test/emptydir# kubectl apply -f volume-emp.yaml 
pod/volume-emptydir created
root@master:~/test/emptydir# kubectl get pods -n dev
NAME              READY   STATUS    RESTARTS   AGE
volume-emptydir   2/2     Running   0          11s
root@master:~/test/emptydir# kubectl get pods -n dev -o wide
NAME              READY   STATUS    RESTARTS   AGE   IP               NODE   NOMINATED NODE   READINESS GATES
volume-emptydir   2/2     Running   0          20s   10.244.167.137   node   <none>           <none>
root@master:~/test/emptydir# curl 10.244.167.137 
<!DOCTYPE html>
<html>
<head>
<title>myapp</title>
</head>
<body>
<h1>this is app v1</h1>
<p>the hostname is volume-emptydir</p>
</body>
</html>
root@master:~/test/emptydir# kubectl logs -f volume-emptydir -n dev -c busybox
10.244.219.64 - - [26/Mar/2025:14:30:30 +0000] "GET / HTTP/1.1" 200 156 "-" "curl/7.81.0" "-"
```

### hostpath

hostpath类型的卷允许 Pod 使用宿主机（Node）上的文件系统资源。通过 `hostPath` 卷，容器可以挂载宿主机上的指定路径，使得容器可以直接访问宿主机的文件系统

在 `hostPath` 卷中，可以指定 `type` 字段，它控制路径的==类型和验证方式==：

- **`DirectoryOrCreate`**：如果指定的路径不存在，它会在宿主机上创建该路径，并确保它是一个目录。
- **`FileOrCreate`**：如果指定的路径不存在，它会在宿主机上创建该路径，并确保它是一个文件。
- **`Directory`**：要求宿主机路径是一个目录。如果路径不存在，Kubernetes 会报错。
- **`File`**：要求宿主机路径是一个文件。如果路径不存在，Kubernetes 会报错。
- **`Socket`**：要求宿主机路径是一个 Unix socket 文件。
- **`CharDevice`**：要求宿主机路径是一个字符设备文件。
- **`BlockDevice`**：要求宿主机路径是一个块设备文件。

hostPath 用途如下：

1. **数据持久化**： `hostPath` 可以将容器的数据持久化到宿主机的本地存储。容器停止或重启时，数据仍然可以保存在宿主机的文件系统中，不会丢失。
2. **共享数据**： 容器可以通过 `hostPath` 共享宿主机上的某些数据或目录。例如，多个容器可以通过相同的 `hostPath` 共享数据，进行数据交换。
3. **访问宿主机资源**： 有些容器需要访问宿主机上的资源或特定文件（例如，配置文件、日志文件等）。使用 `hostPath` 可以让容器直接挂载宿主机上的资源路径。
4. **开发和调试**： 在开发或调试过程中，`hostPath` 可以用来快速挂载宿主机上的代码目录，方便容器访问宿主机上的源代码进行开发和调试。

>  需要注意的是，hostpath在底层主机创建的文件或目录只能有root写入。所有容器需要以root身份运行进程，或修改文件权限。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-pod
spec:
  containers:
  - image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/library/busybox:1.28.4
    name: busybox
    command: ["/bin/sh", "-c", "while true; do sleep 3600; done"]
    volumeMounts:
    - mountPath: /pod/data
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      # directory location on host
      path: /data
      # this field is optional
      type: DirectoryOrCreate
```

### NFS

HostPath可以解决数据持久化的问题，但是一旦Node节点故障了，Pod如果转移到了别的节点，又会出现问题了，此时需要准备单独的网络存储系统，比较常用的用NFS、CIFS。

NFS是一个网络文件存储系统，可以搭建一台NFS服务器，然后将Pod中的存储直接连接到NFS系统上，这样的话，无论Pod在节点上怎么转移，只要Node跟NFS的对接没问题，数据就可以成功访问。

![image-20250309165627911](https://gitee.com/vkermt/blogimage/raw/master/images/202503091656963.png)

1）首先要准备nfs的服务器，这里为了简单，我直接是在node节点做nfs服务器

```shell
# ubuntu操作系统
apt install nfs-kernel-server

# 创建一个目录用于nfs服务器将文件共享给客户端，这个目录将会写入到nfs配置文件中
mkdir /nfsdata
chmod 777 /nfsdata

vim /etc/exports
#/nfsdata  *(rw,sync,no_root_squash)
解析：
/nfsroot：指定/nfsroot为nfs服务器的共享目录
*：允许所有的网段访问，也可以使用具体的IP
rw：挂接此目录的客户端对该共享目录具有读写权限
sync：资料同步写入内存和硬盘
no_root_squash：root用户具有对根目录的完全管理访问权限
no_subtree_check：不检查父目录的权限

# 重启
service nfs-kernel-server restart

# 查看当前共享目录
showmount -e localhost

```

2）配置nfs客户端

```shell
apt install nfs-common

# 在nfs挂载服务端的共享目录
mkdir /nfsdata
mount -t nfs -o nolock 172.28.71.240:/nfsdata /nfsdata

-t：挂载的文件系统类型
-o nolock：不要文件锁
172.28.71.240:/nfsroot：nfs服务器ip:服务器共享目录
nfsdata：客户端已存在的目录

mount
umount /nfsdata
```

3）接下来，就可以编写pod的配置文件了，创建volume-nfs.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-nfs
  namespace: dev
spec:
  containers:
  - name: nginx
    image: harbor.lekai.cn:8443/le/myapp:v1
    ports:
    - containerPort: 80
    volumeMounts:
    - name: logs-volume
      mountPath: /var/log/nginx
  - name: busybox
    image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/library/busybox:1.28.4
    command: ["/bin/sh","-c","tail -f /logs/access.log"] 
    volumeMounts:
    - name: logs-volume
      mountPath: /logs
  volumes:
  - name: logs-volume
    nfs:
      server: 172.28.71.240  #nfs服务器地址
      path: /nfsdata #共享文件路径
```

4）最后，运行下pod，观察结果

```shell
# 创建pod
[root@k8s-master01 ~]# kubectl create -f volume-nfs.yaml
pod/volume-nfs created

# 查看pod
[root@k8s-master01 ~]# kubectl get pods volume-nfs -n dev
NAME                  READY   STATUS    RESTARTS   AGE
volume-nfs        2/2     Running   0          2m9s

# 查看nfs服务器上的共享目录，发现已经有文件了
[root@k8s-master01 ~]# ls /root/data/
access.log  error.log
```

### PV

前面已经学习了使用NFS提供存储，此时就要求用户会搭建NFS系统，并且会在yaml配置nfs。由于kubernetes支持的存储系统有很多，要求客户全都掌握，显然不现实。为了能够屏蔽底层存储实现的细节，方便用户使用， kubernetes引入PV和PVC两种资源对象。

PV（Persistent Volume）是持久化卷的意思，是对底层的共享存储的一种抽象。一般情况下PV由kubernetes管理员进行创建和配置，它与底层具体的共享存储技术有关，并通过插件完成与共享存储的对接。

PVC（Persistent Volume Claim）是持久卷声明的意思，是用户对于存储需求的一种声明。换句话说，PVC其实就是用户向kubernetes系统发出的一种资源需求申请。

![image-20250309170230077](https://gitee.com/vkermt/blogimage/raw/master/images/202503091702133.png)

使用了PV和PVC之后，工作可以得到进一步的细分：

- 存储：存储工程师维护
- PV： kubernetes管理员维护
- PVC：kubernetes用户维护

PV是存储资源的抽象，下面是资源清单文件:

```yaml
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

PV 的关键配置参数说明：

- **存储类型**

  底层实际存储的类型，kubernetes支持多种存储类型，每种存储类型的配置都有所差异

- **存储能力（capacity）**

目前只支持存储空间的设置( storage=1Gi )，不过未来可能会加入IOPS、吞吐量等指标的配置

- **访问模式（accessModes）**

  用于描述用户应用对存储资源的访问权限，访问权限包括下面几种方式：

  - ReadWriteOnce（RWO）：读写权限，但是只能被单个节点挂载
  - ReadOnlyMany（ROX）： 只读权限，可以被多个节点挂载
  - ReadWriteMany（RWX）：读写权限，可以被多个节点挂载

  `需要注意的是，底层不同的存储类型可能支持的访问模式不同`

- **回收策略（persistentVolumeReclaimPolicy）**

  当PV不再被使用了之后，对其的处理方式。目前支持三种策略：

  - Retain （保留） 保留数据，需要管理员手工清理数据
  - Recycle（回收） 清除 PV 中的数据，效果相当于执行 rm -rf /thevolume/*
  - Delete （删除） 与 PV 相连的后端存储完成 volume 的删除操作，当然这常见于云服务商的存储服务

  `需要注意的是，底层不同的存储类型可能支持的回收策略不同`

- **存储类别**

  PV可以通过storageClassName参数指定一个存储类别

  - 具有特定类别的PV只能与请求了该类别的PVC进行绑定
  - 未设定类别的PV则只能与不请求任何类别的PVC进行绑定

- **状态（status）**

  一个 PV 的生命周期中，可能会处于4中不同的阶段：

  - Available（可用）： 表示可用状态，还未被任何 PVC 绑定
  - Bound（已绑定）： 表示 PV 已经被 PVC 绑定
  - Released（已释放）： 表示 PVC 被删除，但是资源还未被集群重新声明
  - Failed（失败）： 表示该 PV 的自动回收失败

**实验**

使用NFS作为存储，来演示PV的使用，创建3个PV，对应NFS中的3个暴露的路径。

1) 准备NFS环境

```shell
# 创建目录
[root@nfs ~]# mkdir /nfs/{pv1,pv2,pv3}

# 暴露服务
root@node:/nfsdata# cat /etc/exports 
/nfsdata/pv1 *(rw,sync,no_root_squash)
/nfsdata/pv2 *(rw,sync,no_root_squash)
/nfsdata/pv3 *(rw,sync,no_root_squash)

# 重启服务
root@node:/nfsdata# systemctl restart nfs-kernel-server

root@node:/nfsdata# showmount -e localhost
Export list for localhost:
/nfsdata/pv3 *
/nfsdata/pv2 *
/nfsdata/pv1 *
```

2) 创建pv.yaml

```yaml
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
    path: /nfsdata/pv1
    server: 172.28.71.240

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
    path: /nfsdata/pv2
    server: 172.28.71.240
    
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
    path: /nfsdata/pv3
    server: 172.28.71.240
```

```shell
root@master:~/test# mkdir nfs
root@master:~/test# cd nfs
root@master:~/test/nfs# vim nfs-pv.yaml
root@master:~/test/nfs# kubectl apply -f nfs-pv.yaml 
persistentvolume/pv1 created
persistentvolume/pv2 created
persistentvolume/pv3 created
root@master:~/test/nfs# kubectl get pv -o wide
NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE   VOLUMEMODE
pv1    1Gi        RWX            Retain           Available                                   8s    Filesystem
pv2    2Gi        RWX            Retain           Available                                   8s    Filesystem
pv3    3Gi        RWX            Retain           Available                                   8s    Filesystem
```

### PVC

PVC是资源的申请，用来声明对存储空间、访问模式、存储类别需求信息。下面是资源清单文件:

```yaml
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
      storage: 1Gi
```

PVC 的关键配置参数说明：

- **访问模式（accessModes）**

用于描述用户应用对存储资源的访问权限

- **选择条件（selector）**

  通过Label Selector的设置，可使PVC对于系统中己存在的PV进行筛选

- **存储类别（storageClassName）**

  PVC在定义时可以设定需要的后端存储的类别，只有设置了该class的pv才能被系统选出

- **资源请求（Resources ）**

  描述对存储资源的请求

**实验**

1) 创建pvc.yaml，申请pv

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
  namespace: dev  # pvc为namespace级别的资源对象
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

```shell
root@master:~/test/nfs# vim nfs-pvc.yaml
root@master:~/test/nfs# kubectl apply 0f nfs-pvc.yaml 
error: Unexpected args: [0f nfs-pvc.yaml]
See 'kubectl apply -h' for help and examples
root@master:~/test/nfs# kubectl apply -f nfs-pvc.yaml 
persistentvolumeclaim/pvc1 created
persistentvolumeclaim/pvc2 created
persistentvolumeclaim/pvc3 created
root@master:~/test/nfs# kubectl get pvc -o wide -n dev
NAME   STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE   VOLUMEMODE
pvc1   Bound    pv1      1Gi        RWX                           8s    Filesystem
pvc2   Bound    pv2      2Gi        RWX                           8s    Filesystem
pvc3   Bound    pv3      3Gi        RWX                           8s    Filesystem
```

2) 创建pods.yaml, 使用pv

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  namespace: dev
spec:
  containers:
  - name: busybox
    image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/library/busybox:1.28.4
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
    image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/library/busybox:1.28.4
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

```shell
root@master:~/test/nfs# kubectl apply -f pods.yaml
pod/pod1 created
pod/pod2 created
root@master:~/test/nfs# kubectl get pods -A
NAMESPACE       NAME                                       READY   STATUS      RESTARTS      AGE
dev             pod1                                       1/1     Running     0             6s
dev             pod2                                       1/1     Running     0             6s
ingress-nginx   ingress-nginx-admission-create-22qd2       0/1     Completed   0             11d
ingress-nginx   ingress-nginx-admission-patch-nd554        0/1     Completed   0             11d
ingress-nginx   ingress-nginx-controller-9bfb4c565-8ww2n   1/1     Running     0             11d
kube-system     calico-kube-controllers-65d77f4899-fbpmp   1/1     Running     1 (11d ago)   12d
kube-system     calico-node-7zkgr                          1/1     Running     0             12d
kube-system     calico-node-pzskm                          1/1     Running     0             12d
kube-system     calico-typha-dd5f68c5d-z778f               1/1     Running     0             12d
kube-system     coredns-66f779496c-9dsxb                   1/1     Running     0             12d
kube-system     coredns-66f779496c-v27gd                   1/1     Running     0             12d
kube-system     etcd-master                                1/1     Running     0             12d
kube-system     kube-apiserver-master                      1/1     Running     0             12d
kube-system     kube-controller-manager-master             1/1     Running     17 (9h ago)   12d
kube-system     kube-proxy-2khz9                           1/1     Running     0             12d
kube-system     kube-proxy-lz84z                           1/1     Running     0             12d
kube-system     kube-scheduler-master                      1/1     Running     18 (9h ago)   12d
root@master:~/test/nfs# kubectl get pv
NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM      STORAGECLASS   REASON   AGE
pv1    1Gi        RWX            Retain           Bound    dev/pvc1                           135m
pv2    2Gi        RWX            Retain           Bound    dev/pvc2                           135m
pv3    3Gi        RWX            Retain           Bound    dev/pvc3                           135m
root@master:~/test/nfs# kubectl get pvc -n dev
NAME   STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc1   Bound    pv1      1Gi        RWX                           129m
pvc2   Bound    pv2      2Gi        RWX                           129m
pvc3   Bound    pv3      3Gi        RWX                           129m

root@master:/nfsdata/pv1# ls
out.txt
root@master:/nfsdata/pv1# cat out.txt 
pod1
pod1
pod1
pod1
```

### 动态pv

[k8s存储卷和动态创建pv-CSDN博客](https://blog.csdn.net/m0_73950916/article/details/144563312)

## 配置

### configmap

**为什么要引入configmap**?

 **分离配置和应用代码**

- `ConfigMap` 将配置信息与应用程序代码分离。这意味着你可以在不修改应用程序代码的情况下改变配置信息，避免了每次更改配置都需要重建镜像。
- 配置可以存储为键值对或者文件，可以通过挂载到容器中来读取。

 **灵活的配置管理**

- 配置数据可以非常灵活地存储在 `ConfigMap` 中，可以包括数据库连接字符串、API 密钥、应用程序的运行参数等。
- 在 Kubernetes 中，`ConfigMap` 可以作为环境变量、命令行参数，或者挂载为文件的形式传递给 Pod 中的容器。

 **支持多个环境**

- 使用 `ConfigMap` 可以轻松地在不同的环境（如开发、测试、生产）之间切换配置。通过简单地替换 `ConfigMap`，无需修改应用程序或重新构建容器镜像。
- 同一个应用可以通过不同的 `ConfigMap` 配置，在不同的环境下适配不同的需求。

 **便于管理**

- Kubernetes 提供了管理配置的工具，比如 `kubectl`，可以通过命令行方便地查看、编辑和更新 `ConfigMap`。
- `ConfigMap` 可以作为版本控制的配置存储，便于追踪和更改配置。

 **解耦与重用**

- 配置可以在多个 Pod 或者多个服务之间共享，避免了在每个 Pod 内部重复配置。例如，你可以在多个微服务之间共享同一份 `ConfigMap`。

 **动态更新配置**

- `ConfigMap` 支持在运行时动态更新配置信息。通过监视配置文件的变化，容器可以在不重新启动的情况下读取更新后的配置。这使得在生产环境中更新配置变得更加容易和灵活。

 **安全和管理**

- 配置可以与 Kubernetes 的其他资源一起使用，比如 Secrets（对于敏感配置），确保敏感数据和普通配置的分离与保护。
- 与 Kubernetes 的 RBAC（基于角色的访问控制）结合使用，可以限制哪些用户和服务账户可以读取或修改配置。

### configmap示例

#### **将configmap作为环境变量注入容器**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: user-config
  namespace: default
data:
  name: lanshan
  password: nihao

---

apiVersion: v1
kind: ConfigMap
metadata:
  name: env-config
  namespace: default
data:
  log_level: INFO

---

apiVersion: v1
kind: Pod
metadata:
  name: cm-env-pod
spec:
  containers:
    - name: myapp-container
      image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/library/busybox:1.28.4
      command: [ "/bin/sh", "-c", "while true; do sleep 3600; done" ]
      env:
        - name: USERNAME
          valueFrom:
            configMapKeyRef:
              name: user-config
              key: name
        - name: PASSWORD
          valueFrom:
            configMapKeyRef:
              name: user-config
              key: password  
      envFrom:
        - configMapRef:
            name: env-config
  restartPolicy: Never
```

#### **将configmap作为文件注入**

注入时，key为文件名，value为文件内容

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: user-config
  namespace: default
data:
  name: lekai
  password: hi

---

apiVersion: v1
kind: Pod
metadata:
  name: cm-file-pod
spec:
  containers:
    - name: myapp-container
      image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/library/busybox:1.28.4
      command: [ "/bin/sh", "-c", "while true ; do sleep 3600; done" ]
      volumeMounts:
      - name: config-volume         # 引用Pod定义的共享存储卷的名称
        mountPath: /etc/data            # 容器内的挂载路径
        readOnly: false
  restartPolicy: Never
  volumes:
    - name: config-volume
      configMap:
        name: user-config
```

```sh
kubectl exec -it pod_name进入容器，查看文件是链接文件
```

> configmap可以做到热更新，就是通过链接文件的机制做到的

### 热更新案例

热更新nginx里面的配置文件：

对于需要创建多行数据时，可以使用 `|` 保留换行符，或者使用 `>` 以折叠格式存储数据。

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-default
  namespace: default
data:
  default.conf: |
    server {
        listen       80;
        server_name  localhost;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
    }
  
---

apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-html
  namespace: default
data:
  index.html: "hello Kubernetes"
  
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: config-nginx-deploy
  namespace: default
  labels:
    app: hot-nginx-deploy
spec:
  replicas: 5
  selector:
    matchLabels:
      app: hot-update-nginx
  template:
    metadata:
      name: config-nginx
      labels:
        app: hot-update-nginx
    spec:
      containers:
        - name: myapp-container
          image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:alpine3.20
          volumeMounts:
          - name: default                     # 与volumes命名一致
            mountPath: /etc/nginx/conf.d/           # 容器内的挂载路径
            # subPath: default.conf                # 通过subPath指定挂载的文件名
            readOnly: false
          - name: html
            mountPath: /usr/share/nginx/html/
            # subPath: index.html
      restartPolicy: Always
      volumes:
        - name: default
          configMap:
            name: nginx-default
        - name: html
          configMap:
            name: nginx-html
```

测试热更新功能：

```sh
kubectl edit configmap nginx-default
kubectl edit configmap nginx-html
# 查看是否成功
```

对于nginx来说，更新了配置文件之后，需要重启nginx加载配置文件。

```sh
kubectl edit deploy config-nginx-deploy 

# k8s为无法热更新的中间件提供了一个自动触发滚动更新的字段；通过spec.template.metadata.annotations.version/config字段来触发滚动更新
```

![image-20250213185156283](https://gitee.com/vkermt/blogimage/raw/master/images/202502131852476.png)



#### immutable字段

除此之外，configmap还支持一个字段，使configmap永不可改。

```sh
kubectl explain configmap
```



![image-20250213191348243](https://gitee.com/vkermt/blogimage/raw/master/images/202502131913278.png)

### secret

在kubernetes中，还存在一种和ConfigMap非常类似的对象，称为Secret对象。它主要用于存储敏感信息，例如密码、秘钥、证书等等。

Secret 对象类型用来保存敏感信息，例如密码、0Auth 令牌和 SSH 密钥。将这些信息放在 secret 中比放在 Pod 的定义或者 容器镜像 中来说更加安全和灵活。

特性：

* Kubernetes 通过仅仅将 Secret 分发到需要访问 Secret 的 Pod 所在的机器节点来保障其安全性
* Secret 只会存储在节点的内存中，永不写入物理存储，这样从节点删除 secret 时就不需要擦除磁盘数据
* 从 Kubernetes1.7 版本开始，etcd 会以加密形式存储 Secret，一定程度的保证了 Secret 安全性

Secret 支持多种类型，通过 `type` 字段指定：

- **Opaque**：默认类型，存储任意用户定义的键值对。
- **kubernetes.io/dockerconfigjson**：存储 Docker 镜像仓库的认证信息。

拉去私有镜像时需要此类型的secret

- **kubernetes.io/tls**：存储 TLS 证书（`tls.crt` 和 `tls.key`）。
- **kubernetes.io/service-account-token**：Service Account 的令牌，用于 Pod 身份认证。

#### **Opaque**

```sh
echo -n  "mypassword" | base64
echo -n  "admin" | base64
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  password: bXlwYXNzd29yZA==
  username: YWRtaW4=
```

#### secret env

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: opaque-secret-env
  name: opaque-secret-env-deploy
spec:
  replicas: 5
  selector:
    matchLabels:
      app: op-se-env-pod
  template:
    metadata:
      labels:
        app: op-se-env-pod
    spec:
      containers:
        - image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:alpine3.20
          command: ["/bin/sh", "-c", "while true; do sleep 3600; done"]
          name: myapp-container
          ports:
            - containerPort: 80
          env:
            - name: TEST_USER
              valueFrom:
                secretKeyRef:
                  name: mysecret
                  key: username
            - name: TEST_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysecret
                  key: password
```

```sh
[root@k8s-master k8s]# kubectl exec -it opaque-secret-env-deploy-7579c9c8c-4r459 -- sh
/ # env
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
HOSTNAME=opaque-secret-env-deploy-7579c9c8c-4r459
SHLVL=1
HOME=/root
TEST_PASSWORD=mypassword
PKG_RELEASE=1
TEST_USER=admin
DYNPKG_RELEASE=1
TERM=xterm
NGINX_VERSION=1.27.2
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
NJS_VERSION=0.8.6
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
NJS_RELEASE=1
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/
```

#### secret file

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: opaque-secret-env
  name: opaque-secret-env-deploy
spec:
  replicas: 5
  selector:
    matchLabels:
      app: op-se-env-pod
  template:
    metadata:
      labels:
        app: op-se-env-pod
    spec:
      containers:
        - image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:alpine3.20
          command: ["/bin/sh", "-c", "while true; do sleep 3600; done"]
          name: myapp-container
          ports:
            - containerPort: 80
          volumeMounts:
            - name: secret-volume          # 引用Pod定义的共享存储卷的名称
              mountPath: /data/             # 容器内的挂载路径
              readOnly: false              # 是否为只读模式
      volumes:
      - name: secret-volume            # Secret类型存储卷
        secret:
          secretName: mysecret         # 引用Secret对象
          defaultMode: 256            # 设置文件权限为400
          items:                       # 挂载secret中部分键值对
            - key: password
              path: test.txt
```

==同样secret也有immutable字段，可以设置secret为不可改变==

### downwardapi

Downward API 是 Kubernetes 中的一个功能，它允许容器在运行时从 Kubernetes API 服务器获取有关它们自身的信息。这些信息可以作为容器内部的环境变量或文件注入到容器中，以便容器可以获取有关其运行环境的各种信息，如 Pod 名称、命名空间、标签等

#### 获取信息作为env

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: downward-api-env-example
spec:
  containers:
  - name: my-container
    image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:alpine3.20 
    env:
    - name: POD_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.name
    - name: POD_NAMESPACE
      valueFrom:
        fieldRef:
          fieldPath: metadata.namespace
    - name: POD_IP
      valueFrom:
        fieldRef:
          fieldPath: status.podIP
    - name: CPU_REQUEST
      valueFrom:
        resourceFieldRef:
          resource: requests.cpu
    - name: CPU_LIMIT
      valueFrom:
        resourceFieldRef:
          resource: limits.cpu
    - name: MEMORY_REQUEST
      valueFrom:
        resourceFieldRef:
          resource: requests.memory
    - name: MEMORY_LIMIT
      valueFrom:
        resourceFieldRef:
          resource: limits.memory
  restartPolicy: Never
```

#### 获取信息作为卷

可以挂载到指定路径的文件中

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: downward-api-volume-example
spec:
  containers:
  - name: my-container
    image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:alpine3.20 
    resources:
      limits:
        cpu: "1"
        memory: "512Mi"
      requests:
        cpu: "0.5"
        memory: "256Mi"
    volumeMounts:
    - name: downward-api-volume
      mountPath: /podinfo
  volumes:
  - name: downward-api-volume
    downwardAPI:
      items:
      - path: "annotations"
        fieldRef:
          fieldPath: metadata.annotations
      - path: "labels"
        fieldRef:
          fieldPath: metadata.labels
      - path: "name"
        fieldRef:
          fieldPath: metadata.name
      - path: "namespace"
        fieldRef:
          fieldPath: metadata.namespace
      - path: "uid"
        fieldRef:
          fieldPath: metadata.uid
      - path: "cpuRequest"
        resourceFieldRef:
          containerName: my-container
          resource: requests.cpu
      - path: "memoryRequest"
        resourceFieldRef:
          containerName: my-container
          resource: requests.memory
      - path: "cpuLimit"
        resourceFieldRef:
          containerName: my-container
          resource: limits.cpu
      - path: "memoryLimit"
        resourceFieldRef:
          containerName: my-container
          resource: limits.memory
  restartPolicy: Never
```

```sh
[root@k8s-master k8s]# kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
downward-api-env-example      1/1     Running   0          77m
downward-api-volume-example   1/1     Running   0          8m47s
[root@k8s-master k8s]# kubectl exec -it downward-api-volume-example -- sh
/ # cd podinfo
/podinfo # ls
annotations    cpuLimit       cpuRequest     labels         memoryLimit    memoryRequest  name           namespace      uid
/podinfo # cat annotations
kubectl.kubernetes.io/last-applied-configuration="{\"apiVersion\":\"v1\",\"kind\":\"Pod\",\"metadata\":{\"annotations\":{},\"name\":\"downward-api-volume-example\",\"namespace\":\"default\"},\"spec\":{\"containers\":[{\"image\":\"swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:alpine3.20\",\"name\":\"my-container\",\"resources\":{\"limits\":{\"cpu\":\"1\",\"memory\":\"512Mi\"},\"requests\":{\"cpu\":\"0.5\",\"memory\":\"256Mi\"}},\"volumeMounts\":[{\"mountPath\":\"/podinfo\",\"name\":\"downward-api-volume\"}]}],\"restartPolicy\":\"Never\",\"volumes\":[{\"downwardAPI\":{\"items\":[{\"fieldRef\":{\"fieldPath\":\"metadata.annotations\"},\"path\":\"annotations\"},{\"fieldRef\":{\"fieldPath\":\"metadata.labels\"},\"path\":\"labels\"},{\"fieldRef\":{\"fieldPath\":\"metadata.name\"},\"path\":\"name\"},{\"fieldRef\":{\"fieldPath\":\"metadata.namespace\"},\"path\":\"namespace\"},{\"fieldRef\":{\"fieldPath\":\"metadata.uid\"},\"path\":\"uid\"},{\"path\":\"cpuRequest\",\"resourceFieldRef\":{\"containerName\":\"my-container\",\"resource\":\"requests.cpu\"}},{\"path\":\"memoryRequest\",\"resourceFieldRef\":{\"containerName\":\"my-container\",\"resource\":\"requests.memory\"}},{\"path\":\"cpuLimit\",\"resourceFieldRef\":{\"containerName\":\"my-container\",\"resource\":\"limits.cpu\"}},{\"path\":\"memoryLimit\",\"resourceFieldRef\":{\"containerName\":\"my-container\",\"resource\":\"limits.memory\"}}]},\"name\":\"downward-api-volume\"}]}}\n"
kubernetes.io/config.seen="2025-02-14T12:40:27.371063554+08:00"
```

#### 扩展

如果要跨pod获取pod和容器的元数据，以上的方式就不行了；不过可以通过k8s官方给的接口，通过给予一定权限，访问获取数据。此时去访问的pod就是client端就类似于kubectl，k8s集群就是server端。

==通过下一方式查看k8s暴露的接口==

```sh
kubectl proxy --port=8080

curl 127.0.0.1:8080/openapi/v2 > swagger.json

docker run -d --rm -p 8080:8080 -e SWAGGER_JSON=/swagger.json -v ./swagger.json:/swagger.json swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/swaggerapi/swagger-ui:v5.9.1
```

# 作业

level0：熟悉课件上的例子。（必做）

level1：用statefulset控制器部署一个mysql数据库，要求持久化存储数据。（最好使用pv、pvc持久化）

要求：密码等敏感数据使用secret，普通的配置使用configmap。

记录好所有资源文件以及完成过程，最后打包发到lekai@lanshan.email

