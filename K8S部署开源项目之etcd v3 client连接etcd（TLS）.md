大概介绍一下[IM项目](https://github.com/nebula-chat/chatengine)在服务发现这块的实现：每个服务都有一个配置文件，配置文件里面主要包含该服务的serverId(主要用在snowflake)、该服务的addr、LB的类型以及etcd的endpoints等。

首先服务启动读取配置文件，获取etcd的地址，然后以服务名：ipaddr的键值对的形式保存，每个服务通过从etcd读取相关地址进行通信。

所以在不对代码进行大规模的改动情况下如何使用K8S的服务发现及负载均衡呢？我暂时的想法就是，首先项目直接使用K8S的etcd，然后在保存etcd数据时，直接保存ServiceName，也就是每个服务通过ServiceName的方式进行服务间通信，这样K8S就能帮我们实现LB。

由于在搭建K8S时直接使用kubeeasz，所以访问etcd需要相关的证书，不能像IM项目直接启动一个docker然后直接ip：2379去访问，应该加上相关的证书。那么具体说一下如何（基于etcd证书直接通过kubeeasz生成，不是的可参考[ETCD TLS 配置的坑](https://www.cnblogs.com/Tempted/p/7737361.html)）。

首先,在etcd-csr.json中配置etcd节点ip。
```
{
  "CN": "etcd",
  "hosts": [
    "192.168.1.20",
    "127.0.0.1"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "HangZhou",
      "L": "XS",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
```
然后执行
```
$ cfssl gencert -ca=/etc/kubernetes/ssl/ca.pem -ca-key=/etc/kubernetes/ssl/ca-key.pem -config=/etc/kubernetes/ssl/ca-config.json -profile=kubernetes etcd-csr.json | cfssljson -bare etcd

2019/04/19 05:09:44 [INFO] generate received request
2019/04/19 05:09:44 [INFO] received CSR
2019/04/19 05:09:44 [INFO] generating key: rsa-2048
2019/04/19 05:09:44 [INFO] encoded CSR
2019/04/19 05:09:44 [INFO] signed certificate with serial number 614245103568378723148471287249547401020442773577
2019/04/19 05:09:44 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").
```
这样会生产etcd.csr，etcd-key.pem，etcd.pem三个文件。然后将这三个文件保存到你的项目中即可。

进行测试代码：
```
package main

import (
	"context"
	"fmt"
	"go.etcd.io/etcd/clientv3"
	"go.etcd.io/etcd/pkg/transport"
	"time"
)

func main() {
	tlsInfo := transport.TLSInfo{

	}
	config, err := tlsInfo.ClientConfig()
	cli, err := clientv3.New(clientv3.Config{
		Endpoints:   []string{"192.168.1.20:2379"},
		DialTimeout: 5 * time.Second,
		TLS:         config,
	})
	if err != nil {
		fmt.Println("connect failed, err:", err)
		return
	}

	defer cli.Close()
	//设置1秒超时，访问etcd有超时控制
	ctx, cancel := context.WithTimeout(context.Background(), time.Second)
	//操作etcd
	_, err = cli.Put(ctx, "hello", "world")
	//操作完毕，取消etcd
	cancel()
	if err != nil {
		fmt.Println("put failed, err:", err)
		return
	}
	//取值，设置超时为1秒
	ctx, cancel = context.WithTimeout(context.Background(), time.Second)
	resp, err := cli.Get(ctx, "hello")
	cancel()
	if err != nil {
		fmt.Println("get failed, err:", err)
		return
	}
	for _, ev := range resp.Kvs {
		fmt.Printf("%s : %s\n", ev.Key, ev.Value)
	}
}
```
终端显示相关结果即为成功，如果显示`put failed, err: context deadline exceeded`说明存放证书位置或在证书不对。

IM项目中需要添加etcd证书的几个文件分别是etcd_util.go,rpc_client.go,zrpc_client.go
