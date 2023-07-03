# 部署Seatunnel引擎


<!--more-->



>  该文档针对 `Seatunnel v2.3.0`版本. 



## 1. 下载

`Seatunnel Engine`是 Seatunnel 的默认引擎. Seatunnel 的安装包已经包含了所有`Seatunnel Engine`的内容. 



## 2. 配置`SEATUNNEL_HOME`

1. 添加文件: `/etc/profile.d/seatunnel.sh`

2. 添加内容: 

    ```shell
    export SEATUNNEL_HOME=${seatunnel install path}
    export PATH=$PATH:$SEATUNNEL_HOME/bin
    ```



## 3. 配置`Seatunnel Engine`的 JVM 参数

`Seatunnel Engine`支持两种方式设置 jvm 参数

1. 在`$SEATUNNEL_HOME/bin/seatunnel-cluster.sh`中添加.

    修改该文件: 在第一行添加内容`JAVA_OPTS="-Xms2G -Xmx2G"`

2. 在启动`Seatunnel Engine`的时候添加 jvm 参数. 🌰: `seatunnel-cluster.sh -DJvmOption="-Xms2G -Xmx2G"`

    

## 4. 配置`Seatunnel Engine`

>  `Seatunnel Engine`提供了很多需要在`seatunnel.yaml`(在`$SEATUNNEL_HOME/config/`下)中设置的参数. 



### 4.1 Backup count (备份数)

`Seatunnel Engine`实现了基于[Hazelcast IMDG](https://docs.hazelcast.com/imdg/4.1/)的集群管理. 集群的状态数据(任务运行状态, 资源状态)存放在[Hazelcast IMap](https://docs.hazelcast.com/imdg/4.1/data-structures/map)中. 这些状态数据在 Hazelcast IMap 中是分布式的并且是存储在所有集群节点中的. Hazelcast 会将数据分区存储在 Imap 中. 每个分区可以指定备份的数量. 因此, `Seatunnel Engine`可以在不使用其他服务(例如 zookeeper)的情况下实现集群.



`backup count`参数定义了同步备份的数量. 举个🌰, 如果设置为`1`, 分区的备份将会存放在另一个分区上; 设置为`2`, 将会存放在另外两个分区上. 



`backup count`建议为`min(1, max(5, N/2))`, 其中`N`为集群节点数



```shell
seatunnel:
	engine:
		backup-count: 1
		# other config
```



### 4.2 Slot service (插槽服务)

插槽的数量决定了集群节点可以并行运行的任务组的数量. `Seatunnel Engine`是一个数据同步引擎, 并且大多数的任务都是`IO`密集型的. 



建议使用动态插槽:

```shell
seatunnel:
	engine:
		slot-service:
			dynamic-slot: true
		# other config
```



### 4.3 Checkpoint Manager (检查点管理)

>  像是 FLink, `Seatunnel Engine`也支持 Chandy–Lamport 算法(分布式快照算法). `Seatunnel Engine`可以在没有数据丢失和重复的前提下实现数据同步. 

#### interval (间隔)

两个检查点之间的间隔单位是毫秒. 如果`checkpoint.interval`参数在任务的配置文件的`env块`中设置了, 这个值将会被覆盖掉. 

#### timeout (超时)

检查点的超时. 如果一个检查点没有在超时范围内完成的话, 那么这个检查点就触发失败. 因此, 任务就会被恢复. 

#### max-concurrent (最大并发)

最多可以同时执行多少个检查点. 

#### tolerable-failure (重试)

检查点失败后最大重试次数. 

```shell
seatunnel:
	engine:
		backup-count: 1
		print-execution-info-interval: 10
		slot-service:
			dynamic-slot: true
		checkpoint:
			interval: 300000
			timeout: 10000
			max-concurrent: 1
			tolerable-failure: 2
```

#### checkpoint storage (检查点存储)

**todo**



## 5. 配置`Seatunnel Engine`服务

>  所有`Seatunnel Engine`服务配置都在`hazelcast.yaml`文件中. 



### 5.1 cluster-name (集群名称)

`Seatunnel Engine`节点通过集群名来判断对方是否和自身属于同一集群. 如果两个节点之间的集群名是不同的, 那么`Seatunnel Engine`会拒绝服务请求. 



### 5.2 Network (网络)

基于[Hazelcast](https://docs.hazelcast.com/imdg/4.1/clusters/discovery-mechanisms),`Seatunnel Engine`集群是运行`Seatunnel Engine`服务的集群成员网络. 集群成员自动连接在一起组成一个集群. 这种自动加入是通过集群成员用来相互查找的各种发现机制进行的. 



**需要注意一点**, 一旦集群建立, 无论是通过什么样的发现机制, 各集群成员之间的通信总是通过 TCP/IP 完成的. 



`Seatunnel Engine`使用以下发现机制:



#### TCP

可以将`Seatunnel Engine`配置为一个完整的 TCP/IP 集群. 



🌰

```shell
hazelcast:
  cluster-name: seatunnel
  network:
    join:
      tcp-ip:
        enabled: true
        member-list:
          - hostname1
    port:
      auto-increment: false
      port: 5801
  properties:
    hazelcast.logging.type: log4j2
```



**TCP是在单独的SeaTunnel引擎集群中的建议方式.**



另一方面, Hazelcast 提供了一些其他服务发现方法. 点此查看详情: [Hazelcast Network](https://docs.hazelcast.com/imdg/4.1/clusters/setting-up-clusters)



## 6. 配置`SeaTunnel Engine`客户端

>  所有`SeaTunnel Engine`客户端配置都在`hazelcast-client.yaml`文件中. 



### 6.1 cluster-name (集群名称)

客户端必须和`Seatunnel Engine`的`cluster-name`一致. 否则, `Seatunnel Engine`将会拒绝客户端的请求. 



### 6.2 Network (网络)

#### cluster-members (集群成员)

所有`Seatunnel Engine`服务节点地址需要添加在这: 

```shell
hazelcast-client:
  cluster-name: seatunnel
  properties:
      hazelcast.logging.type: log4j2
  network:
    cluster-members:
      - hostname1:5801
```



## 7. 启动`SeaTunnel Engine`服务节点

```shell
mkdir -p $SEATUNNEL_HOME/logs
nohup seatunnel-cluster.sh &
```

日志会写在`$SEATUNNEL_HOME/logs/seatunnel-server.log`中. 



## 8. 安装`Seatunnel Engine`客户端

只需要将`Seatunnel Engine`节点上的`$Seatunnel Engine`目录复制到客户端节点, 并像`Seatunnel Engine`服务节点一样配置`$Seatunnel Engine`就可以了. 
 

---

> 作者:   
> URL: https://buli-home.cn/seatunnel_engine/  

