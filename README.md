## 安装nfs

服务端安装nfs

```shell
yum install nfs-utils -y
systemctl start rpcbind && systemctl enable rpcbind
systemctl start nfs-server && systemctl enable nfs-server

mkdir -p /data-ci/backend-cache/
echo “/data-ci/  *(rw,sync,no_root_squash,no_all_squash)” >>  /etc/exports
showmount -e localhost
```

客户端（运行ci runner的k8s节点）安装nfs

```shell
yum install nfs-utils -y
systemctl start rpcbind && systemctl enable rpcbind
systemctl start nfs-server && systemctl enable nfs-server
```



 ## 部署gitlab runner
文件说明
```shell
# tree .
.
├── backend-cache-pv-pvc.yaml             # runner构建缓存目录挂载nfs存储
├── configmap.yaml                        # runner配置文件,构建缓存目录为/builds/cache
├── deployment_backend_runner.yaml        # runner k8s部署文件
├── gitlab-tls-configmap.yaml             # gitlab自签名证书configmap，如果没用自签名证书可以忽略
├── rolebinding.yaml                      # rolebinding文件
├── secret.yaml                           # secret文件，gitlab token需使用base64编码
└── serviceaccount.yaml                   # serviceaccount文件

0 directories, 7 files
```


创建kube-cicd  namespace

```shell
Kubectl create ns kube-cicd
```


创建私有镜像拉取secret

```shell
kubectl create secret docker-registry registry-ci-key --docker-server=registry.example.com --docker-username=admin --docker-password=1234567 -n kube-cicd
```

运行ci runner的k8s节点打上`function=cicd`标签

```shell
Kubectl label node worker1 function=cicd
```


部署runner

`注意：按需配置yaml文件`

```shell
kubectl apply -f backend-cache-pv-pvc.yaml
kubectl apply -f gitlab-tls-configmap.yaml
Kubectl apply -f configmap.yaml
Kubectl apply -f  rolebinding.yaml
Kubectl apply -f  secret.yaml 
Kubectl apply -f  serviceaccount.yaml
```

查看部署结果
```shell
# kubectl get po -n kube-cicd
NAME                                        READY   STATUS    RESTARTS   AGE
k8s-gitlab-backend-runner-5ff54bb84-zqtxn   1/1     Running   0          13h
```







 

 

 

 