# CDH环境搭建


&lt;!--more--&gt;

&gt; 搭建 CDH v6.2.0 (离线版)


## 简介

### 官网介绍

[CDH](https://www.cloudera.com)是Cloudera的100%开源平台发行版, 包括Apache Hadoop, 专为满足企业需求而构建. CDH 提供开箱即用的企业使用所需的一切. 通过将 Hadoop与十几个其他关键的开源项目集成, Cloudera 创建了一个功能先进的系统, 可帮助您执行端到端的大数据工作流程. 

### 简单来说

CDH 是一个拥有集群自动化安装、中心化管理、集群监控、报警功能的一个工具(软件), 使得集群的安装可以从几天的时间缩短到几个小时, 运维人数也会从几个人降低到几个人, 极大的提高了集群管理的效率. 


&gt; 提示: **所有的操作, 均在`root`用户下进行!!!**

## 环境准备

### 集群机器

|     虚拟机IP     | hostname | Hadoop | MySQL | Spark | ZooKeeper |
| :-------------: | :------: | :----: | :---: | :---: | :-------: |
| 192.168.100.101 |   cdh1   |   ✅   |  ✅   |  ✅   |    ✅     |
| 192.168.100.102 |   cdh2   |   ✅   |       |  ✅   |    ✅     |
| 192.168.100.103 |   cdh3   |   ✅   |       |  ✅   |    ✅     |

### 准备安装包

* **jdk**: 
    oracle-j2sdk1.8-1.8.0&#43;update181-1.x86_64.rpm
* **MySQL**: 
    mysql-5.7.26-1.el7.x86_64.rpm-bundle.tar
    mysql-connector-java-5.1.47.jar
* **cloudera-repos-6.2.0**:
    cloudera-manager-agent-6.2.0-968826.el7.x86_64.rpm
    cloudera-manager-daemons-6.2.0-968826.el7.x86_64.rpm
    cloudera-manager-server-6.2.0-968826.el7.x86_64.rpm
* **parcel-6.2.0**: 
    CDH-6.2.0-1.cdh6.2.0.p0.967373-el7.parcel
    CDH-6.2.0-1.cdh6.2.0.p0.967373-el7.parcel.sha
    PHOENIX-1.0.jar
    PHOENIX-5.0.0-cdh6.2.0.p0.1308267-el7.parcel
    PHOENIX-5.0.0-cdh6.2.0.p0.1308267-el7.parcel.sha
    manifest.json

&gt; 下载地址: [百度网盘](https://pan.baidu.com/s/1102pg8hTSieRQTVCCqr3Xw?pwd=cdh6)

## 环境安装

&gt; 如果没关防火墙, 关一下: `systemctl stop firewalld.service`
&gt; 因为这是我自己的虚拟机, 所以我索性让防火墙不启动了: `systemctl disable firewalld.service`

### 修改 hostname 和 hosts (所有节点)

* 设置 hostname 
    ```shell
    [root@localhost ~]# vi /etc/hostname
    cdh1
    
    [root@localhost ~]# vi /etc/sysconfig/network
    # Created by anaconda
    NETWORKING=yes
    HOSTNAME=server4
    ```

* 设置 hosts

  ```shell
  [root@localhost ~]# vi /etc/hosts
  
  127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
  ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
  
  192.168.100.101 cdh1
  192.168.100.102 cdh2
  192.168.100.103 cdh3
  ```

* 重启!

  ```shell
  [root@localhost ~]# reboot
  ```



**同样的操作在三台服务器上! 记得对应不同的 hostsname**

&gt; 这里之后我还是习惯用`vim`, 所以就安装了`vim`: `yum install -y vim`



### `SSH`免密登录 (所有节点)

* 执行指令: 

  ```shell
  [root@cdh1 ~]# ssh-keygen -t rsa -P &#34;&#34;
  Generating public/private rsa key pair.
  Enter file in which to save the key (/root/.ssh/id_rsa):
  Created directory &#39;/root/.ssh&#39;.
  Your identification has been saved in /root/.ssh/id_rsa.
  Your public key has been saved in /root/.ssh/id_rsa.pub.
  The key fingerprint is:
  SHA256:jGyqRTlDiqwQOP/Id2hVDhKNuigVipeSrkafdVXzP4s root@cdh1
  The key&#39;s randomart image is:
  &#43;---[RSA 2048]----&#43;
  |    .o           |
  |. . ...    o     |
  |=o &#43;o . . . o    |
  |*==o &#43; * .   .   |
  |&#43;*o.= = S     .  |
  |=&#43;.&#43; O .       o |
  |= &#43; X o       . o|
  |.. B .       E . |
  |. .              |
  &#43;----[SHA256]-----&#43;
  ```

* 查看生成的公钥和私钥

  ```shell
  [root@cdh1 ~]# cd .ssh/
  [root@cdh1 .ssh]# ll
  总用量 8
  -rw-------. 1 root root 1679 3月   1 08:20 id_rsa
  -rw-r--r--. 1 root root  391 3月   1 08:20 id_rsa.pub
  ```

* 生成 `authorized_keys` 文件, 并把公钥刚进去

  ```shell
  [root@cdh1 .ssh]# cp id_rsa.pub authorized_keys
  ```

* 然后将 cdh2 和 cdh3 的公钥拷贝进去

  ```shell
  [root@cdh1 .ssh]# cat authorized_keys
  ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDRqj0kcgwR68Ci2otjFu5QYQe/wBlyDpX9eI68XU6q6XzyxTrvLkbEpDpHkNwMpmqfr&#43;y/tOMpk5/dAIBUn3pOiRKBpXVDxA4xZkpIpC1PXHi3BxY8zz58QXgBQbm1tFxFplAzkCjlxsYE08Oq0X/xD3vR6T4ZRsFfMdKMo7R2LmyoPohDTf8aiiqvE6ftF&#43;Tv8YmNdb0TKbNwgD76f5vatrYhRZ&#43;XitHCC8NGffDyOA62ogkd04G9mXOKYEhvsu7eYHa98xddhiXmK0mdeBRcV4BfeEkIEBdlcYl1LpYNjKR1oePFzudfG7czeKltgdPTqy5wPvAEPic9HBw3t2St root@cdh1
  ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDEhVBXXBXUiz21kxafL7FadgIrKgNJlvds&#43;Gwf5G9fB&#43;28kXyDgvRFL5F1twCGlIakt266K2kuZioffcwWJ3Co5N6XlyF/ABHq4cxjD1js9uKbXVENDove8CCR8zCIMmojJqEZO53TUljJjau&#43;TEQQrXBaSUcQ6dkmMSEo7XuPYbCukXcZY4xX4tXX4XQyUFnX0T3xGkFkgHZ6JjPMBEo1deWpJn0igt4LEZcdfBNCywmZr4bIFDVnpER7gmZJB15AW1cEqy8&#43;&#43;2qL5a/acee85olRzLeGWFZTkP4XRH5Rqkva7U924f&#43;8uWZQefz7VTlcdruhmLJNhN&#43;DJntrL/Sr root@cdh2
  ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCsBR&#43;l8JM2NMR9JJ8XeQaFV50VFwto6ZJkkdygj6fQPbGms7XYQLTutNRaERXgQPCNgfz2Y7ZncIu0zNoK5ibmFH8qU1RGvpkscTX0AaBnrRtpESBH5R/Gn0ZCqsjSE829GC&#43;VkSxwHgKkrgS63iWVhUjNnkcoHiBueiyR6c8NHxpC0PrM9ePazYGFo/LvjlFFtHXzQYHlA5Dldk6oPm2pKOof1oEuWYhPvsIPluyExZpDeg0dSZaJSWNXJlaHgnO0lp2O2rUjY&#43;A6FVxVh1VbKZlj0R4ZP5WHKkkGZjibdz810iR8VK&#43;COsGXRwLhc&#43;BmGVf10ik/Z2k2qISHKBUZ root@cdh3
  ```

* 再将 `authorized_keys` 分发到 cdh2 和 cdh3

  ```shell
  [root@cdh1 .ssh]# scp authorized_keys root@cdh2:/root/.ssh/
  [root@cdh1 .ssh]# scp authorized_keys root@cdh3:/root/.ssh/
  ```

* 测试连接

  ```shell
  [root@cdh1 .ssh]# ssh cdh2
  Last failed login: Wed Mar  1 08:29:01 EST 2023 from cdh1 on ssh:notty
  There were 4 failed login attempts since the last successful login.
  Last login: Wed Mar  1 08:10:27 2023 from 192.168.100.1
  [root@cdh2 ~]#
  ```

到此, 免密登录就完成啦! 😁



### 配置 `NTP` 服务 (所有节点)

&gt; 使用 `NTP` 的目的是对网络内所有具有时钟的设备进行时钟同步, 使网络内所有设备的时钟保持一致, 从而使设备能够提供基于统一时间的多种应用. 


* 关闭 `chronyd` 服务 (所有节点)

  因为 CentOS 默认的使用的是 chronyd 服务进行时间同步的, 所以先关掉

  ```shell
  [root@cdh1 ~]# systemctl stop chronyd
  [root@cdh1 ~]# systemctl disable chronyd
  ```

* 修改时区 (改为中国标准时区) (所有节点)

  ```shell
  [root@cdh1 ~]# ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
  ```

* 安装 `NTP` (所有节点)

  ```shell
  [root@cdh1 ~]# yum -y install ntp
  ```

* 修改 `NTP` 服务器配置 (cdh1)

  ```shell
  [root@cdh1 ~]# vim /etc/ntp.conf
  
  ...
  
  # Use public servers from the pool.ntp.org project.
  # Please consider joining the pool (http://www.pool.ntp.org/join.html).
  # server 0.centos.pool.ntp.org iburst
  # server 1.centos.pool.ntp.org iburst
  # server 2.centos.pool.ntp.org iburst
  # server 3.centos.pool.ntp.org iburst
  
  server 0.cn.pool.ntp.org iburst
  server 1.cn.pool.ntp.org iburst
  server 2.cn.pool.ntp.org iburst
  server 3.cn.pool.ntp.org iburst
  
  ...
  ```

* 修改同步配置 (cdh1)

  ```shell
  [root@cdh1 ~]# vim /etc/sysconfig/ntpd
  
  # Command line options for ntpd
  OPTIONS=&#34;-g&#34;
  SYNC_HWCLOCK=yes
  ```

* 重启 `NTP` 服务 (cdh1)

  ```shell
  [root@cdh1 ~]# systemctl restart ntpd
  ```

* 配置其余服务器 (cdh2, cdh3)

  * 修改服务器配置

    ```shell
    [root@cdh2 ~]# vim /etc/ntp.conf
    
    ...
    
    # Use public servers from the pool.ntp.org project.
    # Please consider joining the pool (http://www.pool.ntp.org/join.html).
    #server 0.centos.pool.ntp.org iburst
    #server 1.centos.pool.ntp.org iburst
    #server 2.centos.pool.ntp.org iburst
    #server 3.centos.pool.ntp.org iburst
    
    server cdh1 iburst
    
    ...
    ```

  * 手动同步时间

    ```shell
    [root@cdh3 ~]# systemctl stop ntpd
    [root@cdh3 ~]# date
    2023年 03月 01日 星期三 22:22:46 CST
    [root@cdh3 ~]# ntpdate cdh1
     3 Mar 10:53:47 ntpdate[10943]: step time server 192.168.100.101 offset 131446.778897 sec
    [root@cdh3 ~]# systemctl start ntpd
    [root@cdh3 ~]# date
    ```

* 配置开机启动(所有节点)

  ```shell
  [root@cdh3 ~]# systemctl enable ntpd
  ```

  


### 安装 `httpd` (所有节点)

```shell
[root@cdh1 ~]# yum -y install httpd
[root@cdh1 ~]# systemctl start httpd
[root@cdh1 ~]# systemctl enable httpd
```



### 配置 `xsync` (所有节点)

* `xsync` 是对 `rsync` 的二次封装, 所以先安装 `rsync`

  ```shell
  [root@cdh1 ~]# yum install -y rsync
  ```

* 在用户主目录的`bin`目录下添加脚本:

  ```shell
  [root@cdh1 ~]# mkdir bin
  [root@cdh1 ~]# vim ./bin/xsync
  ```

  脚本内容:

  ```shell
  #!/bin/sh
  
  #1. 判断参数个数
  if [ $# -lt 1 ]
  then
          echo Not Enought Arguement!
          exit;
  fi
  
  #2. 遍历集群所有机器
  for host in cdh1 cdh2 cdh3
  do
          echo ============== $host ==============
          #3. 遍历所有目录, 挨个发送
  
          for file in $@
          do
                  #4. 判断文件是否存在
                  if [ -e $file ]
                  then
                          #5. 获取父目录
                          pdir=$(cd -P $(dirname $file); pwd)
  
                          #6. 获取当前文件的名称
                          fname=$(basename $file)
                          ssh $host &#34;mkdir -p $pdir&#34;
                          rsync -av $pdir/$fname $host:$pdir
                  else
                          echo $file does not exists!
                  fi
          done
  done
  ```

  赋权限:

  ```shell
  [root@cdh1 ~]# chmod 755 ./bin/xsync
  ```

  

* 测试一下:

  ```shell
  [root@cdh1 ~]# xsync ./bin
  
  ============== cdh1 ==============
  sending incremental file list
  
  sent 60 bytes  received 12 bytes  48.00 bytes/sec
  total size is 824  speedup is 11.44
  ============== cdh2 ==============
  sending incremental file list
  ./bin
  ./bin/xsync
  
  sent 934 bytes  received 38 bytes  1,944.00 bytes/sec
  total size is 824  speedup is 0.85
  ============== cdh3 ==============
  sending incremental file list
  ./bin/
  ./bin/xsync
  
  sent 934 bytes  received 38 bytes  1,944.00 bytes/sec
  total size is 824  speedup is 0.85
  ```

  然后到其他机器上看一下文件是不是真的分发下去了. 


### 配置 `xcall` (所有节点)

* `xcall`的作用就是所有节点都执行命令的时候, 只需要在一台服务器上执行就可以了. 

  ```shell
  #!/bin/bash
  
  # 获取控制台指令
  
  cmd=$*
  
  # 判断指令是否为空
  if [ ! -n &#34;$cmd&#34; ]
  then
    echo &#34;command can not be null !&#34;
    exit
  fi
  
  # 获取当前登录用户
  user=`whoami`
  
  # 在从机执行指令,这里需要根据你具体的集群情况配置，host与具体主机名一致
  for host in cdh1 cdh2 cdh3
  do
          echo &#34;================current host is $host=================&#34;
    echo &#34;--&gt; excute command \&#34;$cmd\&#34;&#34;
          ssh $user@$host $cmd
  done
  
  echo &#34;excute successfully !&#34;
  ```

* 测试一下:

  ```shell
  [root@cdh1 ~]# xcall ip addr
  /root/bin/xcall:行8: ((: #ip addr -eq # : 语法错误: 期待操作数 （错误符号是 &#34;#ip addr -eq # &#34;）
  ================current host is cdh1=================
  --&gt; excute command &#34;ip addr&#34;
  1: lo: &lt;LOOPBACK,UP,LOWER_UP&gt; mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
      inet 127.0.0.1/8 scope host lo
         valid_lft forever preferred_lft forever
      inet6 ::1/128 scope host
         valid_lft forever preferred_lft forever
  2: ens33: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
      link/ether 00:0c:29:7a:6b:88 brd ff:ff:ff:ff:ff:ff
      inet 192.168.100.101/24 brd 192.168.100.255 scope global noprefixroute ens33
         valid_lft forever preferred_lft forever
      inet6 fe80::20c:29ff:fe7a:6b88/64 scope link
         valid_lft forever preferred_lft forever
  ================current host is cdh2=================
  --&gt; excute command &#34;ip addr&#34;
  1: lo: &lt;LOOPBACK,UP,LOWER_UP&gt; mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
      inet 127.0.0.1/8 scope host lo
         valid_lft forever preferred_lft forever
      inet6 ::1/128 scope host
         valid_lft forever preferred_lft forever
  2: ens33: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
      link/ether 00:0c:29:ee:a3:5c brd ff:ff:ff:ff:ff:ff
      inet 192.168.100.102/24 brd 192.168.100.255 scope global noprefixroute ens33
         valid_lft forever preferred_lft forever
      inet6 fe80::20c:29ff:feee:a35c/64 scope link
         valid_lft forever preferred_lft forever
  ================current host is cdh3=================
  --&gt; excute command &#34;ip addr&#34;
  1: lo: &lt;LOOPBACK,UP,LOWER_UP&gt; mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
      inet 127.0.0.1/8 scope host lo
         valid_lft forever preferred_lft forever
      inet6 ::1/128 scope host
         valid_lft forever preferred_lft forever
  2: ens33: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
      link/ether 00:0c:29:fe:d4:de brd ff:ff:ff:ff:ff:ff
      inet 192.168.100.103/24 brd 192.168.100.255 scope global noprefixroute ens33
         valid_lft forever preferred_lft forever
      inet6 fe80::20c:29ff:fefe:d4de/64 scope link
         valid_lft forever preferred_lft forever
  excute successfully !
  ```

  


### 安装 `JDK` (所有节点)

* 因为我这里是新的 minimal 的虚拟机, 所以肯定没有安装过 `jdk`, 这里就直接安装了. 

  **有一点需要说明一下, 一定要安装 `cdh` 对应版本的 `jdk`!!!**

  ```shell
  [root@cdh1 ~]# rpm -ivh /opt/oracle-j2sdk1.8-1.8.0&#43;update181-1.x86_64.rpm
  ```

  这一步这三台一起安装, 安装包就可以用 `xsync` 同步啦. 

* 配置环境变量

  ```shell
  [root@cdh1 ~]# vim /etc/profile.d/java.sh
  
  export JAVA_HOME=/usr/java/jdk1.8.0_181-cloudera
  export PATH=$JAVA_HOME/bin:$PATH
  ```

  然后分发一下, 后面需要分发的地方就不写命令了

  ```shell
  [root@cdh1 ~]# xsync /etc/profile.d/java.sh
  ```

* 所有服务器都刷新一下环境变量

  ```shell
  [root@cdh1 ~]# source /etc/profile
  [root@cdh1 ~]# java -version
  java version &#34;1.8.0_181&#34;
  Java(TM) SE Runtime Environment (build 1.8.0_181-b13)
  Java HotSpot(TM) 64-Bit Server VM (build 25.181-b13, mixed mode)
  ```



### 安装 `MySQL` (cdh1)

* 创建目录

  ```shell
  [root@cdh1 ~]# mkdir /usr/share/java
  ```

* 上传 `mysql-connector-java.jar` 

  ```shel
  [root@cdh1 ~]# ll /usr/share/java/
  总用量 984
  -rw-r--r--. 1 root root 1007502 3月   3 11:34 mysql-connector-java.jar
  ```

* 解压安装包

  ```shell
  [root@cdh1 mysql]# cd /opt/mysql/
  [root@cdh1 mysql]# ll
  总用量 1036896
  -rw-r--r--. 1 root root  530882560 3月   3 11:52 mysql-5.7.26-1.el7.x86_64.rpm-bundle.tar
  [root@cdh1 mysql]# tar -xf mysql-5.7.26-1.el7.x86_64.rpm-bundle.tar
  [root@cdh1 mysql]# ll
  总用量 1036896
  -rw-r--r--. 1 root root  530882560 3月   3 11:52 mysql-5.7.26-1.el7.x86_64.rpm-bundle.tar
  -rw-r--r--. 1 7155 31415  25381952 4月  15 2019 mysql-community-client-5.7.26-1.el7.x86_64.rpm
  -rw-r--r--. 1 7155 31415    280904 4月  15 2019 mysql-community-common-5.7.26-1.el7.x86_64.rpm
  -rw-r--r--. 1 7155 31415   3838100 4月  15 2019 mysql-community-devel-5.7.26-1.el7.x86_64.rpm
  -rw-r--r--. 1 7155 31415  47076368 4月  15 2019 mysql-community-embedded-5.7.26-1.el7.x86_64.rpm
  -rw-r--r--. 1 7155 31415  24086952 4月  15 2019 mysql-community-embedded-compat-5.7.26-1.el7.x86_64.rpm
  -rw-r--r--. 1 7155 31415 130023844 4月  15 2019 mysql-community-embedded-devel-5.7.26-1.el7.x86_64.rpm
  -rw-r--r--. 1 7155 31415   2274268 4月  15 2019 mysql-community-libs-5.7.26-1.el7.x86_64.rpm
  -rw-r--r--. 1 7155 31415   2118444 4月  15 2019 mysql-community-libs-compat-5.7.26-1.el7.x86_64.rpm
  -rw-r--r--. 1 7155 31415 173541272 4月  15 2019 mysql-community-server-5.7.26-1.el7.x86_64.rpm
  -rw-r--r--. 1 7155 31415 122249684 4月  15 2019 mysql-community-test-5.7.26-1.el7.x86_64.rpm
  ```

* 安装

  因为我安装的时候, 遇到了 `mariadb` 的依赖冲突, 所以先解决一下这个问题: 
  
  ```shell
  [root@cdh1 mysql]# rpm -qa | grep mariadb
  mariadb-libs-5.5.56-2.el7.x86_64
  [root@cdh1 mysql]# rpm -e --nodeps mariadb-libs-5.5.56-2.el7.x86_64
  ```
  
  然后继续安装 `MySQL` 

  ```shell
  [root@cdh1 mysql]# rpm -ivh mysql-community-common-5.7.26-1.el7.x86_64.rpm
  [root@cdh1 mysql]# rpm -ivh mysql-community-libs-5.7.26-1.el7.x86_64.rpm
  [root@cdh1 mysql]# rpm -ivh mysql-community-client-5.7.26-1.el7.x86_64.rpm
  [root@cdh1 mysql]# rpm -ivh mysql-community-devel-5.7.26-1.el7.x86_64.rpm
  [root@cdh1 mysql]# rpm -ivh mysql-community-server-5.7.26-1.el7.x86_64.rpm
  [root@cdh1 mysql]# rpm -ivh mysql-community-libs-compat-5.7.26-1.el7.x86_64.rpm
  ```
  
* 启动 `MySQL`

  ```shell
  [root@cdh1 mysql]# mysqld --initialize --user=root
  [root@cdh1 mysql]# cat /var/log/mysqld.log
  2023-03-03T04:18:29.973382Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
  2023-03-03T04:18:30.126918Z 0 [Warning] InnoDB: New log files created, LSN=45790
  2023-03-03T04:18:30.149953Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
  2023-03-03T04:18:30.218530Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: 740806b3-b97a-11ed-96b3-000c297a6b88.
  2023-03-03T04:18:30.220950Z 0 [Warning] Gtid table is not ready to be used. Table &#39;mysql.gtid_executed&#39; cannot be opened.
  2023-03-03T04:18:30.221618Z 1 [Note] A temporary password is generated for root@localhost: 7uTGbVWuja,.
  [root@cdh1 mysql]# systemctl enable mysqld
  [root@cdh1 mysql]# systemctl start mysqld
  ```
  
* 更改 `MySQL` 密码, 给 `root` 赋权限, 这几步就不写了, 网上有很多.

* 重新登录 `root`, 创建数据表:

  ```mysql
  create database cmserver default charset utf8 collate utf8_general_ci;
  grant all on cmserver.* to &#39;cmserveruser&#39;@&#39;%&#39; identified by &#39;root&#39;;
  
  create database metastore default charset utf8 collate utf8_general_ci;
  grant all on metastore.* to &#39;hiveuser&#39;@&#39;%&#39; identified by &#39;root&#39;;
  
  create database amon default charset utf8 collate utf8_general_ci;
  grant all on amon.* to &#39;amonuser&#39;@&#39;%&#39; identified by &#39;root&#39;;
  
  create database rman default charset utf8 collate utf8_general_ci;
  grant all on rman.* to &#39;rmanuser&#39;@&#39;%&#39; identified by &#39;root&#39;;
  ```



### 安装 `CM` 组件 (所有节点)

* 上传安装包到各服务器

  ```shell
  [root@cdh1 opt]# ll
  总用量 1169544
  -rw-r--r--. 1 root root   10215488 3月   3 12:43 cloudera-manager-agent-6.2.0-968826.el7.x86_64.rpm
  -rw-r--r--. 1 root root 1187380436 3月   3 12:45 cloudera-manager-daemons-6.2.0-968826.el7.x86_64.rpm
  -rw-r--r--. 1 root root       9984 3月   3 12:47 cloudera-manager-server-6.2.0-968826.el7.x86_64.rpm
  
  [root@cdh2 opt]# ll
  总用量 1169532
  -rw-r--r--. 1 root root   10215488 3月   3 12:43 cloudera-manager-agent-6.2.0-968826.el7.x86_64.rpm
  -rw-r--r--. 1 root root 1187380436 3月   3 12:45 cloudera-manager-daemons-6.2.0-968826.el7.x86_64.rpm
  
  [root@cdh3 opt]# ll
  总用量 1169532
  -rw-r--r--. 1 root root   10215488 3月   3 12:43 cloudera-manager-agent-6.2.0-968826.el7.x86_64.rpm
  -rw-r--r--. 1 root root 1187380436 3月   3 12:45 cloudera-manager-daemons-6.2.0-968826.el7.x86_64.rpm
  ```

* 安装 (`cdh1` 安装 3 个, `cdh2` 和 `cdh3` 安装 2 个)

  ```shell
    yum localinstall -y cloudera-manager-daemons-6.2.0-968826.el7.x86_64.rpm 
    yum localinstall -y cloudera-manager-agent-6.2.0-968826.el7.x86_64.rpm
    
    yum localinstall -y cloudera-manager-server-6.2.0-968826.el7.x86_64.rpm
  ```



### `CDH6.2.0` (cdh1)

* 上传安装包

  ```shell
  [root@cdh1 parcel-repo]# cd /opt/cloudera/parcel-repo/
  [root@cdh1 parcel-repo]# ll
  总用量 2431572
  -rw-r--r--. 1 root root 2087665645 3月   3 13:02 CDH-6.2.0-1.cdh6.2.0.p0.967373-el7.parcel
  -rw-r--r--. 1 root root         41 3月   3 13:00 CDH-6.2.0-1.cdh6.2.0.p0.967373-el7.parcel.sha
  -rw-r--r--. 1 root root      33725 3月   3 13:00 manifest.json
  -rw-r--r--. 1 root root  402216960 3月   3 13:00 PHOENIX-5.0.0-cdh6.2.0.p0.1308267-el7.parcel
  -rw-r--r--. 1 root root         41 3月   3 12:59 PHOENIX-5.0.0-cdh6.2.0.p0.1308267-el7.parcel.sha
  [root@cdh1 parcel-repo]# cd /opt/cloudera/csd/
  [root@cdh1 csd]# ll
  总用量 8
  -rw-r--r--. 1 root root 5306 3月   3 13:03 PHOENIX-1.0.jar
  ```

* 修改配置

  * `cdh1`

    ```shell
    [root@cdh1 ~ ]#  vim /etc/cloudera-scm-server/db.properties
    
    # Auto-generated by scm_prepare_database.sh on 2020年 10月 16日 星期五 14:26:26 CST
    #
    # For information describing how to configure the Cloudera Manager Server
    # to connect to databases, see the &#34;Cloudera Manager Installation Guide.&#34;
    #
    com.cloudera.cmf.db.type=mysql
    com.cloudera.cmf.db.host=localhost
    com.cloudera.cmf.db.name=cmserver
    com.cloudera.cmf.db.user=root
    com.cloudera.cmf.db.setupType=EXTERNAL
    com.cloudera.cmf.db.password=root
    ```

  * `cdh1`, `cdh2`, `cdh3`

    ```shell
    [root@cdh1 ~]#  vi /etc/cloudera-scm-agent/config.ini
    
    ...
    
    [General]
    # Hostname of the CM server.
    server_host=cdh1
    
    ...
    
    [root@cdh1 ~]# xsync /etc/cloudera-scm-agent/config.ini
    ```




### 修改Linux `swappiness`参数 (所有节点）

```shell
[root@cdh1 ~]# vim /etc/sysctl.conf

# sysctl settings are defined through files in
# /usr/lib/sysctl.d/, /run/sysctl.d/, and /etc/sysctl.d/.
#
# Vendors settings live in /usr/lib/sysctl.d/.
# To override a whole file, create a new file with the same in
# /etc/sysctl.d/ and put new settings there. To override
# only specific settings, add a file with a lexically later
# name in /etc/sysctl.d/ and put new settings there.
#
# For more information, see sysctl.conf(5) and sysctl.d(5).

vm.swappiness=0

[root@cdh1 ~]# xsync /etc/sysctl.conf
```

所有节点执行

```
echo 0 &gt; /proc/sys/vm/swappiness 
```



### 禁用透明页 (所有节点)

```shell
[root@cdh1 ~]# vim /etc/rc.local

#!/bin/bash
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
#
# It is highly advisable to create own systemd services or udev rules
# to run scripts during boot instead of using this file.
#
# In contrast to previous versions due to parallel execution during boot
# this script will NOT be run after all other services.
#
# Please note that you must run &#39;chmod &#43;x /etc/rc.d/rc.local&#39; to ensure
# that this script will be executed during boot.

touch /var/lock/subsys/local

echo never &gt; /sys/kernel/mm/transparent_hugepage/defrag
echo never &gt; /sys/kernel/mm/transparent_hugepage/enabled

[root@cdh1 ~]# xsync /etc/rc.local
```

所有节点执行

```shell
[root@cdh1 ~]# chmod &#43;x /etc/rc.d/rc.local
[root@cdh1 ~]# source /etc/rc.local
```







### 启动 `CM` 和 `CDH` 集群

* 启动 `server` (`cdh1`)

  ```shell
  [root@cdh1 ~]# systemctl start cloudera-scm-server
  [root@cdh1 ~]# systemctl status cloudera-scm-server
  ● cloudera-scm-server.service - Cloudera CM Server Service
     Loaded: loaded (/usr/lib/systemd/system/cloudera-scm-server.service; enabled; vendor preset: disabled)
     Active: active (running) since 五 2023-03-03 13:11:11 CST; 7s ago
   Main PID: 13692 (java)
     CGroup: /system.slice/cloudera-scm-server.service
             └─13692 /usr/java/jdk1.8.0_181-cloudera/bin/java -cp .:/usr/share/java/mysql-connector-java.jar:/usr/share/java/oracle-connector-java.jar:/usr/share/java/postgresql-connector-java.jar:lib/* -server -Dlog4j.configuration=fil...
  ```

* 启动 `agent` (所有节点)

  ```shell
  [root@cdh2 opt]# systemctl start cloudera-scm-agent
  [root@cdh2 opt]# systemctl status cloudera-scm-agent
  ● cloudera-scm-agent.service - Cloudera Manager Agent Service
     Loaded: loaded (/usr/lib/systemd/system/cloudera-scm-agent.service; enabled; vendor preset: disabled)
     Active: active (running) since 五 2023-03-03 13:13:03 CST; 10s ago
   Main PID: 11790 (cmagent)
     CGroup: /system.slice/cloudera-scm-agent.service
             └─11790 /usr/bin/python2 /opt/cloudera/cm-agent/bin/cm agent
  ```



### 登录 Web 界面安装

#### 登录

打开地址: `http://cdh1:7180/cmf/login`

登录名/密码: `admin`/`admin`

&lt;center&gt;     &lt;img style=&#34;border-radius: 0.3125em;     box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);&#34;      src=&#34;https://cdn.jsdelivr.net/gh/immustard/gallery/pictures/202303031449815.png&#34; width = &#34;65%&#34; alt=&#34;&#34; onclick=&#34;window.open(this.src)&#34;/&gt;     &lt;br&gt;     &lt;div style=&#34;color:orange; border-bottom: 1px solid #d9d9d9;     display: inline-block;     color: #999;     padding: 2px;&#34;&gt;       登录界面   	&lt;/div&gt; &lt;/center&gt;



#### 勾选同意

&lt;center&gt;     &lt;img style=&#34;border-radius: 0.3125em;     box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);&#34;      src=&#34;https://cdn.jsdelivr.net/gh/immustard/gallery/pictures/202303031452623.png&#34; width = &#34;65%&#34; alt=&#34;&#34; onclick=&#34;window.open(this.src)&#34;/&gt;     &lt;br&gt;     &lt;div style=&#34;color:orange; border-bottom: 1px solid #d9d9d9;     display: inline-block;     color: #999;     padding: 2px;&#34;&gt;       勾选同意   	&lt;/div&gt; &lt;/center&gt;



#### 选择免费版

&lt;center&gt;     &lt;img style=&#34;border-radius: 0.3125em;     box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);&#34;      src=&#34;https://cdn.jsdelivr.net/gh/immustard/gallery/pictures/202303031453986.png&#34; width = &#34;65%&#34; alt=&#34;&#34; onclick=&#34;window.open(this.src)&#34;/&gt;     &lt;br&gt;     &lt;div style=&#34;color:orange; border-bottom: 1px solid #d9d9d9;     display: inline-block;     color: #999;     padding: 2px;&#34;&gt;       选择免费版   	&lt;/div&gt; &lt;/center&gt;



#### 进入安装页面啦!

&lt;center&gt;     &lt;img style=&#34;border-radius: 0.3125em;     box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);&#34;      src=&#34;https://cdn.jsdelivr.net/gh/immustard/gallery/pictures/202303031459324.png&#34; width = &#34;65%&#34; alt=&#34;&#34; onclick=&#34;window.open(this.src)&#34;/&gt;     &lt;br&gt;     &lt;div style=&#34;color:orange; border-bottom: 1px solid #d9d9d9;     display: inline-block;     color: #999;     padding: 2px;&#34;&gt;       安装页面   	&lt;/div&gt; &lt;/center&gt;



#### 输入集群名

&lt;center&gt;     &lt;img style=&#34;border-radius: 0.3125em;     box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);&#34;      src=&#34;https://cdn.jsdelivr.net/gh/immustard/gallery/pictures/202303031459325.png&#34; width = &#34;65%&#34; alt=&#34;&#34; onclick=&#34;window.open(this.src)&#34;/&gt;     &lt;br&gt;     &lt;div style=&#34;color:orange; border-bottom: 1px solid #d9d9d9;     display: inline-block;     color: #999;     padding: 2px;&#34;&gt;       输入集群名   	&lt;/div&gt; &lt;/center&gt;



#### 添加集群

&lt;center&gt;     &lt;img style=&#34;border-radius: 0.3125em;     box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);&#34;      src=&#34;https://cdn.jsdelivr.net/gh/immustard/gallery/pictures/202303031459326.png&#34; width = &#34;65%&#34; alt=&#34;&#34; onclick=&#34;window.open(this.src)&#34;/&gt;     &lt;br&gt;     &lt;div style=&#34;color:orange; border-bottom: 1px solid #d9d9d9;     display: inline-block;     color: #999;     padding: 2px;&#34;&gt;       添加集群   	&lt;/div&gt; &lt;/center&gt;



#### 选择存储库 (默认就可以)

&lt;center&gt;     &lt;img style=&#34;border-radius: 0.3125em;     box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);&#34;      src=&#34;https://cdn.jsdelivr.net/gh/immustard/gallery/pictures/202303031459327.png&#34; width = &#34;65%&#34; alt=&#34;&#34; onclick=&#34;window.open(this.src)&#34;/&gt;     &lt;br&gt;     &lt;div style=&#34;color:orange; border-bottom: 1px solid #d9d9d9;     display: inline-block;     color: #999;     padding: 2px;&#34;&gt;       选择存储库   	&lt;/div&gt; &lt;/center&gt;



#### 开始安装

&lt;center&gt;     &lt;img style=&#34;border-radius: 0.3125em;     box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);&#34;      src=&#34;https://cdn.jsdelivr.net/gh/immustard/gallery/pictures/202303031459328.png&#34; width = &#34;65%&#34; alt=&#34;&#34; onclick=&#34;window.open(this.src)&#34;/&gt;     &lt;br&gt;     &lt;div style=&#34;color:orange; border-bottom: 1px solid #d9d9d9;     display: inline-block;     color: #999;     padding: 2px;&#34;&gt;       安装   	&lt;/div&gt; &lt;/center&gt;

#### 安装完成



---

> 作者:   
> URL: https://buli-home.cn/cdh/  

