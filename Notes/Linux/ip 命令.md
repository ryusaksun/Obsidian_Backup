`ip` 命令是现代 Linux 系统中**最核心、最强大**的网络管理工具。它属于 `iproute2` 软件包，旨在取代老旧的 `ifconfig`、`route`、`arp` 等一堆分散的命令。

你可以把它理解为**网络管理的“瑞士军刀”**，一个命令就能搞定 IP、网卡、路由、ARP 表等所有配置。

## 1. 基础语法结构

`ip` 命令的格式非常有逻辑，采用“**对象 + 动作**”的模式：

bash

`ip [选项] 对象 [动作]`

- **对象 (Object)**：你要操作什么？
    
    - `addr` (address)：IP 地址
        
    - `link`：网络接口（网卡硬件层）
        
    - `route`：路由表
        
    - `neigh` (neighbor)：ARP 缓存表
        
- **动作 (Action)**：你要干什么？
    
    - `show` (查看，默认动作)
        
    - `add` (添加)
        
    - `del` (删除)
        
    - `set` (设置状态)
        

---

## 2. 常用操作速查

## 场景一：查看 IP 地址 (最常用)

bash

`ip addr show # 或者简写 ip a`

- _输出内容_：能看到所有网卡的状态 (UP/DOWN)、MAC 地址、IPv4/IPv6 地址。
    

## 场景二：启用/禁用网卡 (相当于拔网线)

操作的是 `link` (链路层)。

bash

`# 启用 eth0 网卡 sudo ip link set eth0 up # 禁用 eth0 网卡 sudo ip link set eth0 down`

## 场景三：临时添加/删除 IP 地址

注意：用命令添加的 IP **重启后会消失**。永久生效需要改配置文件（如 Netplan 或 /etc/network/interfaces）。

bash

`# 给 eth0 网卡添加一个 IP 192.168.1.100 sudo ip addr add 192.168.1.100/24 dev eth0 # 删除这个 IP sudo ip addr del 192.168.1.100/24 dev eth0`

## 场景四：查看和管理路由 (取代 route 命令)

bash

`# 查看路由表 ip route # 添加默认网关 (比如要把数据包发给 192.168.1.1) sudo ip route add default via 192.168.1.1`

## 场景五：查看 ARP 表 (取代 arp 命令)

bash

`ip neigh`

- _作用_：查看局域网里其他机器的 IP 和 MAC 地址对应关系。
    

## 3. 小技巧

- **彩色输出**：加上 `-c` 选项，输出会有颜色高亮，更易读。
    
    bash
    
    `ip -c a`
    
- **只看 IPv4**：加上 `-4`。
    
    bash
    
    `ip -4 a`
    
- **简短输出**：加上 `-br` (brief)，只显示关键信息，不显示一大堆废话。
    
    bash
    
    `ip -br a`
    
    _(强烈推荐这个，一眼就能看清 IP)_
    

## 总结

|旧命令|新命令 (`ip`)|作用|
|---|---|---|
|`ifconfig`|`ip addr` (或 `ip a`)|查看 IP|
|`ifconfig eth0 up`|`ip link set eth0 up`|启用网卡|
|`route add ...`|`ip route add ...`|加路由|
|`arp -a`|`ip neigh`|看 ARP 表|

现在大多数 Linux 发行版（如 Ubuntu 20.04+）默认甚至都不安装 `ifconfig` 了，**熟练掌握 `ip` 命令是必须的**。

1. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/152616778/ada94f74-ecb0-495f-bbac-865f0c2e0d1f/image.jpg](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/152616778/ada94f74-ecb0-495f-bbac-865f0c2e0d1f/image.jpg)
2. [http://www.runoob.com/linux/linux-comm-ip.html](http://www.runoob.com/linux/linux-comm-ip.html)
3. [https://blog.csdn.net/weixin_43332935/article/details/145585290](https://blog.csdn.net/weixin_43332935/article/details/145585290)
4. [https://docs.redhat.com/zh-cn/documentation/red_hat_enterprise_linux/7/html/networking_guide/sec-configuring_ip_networking_with_ip_commands](https://docs.redhat.com/zh-cn/documentation/red_hat_enterprise_linux/7/html/networking_guide/sec-configuring_ip_networking_with_ip_commands)
5. [https://wangchujiang.com/linux-command/c/ip.html](https://wangchujiang.com/linux-command/c/ip.html)
6. [https://cloud.tencent.com/developer/article/2356234](https://cloud.tencent.com/developer/article/2356234)
7. [https://www.codeplayer.org/Wiki/Router/Linux%20ip%E5%91%BD%E4%BB%A4%E8%AF%A6%E8%A7%A3.html](https://www.codeplayer.org/Wiki/Router/Linux%20ip%E5%91%BD%E4%BB%A4%E8%AF%A6%E8%A7%A3.html)
8. [https://gist.github.com/Linuxpizi/d6c5877e76f4c047f3448906b326fc61](https://gist.github.com/Linuxpizi/d6c5877e76f4c047f3448906b326fc61)
9. [https://www.cnblogs.com/zhongguiyao/p/13942741.html](https://www.cnblogs.com/zhongguiyao/p/13942741.html)
10. [https://my.oschina.net/emacs_8759653/blog/17197069](https://my.oschina.net/emacs_8759653/blog/17197069)
11. [https://blog.csdn.net/qq_37247026/article/details/131625531](https://blog.csdn.net/qq_37247026/article/details/131625531)