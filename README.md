# Netkit CLI 使用说明

中文 | [English](./README.en.md)

> 二进制发布的网络工具箱，用统一、可读、效率高的命令行完成日常排障

**命令：** `nk`（Netkit 的简称）

## 概览

`nk` 的常用命令行用法如下。

## 快速上手

```bash
# 1) 目标能连通吗？
nk ping example.com
nk check tcp example.com 22,80,443

# UDP 检查（支持 payload / preset）
nk check example.com 53 -P udp --preset dns

# 2) DNS 是否正确？
nk dns example.com --type A,AAAA,MX

# 3) 路径卡在哪一跳？
nk mtr example.com
nk trace example.com

# 4) 本机网络与接口
nk ipinfo              # 本机网络概览
nk iface               # 网络接口信息

# 5) 路径 MTU 探测
nk mtu example.com     # 探测到主机的 MTU

# 6) 本地发现
nk mdns --resolve      # 发现并解析 mDNS 服务
```

## 输出格式

Netkit 通过全局 `--output` 标志控制两种输出格式：

- **文本模式（默认）**：带颜色、表格和可视化
- **JSON 模式**：结构化 JSON，用于脚本和自动化

```bash
# 文本输出（默认）
nk dns example.com

# JSON 输出
nk --output json dns example.com | jq .

# 所有命令都支持 JSON
nk --output json iface
nk --output json route
nk --output json neigh
```

## 命令总览

### 核心

#### 连通性
- **`nk ping`** - 增强版 ICMP ping，带可视化
- **`nk check`** - 端口连通性检测，类似 `telnet` / `nc`，支持 `--wait`

#### 名称服务
- **`nk dns`** - DNS 查询，类似 `dig`

#### 路径
- **`nk mtr`** - 网络路径质量监测，结合 `ping` + `traceroute`
- **`nk trace`** - 路由跟踪，支持 `--enrich`

#### 本机概览
- **`nk ipinfo`** - 本机网络概览，默认只看本地
- **`nk iface`** - 接口信息
- **`nk route`** - 路由表
- **`nk neigh`** - ARP / ND 邻居表

### 扩展

#### 发现
- **`nk mdns`** - mDNS / Bonjour 服务发现
- **`nk scan`** - 端口扫描，仅用于已授权目标

#### 安全
- **`nk tls`** - TLS / 证书检查
- **`nk whois`** - WHOIS 查询

#### 可观测性
- **`nk ports`** - 本地端口与进程映射
- **`nk firewall`** - 防火墙状态与规则概览，只读
- **`nk top`** - 按进程统计流量
- **`nk watch`** - 数据包捕获统计 / 实时视图

#### 设备 / 无线
- **`nk wifi`** - WiFi 扫描与检查
- **`nk bt`** - 蓝牙设备扫描与检查

#### 工具
- **`nk http`** - HTTP 客户端
- **`nk mtu`** - 路径 MTU 探测
- **`nk diag`** - 一键诊断
- **`nk speedtest`** - 速度测试
- **`nk iperf`** - 基于 iperf3 的带宽测试

## 命令参考

### HTTP 客户端

```bash
# 带自定义请求头的 GET
nk http https://api.example.com/data -H "Authorization: Bearer token" -H "Accept: application/json"

# JSON POST
nk http https://api.example.com/upload -X POST -d '{"name":"test"}' -H "Content-Type: application/json"

# 带进度下载
nk http https://example.com/file.zip -o file.zip --progress

# 查看耗时拆分
nk http https://example.com --timing
```

### DNS 查询

```bash
# 基础查询
nk dns example.com

# 查询多个记录类型
nk dns example.com --type A,AAAA,MX,TXT

# 指定 DNS 服务器
nk dns example.com --server 8.8.8.8

# JSON 输出
nk --output json dns example.com
```

### 端口检查

```bash
# 单个或多个端口
nk check example.com 80
nk check example.com 22,80,443,8080

# 等待服务启动（Docker / CI）
nk check localhost 3000 --wait --wait-timeout 30

# UDP 检查
nk check example.com 53 -P udp --preset dns
nk check example.com 123 -P udp --preset ntp

# UDP 自定义 payload
nk check example.com 9999 -P udp --payload "hello" --expect any
nk check example.com 9999 -P udp --payload-hex deadbeef --timeout 2
```

### 端口扫描

```bash
# 默认扫描前 100 个端口
nk scan example.com

# 全端口范围
nk scan example.com --all
```

### TLS / 证书检查

```bash
# 检查 TLS 握手和证书链
nk tls example.com:443

# 覆盖 SNI
nk tls 1.1.1.1:443 --sni one.one.one.one

# 提供 ALPN
nk tls example.com:443 --alpn h2,http/1.1

# 跳过验证，但仍展示详情
nk tls example.com:443 --insecure

# 隐藏证书链
nk tls example.com:443 --no-chain
```

