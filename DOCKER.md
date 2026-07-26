# Docker 核心技术手册

   **阅读指南**：由浅入深，每个知识点均配有精简示例。建议按顺序阅读，环环相扣。

   ---

## 目录

1. [Docker基础概念](#1-docker基础概念)
   - 1.1 [Docker的核心概念](#11-docker的核心概念)

2. [Docker安装与配置](#2-docker安装与配置)
   - 2.1 [Docker的安装与配置](#21-docker的安装与配置)
   - 2.2 [Docker配置镜像源](#22-docker配置镜像源)

3. [Docker核心命令](#3-docker核心命令)
   - 3.1 [Docker核心命令_镜像命令](#31-docker核心命令_镜像命令)
   - 3.2 [Docker核心命令_容器命令](#32-docker核心命令_容器命令)
   - 3.3 [Docker核心命令_其他命令](#33-docker核心命令_其他命令)

4. [Docker实战部署](#4-docker实战部署)
   - 4.1 [Docker实战_Java环境](#41-docker实战_java环境)
   - 4.2 [Docker实战_安装Tomcat](#42-docker实战_安装tomcat)
   - 4.3 [Docker实战_MySQL数据库](#43-docker实战_mysql数据库)

5. [Docker数据管理](#5-docker数据管理)
   - 5.1 [Docker数据管理_什么是数据卷](#51-docker数据管理_什么是数据卷)
   - 5.2 [Docker数据管理_配置数据卷](#52-docker数据管理_配置数据卷)
   - 5.3 [Docker数据管理_容器数据卷Volume](#53-docker数据管理_容器数据卷volume)
   - 5.4 [Docker实战_MySQL数据持久化](#54-docker实战_mysql数据持久化)

6. [Dockerfile完全指南](#6-dockerfile完全指南)
   - 6.1 [Dockerfile完全指南_什么是Dockerfile](#61-dockerfile完全指南_什么是dockerfile)
   - 6.2 [Dockerfile完全指南_构建镜像](#62-dockerfile完全指南_构建镜像)
   - 6.3 [Dockerfile完全指南_常见的13种指令上](#63-dockerfile完全指南_常见的13种指令上)
   - 6.4 [Dockerfile完全指南_常见的13种指令下](#64-dockerfile完全指南_常见的13种指令下)
   - 6.5 [Dockerfile完全指南_CMD和ENTRYPOINT的区别](#65-dockerfile完全指南_cmd和entrypoint的区别)
   - 6.6 [Dockerfile综合案例_构建Tomcat镜像](#66-dockerfile综合案例_构建tomcat镜像)

7. [Docker网络管理](#7-docker网络管理)
   - 7.1 [Docker网络管理_Docker0详解](#71-docker网络管理_docker0详解)
   - 7.2 [Docker网络管理_容器互联](#72-docker网络管理_容器互联)
   - 7.3 [Docker网络管理_四种网络模式](#73-docker网络管理_四种网络模式)
   - 7.4 [Docker网络管理_自定义网络](#74-docker网络管理_自定义网络)

8. [Docker仓库管理](#8-docker仓库管理)
   - 8.1 [Docker仓库管理_为什么推送镜像到远程仓库](#81-docker仓库管理_为什么推送镜像到远程仓库)
   - 8.2 [Docker仓库管理_发布镜像到DockerHub](#82-docker仓库管理_发布镜像到dockerhub)
   - 8.3 [Docker仓库管理_发布镜像到阿里云](#83-docker仓库管理_发布镜像到阿里云)

9. [Docker核心技术原理](#9-docker核心技术原理)
   - 9.1 [Docker核心技术_基础架构](#91-docker核心技术_基础架构)
   - 9.2 [Docker核心技术_联合文件系统](#92-docker核心技术_联合文件系统)

---

## 1 Docker基础概念

### 1.1 Docker的核心概念

Docker是一个开源的容器化平台，通过容器技术实现应用的快速部署、运行和管理。核心概念包括：

- **镜像(Image)**：只读的模板，包含运行应用所需的所有文件、配置和依赖
- **容器(Container)**：镜像的运行实例，可独立运行的应用环境
- **仓库(Repository)**：存放和管理镜像的场所

**代码示例：查看本地镜像**
```bash
# 列出本地所有镜像
docker images
```

**代码示例：查看运行中容器**
```bash
# 列出正在运行的容器
docker ps
# 列出所有容器（包括停止的）
docker ps -a
```

---

## 2 Docker安装与配置

### 2.1 Docker的安装与配置

Docker支持Windows、Linux和macOS三大平台。安装完成后，通过以下命令验证：

**代码示例：验证Docker安装**
```bash
# 查看Docker版本信息
docker version
# 运行测试容器
docker run hello-world
```

### 2.2 Docker配置镜像源

由于国内网络环境，建议配置国内镜像源加速下载。

**代码示例：配置Docker镜像源（Linux）**
```bash
# 创建配置文件目录
mkdir -p /etc/docker
# 写入镜像源配置
cat > /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://registry.cn-hangzhou.aliyuncs.com"
  ]
}
EOF
# 重启Docker服务
systemctl restart docker
```

**代码示例：验证镜像源配置**
```bash
# 查看Docker配置信息
docker info | grep -A5 "Registry Mirrors"
```

---

## 3 Docker核心命令

**命令与目录的关系：**
- `docker pull`、`docker run`、`docker ps`、`docker images` 等命令与当前工作目录无关，可在任意目录执行
- `docker build` 需要在Dockerfile所在目录或通过 `-f` 参数指定路径
- `docker compose` 默认在当前目录查找 `docker-compose.yml`，也可用 `-f` 指定文件绝对路径

### 3.1 Docker核心命令_镜像命令

镜像命令用于管理Docker镜像的下载、构建、删除等操作。

**代码示例：镜像操作命令**
```bash
# 搜索镜像
docker search nginx
# 下载镜像（指定版本）
docker pull nginx:1.24
# 查看本地镜像
docker images
# 删除镜像
docker rmi nginx:1.24
# 保存镜像到本地文件
docker save -o nginx.tar nginx:1.24
# 从本地文件加载镜像
docker load -i nginx.tar
```

### 3.2 Docker核心命令_容器命令

容器命令用于管理容器的创建、启动、停止、删除等操作。

**代码示例：容器操作命令**
```bash
# 创建并运行容器（后台运行）
docker run -d --name mynginx -p 80:80 nginx:1.24
# 查看容器日志
docker logs -f mynginx
# 进入容器内部
docker exec -it mynginx /bin/bash
# 停止容器
docker stop mynginx
# 启动容器
docker start mynginx
# 删除容器（需先停止）
docker rm mynginx
# 强制删除运行中的容器
docker rm -f mynginx
```

### 3.3 Docker核心命令_其他命令

**代码示例：其他常用命令**
```bash
# 查看Docker系统信息
docker info
# 查看容器资源使用情况
docker stats
# 查看容器详细信息
docker inspect mynginx
# 清理未使用的资源
docker system prune -a
```

---

## 4 Docker实战部署

### 4.1 Docker实战_Java环境

**代码示例：运行Java环境容器**
```bash
# 拉取Java 8镜像
docker pull openjdk:8-jdk-alpine
# 运行Java容器并执行jar包
docker run -d -v /app:/app openjdk:8-jdk-alpine \
  java -jar /app/myapp.jar
```

### 4.2 Docker实战_安装Tomcat

**代码示例：部署Tomcat容器**
```bash
# 拉取Tomcat镜像
docker pull tomcat:9.0
# 运行Tomcat，映射端口和目录
docker run -d --name mytomcat \
  -p 8080:8080 \
  -v /webapps:/usr/local/tomcat/webapps \
  tomcat:9.0
```

### 4.3 Docker实战_MySQL数据库

**代码示例：部署MySQL容器**
```bash
# 拉取MySQL 8镜像
docker pull mysql:8.0
# 运行MySQL容器
docker run -d --name mymysql \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=123456 \
  mysql:8.0
```

---

## 5 Docker数据管理

### 5.1 Docker数据管理_什么是数据卷

数据卷(Volume)是Docker提供的持久化存储机制，用于在容器之间共享数据或保留容器数据。

**核心特点：**
- 数据卷独立于容器生命周期，容器删除后数据卷仍保留
- 支持容器间共享和重用
- 支持备份、恢复和迁移

### 5.2 Docker数据管理_配置数据卷

**代码示例：创建和管理数据卷**
```bash
# 创建数据卷
docker volume create mydata
# 查看数据卷列表
docker volume ls
# 查看数据卷详细信息
docker volume inspect mydata
# 删除数据卷
docker volume rm mydata
```

### 5.3 Docker数据管理_容器数据卷Volume

**代码示例：使用数据卷运行容器**
```bash
# 使用数据卷运行MySQL
docker run -d --name mymysql \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -v mydata:/var/lib/mysql \
  mysql:8.0
```

**代码示例：使用宿主机目录挂载**
```bash
# 挂载宿主机目录到容器
docker run -d --name mynginx \
  -p 80:80 \
  -v /host/html:/usr/share/nginx/html \
  nginx:1.24
```

### 5.4 Docker实战_MySQL数据持久化

**代码示例：MySQL完整持久化配置**
```bash
# 创建专用数据卷
docker volume create mysql-data
# 运行MySQL，数据持久化到数据卷
docker run -d --name mysql-db \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=mydb \
  -v mysql-data:/var/lib/mysql \
  --restart=always \
  mysql:8.0
```

---

## 6 Dockerfile完全指南

### 6.1 Dockerfile完全指南_什么是Dockerfile

Dockerfile是一个文本文件，包含一系列指令，用于自动化构建Docker镜像。

**核心优势：**
- 代码化描述镜像构建过程
- 版本控制友好
- 可重复构建

### 6.2 Dockerfile完全指南_构建镜像

**代码示例：构建镜像命令**
```bash
# 在Dockerfile所在目录构建
docker build -t myapp:1.0 .
# 指定Dockerfile路径
docker build -t myapp:1.0 -f /path/to/Dockerfile .
```

### 6.3 Dockerfile完全指南_常见的13种指令上

**代码示例：基础指令**
```dockerfile
# FROM：指定基础镜像
FROM openjdk:8-jdk-alpine

# MAINTAINER：维护者信息（已废弃，推荐LABEL）
MAINTAINER "admin@example.com"

# LABEL：添加元数据
LABEL version="1.0"
LABEL description="My Application"

# RUN：执行命令
RUN apk add --no-cache curl

# COPY：复制文件到容器
COPY target/myapp.jar /app/

# ADD：复制文件（支持URL和解压）
ADD https://example.com/file.tar.gz /tmp/
```

### 6.4 Dockerfile完全指南_常见的13种指令下

**代码示例：高级指令**
```dockerfile
# ENV：设置环境变量
ENV JAVA_OPTS="-Xms256m -Xmx512m"

# EXPOSE：声明端口
EXPOSE 8080

# WORKDIR：设置工作目录
WORKDIR /app

# USER：指定运行用户
USER appuser

# VOLUME：声明数据卷
VOLUME /data

# CMD：容器启动命令（可被覆盖）
CMD ["java", "-jar", "myapp.jar"]

# ENTRYPOINT：容器入口点
ENTRYPOINT ["java", "-jar"]
```

### 6.5 Dockerfile完全指南_CMD和ENTRYPOINT的区别

| 特性 | CMD | ENTRYPOINT |
|------|-----|------------|
| 作用 | 设置默认命令和参数 | 设置容器启动的主命令 |
| 覆盖方式 | `docker run` 后接参数覆盖 | `docker run --entrypoint` 覆盖 |
| 组合使用 | 作为ENTRYPOINT的参数 | 作为主命令，CMD作为默认参数 |

**代码示例：CMD和ENTRYPOINT组合使用**
```dockerfile
# ENTRYPOINT指定主命令，CMD指定默认参数
ENTRYPOINT ["java", "-jar"]
CMD ["myapp.jar"]
```

### 6.6 Dockerfile综合案例_构建Tomcat镜像

**代码示例：完整Dockerfile**
```dockerfile
# 基于官方Tomcat镜像
FROM tomcat:9.0

# 设置维护者信息
LABEL maintainer="admin@example.com"

# 复制war包到webapps目录
COPY target/myapp.war /usr/local/tomcat/webapps/

# 设置环境变量
ENV CATALINA_OPTS="-Xms512m -Xmx1024m"

# 暴露端口
EXPOSE 8080

# 启动Tomcat
CMD ["catalina.sh", "run"]
```

---

## 7 Docker网络管理

### 7.1 Docker网络管理_Docker0详解

Docker0是Docker默认创建的虚拟网桥，所有容器默认连接到此网络。

**核心特性：**
- Docker自动分配IP地址（默认172.17.0.0/16网段）
- 容器间可通过IP互相访问
- 宿主机可通过docker0与容器通信

**代码示例：查看Docker网络**
```bash
# 查看Docker网络列表
docker network ls
# 查看docker0详细信息
docker network inspect bridge
```

### 7.2 Docker网络管理_容器互联

**代码示例：容器间通信**
```bash
# 创建两个容器并连接到同一网络
docker run -d --name app1 --network bridge nginx
docker run -d --name app2 --network bridge nginx
# 在app1中ping app2（容器名可解析）
docker exec app1 ping app2
```

### 7.3 Docker网络管理_四种网络模式

| 模式 | 说明 | 应用场景 |
|------|------|----------|
| bridge | 默认模式，容器连接到docker0 | 容器间通信 |
| host | 共享宿主机网络栈 | 性能敏感场景 |
| none | 无网络连接 | 安全隔离场景 |
| container | 共享另一个容器的网络 | 容器间紧密协作 |

**代码示例：使用不同网络模式**
```bash
# bridge模式（默认）
docker run -d --network bridge nginx

# host模式
docker run -d --network host nginx

# none模式
docker run -d --network none nginx
```

### 7.4 Docker网络管理_自定义网络

**代码示例：创建自定义网络**
```bash
# 创建自定义桥接网络
docker network create --driver bridge mynet
# 查看自定义网络信息
docker network inspect mynet
# 运行容器使用自定义网络
docker run -d --name myapp --network mynet nginx
# 连接已有容器到自定义网络
docker network connect mynet existing-container
```

---

## 8 Docker仓库管理

### 8.1 Docker仓库管理_为什么推送镜像到远程仓库

**核心原因：**
- 跨环境共享镜像（开发→测试→生产）
- 团队协作，统一镜像版本
- 备份和灾备

### 8.2 Docker仓库管理_发布镜像到DockerHub

**代码示例：发布到DockerHub**
```bash
# 登录DockerHub
docker login
# 标记镜像（格式：用户名/仓库名:标签）
docker tag myapp:1.0 username/myapp:1.0
# 推送镜像
docker push username/myapp:1.0
# 拉取镜像
docker pull username/myapp:1.0
```

### 8.3 Docker仓库管理_发布镜像到阿里云

**代码示例：发布到阿里云镜像仓库**
```bash
# 登录阿里云容器镜像服务
docker login registry.cn-hangzhou.aliyuncs.com
# 标记镜像
docker tag myapp:1.0 registry.cn-hangzhou.aliyuncs.com/namespace/myapp:1.0
# 推送镜像
docker push registry.cn-hangzhou.aliyuncs.com/namespace/myapp:1.0
```

---

## 9 Docker核心技术原理

### 9.1 Docker核心技术_基础架构

Docker采用客户端-服务器架构：

- **Docker Client**：命令行工具，与Docker Daemon通信
- **Docker Daemon**：后台服务，管理镜像、容器、网络等
- **Docker Registry**：镜像仓库，存储和分发镜像

**核心组件：**
- containerd：容器运行时，管理容器生命周期
- runc：OCI兼容的容器运行时
- CRI-O：Kubernetes兼容的容器运行时

### 9.2 Docker核心技术_联合文件系统

联合文件系统(UnionFS)是Docker镜像分层存储的核心技术。

**核心特点：**
- 分层存储，每层只读
- 容器在镜像层之上添加可写层
- 多个容器共享基础镜像层，节省空间

**镜像层结构：**
```
┌─────────────────────────────────┐
│      容器可写层（Container）     │
├─────────────────────────────────┤
│  镜像层3（应用代码/配置）        │
├─────────────────────────────────┤
│  镜像层2（依赖库）               │
├─────────────────────────────────┤
│  镜像层1（操作系统基础）         │
├─────────────────────────────────┤
│  镜像层0（基础镜像）             │
└─────────────────────────────────┘
```

**代码示例：查看镜像分层**
```bash
# 查看镜像分层信息
docker inspect nginx:1.24 | grep -A10 "RootFS"
# 查看镜像历史
docker history nginx:1.24
```