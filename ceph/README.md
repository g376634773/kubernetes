### ROOK部署ceph

kubernetes 1.15.1

rook release-1.1

yaml文件下载 <https://github.com/rook/rook/tree/release-1.1/cluster/examples/kubernetes/ceph>

1、下载operator.yaml和common.yaml
```
kubectl create -f operator.yaml
kubectl create -f common.yaml

kubectl get pod -n rook-ceph 会看到启动了一个operator的pod

2、下载cluster.yaml
```