### 本地端口与进程映射

```bash
# 显示本地端口、PID 和进程
nk ports

# 过滤条件
nk ports --tcp
nk ports --udp
nk ports --listening
nk ports --established

# 高级过滤
nk ports --state LISTEN,ESTAB --port 80,443 --process nginx --sort peer --limit 20
nk ports --pid 1234 --established
```

> 进程信息可能需要更高权限。

### 路由跟踪

```bash
# 基础 traceroute
nk trace example.com

# 附加 ISP / 地理信息
nk trace example.com --enrich
```

### MTU 探测

```bash
# 自动探测路径 MTU（二分搜索 68-9000）
nk mtu example.com

# 自定义范围
nk mtu vpn.example.com --min-size 1280 --max-size 1500
```

<details>
<summary><b>示例输出</b></summary>

```
MTU Discovery for example.com (93.184.216.34)
Testing range: 68-9000 bytes, timeout 3 seconds

Probing 4534 bytes [68-9000]... ✓ OK
Probing 6767 bytes [4535-9000]... ✗ Too big
Probing 5650 bytes [4535-6766]... ✗ Too big
...

════════════════════════════════════════════════════════════
Discovered MTU: 4535 bytes (IP+ICMP payload)
Total packet size: 4535 bytes (including IP header)
Safe payload size: 4507 bytes (TCP/UDP MSS)
Probes sent: 12 probes
════════════════════════════════════════════════════════════

Interpretation:
  ✓ Standard Ethernet MTU (1500) or higher detected.
  ℹ Jumbo frames are supported on this path.
```
</details>

### MTR - 网络路径质量

`nk mtr` 将 `ping` + `traceroute` 结合起来，用于持续监测网络路径质量，并统计每一跳的指标。

```bash
# 持续监测（Ctrl+C 停止）
nk mtr example.com

# 限定循环次数
nk mtr example.com -c 3

# 自定义参数
nk mtr example.com -m 20 -i 500 -t 5
# -m: 最大跳数（默认 30）
# -i: 循环间隔，单位毫秒（默认 1000）
# -t: 每次探测超时，单位秒（默认 3）
```

<details>
<summary><b>示例输出</b></summary>

**实时输出（每秒更新）：**
```
MTR Report for example.com (93.184.216.34)
Running for: 15.3s

┌─────┬─────────────────┬───────┬──────┬──────┬──────┬──────┬───────┬───────┬────────┐
│ Hop │ Host            │ Loss% │ Sent │ Recv │ Best │ Avg  │ Worst │ StDev │ Jitter │
├─────┼─────────────────┼───────┼──────┼──────┼──────┼──────┼───────┼───────┼────────┤
│ 1   │ 192.168.1.1     │ 0.0%  │ 15   │ 15   │ 0.8  │ 1.2  │ 2.5   │ 0.4   │ 0.3    │
│ 2   │ 10.100.50.1     │ 0.0%  │ 15   │ 15   │ 5.2  │ 6.1  │ 8.3   │ 0.8   │ 0.5    │
│ 3   │ 201.0.112.1     │ 6.7%  │ 15   │ 14   │ 12.5 │ 15.3 │ 22.1  │ 2.3   │ 1.8    │
│ 4   │ 198.51.100.1    │ 0.0%  │ 15   │ 15   │ 18.2 │ 20.4 │ 25.6  │ 1.9   │ 1.2    │
│ 5   │ 93.184.216.34   │ 0.0%  │ 15   │ 15   │ 22.1 │ 24.8 │ 30.2  │ 2.1   │ 1.5    │
└─────┴─────────────────┴───────┴──────┴──────┴──────┴──────┴───────┴───────┴────────┘

Press Ctrl+C to stop and show final summary
```

**指标：**
- **Loss%**：超时百分比
- **Best / Avg / Worst**：延迟，单位 ms
- **StDev**：稳定性指标
- **Jitter**：探测之间的平均波动
</details>

**用途：** 定位问题跳点、排查间歇性故障、长期观察路径质量。

### 网络流量监控（top）

`nk top` 用于实时查看每个进程的网络流量，类似带宽版 `top`。

```bash
# 实时监控（每秒更新）
sudo nk top

# 自定义刷新间隔 / 显示数量
sudo nk top -i 2 -n 10

# 单次快照
sudo nk top --once

# 按接口过滤
sudo nk top --iface eth0

# 按速率排序
sudo nk top --sort total|sent|received

# 总量模式（无需 sudo，仅显示接口级别流量）
nk top --total
```

