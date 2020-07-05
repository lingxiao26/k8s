# Deployment

通过replicaSet管理Pod的数量，更新，回滚等。并且维护历史版本，可用于回滚


## 版本更新

```shell
# 创建deployment
kubectl apply -f deploy-example-v1.yaml

# 版本更新
kubectl apply -f deploy-example-v2.yaml

# 查看更新状态
kubectl rollout status deploy nginx-deploy

# 查看更新后的replicaSet
# kubernetes会维护replicaSet的历史版本用于回滚
root@k8s-master1 ~/resources# k get rs
NAME                      DESIRED   CURRENT   READY   AGE
nginx-deploy-558fc78868   3         3         3       8m1s
nginx-deploy-5bf87f5f59   0         0         0       19m

# 查看历史版本
root@k8s-master1 ~/resources# k rollout history deploy nginx-deploy
deployment.apps/nginx-deploy 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>

# 但通过kubectl apply -f deploy-example-v2.yaml来更新的时候，可以加上 --record=true,这样CHANGE-CAUSE就会显示当时所执行的命令
root@k8s-master1 ~/resources# kubectl apply -f deploy-example-v2.yaml --record=true


# 设置维护的历史版本数量
# 在yaml文件中修改下面的配置
spec.revisionHistoryLimit: 10

# k rollout history deploy nginx-deploy 跟 k get rs 中的历史版本是一一对应的，如果删除某一个rs，history中也会删除对应的版本
root@k8s-master1 ~/resources# k get rs
NAME                      DESIRED   CURRENT   READY   AGE
nginx-deploy-558fc78868   0         0         0       33m
nginx-deploy-5bf87f5f59   0         0         0       44m
nginx-deploy-d969f95c6    3         3         3       14m
root@k8s-master1 ~/resources# k delete rs nginx-deploy-558fc78868
replicaset.apps "nginx-deploy-558fc78868" deleted
root@k8s-master1 ~/resources# k get rs
NAME                      DESIRED   CURRENT   READY   AGE
nginx-deploy-5bf87f5f59   0         0         0       45m
nginx-deploy-d969f95c6    3         3         3       14m
root@k8s-master1 ~/resources# k rollout history deploy nginx-deploy
deployment.apps/nginx-deploy 
REVISION  CHANGE-CAUSE
1         <none>
3         <none>

root@k8s-master1 ~/resources#
```

## 版本回滚

- 回滚到上一个版本
`kubectl rollout undo deploy {deployment-name}`

```shell
root@k8s-master1 ~/resources# k get rs
NAME                      DESIRED   CURRENT   READY   AGE
nginx-deploy-5bf87f5f59   0         0         0       45m
nginx-deploy-d969f95c6    3         3         3       14m
root@k8s-master1 ~/resources# k rollout undo deploy nginx-deploy
deployment.apps/nginx-deploy rolled back
root@k8s-master1 ~/resources# k get rs
NAME                      DESIRED   CURRENT   READY   AGE
nginx-deploy-5bf87f5f59   3         3         3       51m
nginx-deploy-d969f95c6    0         0         0       20m
root@k8s-master1 ~/resources#

```

- 回滚到指定版本
`kubectl rollout undo deploy {deployment-name} --to-revision={rev-num}`

```shell
# 查看当前所在的版本号
root@k8s-master1 ~/resources#  k describe deploy nginx-deploy | grep "Annotations"
Annotations:            deployment.kubernetes.io/revision: 4

# 查看历史版本列表
root@k8s-master1 ~/resources# k rollout history deploy nginx-deploy
deployment.apps/nginx-deploy 
REVISION  CHANGE-CAUSE
3         <none>
4         <none>

# 回滚到指定版本
root@k8s-master1 ~/resources# k rollout undo deploy nginx-deploy --to-revision=3
deployment.apps/nginx-deploy rolled back
root@k8s-master1 ~/resources# 
root@k8s-master1 ~/resources# k rollout history deploy nginx-deploy
deployment.apps/nginx-deploy 
REVISION  CHANGE-CAUSE
4         <none>
5         <none>

root@k8s-master1 ~/resources# k describe deploy nginx-deploy | grep "Annotations"
Annotations:            deployment.kubernetes.io/revision: 5
root@k8s-master1 ~/resources# 
```
