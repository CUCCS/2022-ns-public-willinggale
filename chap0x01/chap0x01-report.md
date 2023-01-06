# 基于 VirtualBox 的网络攻防基础环境搭建
## 实验目的
- 掌握 VirtualBox 虚拟机的安装与使用；
- 掌握 VirtualBox 的虚拟网络类型和按需配置；
- 掌握 VirtualBox 的虚拟硬盘多重加载；

## 实验环境
- VirtualBox 虚拟机
- 攻击者主机（Attacker）：Kali Rolling 2019.2
- 网关（Gateway, GW）：Debian Buster
- 靶机（Victim）：From Sqli to shell / xp-sp3 / Kali

## 实验要求
- 虚拟硬盘配置成多重加载；
- 搭建满足如下拓扑图所示的虚拟机网络拓扑；
![](img/网络拓扑.png)
- 完成以下网络连通性测试；
    - 靶机可以直接访问攻击者主机
    - 攻击者主机无法直接访问靶机
    - 网关可以直接访问攻击者主机和靶机
    - 靶机的所有对外上下行流量必须经过网关
    - 所有节点均可以访问互联网

## 实验过程
### 一、配置虚拟硬盘多重加载
- VirtualBox虚拟机管理 -> 虚拟介质管理
- 选中所需的虚拟盘，将属性 -> 类型 修改为多重加载
- 释放盘片后重新加载虚拟硬盘
![](img/多重加载.jpg)

### 二、配置拓扑所需网络
- **网关**需四块网卡
    - NAT网络，使网关可访问攻击者主机；
    - 仅主机（Host-Only）网络，进行网卡设置；
    - 内部网络intnet1，搭建局域网1；
    - 内部网络intnet2，搭建局域网2；
![](img/网卡2.jpg)
![](img/网络设置.jpg)
![](img/主机网络管理器.jpg)
- **攻击者**需三块网卡
    - NAT网络
    - 两块不同的Host-Only
![](img/attacker网络.jpg)
- **靶机**都需一块网卡
    - 内部网络，使不同组的victim在不同局域网内；
    - victim-xp-1 与 victim-kali-1 在同一局域网内（intnet1），victim-xp2 与 victim-debian-2 在同一局域网内（intnet2）；
![](img/victim网络1.jpg)
![](img/victim网络2.jpg)

### 三、连通性测试
| 节点 | ip地址 |
| :----:| :----: |
| Kali-Attacker | 10.0.2.15 | 
| Kali-Victim1 | 172.16.111.125 | 
| Windows XP-Victim1 | 172.16.111.140 | 
| Debian10-Victim2 | 172.16.222.118 | 
| Windows XP-Victim2 | 172.16.222.112 | 

![](img/xpip1.png)
![](img/xpip2.png)
![](img/victim-kali-ip.png)
![](img/victim-debian-ip.png)
![](img/Attacker-kali-ip.png)

- 靶机可以直接访问攻击者主机
![](img/xp1-ping-attacker.png)
![](img/xp2-ping-attacker.png)
![](img/victimkali-ping-attacker.png)
![](img/victimdebian-ping-attacker.png)

- 攻击者主机无法直接访问靶机
![](img/attacker-ping-fail.png)

- 网关可以直接访问攻击者主机和靶机
![](img/gate-ping-attacker.png)
![](img/gateway-ping-victim1.png)
![](img/gateway-ping-victim2.png)

- 靶机的所有对外上下行流量必须经过网关
安装`tcpdump`，并对对应网卡进行监控。在各个节点上访问互联网，观察捕获到了上下行的所有包。
```
apt update && apt install tcpdump
/usr/sbin/tcpdump -i enp0s9(对应网卡) # etc
```

局域网 1 内的靶机
![](img/xp1-ping-cuc.png)
![](img/gateway-tcpdump-xp1.png)
![](img/kali-ping-cuc.png)
![](img/gateway-tcpdump-kali.png)
局域网 2 内的靶机
![](img/xp2-ping-cuc.png)
![](img/gateway-tcpdump-xp2.png)
![](img/debian-ping-cuc.png)
![](img/gateway-tcpdump-debian.png)



- 所有节点均可以访问互联网
    - 网关
    ![](img/gateway-ping-baidu.png)
    - 攻击者
    ![](img/attacker-ping-baidu.png)
    - 局域网 1 内的靶机
    ![](img/xp1-ping-baidu.png)
    ![](img/kali-ping-baidu.png)
    - 局域网 2 内的靶机
    ![](img/xp2-ping-baidu.png)
    ![](img/debian-ping-baidu.png)


## 遇到的问题及解决方案
- **问题**：
启动 XP-Victim1 和 XP-Victim2 两台虚拟机时，无法发现本地网络。<br/>
**解决方案**：
将网卡选择为 PCnet-FAST III，问题解决。

- **问题**：
NAT 网络无效设置<br/>
**解决方案**：
管理 -> 全局设定 -> 网络，添加 NatNetwork，问题解决。

- **问题**：
ping 不通 XP-Victim1 和 XP-Victim2 两台虚拟机<br/>
**解决方案**：
关闭 xp 系统的防火墙，问题解决。

- **问题**：
所有虚拟机均 ping 不通外网<br/>
**解决方案**：
更改任何设置都没用，从校园网切换到宿舍网，ping 成功了。。助教求解答。


## 参考资料
http://courses.cuc.edu.cn/course/90732/learning-activity/full-screen#/378195
http://courses.cuc.edu.cn/course/90732/learning-activity/full-screen#/378192