**特性：**
- 显示进程级 RX/TX 速率（需要 `nethogs` + capabilities）
- 若 `nethogs` 不可用，会自动退回到总流量模式
- 实时刷新，输出带颜色

### 数据包抓取监控（watch）

`nk watch` 使用 libpcap 进行实时抓包监控，可显示聚合统计或实时流。

```bash
# 聚合统计（所有接口，5 秒刷新）
sudo nk watch

# 自定义间隔 / 限制
sudo nk watch --interval 10 --limit 30

# 单次快照
sudo nk watch --once

# 指定接口
sudo nk watch --iface eth0

# 预设过滤
sudo nk watch --preset synscan    # 检测 SYN 扫描
sudo nk watch --preset arpstorm   # ARP 风暴
sudo nk watch --preset mdns       # mDNS 流量
sudo nk watch --preset broadcast  # 广播包

# 自定义 BPF 过滤
sudo nk watch --filter "tcp port 80"
sudo nk watch --filter "host 192.168.1.100"

# 导出 pcap
sudo nk watch --iface eth0 --preset dns --pcap-out capture.pcap --duration 10
```

**特性：**
- 按源 / 目的 IP、协议、端口做 TOP-N 统计
- 彩色速率显示（绿色 < 100 PPS，黄色 < 1k，红色 > 1k）
- 常见预设过滤器
- 支持自定义 BPF

**用途：** 检测端口扫描、排查 ARP 风暴、监控 mDNS / 广播、诊断网络问题。

### mDNS / Bonjour 发现

`nk mdns` 用于发现通过 mDNS / Bonjour 广播的服务，常见于 IoT 和办公网络。

```bash
# 发现所有服务
nk mdns

# 指定服务类型
nk mdns -s _http._tcp

# 解析地址 / 端口
nk mdns --resolve

# 列出所有服务类型
nk mdns --list-types

# 自定义超时（默认 5 秒）
nk mdns -t 10
```

**常见服务类型：** `_http._tcp`、`_ssh._tcp`、`_printer._tcp`、`_airplay._tcp`、`_homekit._tcp`、`_mqtt._tcp`

### WiFi 扫描

`nk wifi` 可查看当前连接信息，并扫描附近网络。

```bash
# 当前连接
nk wifi

# 扫描（按信号强度排序）
nk wifi --scan

# 按 SSID / MAC / 信号过滤
nk wifi --scan --name "MyNetwork"
nk wifi --scan --mac "00:11:22"
nk wifi --scan --min-rssi -70

# 限制结果数量
nk wifi --scan --limit 10

# 指定接口
nk wifi --scan --iface wlan0
```

### 蓝牙扫描

`nk bt` 用于扫描蓝牙设备，适用于 IoT / 智能家居环境。

```bash
# 扫描设备（10 秒，按信号排序）
nk bt --scan

# 自定义超时
nk bt --scan --timeout 15

# 列出已配对 / 已知设备
nk bt --list

# 设备信息（仅对已配对设备有效）
nk bt --info AA:BB:CC:DD:EE:FF

# 按名称 / MAC / 信号过滤
nk bt --scan --name "sensor"
nk bt --list --mac "AA:BB"
nk bt --scan --min-rssi -70 --limit 10
```

**提示：**
- 设备必须处于可发现模式，才会出现在扫描结果中
- `--info` 仅对**已配对 / 已知设备**有效（先执行 `bluetoothctl pair <MAC>`）
- RSSI 范围：0 最强，-100 最弱，常见范围是 -30 到 -90 dBm

### WHOIS 查询

```bash
# 域名 / IP WHOIS
nk whois example.com
nk whois 8.8.8.8

# 原始输出
nk whois example.com --raw
```

### 速度测试

```bash
# 运行 speedtest（需要 speedtest-cli）
nk speedtest

# 传递 speedtest-cli 参数
nk speedtest --simple
```

### 带宽测试（iperf3）

```bash
# 客户端模式：连接 iperf3 服务端
nk iperf -c server.example.com

# 指定测试时长
nk iperf -c server.example.com -t 30

# UDP 测试并限制带宽
nk iperf -c server.example.com -u -b 100M

# 反向模式（服务器发送，客户端接收）
nk iperf -c server.example.com -R

# 多并发流
nk iperf -c server.example.com -P 4

# 自定义端口
nk iperf -c server.example.com -p 5201

# 服务端模式（在服务端运行）
nk iperf -s
```

`nk iperf` 会自动追加 `-J` 以获取 JSON 输出，并解析成结构化结果。所有标准 iperf3 参数都支持。

### 网络诊断

```bash
# 一键健康检查
nk diag

# 自定义目标
nk diag --target 1.1.1.1

# 跳过部分测试
nk diag --no-public --no-http
```
