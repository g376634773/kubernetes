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
# prometheus-operator 监控mysql

参考链接 <https://blog.51cto.com/zyxjohn/2542020>

1、部署mysql并创建用户
```
CREATE USER 'exporter'@"%"  IDENTIFIED BY 'Sensetime.2018';
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'%';
flush privileges;
```
2、部署mysql-exporter
```
kubectl apply -f mysql-exporter-deployment.yaml
kubectl apply -f mysql-exporter-service.yaml
```

3、创建serviceMonitor
```
要想prometheus能够监控到mysql必须要创建serviceMonitor
kubectl apply -f mysql-serviceMonitor.yaml
且serviceMonitor的selector一定一要关联到exporter-service的label
selector:
    matchLabels:
      k8s-app: mysql-exporter  #这个一定要与exporter-service的labels一致

curl 192.168.100.128:30104/metrics | grep mysql_up
[root@master mysqld_exporter]# curl 192.168.100.128:30104/metrics | grep mysql_up
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  158k    0  158k    0     0  7258k      0 --:--:-- --:--:-- --:--:-- 7528k
# HELP mysql_up Whether the MySQL server is up.
# TYPE mysql_up gauge
mysql_up 1

可以看到mysql_up为1，表示prometheus已经监控到mysql了
```

4、登陆prometheus
```
![image](https://user-images.githubusercontent.com/51428270/114814729-5ee4ef00-9de7-11eb-964c-ca01673fed9b.png)
```
5、登陆grafana
```
添加数据源
![image](https://user-images.githubusercontent.com/51428270/114814788-820f9e80-9de7-11eb-942b-b181d649e354.png)

添加prometheus的ip和端口，并test
![image](https://user-images.githubusercontent.com/51428270/114814832-9ce21300-9de7-11eb-9a37-0aa7ee02c63f.png)

查看dashboard就可以看到mysql的数据库了
![image](https://user-images.githubusercontent.com/51428270/114814989-e4689f00-9de7-11eb-8c06-c8c5c49e27f8.png)
```
![image](https://user-images.githubusercontent.com/51428270/114815062-02ce9a80-9de8-11eb-925d-25f359a95dc1.png)

