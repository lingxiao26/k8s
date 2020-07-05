# replication controller (deprecate)

```shell
# create rc
kubectl apply -f rc-example.yaml

# display rc
[root@k8s-node2 ~]# k get rc
NAME         DESIRED   CURRENT   READY   AGE
rc-example   3         3         3       39m

# modify rc
kubect edit rc rc-example


```