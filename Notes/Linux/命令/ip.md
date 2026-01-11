`ip` 命令是现代 Linux 系统中最核心、最强大的网络管理工具。它属于 `iproute2` 软件包，旨在取代老旧的 `ifconfig`、`route`、`arp` 等一堆分散的命令。

你可以把它理解为网络管理的"瑞士军刀"，一个命令就能搞定 IP、网卡、路由、ARP 表等所有配置。

## 1. 基础语法结构

`ip` 命令的格式非常有逻辑，采用"对象 + 动作"的模式：

```bash
ip [选项] 对象 [动作]
```

- 对象 (Object)：你要操作什么？
    - `addr` (address)：IP 地址
    - `link`：网络接口（网卡硬件层）
    - `route`：路由表
    - `neigh` (neighbor)：ARP 缓存表

- 动作 (Action)：你要干什么？
    - `show` (查看，默认动作)
    - `add` (添加)
    - `del` (删除)
    - `set` (设置状态)

---

## 2. 常用操作速查

### 场景一：查看 IP 地址 (最常用)

```bash
ip addr show
# 或者简写
ip a
```

- 输出内容：能看到所有网卡的状态 (UP/DOWN)、MAC 地址、IPv4/IPv6 地址。

### 场景二：启用/禁用网卡 (相当于拔网线)

操作的是 `link` (链路层)。

```bash
# 启用 eth0 网卡
sudo ip link set eth0 up

# 禁用 eth0 网卡
sudo ip link set eth0 down
```

### 场景三：临时添加/删除 IP 地址

注意：用命令添加的 IP 重启后会消失。永久生效需要改配置文件（如 Netplan 或 /etc/network/interfaces）。

```bash
# 给 eth0 网卡添加一个 IP 192.168.1.100
sudo ip addr add 192.168.1.100/24 dev eth0

# 删除这个 IP
sudo ip addr del 192.168.1.100/24 dev eth0
```

### 场景四：查看和管理路由 (取代 route 命令)

```bash
# 查看路由表
ip route

# 添加默认网关 (比如要把数据包发给 192.168.1.1)
sudo ip route add default via 192.168.1.1
```

### 场景五：查看 ARP 表 (取代 arp 命令)

```bash
ip neigh
```

- 作用：查看局域网里其他机器的 IP 和 MAC 地址对应关系。

## 3. 小技巧

- 彩色输出：加上 `-c` 选项，输出会有颜色高亮，更易读。

```bash
ip -c a
```

- 只看 IPv4：加上 `-4`。

```bash
ip -4 a
```

- 简短输出：加上 `-br` (brief)，只显示关键信息，不显示一大堆废话。

```bash
ip -br a
```

- (强烈推荐这个，一眼就能看清 IP)

## 总结

| 旧命令 | 新命令 (`ip`) | 作用 |
|---|---|---|
| `ifconfig` | `ip addr` (或 `ip a`) | 查看 IP |
| `ifconfig eth0 up` | `ip link set eth0 up` | 启用网卡 |
| `route add ...` | `ip route add ...` | 加路由 |
| `arp -a` | `ip neigh` | 看 ARP 表 |

现在大多数 Linux 发行版（如 Ubuntu 20.04+）默认甚至都不安装 `ifconfig` 了，熟练掌握 `ip` 命令是必须的。

---

**<font color="#2ecc71">✅ 已格式化</font>**
