# mac 配置 VMware 的 CentOS 网络 (NAT 模式)


&lt;!--more--&gt;

 &gt; 因为每次都要去网上找教程 (😂), 所以记录一下自己的配置过程

1. VMware 默认配置就为 NAT 模式
	&lt;center&gt;     &lt;img style=&#34;border-radius: 0.3125em;     box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);&#34;      src=&#34;https://cdn.jsdelivr.net/gh/immustard/gallery/pictures/202307110922359.png&#34; width = &#34;85%&#34; alt=&#34;&#34; onclick=&#34;window.open(this.src)&#34;/&gt;     &lt;br&gt;     &lt;div style=&#34;color:orange; border-bottom: 1px solid #d9d9d9;     display: inline-block;     color: #999;     padding: 2px;&#34;&gt;       NAT 模式   	&lt;/div&gt; &lt;/center&gt;
2. 关闭防火墙
	```bash
	# 关闭
	&gt; systemctl stop firewalld
	# 禁用
	&gt; systemctl disable firewalld
	# 查看状态
	&gt; systemctl status firewalld
	```
3. 禁用 selinux
	```bash
	# 将 SELINUX 设置为 `disabled`
	&gt; vi /etc/selinux/config
	```
	
	&lt;center&gt;     &lt;img style=&#34;border-radius: 0.3125em;     box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);&#34;      src=&#34;https://cdn.jsdelivr.net/gh/immustard/gallery/pictures/202307110914417.png&#34; width = &#34;85%&#34; alt=&#34;&#34; onclick=&#34;window.open(this.src)&#34;/&gt;     &lt;br&gt;     &lt;div style=&#34;color:orange; border-bottom: 1px solid #d9d9d9;     display: inline-block;     color: #999;     padding: 2px;&#34;&gt;       禁用 selinux   	&lt;/div&gt; &lt;/center&gt;
	
	&gt; 安全增强型 Linux（Security-Enhanced Linux）简称 SELinux, 它是一个 Linux 内核模块, 也是 Linux 的一个安全子系统. 
4. 获取 mac vmnet8 的 gateway 地址
	```bash
	&gt; cat /Library/Preferences/VMware\ Fusion/vmnet8/nat.conf
	```
	找到 `# NAT gateway address` 一行, 然后记下 `ip` 和 `netmask`
5. 修改虚拟机中的网卡配置
	```bash
	&gt; vi /etc/sysconfig/network-scripts/ifcfg-ens33
	```
	&lt;center&gt;     &lt;img style=&#34;border-radius: 0.3125em;     box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);&#34;      src=&#34;https://cdn.jsdelivr.net/gh/immustard/gallery/pictures/202307110938671.png&#34; width = &#34;85%&#34; alt=&#34;&#34; onclick=&#34;window.open(this.src)&#34;/&gt;     &lt;br&gt;     &lt;div style=&#34;color:orange; border-bottom: 1px solid #d9d9d9;     display: inline-block;     color: #999;     padding: 2px;&#34;&gt;       ifcfg-ens33   	&lt;/div&gt; &lt;/center&gt;
	
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
	&gt; systemctl restart network
	```
7. `ping` 一下外网就可以啦!
	```bash
	&gt; ping www.baidu.com
	```
8. 修改主机名
	```bash
	&gt; vi /etc/hostname
	# 将其中内容删除, 改为: `buli_server1`
	&gt; vi /etc/hosts
	```
	&lt;center&gt;     &lt;img style=&#34;border-radius: 0.3125em;     box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);&#34;      src=&#34;https://cdn.jsdelivr.net/gh/immustard/gallery/pictures/202307110956306.png&#34; width = &#34;85%&#34; alt=&#34;&#34; onclick=&#34;window.open(this.src)&#34;/&gt;     &lt;br&gt;     &lt;div style=&#34;color:orange; border-bottom: 1px solid #d9d9d9;     display: inline-block;     color: #999;     padding: 2px;&#34;&gt;       /etc/hosts   	&lt;/div&gt; &lt;/center&gt;
9. 重启
	```bash
	&gt; reboot -f
	```


---

> 作者:   
> URL: https://buli-home.cn/mac_vmware_set_net/  

