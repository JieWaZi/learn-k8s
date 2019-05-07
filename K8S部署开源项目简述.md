本项目使用的是github一个开源[IM项目](https://github.com/nebula-chat/chatengine)，编程语言使用的golang，通过该项目完成对K8S部署及相关知识的学习。
需要搭建的相关环境包括：
1. **高可用的k8s**（使用kubeeasz进行部署）
2. **高可用的etcd**（开源项目中使用docker启动了一个etcd，我将直接使用K8S中的etcd）
3. **主从分布式mysql**（开源项目中使用docker启动了一个mysql，我将使用k8s部署一个分布式主从mysql）
4. **主从分布式redis**（开源项目中使用docker启动了一个redis，我将使用k8s部署一个分布式主从redis）
5. **分布式存储系统**（开源项目中直接存在本地，我将使用k8s部署一个分布式存储）

运维及服务治理环境：
1. **Gitlab及Gitlab CI**（项目使用gitlab进行代码管理，并集成gitlab CI/CD流水线）
2. **Prometheus**（进行监控）
3. **EFK日志系统**（日志处理）
4. **实验istio**（流量管理）
5. ...

IM项目一共有8个微服务，原项目使用etcd及代码实现LB，这里将改为直接使用K8S提供的LB，微服务之间通过ServiceName进行内部通信，该项目暂未涉及http/https的七层协议，使用的是tcp／udp四层协议，所以如果使用到ingress暴露IP及端口应支持tcp/udp，或者直接使用NodeIp进行暴露（暂时的解决方法）。

由于项目中需要存储图片文件等，所以需要实现分布式存储，暂时使用glusterfs或ceph；项目中使用的mysql和redis也将实现分布式及主从集群部署。

在运维及服务治理也会进行以上的实验，可能并不是很完善，但作为测试K8S部署相关项目的学习还是很有帮助。

下面我们首先会进行环境及简单功能的测试：
1. [使用Ansible安装k8s环境（All In One）](https://app.yinxiang.com/shard/s37/nl/24154640/62bbda2a-cc0c-47e8-9d8a-f056fa7e3407/)
2. [常规安装和Ansible安装Helm](https://app.yinxiang.com/shard/s37/nl/24154640/2e5b429d-f4da-4f9f-839e-dad678884fab/)
3. [部署GlusterFs并在k8s中使用](https://app.yinxiang.com/shard/s37/nl/24154640/be3b8b31-c6a9-4e39-91bc-e349bee2fc33/)

结合项目实际操作：
1. [K8S部署开源项目之项目连接ETCD（TLS）](https://app.yinxiang.com/shard/s37/nl/24154640/5f5b6906-8c38-474c-ad9c-a3af37a89551/)
2. [K8S部署开源项目之Gitlab的CI/CD与K8S集成](https://app.yinxiang.com/shard/s37/nl/24154640/2521a94b-c14e-4baa-81fe-29716b2dc209/)