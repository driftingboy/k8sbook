# 数据管理
本章内容：k8s存储基本概念、实验
⚠️ 如果在实验的途中遇到问题，可以去 issue 目录下查找解决方案。

## volume
### 简介
pod的生命周期是短暂的，会频繁的销毁、创建，里面的内容也会被清空；
volume的生命周期独立于pod, 用于保存pod的持久化数据；它的存在形式和 docker volume 类似，可以被 mount 到 pod，这样pod中的所有容器都可以访问该 volume.
### 类型
volume 有如下几个类型：
- emptyDir

如它的名字所说，它是host上的一个空目录。
`emptyDir volume` 对于容器来说是持久的，但是对pod则不是。当pod被销毁 volume 内容也会被清除，如果只是pod 中的容器被销毁则不会。也就是说 `emptyDir volume` 和 pod 生命周期一致。

- hostPath

`hostPath volume` 是将 host  文件系统上的一个目录 mount 到 pod中，但这样 pod 就和节点产生啦耦合。
一般只有需要访问k8s内部数据的应用才会使用，比如 kube-apiserver。
如果pod销毁， `hostPath volume` 不会销毁，但是可用性仍然不足，如果 Host 宕机，那host path 也无法访问了。

- 外部 storage provider
如果K8s 部署在云上那么还可以直接使用云盘操作 volume， 这些云上的存储和k8s的集群是分离的，可靠性、可用性很高。


## pv和pvc

前面的volume提供了很好的数据持久化方案，但是对于可管理性上还有所不足。
想要使用云盘，我们需要知道：
1. 当前的 volume 来自哪（如 aws ebs）
2. volume已经创建，并且已经得知相应的 volume id

一般是由开发人员去管理维护，而 volume 是存储系统的管理员维护。这样以来开发人员和系统管理员的职责就耦合了。
因此 k8s 提供了解决方案： `pv、pvc`。
`pv` 和 volume 一样是独立于pod的一块存储空间， 由存储系统管理员配置、维护。
`pvc` 是对pv的申请，根据pvc的声明，k8s会去找到最符合要求的pv。

通过pvc，开发人员可以无需关心 volume 来自哪里，底层的访问细节，只需要告诉 k8s 自己需要的存储资源信息就好。

## 案例-数据库
### nfs搭建

master 或 单独一个fs node 安装 nfs server
其余结点都安装 nfs client

具体可以参考：
https://www.jianshu.com/p/5314f90330a6

### 创建K8s资源
创建pv、pvc、deployment、service
```bash
kubectl apply -f mysql-pv.yaml
kubectl apply -f mysql-pvc.yaml
kubectl apply -f mysql-deployment.yaml
kubectl apply -f mysql-service.yaml
```
![20211016111556](https://i.loli.net/2021/10/16/hixXBP3vz2Q7AME.png)

### 模拟节点故障，数据可靠性
1. 执行如下命令，进入mysql pod
```
kubectl exec -ti mysql-deployment-76857cdc5f-xvmr4 /bin/bash
```

2. 进入mysql，添加数据
```mysql -ppassword```

![20211015001425](https://i.loli.net/2021/10/15/gQwuHTa4CoRVLYp.png)
![20211015001456](https://i.loli.net/2021/10/15/KArotlUFDEvWjTI.png)

3. 模拟宕机
我把 node2 关机了, 模拟 node2故障

4. 查看数据可靠性
可以看到一段时间后，pod running在了 node1 上
![20211014234738](https://i.loli.net/2021/10/14/zWn6ZEfIXQYLGRx.png)
进入新的 pod，执行sql查看数据，可以发现数据仍然存在。
