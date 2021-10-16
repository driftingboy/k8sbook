
## issue-001
在我执行kubectl exec时报了如下错误
```
“kubectl exec” results in "error: unable to upgrade connection: pod does not exist"
```
查看节点网络发现 Kubernetes 集群中节点的 INTERNAL-IP 相同
![20211013131753](https://i.loli.net/2021/10/13/WG8zbXiqdrvPBAV.png)

![20211014110531](https://i.loli.net/2021/10/14/oeTKizFXMfbEHwr.png)
k8s 默认使用了 enp0s3 这张 nat网卡，我们发现master和slave节点的enp0s3 addr 都一样，但 enp0s8 不同。可以把 INTERNAL-IP 替换为 enp0s8 addr。替换方式如下：

在每个节点都执行如下操作




![20211014110849](https://i.loli.net/2021/10/14/Lpl4XO82om9Gqy6.png)

解决方案参考：

https://yanbin.blog/kubernetes-cluster-internal-ip-issue/
https://stackoverflow.com/questions/51154911/kubectl-exec-results-in-error-unable-to-upgrade-connection-pod-does-not-exi

## issue-002
https://github.com/kubernetes/website/issues/6552

## issue-003
pv pvc的处理

删除terminate pvc
https://blog.csdn.net/nklinsirui/article/details/112465242
terminate pvc 恢复
https://blog.51cto.com/u_14086194/2718317