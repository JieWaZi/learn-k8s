#### Glusterfs 安装
##### 1. 每台机器上都配置hosts
```
192.168.1.20 master
192.168.1.21 node1
192.168.1.22 node2
```


##### 2. 安装启动 
在每个节点都安装glusterfs
```
$ yum install centos-release-gluster

$ yum install -y glusterfs glusterfs-server glusterfs-fuse glusterfs-rdma
```
启动 glusterFS 
```
$ systemctl start glusterd.service

$ systemctl enable glusterd.service
```

##### 3. 在master节点上配置，将其它节点加入到集群中

```
$ gluster peer probe master

$ gluster peer probe node1

$ gluster peer probe node2
```
##### 4. 查看集群状态
```
$ gluster peer status

Number of Peers: 2

Hostname: node1
Uuid: 71dc6a51-2790-4e5d-8c34-6fde94b25938
State: Peer in Cluster (Connected)

Hostname: node2
Uuid: a69798b1-f6d3-48ac-b612-ac2c8b245f4b
State: Peer in Cluster (Connected)
```

##### 5.创建数据存储目录：
每个节点都需要创建
```
$ mkdir -p /opt/gluster/data
```

##### 6.创建GlusterFS磁盘：
```
gluster volume create gfs master:/opt/gluster/data node1:/opt/gluster/data node2:/opt/gluster/data force
```

##### 7.查看 volume 状态
```
$ gluster volume info

Volume Name: gfs
Type: Distribute
Volume ID: 75639af5-59b8-477b-aeac-b4d34cb28f32
Status: Started
Snapshot Count: 0
Number of Bricks: 3
Transport-type: tcp
Bricks:
Brick1: master:/opt/gluster/data
Brick2: node1:/opt/gluster/data
Brick3: node2:/opt/gluster/data
Options Reconfigured:
performance.write-behind: on
performance.io-thread-count: 32
performance.flush-behind: on
performance.cache-size: 4GB
features.quota-deem-statfs: on
features.inode-quota: on
features.quota: on
transport.address-family: inet
nfs.disable: on
```

##### 8.开启gfs
```
$ gluster volume start gfs

volume start: gfs: success
```

##### 9. gluster性能调优
   
* 开启 指定 volume 的配额： (models 为 volume 名称)
`$ gluster volume quota gfs enable`

* 限制 gfs 中 / (既总目录) 最大使用 80GB 空间
`$ gluster volume quota gfs limit-usage / 80GB`

* 设置 cache 4GB
` $ gluster volume set gfs performance.cache-size 4GB`

* 开启 异步 ， 后台操作
` $ gluster volume set gfs performance.flush-behind on`

* 设置 io 线程 32
` $ gluster volume set gfs performance.io-thread-count 32`

* 设置 回写 (写数据时间，先写入缓存内，再写入硬盘)
` $ gluster volume set gfs performance.write-behind on`

##### 10. 其他的维护命令
* 查看GlusterFS中所有的volume:
` $ gluster volume list`
     
* 停止名字为 gfs 的磁盘
`$ gluster volume stop gfs `
    
* 删除名字为 gfs 的磁盘
`$ gluster volume delete gfs `
*注：删除磁盘以后，必须删除磁盘(/opt/gluster/data)中的(.glusterfs/ .trashcan/)目录。否则创建新volume相同的磁盘会出现文件不分布，或者类型错乱的问题。*

* 卸载某个节点GlusterFS磁盘
`$ gluster peer detach node1`

