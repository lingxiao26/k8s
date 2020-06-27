# job && daemoSet

## job 背景问题

我们可以通过Pod来直接运行任务进程吗？

如果这样做，以下问题有什么方式来解决

1. 如何保证Pod内进程正确的结束
2. 如果进程运行失败，如何重试
3. 如何管理多个任务且任务之间有互相依赖关系
4. 如何并行运行任务并管理他们的队列大小

## Job: 管理任务的控制器

Job能帮助我们做什么事情？

1. 创建一个或多个Pod确保指定数量的Pod可以成功的运行终止
2. 跟踪Pod状态，根据配置及时重试失败的Pod
3. 确定依赖关系，保证上一个任务运行完毕后再运行下一个任务
4. 控制任务并行度，并根据配置确保Pod队列大小

## Job语法

`job.yaml`

```yaml
apiVersion: batch/v1
# Job类型以及元信息
kind: Job
metadata:
  name: pi

spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      # 重启策略
      restartPolicy: Never
    # 重试次数（restartPolicy为Never的时候，不会重试）
  backOffLimit: 4
```

## DaemonSet背景问题

我们可以让每个集群内的节点都运行一个相同的Pod吗？

如果这样做，以下的问题有什么方式来解决？

1. 如何保证每个节点都运行一个Pod？
2. 如果新节点加入集群，如何感知并部署对应的Pod？
3. 如果又节点退出，如何删除对应的Pod？
4. 如果Pod状态异常，如何监控并恢复Pod的状态？


## DaemonSet: 守护进程控制器

DaemonSet 能帮助我们做什么事情？

1. 保证集群内每一个（或者一些）节点都运行一组相同的Pod
2. 跟踪集群节点状态，保证新加入的节点自动创建对应的Pod
3. 跟踪集群节点状态，保证移除的节点删除对应的Pod
4. 跟踪Pod状态，保证每个节点Pod处于运行状态


## DaemonSet语法

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: fluentd/fluentd:v1.4-1
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

## 适用场景

1. 集群存储进程： glasterd，ceph
2. 日志收集过程：fluentd，logstash
3. 需要在每个节点运行的监控收集器


## 查看DaemonSet的状态

`kubectl get ds`

状态描述：（本集群共有四个节点）
DESIRED: 需要的pod个数
CURRENT: 当前已存在的pod个数
READY: 就绪的个数
UP-TO-DATE: 最新创建的个数
AVAILABLE: 可用的pod个数
NODE SELECTOR: 节点选择标签

## 更新DaemonSet

更新策略：

1. RollingUpdate：DaemonSet默认更新策略，当更新DaemonSet模板后，老的Pod会被先删除，然后再去创建新的Pod，可以配合健康检查做滚动更新
2. OnDelete：当DaemonSet模板更新后，只有手动的删除某一个对应的Pod，此节点Pod才会被更新

```shell
kubectl set image ds/fluentd-elasticsearch fluentd-elasticsearch=fluent/fluentd:v1.4

kubectl rollout status ds/fluentd-elasticsearch
```
