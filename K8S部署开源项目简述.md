# 简介
本项目使用到的项目：

* github一个开源[IM项目](https://github.com/nebula-chat/chatengine)

* 自己编写的[blog网站-前端](https://github.com/JieWaZi/Blog)及[blog网站-后端](https://github.com/JieWaZi/blog-rpc)

编程语言使用的golang，通过这两个项目完成对K8S部署及相关知识的学习。
需要搭建的相关环境包括：

- [ ] **高可用的k8s**（使用kubeeasz进行部署）

- [ ] **高可用的etcd**（开源项目中使用docker启动了一个etcd，我将直接使用K8S中的etcd）

- [ ] **主从分布式mysql**（开源项目中使用docker启动了一个mysql，我将使用k8s部署一个分布式主从mysql）

- [ ] **主从分布式redis**（开源项目中使用docker启动了一个redis，我将使用k8s部署一个分布式主从redis）

- [ ] **分布式存储系统**（开源项目中直接存在本地，我将使用k8s部署一个分布式存储）

6. ...

运维及服务治理环境：

- [ ] **Gitlab及Gitlab CI**（项目使用gitlab进行代码管理，并集成gitlab CI/CD流水线）

- [ ] **Prometheus**（进行监控）

- [ ] **EFK日志系统**（日志处理）

- [ ] **实验istio**（流量管理）

5. ...


### IM项目
IM项目一共有8个微服务，原项目使用etcd及代码实现LB，这里将改为直接使用K8S提供的LB，微服务之间通过ServiceName进行内部通信，该项目暂未涉及http/https的七层协议，使用的是tcp／udp四层协议，所以如果使用到ingress暴露IP及端口应支持tcp/udp，或者直接使用NodeIp进行暴露（暂时的解决方法）。

由于项目中需要存储图片文件等，所以需要实现分布式存储，暂时使用glusterfs或ceph；项目中使用的mysql和redis也将实现分布式及主从集群部署。

### Blog项目
TODO


在运维及服务治理也会进行以上的实验，可能并不是很完善，但作为测试K8S部署相关项目的学习还是很有帮助。

下面我们首先会进行环境及简单功能的测试：
* [使用Ansible安装k8s环境（All In One）](使用Ansible安装k8s环境.md)
* [常规安装和Ansible安装Helm](安装Helm.md)
* [部署GlusterFs并在k8s中使用](部署GlusterFs并在k8s中使用.md)
* [Gitlab的CI/CD与K8S集成](Gitlab的CI/CD与K8S集成.md)

结合IM项目实际操作：
* [K8S部署开源项目之连接etcd(TLS)](K8S部署开源项目之连接etcd(TLS).md)