* 设置访问限制,按照每个volume 来限制
`$ gluster volume set gfs auth.allow 10.6.0.*,10.7.0.*

* 添加GlusterFS节点：
`$ gluster peer probe node3`
`$ gluster volume add-brick gfs node3:/opt/gluster/data`
*注：如果是复制卷或者条带卷，则每次添加的Brick数必须是replica或者stripe的整数倍*
* 配置卷
`$ gluster volume set`
* 缩容volume:
先将数据迁移到其它可用的Brick，迁移结束后才将该Brick移除：
`$ gluster volume remove-brick gfs node2:/opt/gluster/data node3:/opt/gluster/data start`
在执行了start之后，可以使用status命令查看移除进度：
`$ gluster volume remove-brick gfs node2:/opt/gluster/data node3:/opt/gluster/data status`
不进行数据迁移，直接删除该Brick：
`$ gluster volume remove-brick gfs node2:/opt/gluster/data node3:/opt/gluster/data commit`
*注意，如果是复制或者条带卷，则每次移除的Brick数必须是replica或者stripe的整数倍。
* 扩容：
`$ gluster volume add-brick gfs node2:/opt/gluster/data` 
* 修复命令:
`$ gluster volume replace-brick gfs node2:/opt/gluster/data node3:/opt/gluster/data commit -force`
* 迁移volume:
`$ gluster volume replace-brick gfs node2:/opt/gluster/data node3:/opt/gluster/data start`
pause 为暂停迁移
`$ gluster volume replace-brick gfs node2:/opt/gluster/data node3:/opt/gluster/data pause`
abort 为终止迁移
`$ gluster volume replace-brick gfs node2:/opt/gluster/data node3:/opt/gluster/data abort`
status 查看迁移状态
`$ gluster volume replace-brick gfs node2:/opt/gluster/data node3:/opt/gluster/data status`
迁移结束后使用commit 来生效
`$ gluster volume replace-brick gfs node2:/opt/gluster/data node3:/opt/gluster/data commit`
* 均衡volume: 
    ` $ gluster volume gfs lay-outstart`
    `$ gluster volume gfs start`
    `$ gluster volume gfs startforce`
    `$ gluster volume gfs status`
    `$ gluster volume gfs stop`

#### Kubernetes配置使用GlusterFS存储卷
我直接将endpoints、services、pv以及pvc写在一个文件里，亦可分开。
```
apiVersion: v1
kind: Endpoints
metadata:
  name: glusterfs
subsets:
- addresses:
  - ip: 192.168.1.20
  - ip: 192.168.1.21
  - ip: 192.168.1.22
  ports:
  - port: 1000
    protocol: TCP
---

apiVersion: v1
kind: Service
metadata:
  name: glusterfs
spec:
  ports:
  - port: 1000

---

apiVersion: v1
kind: PersistentVolume
metadata:
  name: glusterfs-pv
  labels:
    name: glusterfs-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteMany
  glusterfs:
    endpoints: "glusterfs"
    path: "gfs"
    readOnly: false

---

apiVersion: v1
kind: PersistentVolumeClaim
metadata: 
  name: glusterfs-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 2Gi
  selector:
    matchLabels:
      name: "glusterfs-pv"
```

使用nginx进行测试:

```
apiVersion: v1
kind: Service
metadata:
  name: svc-nginx
  labels:
    name: nginx
spec:
  type: NodePort
  ports:
  - name: nginx
    port: 80
    nodePort: 31180
  selector:
    name: nginx
---
kind: Deployment 
apiVersion: extensions/v1beta1 
metadata: 
  name: nginx
spec: 
  replicas: 2
  template: 
    metadata: 
      labels: 
        name: nginx 
    spec: 
      containers: 
        - name: nginx 
          image: nginx
          ports: 
            - containerPort: 80
          volumeMounts:
            - name: glusterfs-vol-nginx
              mountPath: "/usr/share/nginx/html"
      volumes:
      - name: glusterfs-vol-nginx
        persistentVolumeClaim:
          claimName: glusterfs-pvc
 ```
 
*我在／opt/gluster/data中添加了一个index.html文件*

```
<html>
  <title>demo</title>
  <body>Hello world</body>
</html>
```

访问IP：NodePort，如192.168.1.21:31180，看到hello world即为成功。