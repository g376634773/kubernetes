# kube-operator部署prometheus

1、github官网上克隆，
```
gitclone https://github.com/prometheus-operator/kube-prometheus.git
```
2、选择与自己k8s集群版本适合的operator版本，我得集群是1.15.1的，使用的是release-0.3
```
查看分支
[root@master manifests]# git branch -a
  main
* release-0.3
  remotes/origin/HEAD -> origin/main
  remotes/origin/main
  remotes/origin/release-0.1
  remotes/origin/release-0.2
  remotes/origin/release-0.3
  remotes/origin/release-0.4
  remotes/origin/release-0.5
  remotes/origin/release-0.6
  remotes/origin/release-0.7

切换版本branch
git checkout remotes/origin/release-0.3
```
3、进入目录并启动
```
cd ./kube-prometheus/manifests
kubectl apply -f .
[root@master manifests]# kubectl get pod -n monitoring 
NAME                                  READY   STATUS             RESTARTS   AGE
alertmanager-main-0                   1/2     CrashLoopBackOff   111        3d
alertmanager-main-1                   1/2     CrashLoopBackOff   111        3d
alertmanager-main-2                   1/2     CrashLoopBackOff   113        3d
app-6dd7f876c4-6wskb                  1/1     Running            16         26d
grafana-77978cbbdc-twwc9              1/1     Running            3          3d
kube-state-metrics-7f6d7b46b4-xrfsr   3/3     Running            9          3d
mysql-exporter-854587464d-9k6xm       1/1     Running            0          37m
node-exporter-5c6sb                   0/2     Pending            0          3d
node-exporter-q7q2l                   0/2     Pending            0          3d
node-exporter-qhbnk                   2/2     Running            6          3d
prometheus-adapter-68698bc948-wtsvh   1/1     Running            3          3d
prometheus-k8s-0                      3/3     Running            19         3d
prometheus-k8s-1                      3/3     Running            19         3d
prometheus-operator-6685db5c6-lkmx4   1/1     Running            21         41d
```
4、登陆
![image](https://user-images.githubusercontent.com/51428270/114812889-820d9f80-9de3-11eb-9f58-c009c2e1e350.png)
