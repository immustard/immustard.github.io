# mac 配置 VMware 的 CentOS 网络 (NAT 模式)


<!--more-->

 > 因为每次都要去网上找教程 (😂), 所以记录一下自己的配置过程

1. VMware 默认配置就为 NAT 模式
	<center>     <img style="border-radius: 0.3125em;     box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"      src="https://cdn.jsdelivr.net/gh/immustard/gallery/pictures/202307110922359.png" width = "85%" alt="" onclick="window.open(this.src)"/>     <br>     <div style="color:orange; border-bottom: 1px solid #d9d9d9;     display: inline-block;     color: #999;     padding: 2px;">       NAT 模式   	</div> </center>
2. 关闭防火墙
	```bash
	# 关闭
	> systemctl stop firewalld
	# 禁用
	> systemctl disable firewalld
	# 查看状态
	> systemctl status firewalld
	```
3. 禁用 selinux
	```bash
	# 将 SELINUX 设置为 `disabled`
	> vi /etc/selinux/config
	```
	
	<center>     <img style="border-radius: 0.3125em;     box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"      src="https://cdn.jsdelivr.net/gh/immustard/gallery/pictures/202307110914417.png" width = "85%" alt="" onclick="window.open(this.src)"/>     <br>     <div style="color:orange; border-bottom: 1px solid #d9d9d9;     display: inline-block;     color: #999;     padding: 2px;">       禁用 selinux   	</div> </center>
	
	> 安全增强型 Linux（Security-Enhanced Linux）简称 SELinux, 它是一个 Linux 内核模块, 也是 Linux 的一个安全子系统. 
4. 获取 mac vmnet8 的 gateway 地址
	```bash
	> cat /Library/Preferences/VMware\ Fusion/vmnet8/nat.conf
	```
	找到 `# NAT gateway address` 一行, 然后记下 `ip` 和 `netmask`
5. 修改虚拟机中的网卡配置
	```bash
	> vi /etc/sysconfig/network-scripts/ifcfg-ens33
	```
	<center>     <img style="border-radius: 0.3125em;     box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"      src="https://cdn.jsdelivr.net/gh/immustard/gallery/pictures/202307110938671.png" width = "85%" alt="" onclick="window.open(this.src)"/>     <br>     <div style="color:orange; border-bottom: 1px solid #d9d9d9;     display: inline-block;     color: #999;     padding: 2px;">       ifcfg-ens33   	</div> </center>
	
	上图红框为修改内容, 绿框为新增内容
	```bash
	BOOTPROTO=static
	ONBOOT=yes
	IPADDR=172.16.143.101
	GATEWAY=172.16.143.2
	NETMASK=255.255.255.0
	DNS1=114.114.114.114
	DNS2=8.8.8.8
	
	# 其中 GATEWAY 为第 4 步中的 ip, NETMASK 为第 4 步 中的 netmask
	```
6. 重启虚拟机网卡
	```bash
	> systemctl restart network
	```
7. `ping` 一下外网就可以啦!
	```bash
	> ping www.baidu.com
	```
8. 修改主机名
	```bash
	> vi /etc/hostname
	# 将其中内容删除, 改为: `buli_server1`
	> vi /etc/hosts
	```
	<center>     <img style="border-radius: 0.3125em;     box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"      src="https://cdn.jsdelivr.net/gh/immustard/gallery/pictures/202307110956306.png" width = "85%" alt="" onclick="window.open(this.src)"/>     <br>     <div style="color:orange; border-bottom: 1px solid #d9d9d9;     display: inline-block;     color: #999;     padding: 2px;">       /etc/hosts   	</div> </center>
9. 重启
	```bash
	> reboot -f
	```


---

> 作者: <no value>  
> URL: https://buli-home.cn/mac_vmware_set_net/  

