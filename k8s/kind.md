# 使用kind进行本地k8s部署

[kind Kubernetes部署简单应用 上篇](https://zhuanlan.zhihu.com/p/431311301)
[Kind Kubernetes 部署简单应用 下篇 Ingress](https://zhuanlan.zhihu.com/p/433351257)

## kind下载
[快速下载](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)


## 操作过程

### 创建集群

```
kind create cluster --name tsk8s --config=k8s/kind/cluster.yaml
```

### 部署Deployment

```
kubectl apply -f k8s/kind/my-dep.yaml

kubectl get pods -o wide
```

### 部署Service

```
kubectl apply -f k8s/kind/my-svc.yaml

kubectl get svc/httpd-svc

kubectl describe svc/httpd-svc

kubectl apply -f k8s/kind/new-my-svc.yaml

kubectl get svc/httpd-svc

kubectl describe svc/httpd-svc
```

### 部署nginx-ingress

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.47.0/deploy/static/provider/kind/deploy.yaml

https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/kind/deploy.yaml

kubectl get pods --namespace ingress-nginx
```

