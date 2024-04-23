# 操作系统的启动


&lt;!--more--&gt;



首先要知道, **OS**并没有放在内存当中, 而是放在了**DISK**中的. 



当开机的时候, 首先是由**BIOS(Basic Input Output System)**检测各种各样的外设, 通过了之后才会去加载**OS**(由`BootLoader`完成). 



**BIOS**在内存中是有一个固定的地址的, 以`x86`为例, **BIOS**是存放在`CS:IP = 0xf000:fff0`这个地址中的(CS: 段寄存器; IP: 指令寄存器) . 

&lt;center&gt;
    &lt;img style=&#34;border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);&#34; 
    src=&#34;https://cdn.jsdelivr.net/gh/immustard/gallery/pictures/2022-03/202203151607693.png&#34; width = &#34;65%&#34; alt=&#34;&#34; onclick=&#34;window.open(this.src)&#34;/&gt;
    &lt;br&gt;
    &lt;div style=&#34;color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;&#34;&gt;
      BootLoader启动过程
  	&lt;/div&gt;
&lt;/center&gt;




### 操作系统与设备和程序交互

操作系统的**Interface**包含三个: 

1. **系统调用(system call)**: 来源于应用程序主动向操作系统发出服务请求
2. **异常(exception)**: 来源于不良的操作程序, 非法指令或者其他坏的处理状态(🌰 内存出错)
3. **中断(interrupt)**: 来源于外设. 来自不同的硬件设备的计时器和网络的中断



### 那为什么应用程序不能直接去访问外设呢, 而是通过操作系统? 

首先从安全的角度, **OS**是一个特殊的应用软件, 和其他的应用程序最大的不同就是, **OS**对整个计算机有控制权, 它是可信任的. 如果应用程序直接访问外设的话, 很容易造成整个系统的崩溃. 

另一个方面, 希望通过操作系统, 给上层的应用提供简单, 一致的接口, 使得应用程序不用关注底层设备的复杂性和差异性. 



---

> 作者:   
> URL: https://buli-home.cn/os_bootloader/  

