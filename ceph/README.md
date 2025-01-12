# ROOK部署ceph

kubernetes 1.15.1

rook release-1.1

yaml文件下载 <https://github.com/rook/rook/tree/release-1.1/cluster/examples/kubernetes/ceph>

## ceph集群最少需要三节点，否则ceph -s 查看集群状态是不正常的，部署ceph前服务器上需要有一块未格式化、未分区的盘，且大于5GB，否则osd无法初始化，osd的pod不会启动，ceph -s 查看到的osd数量会为0

1、下载operator.yaml和common.yaml
```
kubectl create -f operator.yaml
kubectl create -f common.yaml

kubectl get pod -n rook-ceph 会看到启动了一个operator的pod
```
2、下载cluster.yaml
```
kubectl create -f cluster.yaml
kubectl get pod -n rook-ceph
rook-ceph       csi-cephfsplugin-2v69k                          3/3     Running             15         42h
rook-ceph       csi-cephfsplugin-9x7l4                          3/3     Running             15         42h
rook-ceph       csi-cephfsplugin-j42v6                          3/3     Running             12         42h
rook-ceph       csi-cephfsplugin-provisioner-75b758f6b9-5ddl4   4/4     Running             20         42h
rook-ceph       csi-cephfsplugin-provisioner-75b758f6b9-6r4f2   4/4     Running             12         25h
rook-ceph       csi-rbdplugin-j4td7                             3/3     Running             15         42h
rook-ceph       csi-rbdplugin-mgbsh                             3/3     Running             12         42h
rook-ceph       csi-rbdplugin-provisioner-7bbfc4f486-4j92x      4/5     CrashLoopBackOff    8          18m
rook-ceph       csi-rbdplugin-provisioner-7bbfc4f486-hxshf      4/5     CrashLoopBackOff    8          17m
rook-ceph       csi-rbdplugin-scrvw                             3/3     Running             15         42h
rook-ceph       rook-ceph-mgr-a-74c7d4cc9f-t57x6                1/1     Running             5          42h
rook-ceph       rook-ceph-mon-a-67b8bb9bf5-hqhpn                1/1     Running             3          25h
rook-ceph       rook-ceph-mon-d-845cc75b5f-hk4mz                1/1     Running             4          42h
rook-ceph       rook-ceph-mon-f-654f94fd98-q2h8f                1/1     Running             5          42h
rook-ceph       rook-ceph-operator-5b78f5857d-tb5n9             1/1     Running             10         42h
rook-ceph       rook-ceph-osd-0-6dfbdf7f75-w4kg5                1/1     Running             1          24h
rook-ceph       rook-ceph-osd-1-b5c4f776c-sqb26                 1/1     Running             1          24h
rook-ceph       rook-ceph-osd-prepare-master-4rz4d              0/1     Completed           0          26m
rook-ceph       rook-ceph-osd-prepare-node-129-9lwst            0/1     Completed           0          26m
rook-ceph       rook-ceph-osd-prepare-node-130-rpdcb            0/1     Completed           0          26m
rook-ceph       rook-ceph-tools-7564599895-f6c8w                1/1     Running             4          41h
rook-ceph       rook-discover-bhw5q                             1/1     Running             5          42h
rook-ceph       rook-discover-jtg7n                             1/1     Running             5          42h
rook-ceph       rook-discover-p48c8                             1/1     Running             4          42h
此时可以看到pod已经全部起来了，因为我有一个虚拟机没有新增加磁盘，所以只能看到有连个osd的pod
```
3、下载toolbox.yaml 工具箱，里面会有全部的ceph命令
```  
查看集群状态
$ ceph -s
ceph status
  cluster:
    id:     dae083e6-8487-447b-b6ae-9eb321818439
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum a,b,c (age 15m)
    mgr: a(active, since 2m)
    osd: 2 osds: 2 up (since 6m), 2 in (since 6m)

  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   79 GiB used, 314 GiB / 393 GiB avail
    pgs:
    
 创建pool 用于Storageclass 动态分配pv
 ceph create pool replicapool 128 128
 ```
 # Storageclass自动分配pv
 
 1、创建storageclass.yaml
 ```
 kubectl apply -f storageclass.yaml
 ```
 2、创建nginx.yaml,其中包含pvc
 ```
 kubectl create -f nginx.yaml
 [root@master ~]# kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM                     STORAGECLASS      REASON   AGE
opspv                                      5Gi        RWX            Delete           Bound      kube-ops/opspvc                                      10d
pvc-060b63d4-3c14-4765-88ce-6a91e55da807   20Mi       RWO            Delete           Bound      default/ceph-rdb-claim    rook-ceph-block            24h
pvc-6b125d52-d547-46b9-8ec4-deed312b56f0   20Mi       RWO            Delete           Bound      default/nginx-rdb-claim   rook-ceph-block            20h
pvc-9cb8e660-eb9c-4f76-9cd4-9fba5343447f   954Mi      RWO            Delete           Released   default/mysql-pv-claim    rook-ceph-block            21h
可以看到pv已经自动被创建

[root@node-129 mount]# lsblk
NAME                                                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                                                    8:0    0   20G  0 disk 
├─sda1                                                 8:1    0  500M  0 part /boot
└─sda2                                                 8:2    0 19.5G  0 part 
  ├─centos-root                                      253:0    0 17.5G  0 lvm  /
  └─centos-swap                                      253:1    0    2G  0 lvm  [SWAP]
sdb                                                    8:16   0    6G  0 disk 
└─ceph--93b76e1c--575e--4377--b39f--d004366572b5-osd--data--a2686e01--114d--4d40--a38c--4f505f2202df
                                                     253:2    0    5G  0 lvm  
sr0                                                   11:0    1  4.2G  0 rom  
rbd0                                                 252:0    0   20M  0 disk /var/lib/kubelet/pods/a65babec-7ab9-4b65-ab11-8a423d49b2a6/volumes/kubernetes.io~csi/pvc-060b63d4-3c14-4765-88ce-6a91e55da807/mount
可以看到rbd0 已经被挂载
```
