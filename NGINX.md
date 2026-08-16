# Nginx 学习笔记

> 本文档由浅入深、环环相扣地梳理 Nginx 的核心知识体系，每个知识点均配有精简代码示例与注释，便于快速上手与查漏补缺。

## 目录

- [58. Nginx 简介](#58-nginx-简介)
- [59. Nginx 概述](#59-nginx-概述)
- [60. Nginx 四大应用场景](#60-nginx-四大应用场景)
- [61. 为什么要使用 Nginx](#61-为什么要使用-nginx)
- [62. 环境准备](#62-环境准备)
- [63. Nginx 下载与安装](#63-nginx-下载与安装)
- [64. Nginx 目录详解](#64-nginx-目录详解)
- [65. Docker 安装 Nginx 服务](#65-docker-安装-nginx-服务)
- [66. Nginx 启停服务](#66-nginx-启停服务)
- [67. Nginx 配置详解_全局块](#67-nginx-配置详解_全局块)
- [68. Nginx 配置指令详解_events 块](#68-nginx-配置指令详解_events-块)
- [69. Nginx 配置指令详解_HTTP 块](#69-nginx-配置指令详解_http-块)
- [70. Nginx 配置指令详解_location 指令](#70-nginx-配置指令详解_location-指令)
- [71. Nginx 虚拟主机分类](#71-nginx-虚拟主机分类)
- [72. Nginx 虚拟主机_基于单网卡多 IP 虚拟主机配置](#72-nginx-虚拟主机_基于单网卡多ip虚拟主机配置)
- [73. Nginx 虚拟主机_基于域名虚拟主机配置](#73-nginx-虚拟主机_基于域名虚拟主机配置)
- [74. Nginx 虚拟主机_基于多端口虚拟主机配置](#74-nginx-虚拟主机_基于多端口虚拟主机配置)
- [75. Nginx 核心指令_root 和 alias 指令区别](#75-nginx-核心指令_root-和-alias-指令区别)
- [76. Nginx 核心指令_return 指令](#76-nginx-核心指令_return-指令)
- [77. Nginx 核心指令_rewrite 指令](#77-nginx-核心指令_rewrite-指令)
- [78. Nginx 核心指令_rewrite 实战域名跳转](#78-nginx-核心指令_rewrite-实战域名跳转)
- [79. Nginx 核心指令_if 指令](#79-nginx-核心指令_if-指令)
- [80. Nginx 核心指令_set 和 break 指令](#80-nginx-核心指令_set-和-break-指令)
- [81. Nginx 核心指令_Gzip 压缩指令](#81-nginx-核心指令_gzip-压缩指令)
- [82. Nginx 场景实践_浏览器缓存](#82-nginx-场景实践_浏览器缓存)
- [83. Nginx 场景实践_防盗链技术](#83-nginx-场景实践_防盗链技术)
- [84. Nginx 场景实践_代理服务](#84-nginx-场景实践_代理服务)
- [85. Nginx 场景实践_反向代理](#85-nginx-场景实践_反向代理)
- [86. Nginx 场景实践_负载均衡](#86-nginx-场景实践_负载均衡)
- [87. Nginx 场景实践_负载均衡算法](#87-nginx-场景实践_负载均衡算法)
- [88. Nginx 场景实践_第三方 fair 模块安装](#88-nginx-场景实践_第三方fair模块安装)
- [89. Nginx 场景实践_Nginx 配置故障转移](#89-nginx-场景实践_nginx-配置故障转移)
- [90. Nginx 场景实践_跨域问题](#90-nginx-场景实践_跨域问题)
- [91. Nginx 解决跨域问题](#91-nginx-解决跨域问题)
- [92. Nginx 场景实践_动静分离](#92-nginx-场景实践_动静分离)
- [93. Nginx 场景实践_动静分离实战](#93-nginx-场景实践_动静分离实战)
- [94. Nginx 场景实践_什么是限流](#94-nginx-场景实践_什么是限流)
- [95. Nginx 场景实践_限流算法](#95-nginx-场景实践_限流算法)
- [96. Nginx 场景实践_限流实现](#96-nginx-场景实践_限流实现)
- [97. Nginx 场景实践_WEB 缓存机制](#97-nginx-场景实践_web-缓存机制)
- [98. Nginx 场景实践_Nginx 高可用](#98-nginx-场景实践_nginx-高可用)
- [99. Nginx 场景实践_LVS 负载均衡](#99-nginx-场景实践_lvs-负载均衡)
- [100. Nginx 场景实践_Keepalived 健康监测](#100-nginx-场景实践_keepalived-健康监测)
- [101. Nginx 场景实践_企业双机热备方案](#101-nginx-场景实践_企业双机热备方案)

---

## 58. Nginx 简介

Nginx（发音 "engine x"）是一款由俄罗斯工程师 Igor Sysoev 于 2004 年开源的高性能 HTTP 服务器与反向代理服务器。它以事件驱动（Event-Driven）、异步非阻塞的架构设计著称，能够以极低的内存消耗支撑数万级并发连接。

**核心定位：**

- HTTP/Web 服务器（静态资源服务）
- 反向代理服务器
- 邮件（IMAP/POP3）代理服务器
- 通用的 TCP/UDP 代理服务器（1.9.0+）

**与传统 Apache 的差异：**

| 维度 | Apache（MPM prefork） | Nginx |
|------|----------------------|-------|
| 并发模型 | 进程/线程，每连接一线程 | 事件驱动，单 worker 多连接 |
| 内存占用 | 高（连接数线性增长） | 低（连接数近乎无关） |
| 抗压能力 | C10K 瓶颈明显 | 轻松突破 C100K |
| 静态资源 | 性能优秀 | 性能更优 |

---

## 59. Nginx 概述

### 59.1 核心特性

1. **高并发**：单机可达数万至数十万并发连接
2. **低内存**：10K 空闲连接约消耗 2.5MB 内存
3. **热部署**：支持不停机升级二进制与配置
4. **模块化**：官方模块 + 第三方模块灵活扩展
5. **高可靠**：Master/Worker 架构，Worker 异常由 Master 拉起

### 59.2 Master/Worker 架构

```
        ┌──────────┐
        │  Master   │  负责管理 Worker、读取配置、平滑升级
        └─────┬─────┘
   ┌──────────┼──────────┐
┌──▼──┐    ┌──▼──┐    ┌──▼──┐
│ W1  │    │ W2  │    │ Wn  │  实际处理请求，相互独立
└─────┘    └─────┘    └─────┘
```

### 59.3 事件驱动模型

Nginx Worker 利用操作系统的 I/O 多路复用机制（Linux 上为 epoll、FreeBSD 上为 kqueue），单 Worker 即可同时管理数以万计的连接：

```c
// 伪代码示意：Worker 主循环
while (1) {
    // 阻塞等待所有连接上的事件（读/写/超时）
    events = epoll_wait(epfd, event_list, max_events, timeout);
    for (e in events) {
        // 非阻塞处理：连接 1 处理时，连接 2 不会被卡住
        process_event(e);  
    }
}
```

---

## 60. Nginx 四大应用场景

### 60.1 HTTP 服务器（静态资源）

直接对外提供 HTML、图片、JS/CSS 等静态文件服务，性能远超 Apache 与 Tomcat。

```nginx
server {
    listen 80;
    server_name static.example.com;
    root /data/www;          # 静态资源根目录
    index index.html;
}
```

### 60.2 反向代理

隐藏后端真实服务，客户端只与 Nginx 通信，由 Nginx 转发请求到上游服务器。

```nginx
location /api/ {
    proxy_pass http://127.0.0.1:8080/;   # 转发到后端 Tomcat
    proxy_set_header Host $host;          # 透传客户端 Host
}
```

### 60.3 负载均衡

通过 `upstream` 将请求按策略分发到多台后端服务器，提升吞吐与可用性。

```nginx
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
}
server {
    location / {
        proxy_pass http://backend;        # 命名 upstream
    }
}
```

### 60.4 动静分离

静态资源由 Nginx 直接响应，动态请求转发到后端应用服务器，各取所长。

```nginx
server {
    location ~ \.(jpg|css|js)$ {
        root /data/static;                # 静态资源本地直出
    }
    location ~ \.jsp$ {
        proxy_pass http://tomcat_cluster; # 动态请求转发
    }
}
```

---

## 61. 为什么要使用 Nginx

### 61.1 性能维度

- **高并发**：事件驱动模型，单机轻松支撑数万并发
- **低消耗**：10K 空闲连接仅约 2.5MB 内存
- **高吞吐**：静态文件吞吐量可达 Apache 的 2~3 倍

### 61.2 功能维度

- 丰富的 HTTP 处理能力：rewrite、gzip、缓存、防盗链、限流
- 强大的代理能力：HTTP/HTTPS/TCP/UDP 全覆盖
- 灵活的扩展机制：Lua 模块（OpenResty）可实现复杂业务逻辑

### 61.3 可靠性维度

- Master/Worker 架构保证单 Worker 崩溃不影响整体
- 支持热部署，升级零停机
- 配合 Keepalived 可实现 99.99% 可用性

### 61.4 生态维度

- 社区活跃，文档齐全
- 几乎所有云厂商默认支持
- 与 Kubernetes、Docker 等云原生组件深度集成

---

## 62. 环境准备

### 62.1 系统要求

| 项目 | 最低 | 推荐 |
|------|------|------|
| OS | CentOS 7 / Ubuntu 18.04 | CentOS 8 / Ubuntu 22.04 |
| CPU | 1 核 | 2 核及以上 |
| 内存 | 512MB | 2GB 及以上 |
| 磁盘 | 1GB | 10GB 及以上 |

### 62.2 依赖安装

```bash
# CentOS / RHEL
yum install -y gcc gcc-c++ make pcre pcre-devel zlib zlib-devel openssl openssl-devel

# Ubuntu / Debian
apt update && apt install -y build-essential libpcre3 libpcre3-dev zlib1g-dev libssl-dev
```

### 62.3 防火墙放行

```bash
# CentOS firewalld
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --reload

# Ubuntu ufw
ufw allow 80/tcp
```

### 62.4 内核参数优化

```bash
# /etc/sysctl.conf 末尾追加
net.ipv4.tcp_max_syn_backlog = 65535      # SYN 队列长度
net.core.somaxconn = 65535                # 全连接队列长度
net.ipv4.tcp_tw_reuse = 1                 # TIME_WAIT 端口复用
fs.file_max = 1000000                     # 系统最大文件句柄
# 生效
sysctl -p
```

---

## 63. Nginx 下载与安装

### 63.1 源码下载

```bash
# 官方地址：http://nginx.org/en/download.html
cd /usr/local/src
wget http://nginx.org/download/nginx-1.24.0.tar.gz
tar -zxvf nginx-1.24.0.tar.gz
cd nginx-1.24.0
```

### 63.2 编译参数

```bash
./configure \
  --prefix=/usr/local/nginx \              # 安装根目录
  --with-http_ssl_module \                 # HTTPS 支持
  --with-http_stub_status_module \         # 状态统计模块
  --with-http_gzip_static_module \         # 预压缩文件支持
  --with-stream \                          # TCP/UDP 代理
  --with-pcre                              # 启用正则支持
```

### 63.3 编译安装

```bash
make && make install
# 验证
/usr/local/nginx/sbin/nginx -v
# 输出示例：nginx version: nginx/1.24.0
```

### 63.4 注册为 systemd 服务

```bash
cat > /usr/lib/systemd/system/nginx.service <<EOF
[Unit]
Description=nginx HTTP Server
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s stop
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable nginx
```

---

## 64. Nginx 目录详解

```bash
$ tree /usr/local/nginx -L 2
├── conf/                 # 配置文件目录
│   ├── nginx.conf        # 主配置文件
│   ├── mime.types        # MIME 类型映射
│   └── fastcgi.conf      # FastCGI 参数
├── html/                 # 默认站点目录
│   ├── index.html        # 默认首页
│   └── 50x.html          # 错误页面
├── logs/                 # 日志目录
│   ├── access.log        # 访问日志
│   └── error.log         # 错误日志
└── sbin/
    └── nginx             # 主程序（唯一可执行文件）
```

| 路径 | 说明 |
|------|------|
| `conf/nginx.conf` | 全局主配置，include 其他配置 |
| `html/` | 默认 web 根目录，可自定义 |
| `logs/access.log` | 记录所有访问请求 |
| `logs/error.log` | 记录错误与警告 |
| `logs/nginx.pid` | Master 进程 PID |
| `sbin/nginx` | 启动/重载/停止的唯一入口 |

---

## 65. Docker 安装 Nginx 服务

### 65.1 拉取镜像

```bash
docker pull nginx:1.24
```

### 65.2 快速启动

```bash
docker run -d \
  --name my-nginx \
  -p 80:80 \
  -v /data/nginx/html:/usr/share/nginx/html \        # 挂载静态资源
  -v /data/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \  # 挂载主配置
  -v /data/nginx/logs:/var/log/nginx \               # 挂载日志
  nginx:1.24
```

### 65.3 docker-compose 编排

```yaml
version: '3.8'
services:
  nginx:
    image: nginx:1.24
    container_name: my-nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./html:/usr/share/nginx/html:ro
      - ./conf/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./logs:/var/log/nginx
      - ./cert:/etc/nginx/cert:ro       # HTTPS 证书
    restart: always                     # 异常自动重启
    network_mode: bridge
```

### 65.4 自定义镜像

```dockerfile
FROM nginx:1.24
COPY nginx.conf /etc/nginx/nginx.conf
COPY html/ /usr/share/nginx/html/
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]      # 前台运行，容器不退出
```

---

## 66. Nginx 启停服务

### 66.1 启动

```bash
/usr/local/nginx/sbin/nginx
# 或 systemd
systemctl start nginx
```

### 66.2 停止

```bash
# 优雅停止：处理完当前请求后退出
/usr/local/nginx/sbin/nginx -s quit
# 强制停止：立即断开所有连接
/usr/local/nginx/sbin/nginx -s stop
```

### 66.3 平滑重载配置

修改配置后无需停机，Master 会重新读取配置并逐步替换 Worker：

```bash
/usr/local/nginx/sbin/nginx -s reload
```

### 66.4 信号机制

| 信号 | 含义 |
|------|------|
| `TERM`/`INT` | 快速停止 |
| `QUIT` | 优雅停止 |
| `HUP` | 平滑重载配置 |
| `USR1` | 重新打开日志（日志切割） |
| `USR2` | 平滑升级可执行文件 |
| `WINCH` | 优雅关闭旧的 Worker |

```bash
# 通过 pid 文件发送信号
kill -HUP $(cat /usr/local/nginx/logs/nginx.pid)
# 日志切割示例
mv access.log access.log.$(date +%Y%m%d)
kill -USR1 $(cat /usr/local/nginx/logs/nginx.pid)
```

### 66.5 验证与测试

```bash
nginx -t              # 测试配置语法
nginx -T              # 打印完整生效配置
nginx -V              # 查看编译参数（大写）
```

---

## 67. Nginx 配置详解_全局块

`nginx.conf` 由三大块组成：**全局块**、**events 块**、**http 块**。全局块从文件开头到 events 块之前，配置影响 Nginx 整体行为。

```nginx
# ===== 全局块 =====
user  nginx;                      # 运行用户（推荐非 root）
worker_processes  auto;           # Worker 数量，auto=CPU 核数
worker_rlimit_nofile 65535;       # Worker 最大文件句柄
error_log  logs/error.log  warn;  # 错误日志路径与级别
pid        logs/nginx.pid;        # PID 文件位置

events {
    worker_connections  10240;    # 单 Worker 最大连接数
}

http {
    # ... HTTP 相关配置
}
```

### 67.1 关键指令说明

| 指令 | 作用 | 推荐值 |
|------|------|--------|
| `user` | Worker 进程运行身份 | `nginx` 或 `www` |
| `worker_processes` | Worker 数量 | `auto` 或 CPU 核数 |
| `worker_rlimit_nofile` | 单 Worker 文件句柄上限 | 65535 |
| `error_log` | 全局错误日志 | `warn` 级别 |
| `pid` | Master PID 文件 | 默认即可 |

### 67.2 Worker 数量选择原则

- **CPU 密集型**：等于 CPU 核数，避免上下文切换损耗
- **I/O 密集型**：可设为 CPU 核数 × 1.5~2
- **通用场景**：`auto` 即可，Nginx 会自动匹配

---

## 68. Nginx 配置指令详解_events 块

events 块配置 Nginx 与用户的网络连接相关参数，影响所有 Worker 的工作方式。

```nginx
events {
    use epoll;                    # 事件模型：Linux 推荐 epoll
    worker_connections  10240;    # 单 Worker 最大并发连接数
    multi_accept on;              # 一次接受多个连接
    accept_mutex on;              # 惊群控制：Worker 串行接受新连接
    accept_mutex_delay 500ms;     # 获取互斥锁失败后的等待时间
}
```

### 68.1 事件模型选择

| 模型 | 平台 | 适用 |
|------|------|------|
| `epoll` | Linux 2.6+ | 推荐，性能最佳 |
| `kqueue` | FreeBSD/macOS | macOS 推荐 |
| `select`/`poll` | 通用 | 性能差，仅兼容用 |

### 68.2 worker_connections 计算

```
最大客户端数 = worker_processes × worker_connections
若作为反向代理，实际客户端数为上述值的 1/2（每客户端占用 2 个连接）
```

```nginx
# 4 核 CPU，目标 2 万并发
worker_processes 4;
events {
    worker_connections 10240;     # 4 × 10240 = 40960 连接
}
```

### 68.3 惊群问题

多个 Worker 同时阻塞在 `accept` 上，新连接到来时所有 Worker 被唤醒，但只有一个能成功处理，其余被浪费。`accept_mutex on` 让 Worker 串行竞争锁，避免惊群。

---

## 69. Nginx 配置指令详解_HTTP 块

http 块是 Nginx 配置的核心，可包含多个 server 块。它定义了 HTTP 服务器的全局行为。

```nginx
http {
    include       mime.types;                  # MIME 类型映射文件
    default_type  application/octet-stream;    # 默认 MIME 类型

    # 日志格式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  logs/access.log  main;         # 访问日志

    sendfile        on;                        # 零拷贝，提升静态资源性能
    tcp_nopush      on;                        # 数据包合并发送
    tcp_nodelay     on;                        # 禁用 Nagle，小包立即发送
    keepalive_timeout  65;                     # 长连接超时秒数
    keepalive_requests 1000;                   # 单连接最大请求数

    # gzip 压缩（详见 81 节）
    gzip  on;

    # 客户端限制
    client_max_body_size  20m;                 # 请求体最大尺寸
    client_body_timeout   60s;                 # 请求体读取超时

    server {
        listen 80;
        server_name localhost;
        location / {
            root   html;
            index  index.html;
        }
    }
}
```

### 69.1 sendfile 零拷贝

传统方式：磁盘 → 内核 → 用户态 → 内核 → 网卡（4 次拷贝）
sendfile：磁盘 → 内核 → 网卡（2 次拷贝）

```nginx
sendfile on;       # 静态资源推荐开启
```

### 69.2 tcp_nopush vs tcp_nodelay

| 指令 | 行为 | 场景 |
|------|------|------|
| `tcp_nopush on` | 等数据包攒够再发 | 大文件下载、sendfile 场景 |
| `tcp_nodelay on` | 立即发送小包 | 交互式应用、长连接 |

两者看似矛盾，但 Nginx 可以智能切换：开启 sendfile 时用 nopush，结束大文件后切回 nodelay。

### 69.3 keepalive 长连接

```nginx
keepalive_timeout 65;        # 65 秒内无新请求则关闭连接
keepalive_requests 1000;     # 单连接最多处理 1000 次请求
```

长连接避免了反复 TCP 握手，显著提升 QPS，但需注意连接占用资源。

---

## 70. Nginx 配置指令详解_location 指令

location 用于匹配 URL，决定请求如何处理，是 Nginx 最核心的指令之一。

### 70.1 匹配语法

```nginx
location [ = | ~ | ~* | ^~ ] uri { ... }
```

| 修饰符 | 匹配方式 | 优先级 |
|--------|----------|--------|
| `=` | 精确匹配 | 1（最高） |
| `^~` | 前缀匹配，不再进入正则 | 2 |
| `~` | 区分大小写的正则 | 3 |
| `~*` | 不区分大小写的正则 | 3 |
| 无 | 普通前缀匹配 | 4（最低） |

### 70.2 匹配示例

```nginx
server {
    location = / {                    # 精确匹配 / 
        # 只匹配根路径，优先级最高
    }
    location ^~ /static/ {            # 前缀匹配 /static/
        # 匹配后不再检查正则
        root /data/web;
    }
    location ~* \.(gif|jpg|png)$ {    # 正则匹配图片
        # 不区分大小写
        expires 1h;
    }
    location /api/ {                  # 普通前缀匹配
        proxy_pass http://backend;
    }
}
```

### 70.3 匹配流程

1. 检查所有 `=` 精确匹配，命中则立即使用
2. 检查 `^~` 与普通前缀匹配，记录最长前缀
3. 若最长前缀是 `^~`，直接使用，跳过正则
4. 按配置文件顺序检查正则 `~` 和 `~*`，命中即用
5. 若正则均未命中，使用第 2 步记录的最长普通前缀

### 70.4 命名 location

```nginx
location /foo {
    error_page 404 = @fallback;       # 错误时跳转到命名 location
}
location @fallback {                  # 内部命名 location，外部不可访问
    proxy_pass http://backup;
}
```

---

## 71. Nginx 虚拟主机分类

虚拟主机（Virtual Host）允许一台 Nginx 同时服务多个网站，按维度分为三类：

| 类型 | 区分依据 | 适用场景 |
|------|----------|----------|
| 基于域名 | Host 请求头 | 最常用，生产标配 |
| 基于端口 | 监听端口 | 测试环境、内网服务 |
| 基于 IP | 服务器多 IP | 历史遗留、特殊网络 |

```nginx
# 一个 http 块内放多个 server 即构成虚拟主机
http {
    server {                          # 站点 A
        listen 80;
        server_name www.a.com;
        root /data/a;
    }
    server {                          # 站点 B
        listen 80;
        server_name www.b.com;
        root /data/b;
    }
}
```

---

## 72. Nginx 虚拟主机_基于单网卡多IP虚拟主机配置

通过给单网卡绑定多个 IP 地址，实现基于 IP 的虚拟主机。

### 72.1 单网卡绑定多 IP

```bash
# 给 eth0 额外绑定两个 IP
ip addr add 192.168.1.101/24 dev eth0
ip addr add 192.168.1.102/24 dev eth0
# 验证
ip addr show eth0
```

### 72.2 Nginx 配置

```nginx
server {
    listen 192.168.1.101:80;          # 绑定 IP1
    server_name site1.example.com;
    root /data/site1;
}
server {
    listen 192.168.1.102:80;          # 绑定 IP2
    server_name site2.example.com;
    root /data/site2;
}
```

### 72.3 适用场景

- 历史遗留系统，无法用域名区分
- 内网服务，需通过 IP 隔离
- HTTPS 多证书场景（SNI 出现前）

> 实际生产中 IPv4 资源稀缺，此方案已较少使用，推荐基于域名。

---

## 73. Nginx 虚拟主机_基于域名虚拟主机配置

生产环境最常用的方案，通过 `Host` 请求头区分不同站点。

```nginx
http {
    server {
        listen 80;
        server_name www.shop.com;     # 主站
        root /data/www/shop;
        index index.html;
    }
    server {
        listen 80;
        server_name m.shop.com;       # 移动站
        root /data/www/mobile;
        index index.html;
    }
    server {
        listen 80;
        server_name *.shop.com;       # 通配符，匹配所有 shop.com 子域
        root /data/www/default;
    }
    server {
        listen 80 default_server;    # 默认服务器，匹配不到任何域名时使用
        server_name _;
        return 444;                   # 直接断开连接
    }
}
```

### 73.1 server_name 匹配优先级

1. 精确匹配：`www.shop.com`
2. 前缀通配符：`*.shop.com`
3. 后缀通配符：`www.*`
4. 正则匹配：`~^www\d+\.shop\.com$`

### 73.2 本地测试

```bash
# 修改 hosts 文件，将域名指向本机
echo "127.0.0.1 www.shop.com m.shop.com" >> /etc/hosts
curl -H "Host: m.shop.com" http://127.0.0.1/
```

---

## 74. Nginx 虚拟主机_基于多端口虚拟主机配置

通过监听不同端口区分站点，适合测试环境或内网服务。

```nginx
server {
    listen 8080;                      # 站点 1：8080 端口
    server_name localhost;
    root /data/site1;
    index index.html;
}
server {
    listen 8081;                      # 站点 2：8081 端口
    server_name localhost;
    root /data/site2;
    index index.html;
}
server {
    listen 8082;                      # 站点 3：8082 端口
    server_name localhost;
    root /data/site3;
    index index.html;
}
```

### 74.1 访问方式

```bash
curl http://localhost:8080/      # 访问站点 1
curl http://localhost:8081/      # 访问站点 2
curl http://localhost:8082/      # 访问站点 3
```

### 74.2 适用场景

- 开发/测试环境，无独立域名
- 内部管理后台，通过端口隔离不同服务
- SSL 证书共用端口（80/443）时，其他服务用高端口

> 缺点：客户端访问需带端口号，不适合对外提供服务。

---

## 75. Nginx 核心指令_root 和 alias 指令区别

`root` 与 `alias` 都用于指定资源路径，但行为差异明显，是面试与实战高频考点。

### 75.1 root 指令

```nginx
location /static/ {
    root /data/web;                  # 将 location 路径拼接在 root 之后
}
# 请求 /static/a.jpg → 实际路径 /data/web/static/a.jpg
```

**特点**：root 值 + 完整 URI 路径 = 实际文件路径

### 75.2 alias 指令

```nginx
location /static/ {
    alias /data/web/;                # 用 alias 值替换 location 路径
}
# 请求 /static/a.jpg → 实际路径 /data/web/a.jpg
```

**特点**：alias 值替换 location 匹配部分 = 实际文件路径

### 75.3 对比表

| 维度 | root | alias |
|------|------|-------|
| 路径计算 | 拼接 location 路径 | 替换 location 路径 |
| 使用位置 | http/server/location/if | 仅 location |
| 结尾斜杠 | 可有可无 | **必须**以 `/` 结尾 |
| 性能 | 略优（少一次替换） | 略差 |

### 75.4 典型陷阱

```nginx
# 错误：alias 未以 / 结尾，导致路径错误
location /img {
    alias /data/images;              # 请求 /img/a.png → /data/imagesa.png ❌
}

# 正确写法
location /img/ {
    alias /data/images/;             # 请求 /img/a.png → /data/images/a.png ✓
}
```

---

## 76. Nginx 核心指令_return 指令

`return` 用于直接返回状态码或重定向，**终止后续处理**，是最高效的响应方式。

### 76.1 语法

```nginx
return code [text];           # 返回状态码与文本
return code URL;              # 返回状态码与跳转 URL（3xx）
return URL;                   # 直接 302 重定向到 URL
```

### 76.2 常用示例

```nginx
# 1. 返回简单状态码
location /maintenance {
    return 503;                       # 返回 503 服务不可用
}

# 2. 返回 JSON
location /api/status {
    default_type application/json;
    return 200 '{"status":"ok","code":0}';   # 直接返回 JSON 字符串
}

# 3. 301 永久重定向（SEO 友好）
location /old {
    return 301 https://www.new.com$request_uri;
}

# 4. 302 临时重定向
location /promo {
    return 302 /new-promo.html;
}

# 5. 拒绝非本域名访问
server {
    listen 80 default_server;
    server_name _;
    return 444;                       # 444 = Nginx 特有，直接断开连接
}
```

### 76.3 return 与 rewrite 区别

| 维度 | return | rewrite |
|------|--------|---------|
| 执行位置 | server/location/if | server/location/if |
| 作用 | 直接返回或重定向 | 改写 URI 后继续匹配 |
| 性能 | 更高（直接终止） | 略低（需重新匹配 location） |
| 状态码 | 任意 | 主要用于 301/302 |

---

## 77. Nginx 核心指令_rewrite 指令

`rewrite` 通过正则表达式改写 URL，是实现 URL 美化、域名跳转的核心指令。

### 77.1 语法

```nginx
rewrite regex replacement [flag];
```

| flag | 含义 |
|------|------|
| `last` | 改写后重新匹配 location（内部跳转） |
| `break` | 改写后不再匹配后续 rewrite，继续当前 location |
| `redirect` | 返回 302 临时重定向 |
| `permanent` | 返回 301 永久重定向 |

### 77.2 flag 区别

```nginx
# last：重新匹配 location
location /a/ {
    rewrite ^/a/(.*)$ /b/$1 last;     # 改写为 /b/xxx，重新匹配 location
}
location /b/ {
    # last 触发后这里会被匹配到
}

# break：不再重新匹配
location /c/ {
    rewrite ^/c/(.*)$ /d/$1 break;    # 改写后停留在当前 location
    # 后续 rewrite 不再执行
}
```

### 77.3 典型示例

```nginx
# 1. URL 美化：/article/123 → /article.php?id=123
location /article/ {
    rewrite ^/article/(\d+)$ /article.php?id=$1 last;
}

# 2. http 强制跳转 https
server {
    listen 80;
    server_name www.example.com;
    rewrite ^(.*)$ https://$host$1 permanent;    # 301 永久跳转
}

# 3. 隐藏 .php 后缀
location / {
    rewrite ^/([a-z]+)/?$ /$1.php last;
}
```

### 77.4 捕获变量

```nginx
# 正则捕获可用 $1, $2 ... 引用
rewrite ^/user/(\w+)/(\d+)$ /profile?type=$1&id=$2 last;
# 请求 /user/admin/1001 → /profile?type=admin&id=1001
```

---

## 78. Nginx 核心指令_rewrite 实战域名跳转

实际业务中常见的域名跳转场景：旧域名迁移、http→https、子域名统一等。

### 78.1 旧域名 301 永久跳转新域名

```nginx
server {
    listen 80;
    server_name old-domain.com www.old-domain.com;
    # 所有请求永久跳转到新域名，保留 URI
    rewrite ^(.*)$ http://www.new-domain.com$1 permanent;
}
```

### 78.2 不带 www 跳转到带 www

```nginx
server {
    listen 80;
    server_name example.com;
    rewrite ^(.*)$ http://www.example.com$1 permanent;
}
server {
    listen 80;
    server_name www.example.com;
    root /data/www;
    # ... 正常业务配置
}
```

### 78.3 http 强制跳转 https（推荐 301）

```nginx
server {
    listen 80;
    server_name www.example.com;
    return 301 https://$host$request_uri;    # 比 rewrite 更简洁高效
}
server {
    listen 443 ssl;
    server_name www.example.com;
    ssl_certificate     /etc/nginx/cert/server.crt;
    ssl_certificate_key /etc/nginx/cert/server.key;
    # ... HTTPS 业务配置
}
```

### 78.4 多级域名统一跳转

```nginx
server {
    listen 80;
    server_name *.example.com;
    # 统一跳转到主域名，保留路径与查询参数
    if ($host != 'www.example.com') {
        rewrite ^(.*)$ http://www.example.com$1 permanent;
    }
}
```

### 78.5 301 vs 302 选择

| 类型 | 场景 |
|------|------|
| 301 永久 | 域名迁移、http→https，SEO 权重传递 |
| 302 临时 | 临时活动、A/B 测试、灰度发布 |

---

## 79. Nginx 核心指令_if 指令

`if` 用于条件判断，但 Nginx 官方明确"if is evil"，应谨慎使用，能用 try_files/map 替代则替代。

### 79.1 语法与可用条件

```nginx
if (condition) { ... }
```

**condition 可为：**

- 变量值：`$var`（非空非 0 为真）
- 等值比较：`$var = "value"`
- 不等比较：`$var != "value"`
- 正则匹配：`$var ~ "^pattern"`、`~*`（不区分大小写）、`!~`、`!~*`
- 文件判断：`-f $file`（存在）、`!-f`（不存在）
- 目录判断：`-d $dir`、`!-d`
- 可执行：`-e`、`-x`

### 79.2 常用示例

```nginx
# 1. UA 判断：移动端跳转
if ($http_user_agent ~* "(iPhone|Android)") {
    rewrite ^/$ /mobile/ redirect;
}

# 2. 禁止特定 IP 访问
if ($remote_addr = "1.2.3.4") {
    return 403;
}

# 3. 文件不存在时返回 404
if (!-f $request_filename) {
    return 404;
}

# 4. 限制请求方法
if ($request_method !~ ^(GET|POST|HEAD)$ ) {
    return 405;
}

# 5. HTTPS 检测（前置代理透传 X-Forwarded-Proto）
if ($http_x_forwarded_proto != "https") {
    return 301 https://$host$request_uri;
}
```

### 79.3 if 的陷阱

```nginx
# 错误：if 内部嵌套 location 不支持
location / {
    if ($var) {
        location /foo { ... }     # ❌ 语法错误
    }
}

# 错误：if 内部使用 rewrite 后行为不可预测
location /a {
    if ($flag) {
        rewrite ^ /b last;        # ⚠ 在 location 内的 if 中使用 last 可能失效
    }
}
```

### 79.4 替代方案

```nginx
# 用 try_files 替代 if -f
location / {
    try_files $uri $uri/ =404;    # 文件存在则返回，否则 404
}

# 用 map 替代复杂的 if 判断
map $http_user_agent $is_mobile {
    default 0;
    ~*iPhone|Android 1;
}
```

---

## 80. Nginx 核心指令_set 和 break 指令

### 80.1 set 指令

`set` 用于定义变量并赋值，作用域为当前 server/location。

```nginx
server {
    set $root_path "/data/www";        # 定义变量
    location / {
        root $root_path;               # 引用变量
    }
}

# 结合 if 设置动态变量
location / {
    set $cache_key "$host$request_uri";
    if ($http_x_cache = "bypass") {
        set $cache_key "${cache_key}_nocache";
    }
    proxy_cache_key $cache_key;
}
```

### 80.2 内置变量

| 变量 | 含义 |
|------|------|
| `$args` | 请求查询参数 |
| `$uri` | 当前 URI（不含参数） |
| `$request_uri` | 完整请求 URI（含参数） |
| `$host` | 请求 Host 头 |
| `$remote_addr` | 客户端 IP |
| `$http_xxx` | 请求头（xxx 为头名称） |
| `$scheme` | 协议 http/https |
| `$server_port` | 服务监听端口 |

### 80.3 break 指令

`break` 终止当前作用域内后续 rewrite 指令的执行，但继续处理当前 location 的其他指令。

```nginx
location /download/ {
    # 防止 rewrite 死循环
    if ($args ~ "token=secret") {
        break;                          # 跳出 if，继续后续配置
    }
    rewrite ^/download/(.*)$ /auth/$1 last;
}

# break 与 rewrite break 的区别
location /a {
    rewrite ^/a/(.*)$ /b/$1 break;      # rewrite 的 break flag，改写后停止 rewrite
    # 上面这种 break 是 rewrite 的参数，不是独立指令
}
```

### 80.4 实战：动态缓存 key

```nginx
location /api/ {
    set $cache_key "$scheme$host$request_uri";
    # 登录用户不缓存
    if ($http_cookie ~ "session_id") {
        set $cache_key "";
    }
    proxy_cache api_cache;
    proxy_cache_key $cache_key;
    proxy_pass http://backend;
}
```

---

## 81. Nginx 核心指令_Gzip 压缩指令

Gzip 压缩可显著减少传输体积，降低带宽与首屏时间，是 Web 性能优化必备。

```nginx
http {
    gzip on;                                  # 开启压缩
    gzip_min_length 1k;                       # 小于 1KB 不压缩（压缩收益低）
    gzip_comp_level 6;                        # 压缩级别 1-9，推荐 6
    gzip_types text/plain text/css text/xml application/javascript
               application/json application/xml;   # 压缩的 MIME 类型
    gzip_vary on;                             # 响应头加 Vary: Accept-Encoding
    gzip_disable "MSIE [1-6]\.";              # IE6 不支持 gzip
    gzip_buffers 4 16k;                       # 压缩缓冲区
    gzip_proxied any;                         # 反向代理也压缩
}
```

### 81.1 关键参数说明

| 参数 | 作用 | 推荐值 |
|------|------|--------|
| `gzip_min_length` | 触发压缩的最小体积 | 1k |
| `gzip_comp_level` | 压缩级别 | 6 |
| `gzip_types` | 压缩的文件类型 | 文本类，不含图片/视频 |
| `gzip_vary` | 添加 Vary 头 | on |

### 81.2 压缩级别选择

- 级别 1：速度最快，压缩率约 70%
- 级别 6：推荐平衡点，压缩率约 80%
- 级别 9：最慢，压缩率约 82%，CPU 占用高

```nginx
# CPU 富余场景
gzip_comp_level 9;
# 高并发场景，优先响应速度
gzip_comp_level 4;
```

### 81.3 不应压缩的类型

```nginx
# 图片/视频已是有损压缩，再次压缩无收益且浪费 CPU
gzip_types text/plain text/css application/javascript;  # 仅文本类
```

### 81.4 预压缩（gzip_static）

```nginx
# 启用 gzip_static 模块（编译时需 --with-http_gzip_static_module）
gzip_static on;
# Nginx 优先查找 .gz 文件，找到则直接发送，省去实时压缩
# 构建：gzip -k -9 index.html  → index.html.gz
```

---

## 82. Nginx 场景实践_浏览器缓存

通过响应头控制浏览器缓存策略，减少重复请求，提升用户体验。

### 82.1 缓存相关响应头

| 头 | 作用 |
|----|------|
| `Expires` | 绝对过期时间（HTTP 1.0） |
| `Cache-Control: max-age=N` | 相对过期秒数（HTTP 1.1，优先级高） |
| `Last-Modified` | 资源最后修改时间 |
| `ETag` | 资源唯一标识 |

### 82.2 expires 指令

```nginx
location ~* \.(jpg|jpeg|png|gif|ico)$ {
    expires 30d;                        # 浏览器缓存 30 天
    add_header Cache-Control "public";
}

location ~* \.(css|js)$ {
    expires 7d;                         # 7 天
    add_header Cache-Control "public";
}

location ~* \.(html)$ {
    expires -1;                         # 不缓存
    add_header Cache-Control "no-cache";
}
```

### 82.3 expires 取值

| 取值 | 含义 |
|------|------|
| `30d` / `30 days` | 缓存 30 天 |
| `1h` | 缓存 1 小时 |
| `@15h30m` | 今天 15:30 过期 |
| `-1` | 永远过期（不缓存） |
| `epoch` | 设为 1970-01-01 |
| `max` | 永不过期（10 年） |

### 82.4 Cache-Control 取值

| 值 | 含义 |
|----|------|
| `public` | 客户端与代理都可缓存 |
| `private` | 仅客户端缓存 |
| `no-cache` | 必须向服务器验证后才能用 |
| `no-store` | 完全不缓存 |
| `max-age=N` | N 秒内有效 |

### 82.5 协商缓存

```nginx
# Last-Modified 协商（默认开启）
location / {
    root /data/www;
    # 客户端发 If-Modified-Since，未修改则返回 304
}

# ETag 协商（默认开启）
# 客户端发 If-None-Match，匹配则返回 304
```

### 82.6 完整缓存策略示例

```nginx
server {
    location ~* \.(jpg|png|gif|woff2)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";   # immutable 防止刷新时重新验证
    }
    location ~* \.(css|js)$ {
        expires 7d;
        add_header Cache-Control "public";
    }
    location / {
        add_header Cache-Control "no-cache";            # HTML 必须每次验证
    }
}
```

---

## 83. Nginx 场景实践_防盗链技术

防盗链防止其他网站直接引用本站图片/视频资源，消耗本站带宽。

### 83.1 原理

通过校验请求头 `Referer`，判断请求来源是否合法。合法来源放行，否则返回 403 或替代图。

```nginx
location ~* \.(jpg|png|gif|mp4)$ {
    valid_referers none blocked server_names
        *.example.com example.*;        # 允许的来源
    if ($invalid_referer) {
        return 403;                     # 非法来源返回 403
    }
}
```

### 83.2 valid_referers 参数

| 参数 | 含义 |
|------|------|
| `none` | 请求头无 Referer（直接访问） |
| `blocked` | Referer 被代理删除 |
| `server_names` | 当前 server_name |
| `*.example.com` | 通配符域名 |
| `~regex` | 正则匹配 |

### 83.3 返回替代图片

```nginx
location ~* \.(jpg|png|gif)$ {
    root /data/images;
    valid_referers none blocked *.yoursite.com;
    if ($invalid_referer) {
        rewrite ^/.*\.(jpg|png|gif)$ /forbidden.png last;  # 重写到警告图
    }
}
```

### 83.4 资源签名（更可靠方案）

防盗链可被伪造 Referer 绕过，更安全的方案是给资源 URL 加签名：

```nginx
# 后端生成带签名的 URL：/img/photo.jpg?sign=md5(secret+uri+expire)
location /img/ {
    # 校验签名（需 Lua 模块，此处示意）
    access_by_lua_block {
        local sign = ngx.md5("mysecret" .. ngx.var.uri .. ngx.var.arg_expire)
        if sign ~= ngx.var.arg_sign then
            ngx.exit(403)
        end
    }
    root /data/images;
}
```

---

## 84. Nginx 场景实践_代理服务

代理（Proxy）是 Nginx 的核心能力之一，分为正向代理与反向代理。

### 84.1 正向代理 vs 反向代理

| 维度 | 正向代理 | 反向代理 |
|------|----------|----------|
| 代理对象 | 客户端 | 服务端 |
| 客户端是否感知 | 是（主动配置） | 否（透明） |
| 典型场景 | VPN、翻墙、内网出口 | 负载均衡、网关 |

```
正向代理：客户端 → [代理] → 服务器     （代理帮客户端访问）
反向代理：客户端 → [代理] → 服务器     （代理代表服务器接收）
```

### 84.2 Nginx 正向代理（HTTP）

```nginx
server {
    listen 8080;
    resolver 8.8.8.8;
    location / {
        proxy_pass $scheme://$host$request_uri;   # 动态代理到任意目标
        proxy_set_header Host $host;
    }
}
# 客户端配置代理：curl -x http://proxy:8080 http://target.com
```

### 84.3 Nginx 正向代理（HTTPS，需 ngx_http_proxy_connect_module）

```nginx
server {
    listen 443;
    proxy_connect;                       # 支持 CONNECT 方法
    proxy_connect_allow 443 563;
    proxy_connect_connect_timeout 10s;
    proxy_connect_data_timeout 10s;
    # ... 其他配置
}
```

### 84.4 代理相关核心指令

```nginx
proxy_pass http://backend;               # 代理目标
proxy_set_header Host $host;             # 透传 Host
proxy_set_header X-Real-IP $remote_addr; # 透传真实 IP
proxy_connect_timeout 5s;                # 连接超时
proxy_read_timeout 60s;                  # 读取超时
proxy_send_timeout 60s;                  # 发送超时
proxy_buffering on;                      # 缓冲后端响应
proxy_buffers 8 4k;                      # 缓冲区大小
```

---

## 85. Nginx 场景实践_反向代理

反向代理是生产环境最常见的 Nginx 用法，隐藏后端、统一入口、提供附加能力。

### 85.1 基础反向代理

```nginx
server {
    listen 80;
    server_name www.example.com;

    location / {
        proxy_pass http://127.0.0.1:8080;        # 转发到本地 Tomcat
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 85.2 透传客户端真实信息

```nginx
# 后端通过 X-Forwarded-For 获取真实 IP（注意防伪造）
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
# $proxy_add_x_forwarded_for = "已有 XFF 值, $remote_addr"
```

### 85.3 路径改写

```nginx
location /api/ {
    # /api/users → http://backend/users （去掉 /api 前缀）
    proxy_pass http://backend/;
    # 注意：proxy_pass 带 / 时，location 匹配部分被替换
}

location /v2/ {
    # /v2/users → http://backend/api/users
    rewrite ^/v2/(.*)$ /api/$1 break;
    proxy_pass http://backend;
}
```

### 85.4 WebSocket 代理

```nginx
location /ws/ {
    proxy_pass http://backend;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;        # 升级协议
    proxy_set_header Connection "upgrade";         # 保持连接
    proxy_read_timeout 3600s;                      # WS 长连接超时
}
```

### 85.5 代理 HTTPS 后端

```nginx
location /secure/ {
    proxy_pass https://backend;                    # 代理到 HTTPS 后端
    proxy_ssl_server_name on;
    proxy_ssl_name backend.example.com;
    proxy_ssl_verify on;                           # 校验后端证书
    proxy_ssl_trusted_certificate /etc/ssl/certs/ca-bundle.crt;
}
```

### 85.6 完整反向代理配置

```nginx
upstream backend {
    server 127.0.0.1:8080;
    keepalive 32;                                  # 与后端保持长连接
}

server {
    listen 80;
    server_name www.example.com;

    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Connection "";            # 清空 Connection，启用长连接
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        # 超时配置
        proxy_connect_timeout 5s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;

        # 错误处理
        proxy_next_upstream error timeout http_500;  # 失败自动切换上游
        proxy_intercept_errors on;                   # 拦截后端错误页
    }
}
```

---

## 86. Nginx 场景实践_负载均衡

通过 `upstream` 模块将请求分发到多台后端服务器，提升系统吞吐与可用性。

### 86.1 基础负载均衡

```nginx
http {
    upstream backend {
        server 192.168.1.10:8080;          # 后端 1
        server 192.168.1.11:8080;          # 后端 2
        server 192.168.1.12:8080;          # 后端 3
    }

    server {
        listen 80;
        location / {
            proxy_pass http://backend;
        }
    }
}
```

### 86.2 server 指令参数

```nginx
server address [parameters];
```

| 参数 | 含义 |
|------|------|
| `weight=N` | 权重，默认 1 |
| `max_fails=N` | 失败 N 次后标记宕机，默认 1 |
| `fail_timeout=T` | 失败判定时间窗口与恢复探测周期，默认 10s |
| `backup` | 备用服务器，仅当主服务器全宕机时启用 |
| `down` | 标记为不可用，用于灰度下线 |
| `max_conns=N` | 最大并发连接数（限流） |
| `resolve` | 动态解析域名（商业版或 1.27+） |

### 86.3 完整配置示例

```nginx
upstream backend {
    server 192.168.1.10:8080 weight=3 max_fails=3 fail_timeout=30s;
    server 192.168.1.11:8080 weight=2 max_fails=3 fail_timeout=30s;
    server 192.168.1.12:8080 weight=1 backup;          # 备用
    keepalive 32;                                       # 与后端长连接池
}

server {
    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }
}
```

### 86.4 健康检查机制

```nginx
# Nginx 开源版被动健康检查
server 192.168.1.10:8080 max_fails=3 fail_timeout=30s;
# 含义：30 秒内失败 3 次则标记宕机，30 秒后开始探测恢复

# Nginx Plus（商业版）主动健康检查
health_check interval=10s fails=3 passes=2 uri=/health;
```

---

## 87. Nginx 场景实践_负载均衡算法

### 87.1 轮询（Round Robin，默认）

```nginx
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
    # 默认轮询，按顺序逐一分配
}
```

### 87.2 加权轮询（Weighted Round Robin）

```nginx
upstream backend {
    server 192.168.1.10:8080 weight=3;    # 性能强，权重高
    server 192.168.1.11:8080 weight=2;
    server 192.168.1.12:8080 weight=1;
    # 6 次请求分配：3→1，2→2，1→3
}
```

### 87.3 ip_hash（基于 IP 哈希）

保证同一客户端 IP 始终访问同一后端，解决 Session 粘性问题。

```nginx
upstream backend {
    ip_hash;                               # 必须写在 server 之前
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
}
# 哈希算法：hash(client_ip) % server_count
```

> 注意：使用 ip_hash 时不能使用 `backup`，且后端摘除需谨慎。

### 87.4 least_conn（最少连接）

将请求分配给当前连接数最少的后端，适合长连接或处理时长不均的场景。

```nginx
upstream backend {
    least_conn;                            # 最少连接算法
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
}
```

### 87.5 url_hash（基于 URL 哈希）

同一 URL 始终落到同一后端，提升后端缓存命中率（需第三方模块）。

```nginx
upstream backend {
    hash $request_uri;                     # 按 URI 哈希
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
}
```

### 87.6 一致性哈希（consistent）

```nginx
upstream backend {
    hash $request_uri consistent;          # 一致性哈希，节点增减时影响小
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
}
```

### 87.7 算法对比

| 算法 | 特点 | 适用 |
|------|------|------|
| 轮询 | 简单公平 | 后端性能一致 |
| 加权轮询 | 按能力分配 | 后端性能不一致 |
| ip_hash | 会话保持 | Session 强依赖 |
| least_conn | 负载均衡 | 长连接、请求时长不均 |
| url_hash | 缓存友好 | 后端有缓存 |
| 一致性哈希 | 节点变动影响小 | 大规模集群 |

---

## 88. Nginx 场景实践_第三方fair模块安装

`fair` 模块按后端响应时间分配请求，响应快的多分配，响应慢的少分配，实现更智能的负载均衡。

### 88.1 fair 算法原理

```
请求 → Nginx 记录每个后端的最近响应时间
     → 优先将请求分配给响应时间最短的后端
     → 动态调整，自适应负载
```

### 88.2 编译安装 fair 模块

```bash
# 1. 下载 fair 源码
cd /usr/local/src
git clone https://github.com/gnosek/nginx-upstream-fair.git

# 2. 重新编译 Nginx
cd /usr/local/src/nginx-1.24.0
./configure \
  --prefix=/usr/local/nginx \
  --add-module=/usr/local/src/nginx-upstream-fair \   # 添加 fair 模块
  --with-http_ssl_module

# 3. 编译（不要 make install，避免覆盖配置）
make
# 4. 替换二进制（先备份旧的）
mv /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.bak
cp objs/nginx /usr/local/nginx/sbin/
# 5. 平滑重启
/usr/local/nginx/sbin/nginx -s reload
```

### 88.3 配置使用

```nginx
upstream backend {
    fair;                                  # 启用 fair 算法
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}
```

### 88.4 注意事项

- fair 模块需重新编译，每次 Nginx 升级需重新集成
- 老版本 fair 模块可能不兼容新版 Nginx，建议用 `nginx-upstream-fair` 的 fork 版本
- 现代替代方案：Nginx Plus 的 `least_time` 指令，或使用 OpenResty + Lua 自定义算法

---

## 89. Nginx 场景实践_Nginx 配置故障转移

故障转移（Failover）指后端节点故障时，自动将请求切换到健康节点的机制。

### 89.1 被动健康检查（开源版）

```nginx
upstream backend {
    server 192.168.1.10:8080 max_fails=3 fail_timeout=30s;
    server 192.168.1.11:8080 max_fails=3 fail_timeout=30s;
    server 192.168.1.12:8080 max_fails=3 fail_timeout=30s;
}
# 30 秒内失败 3 次 → 标记宕机 → 30 秒后开始探测恢复
```

### 89.2 proxy_next_upstream 自动重试

```nginx
location / {
    proxy_pass http://backend;
    # 以下情况自动切换到下一台后端
    proxy_next_upstream error timeout http_500 http_502 http_503 http_504;
    # 重试限制
    proxy_next_upstream_tries 3;           # 最多重试 3 次
    proxy_next_upstream_timeout 10s;       # 重试总时长上限
    # 请求方法限制：默认仅 GET/HEAD/POST 等幂等方法重试
    proxy_next_upstream non_idempotent;    # 允许非幂等方法重试（慎用）
}
```

### 89.3 backup 备用服务器

```nginx
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.99:8080 backup;       # 仅当主服务器全宕机时启用
}
```

### 89.4 自定义错误页

```nginx
server {
    location / {
        proxy_pass http://backend;
        proxy_intercept_errors on;         # 拦截后端错误响应
        error_page 500 502 503 504 /50x.html;
    }
    location = /50x.html {
        root /data/www/error;
        internal;                          # 仅内部跳转可访问
    }
}
```

### 89.5 主动健康检查（开源方案：nginx_upstream_check_module）

```nginx
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    check interval=3000 rise=2 fall=3 timeout=2000 type=http;
    # interval: 探测间隔 3 秒
    # rise: 连续成功 2 次标记健康
    # fall: 连续失败 3 次标记宕机
    # type: 探测协议
    check_http_send "HEAD /health HTTP/1.0\r\n\r\n";
    check_http_expect_alive http_2xx http_3xx;
}
```

### 89.6 优雅下线

```nginx
# 方式 1：临时标记 down
upstream backend {
    server 192.168.1.10:8080 down;        # 临时下线，平滑重载后生效
    server 192.168.1.11:8080;
}

# 方式 2：权重置 0
upstream backend {
    server 192.168.1.10:8080 weight=0;    # 不再接收新请求
    server 192.168.1.11:8080 weight=1;
}
```

---

## 90. Nginx 场景实践_跨域问题

### 90.1 什么是跨域

**同源策略**：浏览器出于安全，限制 JS 脚本发起跨域请求。同源 = 协议 + 域名 + 端口完全一致。

```
http://www.a.com/page → http://www.a.com/api          ✅ 同源
http://www.a.com      → https://www.a.com              ❌ 协议不同
http://www.a.com      → http://api.a.com               ❌ 域名不同
http://www.a.com:80   → http://www.a.com:8080          ❌ 端口不同
```

### 90.2 CORS（跨域资源共享）

W3C 标准，通过 HTTP 头声明允许跨域。Nginx 作为服务端（或反向代理）添加响应头即可解决。

**简单请求**：GET/HEAD/POST（仅 text/plain 等简单 Content-Type）
**预检请求**：浏览器先发 OPTIONS 探测，服务端同意后才发真实请求

### 90.3 关键响应头

| 头 | 作用 |
|----|------|
| `Access-Control-Allow-Origin` | 允许的源（精确域名或 *） |
| `Access-Control-Allow-Methods` | 允许的方法 |
| `Access-Control-Allow-Headers` | 允许的请求头 |
| `Access-Control-Allow-Credentials` | 是否允许携带 Cookie |
| `Access-Control-Max-Age` | 预检结果缓存时间 |

### 90.4 跨域问题示例

```
前端：http://www.a.com
后端：http://api.a.com

浏览器报错：
No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

---

## 91. Nginx 解决跨域问题

### 91.1 方案一：响应头添加（后端配置）

```nginx
server {
    listen 80;
    server_name api.a.com;

    location / {
        # 允许指定域名跨域
        add_header Access-Control-Allow-Origin "http://www.a.com";
        add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS";
        add_header Access-Control-Allow-Headers "Content-Type, Authorization";
        add_header Access-Control-Allow-Credentials "true";
        add_header Access-Control-Max-Age 3600;

        # 处理预检请求
        if ($request_method = OPTIONS) {
            return 204;                    # 预检请求直接返回 204
        }

        proxy_pass http://backend;
    }
}
```

### 91.2 方案二：动态回填 Origin

```nginx
location / {
    # 回填客户端 Origin，支持多域名
    add_header Access-Control-Allow-Origin $http_origin;
    add_header Access-Control-Allow-Methods "GET, POST, OPTIONS";
    add_header Access-Control-Allow-Headers "*";
    add_header Access-Control-Allow-Credentials "true";

    if ($request_method = OPTIONS) {
        return 204;
    }
    proxy_pass http://backend;
}
```

### 91.3 方案三：白名单校验

```nginx
map $http_origin $allow_origin {
    default "";
    "~^http://(www|admin|m)\.a\.com$" "$http_origin";   # 白名单
    "~^https://.*\.a\.com$"            "$http_origin";
}

server {
    location / {
        add_header Access-Control-Allow-Origin $allow_origin;
        add_header Access-Control-Allow-Methods "GET, POST, OPTIONS";
        add_header Access-Control-Allow-Credentials "true";

        if ($request_method = OPTIONS) {
            add_header Access-Control-Allow-Origin $allow_origin;
            add_header Access-Control-Allow-Methods "GET, POST, OPTIONS";
            add_header Access-Control-Allow-Credentials "true";
            return 204;
        }
        proxy_pass http://backend;
    }
}
```

### 91.4 方案四：反向代理同源化（推荐）

通过 Nginx 代理，让前后端看起来同源，从根本上避免跨域。

```nginx
server {
    listen 80;
    server_name www.a.com;

    # 前端静态资源
    location / {
        root /data/www;
        index index.html;
    }

    # 后端 API（看起来还是 www.a.com）
    location /api/ {
        proxy_pass http://api-server:8080/;   # 转发到后端真实地址
    }
}
# 前端请求 /api/users，实际是同源，无跨域问题
```

### 91.5 注意事项

- `Access-Control-Allow-Origin: *` 与 `Allow-Credentials: true` **不能同时使用**
- 预检 OPTIONS 请求**必须返回 200/204**，且需带齐 CORS 响应头
- Cookie 跨域需 `Allow-Credentials: true`，且前端 XHR 需 `withCredentials: true`

---

## 92. Nginx 场景实践_动静分离

### 92.1 什么是动静分离

将静态资源（HTML/CSS/JS/图片/视频）与动态请求（JSP/PHP/接口）由不同服务器处理，各取所长。

```
客户端 → Nginx
         ├─ 静态资源 → Nginx 本地直出（高性能）
         └─ 动态请求 → 后端应用服务器（Tomcat/Node/PHP-FPM）
```

### 92.2 为什么需要动静分离

| 维度 | 静态资源 | 动态请求 |
|------|----------|----------|
| 处理特点 | 文件 I/O 为主 | CPU/DB 为主 |
| 适合服务 | Nginx（事件驱动，文件 IO 强） | Tomcat/PHP-FPM（线程模型，业务处理强） |
| 缓存策略 | 浏览器/CDN 长缓存 | 不缓存或短缓存 |
| 扩展方式 | 增加静态服务器 | 增加应用服务器 |

Tomcat 处理静态资源能力远弱于 Nginx（约 1:5），混合部署会拖累动态请求处理。

### 92.3 部署架构

```
                 ┌─ 静态资源服务器集群（Nginx）
Nginx 入口 ──────┤
                 └─ 动态应用服务器集群（Tomcat/Node）
```

- 静态资源部署在 Nginx 本地或独立静态服务器
- 动态请求通过反向代理转发到应用服务器集群

---

## 93. Nginx 场景实践_动静分离实战

### 93.1 通过 location 区分

```nginx
server {
    listen 80;
    server_name www.example.com;

    # 静态资源：Nginx 直接响应
    location ~* \.(html|css|js|jpg|png|gif|ico|woff2|svg)$ {
        root /data/www/static;
        expires 7d;                              # 浏览器缓存
    }

    # 动态请求：转发到 Tomcat 集群
    location ~ \.(jsp|do)$ {
        proxy_pass http://tomcat_cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # RESTful API：转发到应用服务器
    location /api/ {
        proxy_pass http://app_cluster;
    }

    # 默认：静态首页
    location / {
        root /data/www;
        index index.html;
    }
}

upstream tomcat_cluster {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
}

upstream app_cluster {
    server 192.168.1.20:3000;
    server 192.168.1.21:3000;
}
```

### 93.2 通过 root/alias 区分目录

```nginx
server {
    root /data/www;                              # 默认根目录

    location /static/ {
        alias /data/static/;                     # 静态资源独立目录
        expires 30d;
    }
    location /upload/ {
        alias /data/upload/;                     # 用户上传文件
    }
    location /api/ {
        proxy_pass http://backend;
    }
}
```

### 93.3 静态资源独立服务器

```nginx
# 静态服务器（独立部署）
server {
    listen 80;
    server_name static.example.com;
    root /data/static;
    location / {
        expires 30d;
        add_header Cache-Control "public";
    }
}

# 应用入口（独立部署）
server {
    listen 80;
    server_name www.example.com;
    location /static/ {
        proxy_pass http://static.example.com/;   # 转发到静态服务器
    }
    location /api/ {
        proxy_pass http://backend;
    }
}
```

### 93.4 完整生产配置

```nginx
http {
    upstream web_cluster {
        server 192.168.1.10:8080;
        server 192.168.1.11:8080;
    }

    upstream static_server {
        server 192.168.1.20:80;
    }

    server {
        listen 80;
        server_name www.example.com;

        # 静态资源（本地直出）
        location ~* \.(css|js|jpg|png|gif|woff2)$ {
            root /data/www/static;
            expires 7d;
            access_log off;                      # 静态资源不记访问日志
        }

        # 大文件（独立静态服务器）
        location /download/ {
            proxy_pass http://static_server;
        }

        # 动态请求
        location / {
            proxy_pass http://web_cluster;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

---

## 94. Nginx 场景实践_什么是限流

### 94.1 为什么需要限流

高并发场景下，突发流量可能压垮后端，限流是保护系统的最后一道防线。

```
正常 QPS：1K
突发 QPS：100K（被刷、热点事件、DDoS）

无限流：后端直接被打挂，所有用户不可用
有限流：Nginx 拦截超额请求，保证核心业务可用
```

### 94.2 限流的层次

| 层次 | 实现 | 特点 |
|------|------|------|
| 网关层 | Nginx 限流 | 最前段，快速拒绝 |
| 应用层 | Sentinel/Hystrix | 业务级，精细控制 |
| DB 层 | 连接池上限 | 最后防线 |

### 94.3 限流维度

- **按 IP**：限制单 IP 请求频率（防爬虫、防刷）
- **按 URL**：限制特定接口频率（保护慢接口）
- **按用户**：限制单用户操作频率（防滥用）
- **按总量**：限制全局 QPS（保护后端容量）

### 94.4 限流策略

| 策略 | 行为 | 适用 |
|------|------|------|
| 拒绝 | 超额请求直接返回 503 | 防刷、保护后端 |
| 等待 | 超额请求排队 | 排队买票、抢购 |
| 降级 | 超额请求返回降级数据 | 推荐列表、热点查询 |

---

## 95. Nginx 场景实践_限流算法

### 95.1 计数器算法（固定窗口）

```
每秒最多 100 请求：
- 0~1s 内请求数 < 100，放行
- 超过 100 拒绝
- 1s 后清零重新计数
```

**缺点**：临界问题。0.9s 时来 100 个，1.1s 时又来 100 个，0.2s 内实际通过 200 个。

### 95.2 滑动窗口算法

将固定窗口切分为多个小窗口，平滑计数，缓解临界问题。

```
1 秒窗口分为 4 个 250ms 子窗口：
[t-750, t-500]: 30
[t-500, t-250]: 40
[t-250, t]:     25
当前累计：95，还能通过 5 个
```

### 95.3 漏桶算法（Leaky Bucket）

请求像水滴进入漏桶，桶以固定速率漏水。桶满则拒绝，桶未满则入桶等待。

```
        入请求（突发）
           ↓
       ┌───────┐
       │ 桶(容量N) │  ← 桶满则拒绝新请求
       └───┬───┘
           ↓ 固定速率流出（处理）
       后端服务器
```

**特点**：平滑输出，无法应对突发流量。Nginx `limit_req` 默认基于漏桶。

### 95.4 令牌桶算法（Token Bucket）

固定速率向桶中放令牌，桶满则丢弃。请求需拿到令牌才处理，无令牌则拒绝/等待。

```
       固定速率产生令牌
              ↓
         ┌─────────┐
         │ 令牌桶(N) │  ← 桶满则丢弃多余令牌
         └────┬────┘
              ↓ 取令牌
          请求 → 有令牌：放行；无令牌：拒绝
```

**特点**：允许突发（桶中令牌可瞬时消费），适合有突发流量的业务。

### 95.5 漏桶 vs 令牌桶

| 维度 | 漏桶 | 令牌桶 |
|------|------|--------|
| 输出速率 | 恒定 | 可突发 |
| 突发流量 | 拒绝/排队 | 允许（桶中令牌） |
| 适用 | 保护后端，平滑流量 | 允许合理突发 |
| Nginx 实现 | `limit_req` | 需 Lua/OpenResty 实现 |

---

## 96. Nginx 场景实践_限流实现

### 96.1 limit_req 模块（请求频率限流）

基于漏桶算法，按请求速率限流。

```nginx
http {
    # 定义限流维度：按 IP 限流，桶容量 1，速率 1 请求/秒
    limit_req_zone $binary_remote_addr zone=req_zone:10m rate=1r/s;

    server {
        location /api/ {
            limit_req zone=req_zone burst=5 nodelay;
            # burst=5：桶容量 5，允许突发 5 个排队
            # nodelay：突发请求不延迟，立即处理
            proxy_pass http://backend;
        }
    }
}
```

**参数说明：**

| 参数 | 含义 |
|------|------|
| `zone=name:size` | 共享内存区名称与大小（10m ≈ 16万 IP） |
| `rate=1r/s` | 每秒 1 个请求（也支持 `30r/m` 每分钟 30 个） |
| `burst=N` | 桶容量，允许突发排队 N 个 |
| `nodelay` | 突发请求立即处理，不延迟 |
| `delay=N` | 前 N 个立即处理，超过则延迟 |

### 96.2 limit_conn 模块（并发连接数限流）

限制单 IP 的并发连接数。

```nginx
http {
    limit_conn_zone $binary_remote_addr zone=conn_zone:10m;

    server {
        location /download/ {
            limit_conn conn_zone 1;          # 单 IP 同时只允许 1 个连接
            limit_rate 100k;                 # 单连接限速 100KB/s
            proxy_pass http://backend;
        }
    }
}
```

### 96.3 limit_rate（响应带宽限流）

```nginx
location /download/ {
    limit_rate 100k;                         # 每连接 100KB/s
    limit_rate_after 1m;                     # 下载 1MB 后开始限速
    root /data/files;
}
```

### 96.4 完整限流配置

```nginx
http {
    # 按 IP 限流：每秒 10 个请求
    limit_req_zone $binary_remote_addr zone=ip_limit:10m rate=10r/s;
    # 按 URI 限流：每个 URI 每秒 100 请求
    limit_req_zone $server_name zone=server_limit:10m rate=100r/s;
    # 并发连接数限流
    limit_conn_zone $binary_remote_addr zone=conn_limit:10m;

    server {
        # 全局限流
        limit_req zone=server_limit burst=20 nodelay;
        limit_conn conn_limit 50;

        location /api/login {
            # 登录接口更严格：每秒 1 个，防止暴力破解
            limit_req zone=ip_limit burst=3 nodelay;
            proxy_pass http://backend;
        }

        location /api/ {
            # 普通接口：每秒 10 个
            limit_req zone=ip_limit burst=10 nodelay;
            proxy_pass http://backend;
        }

        # 自定义限流响应
        limit_req_status 429;                # 限流时返回 429
        limit_conn_status 429;
    }
}
```

### 96.5 限流响应优化

```nginx
location /api/ {
    limit_req zone=ip_limit burst=10 nodelay;

    # 限流时返回 JSON 而非默认 503
    limit_req_status 429;
    error_page 429 = @limited;
}

location @limited {
    default_type application/json;
    return 429 '{"code":429,"msg":"请求过于频繁，请稍后再试"}';
}
```

### 96.6 高级：基于 Lua 的动态限流

```nginx
# OpenResty 实现：按用户限流
location /api/ {
    access_by_lua_block {
        local limit = require "resty.limit.req"
        -- 每秒 100 请求，突发 50
        local lim = limit.new("user_limit", 100, 50)
        local key = ngx.var.http_x_user_id or ngx.var.remote_addr
        local delay, err = lim:incoming(key, true)
        if not delay then
            if err == "rejected" then
                ngx.header["Retry-After"] = "1"
                ngx.exit(429)
            end
        end
    }
    proxy_pass http://backend;
}
```

---

## 97. Nginx 场景实践_WEB缓存机制

### 97.1 Nginx 缓存层级

```
客户端浏览器缓存 ←→ Nginx 代理缓存 ←→ 后端应用缓存 ←→ DB
        ↑                  ↑                ↑
    (82节)             (本节)            (应用层)
```

### 97.2 proxy_cache 代理缓存

缓存后端响应，减少重复请求打到后端，大幅提升性能。

```nginx
http {
    # 定义缓存区
    proxy_cache_path /data/nginx/cache
        levels=1:2                          # 目录层级：1级1字符 + 2级2字符
        keys_zone=api_cache:10m             # 共享内存区，10m ≈ 8万 key
        max_size=10g                        # 磁盘缓存上限
        inactive=60m                        # 60 分钟未被访问则删除
        use_temp_path=off;                  # 直接写入缓存目录，减少 IO

    server {
        location /api/ {
            proxy_cache api_cache;          # 启用缓存
            proxy_cache_key "$scheme$host$request_uri";   # 缓存 key
            proxy_cache_valid 200 302 10m;  # 200/302 缓存 10 分钟
            proxy_cache_valid 404 1m;       # 404 缓存 1 分钟
            proxy_cache_valid any 5s;       # 其他状态缓存 5 秒

            # 响应头加缓存命中状态
            add_header X-Cache-Status $upstream_cache_status;

            proxy_pass http://backend;
        }
    }
}
```

### 97.3 upstream_cache_status 状态值

| 值 | 含义 |
|----|------|
| `MISS` | 未命中，请求转发到后端 |
| `HIT` | 命中缓存，直接返回 |
| `EXPIRED` | 缓存已过期，重新请求后端 |
| `STALE` | 命中过期缓存，后台更新（stale 模式） |
| `UPDATING` | 正在更新缓存，返回旧内容 |
| `BYPASS` | 跳过缓存（proxy_cache_bypass 命中） |
| `REVALIDATED` | 启用 proxy_cache_revalidate 后，304 校验通过 |

### 97.4 不缓存特定请求

```nginx
location /api/ {
    proxy_cache api_cache;
    # POST 请求不缓存
    proxy_cache_methods GET HEAD;           # 默认仅缓存 GET/HEAD
    # 带 Cookie 不缓存
    proxy_cache_bypass $http_cookie;        # 有 Cookie 时跳过缓存
    proxy_no_cache $http_cookie;            # 有 Cookie 时不写入缓存
    # 特定参数不缓存
    set $skip_cache 0;
    if ($arg_nocache = "1") {
        set $skip_cache 1;
    }
    proxy_cache_bypass $skip_cache;
    proxy_no_cache $skip_cache;
    proxy_pass http://backend;
}
```

### 97.5 缓存清理

```nginx
# 方式 1：删除缓存文件（暴力）
rm -rf /data/nginx/cache/*
nginx -s reload

# 方式 2：proxy_cache_purge 模块（精确清理）
location ~ /purge(/.*) {
    allow 127.0.0.1;                        # 仅允许本机清理
    deny all;
    proxy_cache_purge api_cache $scheme$host$1;
}
# 访问 http://www.example.com/purge/api/users 即清理 /api/users 的缓存
```

### 97.6 stale 降级策略

```nginx
location /api/ {
    proxy_cache api_cache;
    proxy_cache_valid 200 10m;

    # 后端故障时返回过期缓存，保证可用性
    proxy_cache_use_stale error timeout updating
                          http_500 http_502 http_503 http_504;
    # 后端故障时优先用过期缓存，而不是返回错误页
    proxy_cache_background_update on;       # 后台异步更新缓存
    proxy_cache_lock on;                    # 同一 key 并发请求只放一个到后端

    proxy_pass http://backend;
}
```

### 97.7 FastCGI 缓存（PHP 场景）

```nginx
http {
    fastcgi_cache_path /data/nginx/fcgi_cache
        levels=1:2 keys_zone=fcgi:10m max_size=5g inactive=30m;

    server {
        location ~ \.php$ {
            fastcgi_cache fcgi;
            fastcgi_cache_key $scheme$host$request_uri;
            fastcgi_cache_valid 200 302 10m;
            fastcgi_cache_valid 404 1m;
            fastcgi_pass 127.0.0.1:9000;
            include fastcgi_params;
        }
    }
}
```

---

## 98. Nginx 场景实践_Nginx 高可用

### 98.1 单点问题

单台 Nginx 是整个系统的入口，一旦宕机，所有服务不可用。

```
客户端 → Nginx（单点） → 后端集群
           ↑
        宕机则全线不可用
```

### 98.2 高可用方案对比

| 方案 | 实现 | 成本 | 适用 |
|------|------|------|------|
| Keepalived + Nginx | VIP 漂移 | 低 | 主流方案 |
| LVS + Keepalived | 四层负载 + VIP | 中 | 超大流量 |
| 云厂商 SLB | 托管负载均衡 | 按量付费 | 云上业务 |
| DNS 轮询 | 多 IP 解析 | 低 | 简单容灾 |
| Heartbeat/Corosync | 集群高可用 | 中 | 复杂场景 |

### 98.3 Keepalived + Nginx 架构

```
                  VIP: 192.168.1.100
                        │
        ┌───────────────┴───────────────┐
        │                               │
   Master Nginx                    Backup Nginx
   192.168.1.10                   192.168.1.11
   Keepalived(主)                 Keepalived(备)
        │                               │
        └───────────────┬───────────────┘
                        │
                  后端服务器集群
```

### 98.4 高可用原理

1. Master/Backup 同时运行 Keepalived，通过 VRRP 协议通信
2. Master 持有 VIP，对外提供服务
3. Master 故障时，Backup 检测心跳超时，接管 VIP
4. 客户端通过 VIP 访问，无感知切换

### 98.5 双机主备 vs 双主模式

**主备模式**：一主一备，备机闲置
**双主模式**：两台均为主，互为备份，资源利用率高

```
双主模式：
Nginx A：VIP1 主，VIP2 备
Nginx B：VIP1 备，VIP2 主

DNS 解析：
www.example.com → VIP1, VIP2
```

---

## 99. Nginx 场景实践_LVS负载均衡

### 99.1 LVS 简介

LVS（Linux Virtual Server）是章文嵩博士开源的四层负载均衡，工作在内核态，性能远超 Nginx（七层）。

### 99.2 LVS vs Nginx

| 维度 | LVS | Nginx |
|------|-----|-------|
| 工作层级 | 四层（传输层） | 七层（应用层） |
| 性能 | 极高（内核态，百万 QPS） | 高（用户态，10万级 QPS） |
| 功能 | 仅负载均衡 | 负载均衡 + HTTP 处理 |
| 配置复杂度 | 较高 | 简单 |
| 健康检查 | 需配合 Keepalived | 内置 |

### 99.3 LVS 三种工作模式

#### 99.3.1 NAT 模式

```
客户端 → LVS(改写目标IP) → RS → LVS(改写源IP) → 客户端
```
- LVS 承担双向流量，成为瓶颈
- RS 可任意 OS，无需改动
- 适合小规模集群

#### 99.3.2 DR 模式（直接路由，推荐）

```
客户端 → LVS(改写MAC) → RS(直接响应) → 客户端
```
- LVS 仅处理请求，响应由 RS 直发，性能最高
- LVS 与 RS 必须在同一物理网络（同一 VLAN）
- RS 需配置 loopback 上的 VIP，并抑制 ARP

#### 99.3.3 TUN 模式（IP 隧道）

```
客户端 → LVS(IP隧道封装) → RS(解封装,直接响应) → 客户端
```
- LVS 与 RS 可跨网段
- RS 需支持 IP 隧道协议
- 适合异地容灾

### 99.4 LVS + Keepalived 配置

```bash
# 安装
yum install -y ipvsadm keepalived

# Keepalived 配置（Master）
cat > /etc/keepalived/keepalived.conf <<EOF
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.100              # VIP
    }
}

# LVS 虚拟服务
virtual_server 192.168.1.100 80 {
    delay_loop 6                   # 健康检查间隔 6 秒
    lb_algo rr                     # 调度算法：轮询
    lb_kind DR                     # 工作模式：DR
    protocol TCP

    real_server 192.168.1.20 80 {  # 后端 RS1
        weight 1
        TCP_CHECK {
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
    real_server 192.168.1.21 80 {  # 后端 RS2
        weight 1
        TCP_CHECK {
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
EOF

systemctl start keepalived
```

### 99.5 LVS 调度算法

| 算法 | 说明 |
|------|------|
| `rr` | 轮询 |
| `wrr` | 加权轮询 |
| `lc` | 最少连接 |
| `wlc` | 加权最少连接（默认） |
| `sh` | 源地址哈希 |
| `dh` | 目标地址哈希 |
| `lblc` | 基于局部性的最少连接 |

---

## 100. Nginx 场景实践_Keepalived 健康监测

### 100.1 Keepalived 简介

Keepalived 基于 VRRP 协议实现高可用，同时提供后端健康检查能力。

### 100.2 VRRP 协议

虚拟路由冗余协议（Virtual Router Redundancy Protocol）：

- 多台路由器组成一个虚拟路由器组
- 选举 Master 持有 VIP
- Master 定期发送 VRRP 通告（默认 1 秒）
- Backup 优先级低于 Master，超时未收到通告则选举新 Master

### 100.3 健康检查脚本

```bash
# 检测 Nginx 是否存活
cat > /etc/keepalived/check_nginx.sh <<'EOF'
#!/bin/bash
if ! killall -0 nginx 2>/dev/null; then
    systemctl stop keepalived          # Nginx 挂了，停掉 Keepalived，触发 VIP 漂移
fi
EOF
chmod +x /etc/keepalived/check_nginx.sh
```

### 100.4 Keepalived 配置（Master）

```nginx
vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 2                              # 每 2 秒检查一次
    weight -20                              # 失败则优先级减 20
    fall 2                                  # 连续失败 2 次才判定
    rise 1                                  # 成功 1 次即恢复
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.100
    }
    track_script {
        check_nginx                          # 关联健康检查脚本
    }
}
```

### 100.5 Keepalived 配置（Backup）

```nginx
vrrp_instance VI_1 {
    state BACKUP                            # 备机
    interface eth0
    virtual_router_id 51                    # 必须与 Master 一致
    priority 90                             # 优先级低于 Master
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.100
    }
    track_script {
        check_nginx
    }
}
```

### 100.6 健康检查方式

```nginx
# TCP_CHECK：检测端口连通性
TCP_CHECK {
    connect_port 80
    connect_timeout 3
}

# HTTP_GET：检测 HTTP 响应
HTTP_GET {
    url {
        path /health
        status_code 200
    }
    connect_timeout 3
    nb_get_retry 3
    delay_before_retry 3
}

# SSL_GET：检测 HTTPS
SSL_GET {
    url {
        path /health
        digest 2432c9b...
    }
}

# MISC_CHECK：自定义脚本
MISC_CHECK {
    misc_path "/etc/keepalived/check_app.sh"
    misc_timeout 5
}
```

### 100.7 通知机制

```nginx
vrrp_instance VI_1 {
    # ...
    notify_master "/etc/keepalived/notify.sh master"
    notify_backup "/etc/keepalived/notify.sh backup"
    notify_fault   "/etc/keepalived/notify.sh fault"
}
```

```bash
# notify.sh 示例
#!/bin/bash
STATE=$1
case $STATE in
    master)
        echo "$(date) - Became MASTER" >> /var/log/keepalived.log
        systemctl start nginx
        ;;
    backup)
        echo "$(date) - Became BACKUP" >> /var/log/keepalived.log
        ;;
esac
```

---

## 101. Nginx 场景实践_企业双机热备方案

### 101.1 双机热备架构

生产环境标准高可用方案：两台 Nginx + Keepalived + VIP，对外暴露单一入口。

```
                  DNS
                   │
            www.example.com
                   │
              VIP: 192.168.1.100
                   │
       ┌───────────┴───────────┐
       │                       │
  Nginx Master              Nginx Backup
  192.168.1.10              192.168.1.11
  Keepalived(主)            Keepalived(备)
       │                       │
       └───────────┬───────────┘
                   │
            后端应用集群
       192.168.1.20 ~ 192.168.1.25
```

### 101.2 双机热备 vs 双主模式

**双机热备（主备）**：一主一备，备机闲置，Master 故障时 Backup 接管
**双机双主**：两台均为主，互为备份，资源利用率翻倍

### 101.3 完整配置示例

#### 101.3.1 Master 节点 Keepalived 配置

```nginx
global_defs {
    router_id NGINX_MASTER              # 路由器标识
}

vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 2
    weight -20
    fall 2
    rise 1
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51                # VRID，主备必须一致
    priority 100                        # 主优先级高
    advert_int 1                        # VRRP 通告间隔

    authentication {
        auth_type PASS
        auth_pass 1q2w3e                # 主备密码一致
    }

    virtual_ipaddress {
        192.168.1.100/24                # VIP
    }

    track_script {
        check_nginx
    }

    notify_master "/etc/keepalived/notify.sh master"
    notify_backup "/etc/keepalived/notify.sh backup"
    notify_fault   "/etc/keepalived/notify.sh fault"
}
```

#### 101.3.2 Backup 节点 Keepalived 配置

```nginx
global_defs {
    router_id NGINX_BACKUP
}

vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 2
    weight -20
    fall 2
    rise 1
}

vrrp_instance VI_1 {
    state BACKUP                        # 备机
    interface eth0
    virtual_router_id 51                # 必须与 Master 一致
    priority 90                         # 优先级低于 Master
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass 1q2w3e
    }

    virtual_ipaddress {
        192.168.1.100/24
    }

    track_script {
        check_nginx
    }

    notify_master "/etc/keepalived/notify.sh master"
    notify_backup "/etc/keepalived/notify.sh backup"
}
```

#### 101.3.3 健康检查脚本

```bash
#!/bin/bash
# /etc/keepalived/check_nginx.sh
if [ -f /var/run/nginx.pid ]; then
    nginx_pid=$(cat /var/run/nginx.pid)
    if kill -0 $nginx_pid 2>/dev/null; then
        exit 0                          # Nginx 正常
    fi
fi
# Nginx 异常，尝试重启
systemctl restart nginx
sleep 2
if ! killall -0 nginx 2>/dev/null; then
    systemctl stop keepalived          # 重启失败，停止 Keepalived，触发 VIP 漂移
fi
```

#### 101.3.4 通知脚本

```bash
#!/bin/bash
# /etc/keepalived/notify.sh
STATE=$1
VIP=192.168.1.100

case $STATE in
    master)
        logger "Keepalived: became MASTER, VIP=$VIP"
        # 可选：发送告警
        curl -s "https://qyapi.weixin.com/cgi-bin/webhook?key=xxx" \
             -H 'Content-Type: application/json' \
             -d "{\"msgtype\":\"text\",\"text\":{\"content\":\"Nginx 主节点切换，新 Master: $(hostname)\"}}"
        ;;
    backup)
        logger "Keepalived: became BACKUP"
        ;;
    fault)
        logger "Keepalived: FAULT detected"
        ;;
esac
```

#### 101.3.5 Nginx 配置（两台一致）

```nginx
upstream backend {
    server 192.168.1.20:8080 max_fails=3 fail_timeout=30s;
    server 192.168.1.21:8080 max_fails=3 fail_timeout=30s;
    server 192.168.1.22:8080 max_fails=3 fail_timeout=30s;
    keepalive 32;
}

server {
    listen 80;
    server_name www.example.com;

    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_connect_timeout 5s;
        proxy_read_timeout 30s;
        proxy_next_upstream error timeout http_500 http_502 http_503 http_504;

        proxy_cache api_cache;
        proxy_cache_valid 200 10m;
        proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
    }

    location /health {
        access_log off;
        return 200 "ok\n";
    }
}
```

### 101.4 脑裂问题与防范

**脑裂**：Master 与 Backup 因网络分区同时持有 VIP，导致客户端请求被双机同时接收，引发数据混乱。

**防范措施：**

1. **冗余心跳线**：除主网卡外，增加直连心跳线
2. **仲裁机制**：引入第三方节点（如网关/数据库）做仲裁
3. **fencing 设备**：强制关闭异常节点电源
4. **脚本检测**：检测到 Backup 也持有 VIP 时主动让出

```bash
# 防脑裂脚本：检测 Backup 是否能 ping 通 Master
#!/bin/bash
# /etc/keepalived/check_split_brain.sh
MASTER_IP=192.168.1.10
GATEWAY=192.168.1.1

# 能 ping 网关但 ping 不通 Master，说明 Master 可能还活着但网络分区
if ping -c 1 -W 1 $GATEWAY >/dev/null && ! ping -c 1 -W 1 $MASTER_IP >/dev/null; then
    logger "Split brain detected! Stopping keepalived."
    systemctl stop keepalived
fi
```

### 101.5 故障演练

```bash
# 1. 模拟 Master Nginx 宕机
ssh root@192.168.1.10 "systemctl stop nginx"

# 2. 观察 VIP 漂移
ip addr show eth0                     # 在 Backup 上看到 VIP

# 3. 验证业务可用性
curl http://www.example.com/          # 应正常响应

# 4. 恢复 Master Nginx
ssh root@192.168.1.10 "systemctl start nginx"

# 5. 观察 VIP 是否抢占（依赖 priority 配置）
```

### 101.6 运维最佳实践

1. **配置一致性**：两台 Nginx 配置完全一致，建议用 Ansible/Puppet 统一管理
2. **日志集中**：日志统一采集到 ELK，便于排查切换问题
3. **监控告警**：监控 VIP 状态、Keepalived 进程、Nginx 存活，切换即告警
4. **定期演练**：每月一次故障切换演练，确保高可用真正可用
5. **灰度发布**：先停 Backup，升级 Master，再切换 VIP，升级 Backup
6. **容量规划**：单台 Nginx 需能独立承担全量流量，否则故障切换后雪崩

---

## 附录：常用命令速查

```bash
nginx -t              # 测试配置
nginx -T              # 打印完整生效配置
nginx -s reload       # 平滑重载
nginx -s stop         # 快速停止
nginx -s quit         # 优雅停止
nginx -V              # 查看编译参数
nginx -v              # 查看版本

# 平滑升级
kill -USR2 $(cat /var/run/nginx.pid)    # 启动新 Master
kill -WINCH $(cat /var/run/nginx.pid.oldbin)  # 优雅关闭旧 Worker
kill -QUIT $(cat /var/run/nginx.pid.oldbin)   # 退出旧 Master

# 日志切割
mv access.log access.log.$(date +%Y%m%d)
kill -USR1 $(cat /var/run/nginx.pid)

# 查看连接数
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'

# 查看 Nginx 状态（需 stub_status 模块）
curl http://localhost/nginx_status
```

## 附录：核心配置模板

```nginx
user nginx;
worker_processes auto;
worker_rlimit_nofile 65535;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    use epoll;
    worker_connections 10240;
    multi_accept on;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for" $request_time';
    access_log /var/log/nginx/access.log main;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    keepalive_requests 1000;
    client_max_body_size 20m;

    gzip on;
    gzip_min_length 1k;
    gzip_comp_level 6;
    gzip_types text/plain text/css application/javascript application/json;

    open_file_cache max=10000 inactive=60s;
    open_file_cache_valid 30s;
    open_file_cache_min_uses 2;

    include /etc/nginx/conf.d/*.conf;
}
```
