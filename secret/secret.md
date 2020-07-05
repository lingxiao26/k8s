# secret

可用于存储一些敏感信息，用户名密码之类的，比如docker私有仓库的认证信息


1. 创建存储docker-registry的secret

`k create secret docker-registry {secret-name} --docker-username={username} --docker-password={pass}--docker-email={email}`

```shell
# 1. 创建私有仓库
k create secret docker-registry registry-secret --docker-username=user --docker-password=pass --docker-email=445864543@qq.com
```

2. 在创建资源的时候使用该secret

在`spec`下面添加`imagePullSecrets`字段

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: registry-demo
spec:
  containers:
    - name: nginx
      image: registry.cn-shanghai.aliyuncs.com/lxli/nginx:v1.0
      ports:
        - containerPort: 80
  imagePullSecrets:
    - name: registry-secret
```
