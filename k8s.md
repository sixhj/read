
# kube-proxy的实现细节

包括一下部分

- 建立服务代理对象
  
  kube-proxy 通过查询监听API Service中的Service 与Endpoints的变化，为每个Service都建立一个服务代理对象，并自动同步。这个代理对象就是程序内部的一种数据结构

- SocketServer

    监听此服务请求的

- LoadBalancer


--- 

##  Service 列表发生变化是的处理流程

- 1、如果该 Service 没有设置 Cluster IP，则不做任何处理，否则，获取该 Service 的所有端口定义列表（ spec.ports 域）。

- 2、逐个读取服务端口定义列表中的端口信息，
    根据端口名称、Service 名称和 Namespace判断本地是否已经存在对应的服务代理对象，

    如果不存在则新建：

    如果存在并且 Service 端口被修改过 ，则先删除 IP tables 中和该 Service 端口相关的规则，关闭服务代理对象，
    然后走新建流程，即为该 Service 端 口分配服务代理对象并为该 Service 创建相关的 IP tables 规则 。

```
1、KUBE-PORTALS-CONTAINER ：从容器中通过 Service Cluster IP 和端口号访问 Service 的请求。 

2、KUBE-PORTALS-HOST：从主机中通过 Service Cluster IP 和端口号访 问 Service 的请求。

3、KUBE-NODEPORT-CONTAINER：从容器中通过 Service 的 NodePort 端口号访问 Service 的请求。

4、KUBE-NODEPOR1工HOST：从主机中通过 Service 的 NodePort 端 口号访问 Service 的请求。
```
    
    

- 3、更新负载均衡器组件中对应Service的转发地址列表，对于新建的service，确定转发时的会话保持策略。 


- 4、对于己经删 除 的 Service 则进行清理。 而针对 Endpoint 的变化 ， kube-proxy 会自动更新负载均衡器 中对应 下面讲解 kube-proxy 针对 Iptables 所做的一些细节操作。


# 网络模型
每个Pod 都拥有一个独立的IP地址，不管它们是否运行在同一个Node中，都可以直接通过对对方的IP进行访问

IP 是以Pod为单位进行分配的。一个Pod内部的所有容器共享一个网络堆栈，相当于一个网络命名空间。这样的设计模型被称作IP-per-Pod模型。

一个Pod内部的应用程序看到自己的IP地址和端口与集群内其他Pod看到的一样。

Docker原生的通过动态端口映射方式会引入端口管理的复杂性，经过NAT，导致访问者看到的IP、端口与服务提供者实际绑定的不同


K8s对集群的网络要求
- 所有容器都可以在不用NAT的方式下同别的容器通信。
- 所有节点都可以在不用NAT的方式下同所有容器通信。
- 容器的地址和别人看到的地址是同一个地址