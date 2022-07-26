# Kubernetes Local Path Provisioner

##**external-provisioner-localpath**

###基于github.com/kubernetes-sigs/nfs-subdir-external-provisioner项目修改
原项目在监听到pvc请求后，调用nfs接口创建pv，此处修改为直接调用系统本地os.MkdirAll创建localpath模式的pv返回。  
用于监听PVC请求，并创建localpath模式的pv资源给应用。

### 编译
```console
cd ./external-provisioner
执行make，会将cmd/external-provisioner-localpath/provisioner.go进行编译，生成 
bin/external-provisioner-localpath 的二进制文件，编译过程会分别对amd64和arm架构进行分开编译
```

### 镜像打包
```console
cd ./external-provisioner
docker build -t 192.168.100.105/storage/external-provisioner-localpath:v0.0.1 ./
```

### K8S部署
```console
cd ./external-provisioner/deploy
kubectl apply -f rbac.yaml
kubectl apply -f deployment.yaml
kubectl apply -f class.yaml
```

### 功能验证
```console
cd ./external-provisioner/deploy
kubectl apply -f sts_nettoolbox.yaml
kubectl get pv && kubectl get pvc && kubectl get pod -owide | grep nettoolbox
kubectl delete pod nettoolbox-0/1/2/3/XXX 
查看pod删除后pv永久保留且在pod重建后恢复
kubectl patch statefulset/nettoolbox -p '{"spec":{"replicas":0}}' --type=merge
查看replicas副本数至零后，pv和pvc任保留在本地，且在副本数提升后，pv能携带数据重新关联
```
