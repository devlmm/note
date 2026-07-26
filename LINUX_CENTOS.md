# Linux CentOS 运维核心技术手册

**阅读指南**：由浅入深，每个知识点均配有精简示例。建议按顺序阅读，环环相扣。

---

## 目录

### 第一章：基础入门
- [1.1 Linux下载安装](#11-linux下载安装)
- [1.2 Linux三种网络模式](#12-linux三种网络模式)
- [1.3 Linux远程登录](#13-linux远程登录)
- [1.4 Linux系统目录结构](#14-linux系统目录结构)

### 第二章：用户与权限
- [2.1 Linux用户和用户组](#21-linux用户和用户组)
- [2.2 Linux用户管理](#22-linux用户管理)
- [2.3 Linux用户组管理](#23-linux用户组管理)
- [2.4 Linux超级用户和伪用户](#24-linux超级用户和伪用户)
- [2.5 Linux文件基本属性](#25-linux文件基本属性)
- [2.6 Linux权限字与权限操作](#26-linux权限字与权限操作)

### 第三章：文件系统
- [3.1 Linux路径](#31-linux路径)
- [3.2 Linux目录文件操作常用命令](#32-linux目录文件操作常用命令)
- [3.3 Linux文件编辑工具vim](#33-linux文件编辑工具vim)
- [3.4 Linux文件内容查看命令](#34-linux文件内容查看命令)
- [3.5 Linux打包压缩与搜索命令](#35-linux打包压缩与搜索命令)

### 第四章：系统管理
- [4.1 Linux常用系统工作命令](#41-linux常用系统工作命令)
- [4.2 Linux重定向、管道符和环境变量](#42-linux重定向管道符和环境变量)
- [4.3 Linux磁盘管理](#43-linux磁盘管理)
- [4.4 Linux挂载硬盘](#44-linux挂载硬盘)
- [4.5 Linux系统状态检测命令](#45-linux系统状态检测命令)

### 第五章：软件与服务
- [5.1 Linux软件安装命令](#51-linux软件安装命令)
- [5.2 Linux常用软件安装_JDK和Tomcat安装](#52-linux常用软件安装_jdk和tomcat安装)
- [5.3 Linux常用软件安装_Mysql数据库安装](#53-linux常用软件安装_mysql数据库安装)
- [5.4 Linux常用软件安装_Mysql数据库卸载](#54-linux常用软件安装_mysql数据库卸载)
- [5.5 Linux进程管理](#55-linux进程管理)
- [5.6 Linux系统服务](#56-linux系统服务)

### 第六章：高级运维
- [6.1 Linux定时任务](#61-linux定时任务)
- [6.2 Linux网络防火墙](#62-linux网络防火墙)
- [6.3 Linux内核机制](#63-linux内核机制)
- [6.4 SSH密钥认证](#64-ssh密钥认证)
- [6.5 SELinux基础](#65-selinux基础)
- [6.6 日志管理](#66-日志管理)
- [6.7 网络配置](#67-网络配置)

### 第七章：Shell脚本
- [7.1 Shell脚本入门](#71-shell脚本入门)
- [7.2 Shell变量_系统预定义变量](#72-shell变量_系统预定义变量)
- [7.3 Shell变量_用户自定义变量](#73-shell变量_用户自定义变量)
- [7.4 Shell变量_只读变量和撤销变量](#74-shell变量_只读变量和撤销变量)
- [7.5 Shell变量_特殊变量](#75-shell变量_特殊变量)
- [7.6 Shell_运算符](#76-shell_运算符)
- [7.7 Shell_条件判断](#77-shell_条件判断)
- [7.8 Shell_流程控制](#78-shell_流程控制)
- [7.9 Shell_读取控制台输入](#79-shell_读取控制台输入)
- [7.10 Shell函数_系统函数](#710-shell函数_系统函数)
- [7.11 Shell函数_自定义函数](#711-shell函数_自定义函数)
- [7.12 Shell综合应用案例_归档文件](#712-shell综合应用案例_归档文件)
- [7.13 Shell综合应用案例_定时归档文件](#713-shell综合应用案例_定时归档文件)
- [7.14 Shell_正则表达式](#714-shell_正则表达式)
- [7.15 Shell文本处理工具_cut](#715-shell文本处理工具_cut)
- [7.16 Shell文本处理工具_awk](#716-shell文本处理工具_awk)

---

## 第一章：基础入门

### 1.1 Linux下载安装

**镜像下载地址：**
- 官方地址：[centos.org](https://www.centos.org/)
- 国内镜像：[mirrors.aliyun.com/centos](https://mirrors.aliyun.com/centos/)

**最小化安装命令行（推荐生产环境）：**
```bash
# 安装过程中选择"最小安装"，仅安装必要组件
# 安装完成后更新系统
yum update -y
```

**安装常用工具：**
```bash
# 安装基础工具包
yum install -y vim wget net-tools tree
```

### 1.2 Linux三种网络模式

| 模式 | 说明 | 适用场景 |
|------|------|----------|
| **桥接模式** | 虚拟机与宿主机在同一网段，独立IP | 需要虚拟机对外提供服务 |
| **NAT模式** | 虚拟机通过宿主机上网，内网IP | 日常开发、学习 |
| **仅主机模式** | 虚拟机与宿主机组成私有网络 | 网络隔离测试 |

**查看网络配置：**
```bash
# 查看网卡信息
ip addr show

# 查看路由表
ip route
```

### 1.3 Linux远程登录

**使用SSH登录：**
```bash
# 基本格式：ssh 用户名@IP地址
ssh root@192.168.1.100

# 指定端口登录（非默认22端口）
ssh -p 2222 root@192.168.1.100
```

**SSH配置优化（安全加固）：**
```bash
# 编辑SSH配置文件
vim /etc/ssh/sshd_config

# 修改配置项（推荐）
Port 2222          # 修改默认端口
PermitRootLogin no # 禁止root远程登录
PasswordAuthentication no # 禁用密码登录（启用密钥认证后）
```

**重启SSH服务：**
```bash
systemctl restart sshd
systemctl enable sshd
```

### 1.4 Linux系统目录结构

**核心目录说明：**

| 目录 | 说明 |
|------|------|
| `/` | 根目录，所有文件的起点 |
| `/bin` | 普通用户可执行的命令 |
| `/sbin` | 管理员可执行的系统命令 |
| `/etc` | 系统配置文件目录 |
| `/home` | 用户主目录 |
| `/root` | 超级用户root的主目录 |
| `/var` | 可变数据（日志、缓存等） |
| `/tmp` | 临时文件目录 |
| `/usr` | 用户应用程序目录 |

**查看目录结构：**
```bash
# 查看根目录结构
ls -la /

# 树形显示目录结构
tree / -L 2
```

---

## 第二章：用户与权限

### 2.1 Linux用户和用户组

**用户分类：**
- **超级用户（root）**：UID=0，拥有最高权限
- **普通用户**：UID=1000+，受权限限制
- **系统用户**：UID=1-999，用于运行系统服务

**用户组：**
- 每个用户至少属于一个主组（primary group）
- 可以属于多个附属组（secondary group）

**查看用户信息：**
```bash
# 查看当前用户
whoami

# 查看用户所属组
groups

# 查看用户详细信息
id username
```

### 2.2 Linux用户管理

**创建用户：**
```bash
# 创建用户（自动创建同名主组）
useradd alice

# 创建用户并指定主目录
useradd -d /home/alice alice

# 创建用户并指定UID和主组
useradd -u 1001 -g developers alice
```

**设置密码：**
```bash
# 设置用户密码
passwd alice

# 强制用户下次登录修改密码
passwd -e alice
```

**修改用户属性：**
```bash
# 修改用户主目录
usermod -d /home/new_alice alice

# 添加用户到附属组
usermod -aG wheel alice

# 锁定用户
usermod -L alice

# 解锁用户
usermod -U alice
```

**删除用户：**
```bash
# 删除用户（保留主目录）
userdel alice

# 删除用户及主目录
userdel -r alice
```

### 2.3 Linux用户组管理

**创建用户组：**
```bash
# 创建用户组
groupadd developers

# 创建用户组并指定GID
groupadd -g 2000 developers
```

**修改用户组：**
```bash
# 修改用户组名称
groupmod -n devs developers

# 修改用户组GID
groupmod -g 2001 developers
```

**删除用户组：**
```bash
# 删除用户组（必须为空组）
groupdel developers
```

**管理组成员：**
```bash
# 将用户添加到组
gpasswd -a alice developers

# 将用户从组移除
gpasswd -d alice developers

# 设置组管理员
gpasswd -A alice developers
```

### 2.4 Linux超级用户和伪用户

**超级用户（root）：**
```bash
# 切换到root用户
su - root

# 执行单条命令（临时获取root权限）
sudo command

# 配置sudo权限（编辑sudoers文件）
visudo
```

**伪用户（系统用户）：**
- `bin`、`daemon`、`adm`、`nobody`等
- 用于运行系统服务，无登录权限
- 禁止修改或删除这些用户

**查看系统用户：**
```bash
# 查看所有用户
cat /etc/passwd

# 查看所有用户组
cat /etc/group
```

### 2.5 Linux文件基本属性

**文件属性详解：**
```bash
# 查看文件属性示例
ls -la

# 输出格式：权限 链接数 所有者 所属组 大小 时间 名称
# -rw-r--r-- 1 root root 1024 Jul 1 10:00 file.txt
```

**属性字段说明：**
| 字段 | 说明 |
|------|------|
| `-rw-r--r--` | 文件类型和权限 |
| `1` | 硬链接数 |
| `root` | 文件所有者 |
| `root` | 所属组 |
| `1024` | 文件大小（字节） |
| `Jul 1 10:00` | 修改时间 |
| `file.txt` | 文件名 |

**文件类型标识：**
- `-`：普通文件
- `d`：目录
- `l`：软链接（符号链接）
- `b`：块设备
- `c`：字符设备
- `s`：套接字文件
- `p`：管道文件

### 2.6 Linux权限字与权限操作

**权限分类：**
| 权限 | 符号 | 数字 | 说明 |
|------|------|------|------|
| 读 | r | 4 | 查看文件内容 |
| 写 | w | 2 | 修改文件内容 |
| 执行 | x | 1 | 运行程序/进入目录 |

**三类权限主体：**
- **所有者（u）**：文件拥有者
- **所属组（g）**：文件所属组的成员
- **其他用户（o）**：系统中其他所有用户

**权限操作命令：**
```bash
# 为所有者添加执行权限
chmod u+x file.txt

# 为所属组添加读写权限
chmod g+rw file.txt

# 移除其他用户的所有权限
chmod o-rwx file.txt

# 使用数字设置权限（755：所有者rwx，组和其他rx）
chmod 755 file.txt

# 设置目录及所有子文件权限
chmod -R 755 /path/to/dir
```

**修改所有者和所属组：**
```bash
# 修改文件所有者
chown alice file.txt

# 修改文件所属组
chgrp developers file.txt

# 同时修改所有者和所属组
chown alice:developers file.txt

# 递归修改目录权限
chown -R alice:developers /path/to/dir
```

**特殊权限：**
```bash
# 设置SUID（执行时继承文件所有者权限）
chmod u+s /usr/bin/passwd

# 设置SGID（目录下新文件继承目录所属组）
chmod g+s /var/www/html

# 设置粘滞位（只有所有者能删除自己的文件）
chmod o+t /tmp
```

---

## 第三章：文件系统

### 3.1 Linux路径

**绝对路径与相对路径：**
```bash
# 绝对路径：从根目录开始的完整路径
cd /home/alice/documents

# 相对路径：相对于当前目录的路径
cd documents/reports

# 当前目录
pwd

# 返回上一级目录
cd ..

# 返回当前用户主目录
cd ~

# 返回上次所在目录
cd -
```

**路径通配符：**
```bash
# 匹配任意字符
ls *.txt

# 匹配单个字符
ls file?.txt

# 匹配多个字符中的一个
ls file[1-3].txt

# 递归查找
find /home -name "*.log"
```

### 3.2 Linux目录文件操作常用命令

**目录操作：**
```bash
# 创建目录
mkdir project

# 创建多级目录
mkdir -p project/src/main

# 切换目录
cd /path/to/dir

# 查看当前目录
pwd

# 删除空目录
rmdir empty_dir

# 删除目录及内容
rm -rf directory
```

**文件操作：**
```bash
# 创建空文件
touch newfile.txt

# 创建多个文件
touch file1.txt file2.txt

# 复制文件
cp source.txt dest.txt

# 复制目录
cp -r /path/to/source /path/to/dest

# 移动文件
mv oldname.txt newname.txt

# 移动目录
mv /path/old /path/new

# 删除文件
rm file.txt

# 删除前确认
rm -i file.txt

# 删除多个文件
rm file1.txt file2.txt
```

**链接文件：**
```bash
# 创建硬链接（同一文件的多个入口）
ln source.txt hardlink.txt

# 创建软链接（快捷方式）
ln -s /path/to/source /path/to/link

# 查看链接目标
ls -la
```

### 3.3 Linux文件编辑工具vim

**vim三种模式：**
| 模式 | 说明 | 进入方式 |
|------|------|----------|
| **命令模式** | 执行命令、移动光标 | 默认进入 |
| **编辑模式** | 输入文本 | 按 i/I/a/A/o/O |
| **末行模式** | 保存、退出、查找等 | 按 : |

**命令模式常用操作：**
```bash
# 移动光标
h j k l          # 左 下 上 右
gg               # 跳转到文件开头
G                # 跳转到文件结尾
:n               # 跳转到第n行

# 删除操作
dd               # 删除当前行
ndd              # 删除n行
x                # 删除光标所在字符
dw               # 删除当前单词

# 复制粘贴
yy               # 复制当前行
nyy              # 复制n行
p                # 粘贴到光标下方
P                # 粘贴到光标上方

# 撤销与恢复
u                # 撤销
Ctrl+r           # 恢复
```

**末行模式常用命令：**
```bash
# 保存退出
:w               # 保存
:q               # 退出
:wq              # 保存并退出
:q!              # 强制退出（不保存）

# 搜索替换
:/pattern        # 向下搜索
:?pattern        # 向上搜索
:%s/old/new/g    # 全局替换
:1,10s/old/new/g # 替换第1-10行

# 显示行号
:set nu          # 显示行号
:set nonu        # 隐藏行号
```

**编辑模式进入方式：**
```bash
i                # 在光标前插入
I                # 在行首插入
a                # 在光标后插入
A                # 在行尾插入
o                # 在下方新建一行
O                # 在上方新建一行
```

**vim配置：**
```bash
# 编辑vim配置文件
vim ~/.vimrc

# 添加常用配置
set nu           # 显示行号
set tabstop=4    # Tab宽度为4
set shiftwidth=4 # 自动缩进宽度为4
set autoindent   # 自动缩进
set hlsearch     # 高亮搜索结果
```

### 3.4 Linux文件内容查看命令

**cat命令（查看文件内容）：**
```bash
# 查看文件内容
cat file.txt

# 查看文件并显示行号
cat -n file.txt

# 查看多个文件
cat file1.txt file2.txt

# 创建文件（EOF结束）
cat > newfile.txt <<EOF
这是文件内容
第二行内容
EOF
```

**more命令（分页查看）：**
```bash
# 分页查看文件
more file.txt

# 操作键：空格翻页，Enter换行，q退出
```

**less命令（增强分页查看）：**
```bash
# 分页查看文件（支持上下滚动）
less file.txt

# 操作键：方向键滚动，/搜索，q退出
```

**head命令（查看文件开头）：**
```bash
# 查看前10行（默认）
head file.txt

# 查看前20行
head -n 20 file.txt

# 查看前100字节
head -c 100 file.txt
```

**tail命令（查看文件结尾）：**
```bash
# 查看后10行（默认）
tail file.txt

# 查看后20行
tail -n 20 file.txt

# 实时监控文件变化
tail -f /var/log/messages

# 持续监控并显示行数
tail -n 50 -f /var/log/messages
```

**grep命令（搜索内容）：**
```bash
# 在文件中搜索关键字
grep "error" log.txt

# 搜索并显示行号
grep -n "error" log.txt

# 忽略大小写搜索
grep -i "Error" log.txt

# 显示匹配行的前后各5行
grep -C 5 "error" log.txt

# 递归搜索目录
grep -r "error" /var/log/
```

### 3.5 Linux打包压缩与搜索命令

**tar打包命令：**
```bash
# 创建tar包
tar -cvf archive.tar /path/to/files

# 创建tar.gz压缩包
tar -zcvf archive.tar.gz /path/to/files

# 创建tar.bz2压缩包
tar -jcvf archive.tar.bz2 /path/to/files

# 查看压缩包内容
tar -tvf archive.tar.gz

# 解压tar包
tar -xvf archive.tar

# 解压到指定目录
tar -zxvf archive.tar.gz -C /path/to/dest
```

**zip压缩命令：**
```bash
# 创建zip压缩包
zip archive.zip file1.txt file2.txt

# 递归压缩目录
zip -r archive.zip /path/to/dir

# 解压zip包
unzip archive.zip

# 解压到指定目录
unzip archive.zip -d /path/to/dest
```

**find搜索命令：**
```bash
# 按名称搜索文件
find / -name "*.log"

# 按大小搜索（大于100MB）
find / -size +100M

# 按类型搜索（f文件，d目录）
find / -type f -name "*.txt"

# 按时间搜索（最近7天修改的文件）
find / -mtime -7

# 搜索并执行命令
find / -name "*.tmp" -exec rm {} \;

# 搜索并删除（安全方式）
find / -name "*.tmp" -delete
```

**locate搜索命令（快速搜索）：**
```bash
# 更新数据库
updatedb

# 快速搜索文件
locate filename

# 不区分大小写
locate -i filename
```

**which命令（查找命令位置）：**
```bash
# 查找命令位置
which ls

# 查找命令位置及别名
which -a ls
```

**whereis命令（查找命令、源文件和手册）：**
```bash
# 查找命令相关文件
whereis ls

# 只查找命令
whereis -b ls

# 只查找手册
whereis -m ls
```

---

## 第四章：系统管理

### 4.1 Linux常用系统工作命令

**日期时间命令：**
```bash
# 查看当前时间
date

# 查看当前日期（格式：YYYY-MM-DD）
date +%Y-%m-%d

# 查看当前时间（格式：HH:MM:SS）
date +%H:%M:%S

# 设置系统时间
date -s "2024-01-15 10:30:00"

# 查看日历
cal

# 查看指定月份日历
cal 2024 12
```

**关机重启命令：**
```bash
# 立即关机
shutdown -h now

# 指定时间关机
shutdown -h 22:00

# 取消关机计划
shutdown -c

# 重启系统
reboot

# 立即重启
shutdown -r now

# 切换到单用户模式（维护模式）
init 1
```

**用户会话命令：**
```bash
# 查看当前登录用户
who

# 查看当前登录用户详细信息
w

# 查看用户登录历史
last

# 查看失败登录记录
lastb

# 切换用户
su - username

# 退出当前用户
exit
```

**系统信息命令：**
```bash
# 查看系统内核版本
uname -r

# 查看系统完整信息
uname -a

# 查看系统架构
uname -m

# 查看系统主机名
hostname

# 修改系统主机名
hostnamectl set-hostname newhostname

# 查看系统发行版信息
cat /etc/centos-release

# 查看系统启动时间
uptime

# 查看系统运行时间
uptime -p
```

**进程相关命令：**
```bash
# 查看当前进程
ps

# 查看所有进程
ps aux

# 查看进程树
pstree

# 查看进程详细信息
ps -ef

# 实时监控进程
top

# 按内存排序
top -o %MEM

# 按CPU排序
top -o %CPU

# 退出top
q
```

### 4.2 Linux重定向、管道符和环境变量

**重定向：**
```bash
# 标准输出重定向（覆盖）
command > file.txt

# 标准输出重定向（追加）
command >> file.txt

# 标准错误重定向
command 2> error.txt

# 标准输出和错误同时重定向
command > output.txt 2>&1

# 重定向到空设备（丢弃输出）
command > /dev/null

# 输入重定向
command < input.txt

# 输入输出同时重定向
command < input.txt > output.txt
```

**管道符：**
```bash
# 将前一个命令的输出作为后一个命令的输入
ps aux | grep "nginx"

# 多级管道
cat access.log | grep "404" | wc -l

# 分页查看输出
ls -la /etc | more

# 排序输出
ls -la | sort -k 5 -r

# 去重
cat file.txt | sort | uniq

# 统计行数
cat file.txt | wc -l
```

**环境变量：**
```bash
# 查看所有环境变量
env

# 查看指定环境变量
echo $PATH

# 设置环境变量（临时）
export MY_VAR="hello"

# 设置环境变量（永久）
echo 'export MY_VAR="hello"' >> ~/.bashrc
source ~/.bashrc

# 添加路径到PATH
export PATH=$PATH:/new/path

# 查看环境变量文件
cat /etc/profile
cat ~/.bashrc
cat ~/.bash_profile
```

**常用环境变量：**
| 变量 | 说明 |
|------|------|
| `PATH` | 命令搜索路径 |
| `HOME` | 用户主目录 |
| `USER` | 当前用户名 |
| `SHELL` | 当前Shell |
| `LANG` | 语言设置 |
| `PWD` | 当前工作目录 |

### 4.3 Linux磁盘管理

**磁盘查看命令：**
```bash
# 查看磁盘分区
fdisk -l

# 查看磁盘使用情况
df -h

# 查看指定目录使用情况
du -h /home

# 查看目录下各文件大小
du -h --max-depth=1 /home

# 查看inode使用情况
df -i
```

**磁盘分区命令：**
```bash
# 进入分区工具
fdisk /dev/sdb

# 分区操作命令（在fdisk中）
n               # 创建新分区
p               # 创建主分区
e               # 创建扩展分区
d               # 删除分区
w               # 保存并退出
q               # 退出不保存
```

**创建文件系统：**
```bash
# 创建ext4文件系统
mkfs.ext4 /dev/sdb1

# 创建xfs文件系统
mkfs.xfs /dev/sdb1

# 创建swap分区
mkswap /dev/sdb2

# 启用swap分区
swapon /dev/sdb2
```

### 4.4 Linux挂载硬盘

**挂载命令：**
```bash
# 创建挂载点
mkdir /mnt/data

# 挂载分区
mount /dev/sdb1 /mnt/data

# 挂载只读分区
mount -o ro /dev/sdb1 /mnt/data

# 查看已挂载分区
mount

# 卸载分区
umount /mnt/data

# 强制卸载（谨慎使用）
umount -f /mnt/data
```

**自动挂载（fstab配置）：**
```bash
# 编辑fstab文件
vim /etc/fstab

# 添加挂载项格式
/dev/sdb1    /mnt/data    ext4    defaults    0 0

# 测试fstab配置
mount -a

# 查看fstab内容
cat /etc/fstab
```

**挂载U盘/移动硬盘：**
```bash
# 查看磁盘设备
fdisk -l

# 创建挂载点
mkdir /mnt/usb

# 挂载U盘
mount /dev/sdc1 /mnt/usb

# 卸载U盘
umount /mnt/usb
```

### 4.5 Linux系统状态检测命令

**CPU状态检测：**
```bash
# 查看CPU信息
cat /proc/cpuinfo

# 查看CPU核心数
nproc

# 查看CPU使用情况
top

# 查看CPU使用率（简洁版）
mpstat
```

**内存状态检测：**
```bash
# 查看内存使用情况
free -h

# 查看内存详细信息
cat /proc/meminfo

# 查看内存占用排行
top -o %MEM

# 查看内存使用进程
ps aux --sort=-%mem | head -10
```

**网络状态检测：**
```bash
# 查看网卡信息
ip addr show

# 查看网络连接
ss -tuln

# 查看路由表
ip route

# 测试网络连通性
ping -c 4 www.baidu.com

# 测试端口连通性
telnet localhost 80

# 查看网络统计
netstat -s

# 查看活跃连接
ss -tnp
```

**系统负载检测：**
```bash
# 查看系统负载
uptime

# 查看系统负载详细信息
w

# 查看CPU和内存使用情况
top

# 查看系统负载历史
cat /proc/loadavg

# 查看磁盘I/O
iostat

# 查看进程I/O
iotop
```

**服务状态检测：**
```bash
# 查看服务状态
systemctl status nginx

# 查看所有服务状态
systemctl list-units --type=service

# 查看服务是否自启
systemctl is-enabled nginx

# 启动服务
systemctl start nginx

# 停止服务
systemctl stop nginx

# 重启服务
systemctl restart nginx

# 重新加载配置
systemctl reload nginx

# 设置服务自启
systemctl enable nginx

# 禁用服务自启
systemctl disable nginx
```

---

## 第五章：软件与服务

### 5.1 Linux软件安装命令

**YUM包管理：**
```bash
# 安装软件
yum install -y nginx

# 卸载软件
yum remove -y nginx

# 更新软件
yum update -y nginx

# 搜索软件
yum search nginx

# 查看软件信息
yum info nginx

# 列出已安装软件
yum list installed

# 清理缓存
yum clean all

# 更新缓存
yum makecache
```

**RPM包管理：**
```bash
# 安装RPM包
rpm -ivh package.rpm

# 升级RPM包
rpm -Uvh package.rpm

# 卸载RPM包
rpm -e package

# 查看已安装RPM包
rpm -qa

# 查看RPM包信息
rpm -qi package

# 查看RPM包文件列表
rpm -ql package

# 检查文件所属RPM包
rpm -qf /path/to/file
```

**源码编译安装：**
```bash
# 下载源码
wget https://example.com/package.tar.gz

# 解压源码
tar -zxvf package.tar.gz
cd package

# 配置（指定安装路径）
./configure --prefix=/usr/local/package

# 编译
make

# 安装
make install
```

**EPEL源配置：**
```bash
# 安装EPEL源
yum install -y epel-release

# 安装EPEL源的软件
yum --enablerepo=epel install nginx
```

### 5.2 Linux常用软件安装_JDK和Tomcat安装

**JDK安装：**
```bash
# 查看系统自带JDK
rpm -qa | grep java

# 卸载系统自带JDK（如果需要）
rpm -e --nodeps java-1.8.0-openjdk

# 下载JDK（官网或镜像）
wget https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.tar.gz

# 解压到指定目录
tar -zxvf jdk-17_linux-x64_bin.tar.gz -C /usr/local/

# 配置环境变量
vim /etc/profile

# 添加以下内容
export JAVA_HOME=/usr/local/jdk-17
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

# 使配置生效
source /etc/profile

# 验证安装
java -version
```

**Tomcat安装：**
```bash
# 下载Tomcat
wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.0/bin/apache-tomcat-10.1.0.tar.gz

# 解压到指定目录
tar -zxvf apache-tomcat-10.1.0.tar.gz -C /usr/local/

# 重命名
mv /usr/local/apache-tomcat-10.1.0 /usr/local/tomcat

# 启动Tomcat
/usr/local/tomcat/bin/startup.sh

# 停止Tomcat
/usr/local/tomcat/bin/shutdown.sh

# 查看日志
tail -f /usr/local/tomcat/logs/catalina.out

# 设置开机自启
echo '/usr/local/tomcat/bin/startup.sh' >> /etc/rc.d/rc.local
chmod +x /etc/rc.d/rc.local
```

**Tomcat配置：**
```bash
# 修改端口（默认8080）
vim /usr/local/tomcat/conf/server.xml

# 修改端口号
<Connector port="80" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />

# 设置管理员用户
vim /usr/local/tomcat/conf/tomcat-users.xml

# 添加用户
<user username="admin" password="password" roles="manager-gui,admin-gui"/>
```

### 5.3 Linux常用软件安装_Mysql数据库安装

**MySQL 8.0安装：**
```bash
# 下载MySQL YUM源
wget https://dev.mysql.com/get/mysql80-community-release-el8-3.noarch.rpm

# 安装YUM源
rpm -ivh mysql80-community-release-el8-3.noarch.rpm

# 安装MySQL服务器
yum install -y mysql-community-server

# 启动MySQL服务
systemctl start mysqld
systemctl enable mysqld

# 查看初始密码
grep 'temporary password' /var/log/mysqld.log

# 登录MySQL并修改密码
mysql -uroot -p
ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';

# 允许远程访问
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '密码';
FLUSH PRIVILEGES;

# 退出MySQL
exit
```

**MySQL配置：**
```bash
# 编辑MySQL配置文件
vim /etc/my.cnf

# 添加常用配置
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
max_connections=1000

# 重启MySQL服务
systemctl restart mysqld
```

**MySQL常用命令：**
```bash
# 登录MySQL
mysql -u username -p

# 创建数据库
CREATE DATABASE dbname;

# 使用数据库
USE dbname;

# 创建表
CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(50));

# 查看数据库列表
SHOW DATABASES;

# 查看表列表
SHOW TABLES;

# 查看表结构
DESCRIBE users;

# 退出MySQL
EXIT;
```

### 5.4 Linux常用软件安装_Mysql数据库卸载

**MySQL卸载：**
```bash
# 停止MySQL服务
systemctl stop mysqld
systemctl disable mysqld

# 查看已安装的MySQL包
rpm -qa | grep -i mysql

# 卸载MySQL包
yum remove -y mysql-community-server mysql-community-client mysql-community-common mysql-community-libs

# 删除MySQL数据目录
rm -rf /var/lib/mysql

# 删除MySQL配置文件
rm -rf /etc/my.cnf

# 删除MySQL日志
rm -rf /var/log/mysqld.log

# 验证卸载
rpm -qa | grep -i mysql
```

### 5.5 Linux进程管理

**进程查看：**
```bash
# 查看所有进程
ps aux

# 查看进程树
pstree

# 查看进程详细信息
ps -ef

# 实时监控进程
top

# 按CPU排序
top -o %CPU

# 按内存排序
top -o %MEM
```

**进程控制：**
```bash
# 终止进程（按PID）
kill 1234

# 强制终止进程
kill -9 1234

# 终止进程（按名称）
pkill nginx

# 强制终止进程（按名称）
pkill -9 nginx

# 终止进程树
killall nginx

# 挂起进程
kill -STOP 1234

# 恢复进程
kill -CONT 1234
```

**进程优先级：**
```bash
# 查看进程优先级
ps -o pid,ni,cmd

# 启动低优先级进程
nice -n 10 command

# 启动高优先级进程
nice -n -10 command

# 调整运行中进程优先级
renice 10 -p 1234

# 查看进程优先级范围（-20到19，数值越小优先级越高）
nice --help
```

### 5.6 Linux系统服务

**systemd服务管理：**
```bash
# 查看服务状态
systemctl status nginx

# 启动服务
systemctl start nginx

# 停止服务
systemctl stop nginx

# 重启服务
systemctl restart nginx

# 重新加载配置
systemctl reload nginx

# 查看所有服务
systemctl list-units --type=service

# 查看已启用服务
systemctl list-unit-files --type=service | grep enabled

# 设置服务自启
systemctl enable nginx

# 禁用服务自启
systemctl disable nginx

# 查看服务依赖
systemctl list-dependencies nginx

# 查看服务日志
journalctl -u nginx

# 查看服务启动失败原因
systemctl status nginx
```

**自定义systemd服务：**
```bash
# 创建服务文件
vim /etc/systemd/system/myapp.service

# 服务文件内容
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=nobody
ExecStart=/usr/local/myapp/myapp.sh
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target

# 重新加载systemd配置
systemctl daemon-reload

# 启动服务
systemctl start myapp

# 设置自启
systemctl enable myapp
```

**服务运行级别：**
```bash
# 查看当前运行级别
systemctl get-default

# 设置运行级别（多用户模式）
systemctl set-default multi-user.target

# 设置运行级别（图形界面模式）
systemctl set-default graphical.target
```

---

## 第六章：高级运维

### 6.1 Linux定时任务

**cron定时任务：**
```bash
# 查看当前用户的定时任务
crontab -l

# 编辑定时任务
crontab -e

# 删除定时任务
crontab -r

# 查看cron服务状态
systemctl status crond

# 启动cron服务
systemctl start crond

# 设置cron服务自启
systemctl enable crond
```

**cron表达式格式：**
```bash
# 分钟(0-59) 小时(0-23) 日期(1-31) 月份(1-12) 星期(0-7) 命令
# * 表示任意值
# , 表示多个值
# - 表示范围
# / 表示间隔

# 示例：每天凌晨3点执行
0 3 * * * /usr/bin/backup.sh

# 示例：每周一到周五早上8点执行
0 8 * * 1-5 /usr/bin/cleanup.sh

# 示例：每小时执行一次
0 * * * * /usr/bin/check.sh

# 示例：每30分钟执行一次
*/30 * * * * /usr/bin/monitor.sh

# 示例：每月1号和15号执行
0 0 1,15 * * /usr/bin/monthly.sh
```

**系统级定时任务：**
```bash
# 系统级定时任务目录
/etc/cron.d/
/etc/cron.hourly/
/etc/cron.daily/
/etc/cron.weekly/
/etc/cron.monthly/

# 编辑系统级定时任务
vim /etc/cron.d/mycron

# 添加内容
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# 每天凌晨2点执行
0 2 * * * root /usr/bin/system_backup.sh
```

### 6.2 Linux网络防火墙

**firewalld防火墙：**
```bash
# 查看防火墙状态
firewall-cmd --state

# 启动防火墙
systemctl start firewalld

# 停止防火墙
systemctl stop firewalld

# 设置防火墙自启
systemctl enable firewalld

# 查看已开放端口
firewall-cmd --list-ports

# 查看已开放服务
firewall-cmd --list-services

# 开放端口（临时）
firewall-cmd --add-port=80/tcp

# 开放端口（永久）
firewall-cmd --add-port=80/tcp --permanent

# 关闭端口（临时）
firewall-cmd --remove-port=80/tcp

# 关闭端口（永久）
firewall-cmd --remove-port=80/tcp --permanent

# 开放服务（永久）
firewall-cmd --add-service=http --permanent

# 重新加载配置
firewall-cmd --reload
```

**防火墙规则配置：**
```bash
# 创建自定义服务
firewall-cmd --new-service=myapp --permanent

# 配置服务端口
firewall-cmd --service=myapp --add-port=8080/tcp --permanent

# 应用服务
firewall-cmd --add-service=myapp --permanent

# 设置端口转发
firewall-cmd --add-forward-port=port=80:proto=tcp:toport=8080 --permanent

# 设置IP白名单
firewall-cmd --add-source=192.168.1.0/24 --permanent

# 设置IP黑名单
firewall-cmd --remove-source=192.168.1.0/24 --permanent
```

**iptables（旧版防火墙）：**
```bash
# 查看当前规则
iptables -L

# 开放端口
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# 关闭端口
iptables -D INPUT -p tcp --dport 80 -j ACCEPT

# 拒绝所有访问
iptables -P INPUT DROP

# 允许本地回环
iptables -A INPUT -i lo -j ACCEPT

# 允许已建立连接
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# 保存规则
service iptables save

# 重启iptables
service iptables restart
```

### 6.3 Linux内核机制

**内核版本管理：**
```bash
# 查看当前内核版本
uname -r

# 查看系统内核完整信息
uname -a

# 查看已安装内核
rpm -qa | grep kernel

# 安装新内核
yum install kernel

# 设置默认内核（修改grub配置）
vim /etc/default/grub

# 修改默认启动项
GRUB_DEFAULT=0

# 生成grub配置
grub2-mkconfig -o /boot/grub2/grub.cfg

# 删除旧内核
yum remove kernel-旧版本号
```

**内核参数调整：**
```bash
# 查看当前内核参数
sysctl -a

# 修改内核参数（临时）
sysctl -w net.ipv4.ip_forward=1

# 修改内核参数（永久）
vim /etc/sysctl.conf

# 添加参数
net.ipv4.ip_forward = 1
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 65535

# 使配置生效
sysctl -p
```

**常用内核参数：**
| 参数 | 说明 | 推荐值 |
|------|------|--------|
| `net.ipv4.ip_forward` | IP转发 | 1（开启） |
| `net.ipv4.tcp_syncookies` | SYN攻击防护 | 1（开启） |
| `net.ipv4.tcp_max_syn_backlog` | SYN队列长度 | 65535 |
| `vm.swappiness` | 内存交换倾向 | 10 |
| `vm.dirty_ratio` | 脏页比例 | 40 |

**系统调用与进程：**
```bash
# 查看系统调用统计
strace -c ls

# 跟踪进程系统调用
strace -p 1234

# 查看进程打开的文件
lsof -p 1234

# 查看端口占用
lsof -i :80

# 查看内存映射
cat /proc/1234/maps
```

### 6.4 SSH密钥认证

**密钥生成：**
```bash
# 生成RSA密钥对
ssh-keygen -t rsa -b 2048

# 生成ED25519密钥对（更安全）
ssh-keygen -t ed25519

# 指定密钥文件名
ssh-keygen -t ed25519 -f ~/.ssh/mykey
```

**密钥分发：**
```bash
# 复制公钥到远程服务器
ssh-copy-id user@remote_host

# 指定端口复制公钥
ssh-copy-id -p 2222 user@remote_host

# 手动复制公钥
cat ~/.ssh/id_ed25519.pub | ssh user@remote_host "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

**密钥权限配置：**
```bash
# 设置私钥权限（必须600）
chmod 600 ~/.ssh/id_ed25519

# 设置公钥权限（必须644）
chmod 644 ~/.ssh/id_ed25519.pub

# 设置authorized_keys权限（必须600）
chmod 600 ~/.ssh/authorized_keys

# 设置.ssh目录权限（必须700）
chmod 700 ~/.ssh
```

**SSH配置优化：**
```bash
# 编辑SSH客户端配置
vim ~/.ssh/config

# 添加配置
Host server1
    HostName 192.168.1.100
    User root
    Port 2222
    IdentityFile ~/.ssh/id_ed25519

# 使用配置登录
ssh server1
```

### 6.5 SELinux基础

**SELinux状态管理：**
```bash
# 查看SELinux状态
getenforce

# 查看SELinux配置
sestatus

# 临时关闭SELinux
setenforce 0

# 临时开启SELinux
setenforce 1

# 永久关闭SELinux（修改配置文件）
vim /etc/selinux/config

# 修改配置
SELINUX=disabled

# 永久开启SELinux
SELINUX=enforcing

# 重启系统生效
reboot
```

**SELinux策略管理：**
```bash
# 查看SELinux策略
semodule -l

# 安装SELinux策略
semodule -i mypolicy.pp

# 查看文件SELinux上下文
ls -Z

# 修改文件SELinux上下文
chcon -t httpd_sys_content_t /var/www/html

# 永久修改SELinux上下文
semanage fcontext -a -t httpd_sys_content_t "/var/www/html(/.*)?"
restorecon -Rv /var/www/html

# 查看端口SELinux上下文
semanage port -l

# 添加端口到SELinux
semanage port -a -t http_port_t -p tcp 8080
```

**SELinux日志：**
```bash
# 查看SELinux拒绝日志
grep -i "denied" /var/log/audit/audit.log

# 使用sealert分析日志
sealert -a /var/log/audit/audit.log
```

### 6.6 日志管理

**日志文件位置：**
```bash
# 系统日志
/var/log/messages

# 安全日志
/var/log/secure

# 认证日志
/var/log/auth.log

# 应用日志
/var/log/httpd/
/var/log/mysql/
/var/log/tomcat/

# 引导日志
/var/log/dmesg
```

**journalctl日志管理：**
```bash
# 查看所有日志
journalctl

# 查看系统启动日志
journalctl -b

# 查看指定服务日志
journalctl -u nginx

# 查看最近N条日志
journalctl -n 100

# 实时监控日志
journalctl -f

# 按时间范围查看日志
journalctl --since "2024-01-01" --until "2024-01-02"

# 查看错误日志
journalctl -p err

# 导出日志到文件
journalctl -u nginx > nginx.log
```

**logrotate日志轮转：**
```bash
# 查看logrotate配置
cat /etc/logrotate.conf

# 查看应用日志配置
cat /etc/logrotate.d/nginx

# 手动执行日志轮转
logrotate -f /etc/logrotate.d/nginx

# 创建自定义日志轮转配置
vim /etc/logrotate.d/myapp

# 添加配置
/var/log/myapp/*.log {
    daily
    rotate 30
    compress
    delaycompress
    missingok
    notifempty
    create 644 root root
    sharedscripts
    postrotate
        systemctl reload myapp > /dev/null 2>&1 || true
    endscript
}
```

**日志分析工具：**
```bash
# 统计日志中错误出现次数
grep -c "ERROR" /var/log/messages

# 查看最近的错误
grep "ERROR" /var/log/messages | tail -20

# 使用awk分析日志
awk '{print $1, $2, $NF}' /var/log/messages

# 使用sed处理日志
sed -n '/2024-01-15/p' /var/log/messages
```

### 6.7 网络配置

**网络接口配置：**
```bash
# 查看网卡配置文件
ls /etc/sysconfig/network-scripts/

# 编辑网卡配置
vim /etc/sysconfig/network-scripts/ifcfg-eth0

# 静态IP配置
BOOTPROTO=static
IPADDR=192.168.1.100
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=8.8.8.8
DNS2=1.1.1.1
ONBOOT=yes

# 重启网络服务
systemctl restart network
```

**DNS配置：**
```bash
# 查看DNS配置
cat /etc/resolv.conf

# 临时修改DNS
echo "nameserver 8.8.8.8" > /etc/resolv.conf

# 永久修改DNS（修改网卡配置）
vim /etc/sysconfig/network-scripts/ifcfg-eth0

# 添加DNS配置
DNS1=8.8.8.8
DNS2=1.1.1.1
```

**路由配置：**
```bash
# 查看路由表
ip route

# 添加默认路由
ip route add default via 192.168.1.1

# 添加静态路由
ip route add 10.0.0.0/8 via 192.168.1.100

# 删除路由
ip route del 10.0.0.0/8

# 永久添加路由（编辑配置文件）
vim /etc/sysconfig/network-scripts/route-eth0

# 添加路由配置
10.0.0.0/8 via 192.168.1.100
```

**网络诊断工具：**
```bash
# 测试网络连通性
ping -c 4 www.baidu.com

# 测试端口连通性
telnet localhost 80

# 查看网络连接
ss -tuln

# 追踪路由
traceroute www.baidu.com

# 查看DNS解析
nslookup www.baidu.com

# 查看DNS解析（更详细）
dig www.baidu.com

# 查看网络接口统计
netstat -s
```

---

## 第七章：Shell脚本

### 7.1 Shell脚本入门

**创建Shell脚本：**
```bash
# 创建脚本文件
vim hello.sh

# 脚本内容
#!/bin/bash
echo "Hello, World!"

# 赋予执行权限
chmod +x hello.sh

# 执行脚本
./hello.sh
```

**脚本执行方式：**
```bash
# 方式一：直接执行（需要执行权限）
./hello.sh

# 方式二：使用bash解释器执行（不需要执行权限）
bash hello.sh

# 方式三：使用sh解释器执行（兼容模式）
sh hello.sh

# 方式四：在当前Shell环境执行（影响当前环境）
source hello.sh
. hello.sh
```

**脚本规范：**
```bash
#!/bin/bash
# 脚本名称: hello.sh
# 功能描述: 输出Hello World
# 创建日期: 2024-01-01
# 作者: Admin

# 脚本内容
echo "Hello, World!"
```

### 7.2 Shell变量_系统预定义变量

**系统预定义变量：**
```bash
# 查看所有环境变量
env

# 常用系统变量
echo $HOME       # 用户主目录
echo $PATH       # 命令搜索路径
echo $USER       # 当前用户名
echo $SHELL      # 当前Shell
echo $PWD        # 当前工作目录
echo $LANG       # 语言设置
echo $HOSTNAME   # 主机名
echo $TERM       # 终端类型
```

**Bash特殊变量：**
```bash
echo $$          # 当前进程PID
echo $PPID       # 父进程PID
echo $?          # 上一条命令执行结果（0表示成功）
echo $0          # 当前脚本名称
echo $#          # 命令行参数个数
```

**常用系统变量说明：**
| 变量 | 说明 |
|------|------|
| `$HOME` | 当前用户主目录 |
| `$PATH` | 命令搜索路径 |
| `$USER` | 当前用户名 |
| `$SHELL` | 当前Shell类型 |
| `$PWD` | 当前工作目录 |
| `$?` | 命令返回值 |

### 7.3 Shell变量_用户自定义变量

**变量定义：**
```bash
# 定义变量（等号两边无空格）
name="Alice"

# 定义数字变量
age=25

# 定义带空格的字符串变量
greeting="Hello, World!"

# 使用变量
echo $name
echo "My name is $name"
```

**变量命名规则：**
- 变量名由字母、数字、下划线组成
- 变量名不能以数字开头
- 变量名区分大小写
- 不要使用Shell关键字作为变量名

**变量作用域：**
```bash
# 局部变量（仅当前Shell有效）
local_var="local"

# 全局变量（子Shell也能访问）
export global_var="global"

# 在脚本中定义全局变量
export PATH=$PATH:/new/path
```

**变量操作：**
```bash
# 变量赋值
var1="value1"
var2=$var1        # 变量值赋给另一个变量
var3=$(date)      # 命令执行结果赋给变量
var4=`date`       # 命令执行结果赋给变量（反引号方式）

# 变量拼接
full_name="$first_name $last_name"

# 变量长度
echo ${#name}     # 输出变量长度
```

### 7.4 Shell变量_只读变量和撤销变量

**只读变量：**
```bash
# 定义只读变量
readonly PI=3.14

# 定义变量后设为只读
name="Alice"
readonly name

# 尝试修改只读变量（会报错）
PI=3.14159        # error: readonly variable

# 查看只读变量
readonly -p
```

**撤销变量：**
```bash
# 定义变量
name="Alice"

# 撤销变量
unset name

# 查看变量（已不存在）
echo $name        # 输出空

# 注意：不能撤销只读变量
unset PI          # error: cannot unset readonly variable
```

### 7.5 Shell变量_特殊变量

**位置参数变量：**
```bash
# 脚本内容
#!/bin/bash
echo "脚本名称: $0"
echo "参数个数: $#"
echo "第一个参数: $1"
echo "第二个参数: $2"
echo "所有参数: $*"
echo "所有参数: $@"

# 执行脚本
./script.sh arg1 arg2 arg3
```

**位置参数说明：**
| 变量 | 说明 |
|------|------|
| `$0` | 脚本名称 |
| `$1-$9` | 第1-9个命令行参数 |
| `${10}` | 第10个及以上参数 |
| `$#` | 参数个数 |
| `$*` | 所有参数（作为单个字符串） |
| `$@` | 所有参数（作为独立字符串） |

**特殊变量：**
```bash
echo $$          # 当前进程PID
echo $PPID       # 父进程PID
echo $?          # 上一条命令返回值
echo $!          # 后台进程PID
echo $-          # 当前Shell选项
```

**$*与$@的区别：**
```bash
# 脚本内容
#!/bin/bash
echo "使用\$*遍历:"
for arg in "$*"; do
    echo "$arg"
done

echo "使用\$@遍历:"
for arg in "$@"; do
    echo "$arg"
done

# 执行结果
# 使用$*遍历:
# arg1 arg2 arg3
# 使用$@遍历:
# arg1
# arg2
# arg3
```

### 7.6 Shell_运算符

**算术运算符：**
```bash
# 基本运算
a=10
b=5

echo $((a + b))   # 加法
echo $((a - b))   # 减法
echo $((a * b))   # 乘法
echo $((a / b))   # 除法
echo $((a % b))   # 取模
echo $((a ** b))  # 幂运算
```

**赋值运算符：**
```bash
a=10
a=$((a + 1))      # a = a + 1
echo $a           # 输出11

# 简写形式
a=$((a += 1))     # a = a + 1
a=$((a -= 1))     # a = a - 1
a=$((a *= 2))     # a = a * 2
a=$((a /= 2))     # a = a / 2
```

**比较运算符：**
```bash
a=10
b=5

# 整数比较
[ $a -eq $b ]     # 等于
[ $a -ne $b ]     # 不等于
[ $a -gt $b ]     # 大于
[ $a -lt $b ]     # 小于
[ $a -ge $b ]     # 大于等于
[ $a -le $b ]     # 小于等于

# 字符串比较
[ "abc" = "abc" ] # 等于
[ "abc" != "def" ] # 不等于
[ -z "" ]         # 字符串为空
[ -n "abc" ]      # 字符串不为空
```

**逻辑运算符：**
```bash
# 逻辑与（两个条件都为真）
[ $a -gt 0 -a $b -gt 0 ]

# 逻辑或（两个条件任一为真）
[ $a -gt 0 -o $b -gt 0 ]

# 逻辑非（取反）
[ ! $a -gt 0 ]

# 使用&&和||
[ $a -gt 0 ] && echo "a大于0"
[ $a -gt 0 ] || echo "a不大于0"
```

**文件测试运算符：**
```bash
# 文件测试
[ -f file.txt ]   # 文件存在且为普通文件
[ -d dir ]        # 目录存在
[ -e path ]       # 文件或目录存在
[ -r file.txt ]   # 文件可读
[ -w file.txt ]   # 文件可写
[ -x file.txt ]   # 文件可执行
[ -s file.txt ]   # 文件大小不为0
```

### 7.7 Shell_条件判断

**if条件判断：**
```bash
# 基本格式
if [ 条件 ]; then
    # 条件为真时执行
    echo "条件成立"
fi

# if-else格式
if [ $a -gt 0 ]; then
    echo "a大于0"
else
    echo "a不大于0"
fi

# if-elif-else格式
if [ $score -ge 90 ]; then
    echo "优秀"
elif [ $score -ge 60 ]; then
    echo "及格"
else
    echo "不及格"
fi
```

**case条件判断：**
```bash
# case格式
case $choice in
    1)
        echo "选择了1"
        ;;
    2)
        echo "选择了2"
        ;;
    3)
        echo "选择了3"
        ;;
    *)
        echo "无效选择"
        ;;
esac
```

**条件判断示例：**
```bash
# 判断文件是否存在
if [ -f /etc/passwd ]; then
    echo "文件存在"
else
    echo "文件不存在"
fi

# 判断用户是否存在
if id "alice" &>/dev/null; then
    echo "用户存在"
else
    echo "用户不存在"
fi

# 判断命令是否成功执行
if ping -c 1 www.baidu.com &>/dev/null; then
    echo "网络正常"
else
    echo "网络异常"
fi
```

### 7.8 Shell_流程控制

**for循环：**
```bash
# 基本格式
for i in 1 2 3 4 5; do
    echo "数字: $i"
done

# 遍历文件
for file in *.txt; do
    echo "文件: $file"
done

# 数字范围循环
for i in {1..10}; do
    echo "数字: $i"
done

# C风格for循环
for ((i=1; i<=10; i++)); do
    echo "数字: $i"
done
```

**while循环：**
```bash
# 基本格式
count=1
while [ $count -le 5 ]; do
    echo "计数: $count"
    count=$((count + 1))
done

# 无限循环
while true; do
    echo "按Ctrl+C退出"
    sleep 1
done

# 读取文件内容
while read line; do
    echo "行: $line"
done < file.txt
```

**until循环：**
```bash
# 基本格式（条件为假时执行）
count=1
until [ $count -gt 5 ]; do
    echo "计数: $count"
    count=$((count + 1))
done
```

**循环控制：**
```bash
# break：跳出循环
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        break
    fi
    echo "数字: $i"
done

# continue：跳过当前循环
for i in {1..10}; do
    if [ $((i % 2)) -eq 0 ]; then
        continue
    fi
    echo "奇数: $i"
done
```

### 7.9 Shell_读取控制台输入

**read命令：**
```bash
# 基本读取
read name
echo "你输入的是: $name"

# 带提示读取
read -p "请输入姓名: " name
echo "姓名: $name"

# 读取密码（不显示）
read -s -p "请输入密码: " password
echo ""
echo "密码已输入"

# 限制输入时间（5秒）
read -t 5 -p "请在5秒内输入: " input

# 限制输入字符数（最多10个）
read -n 10 -p "请输入最多10个字符: " input
echo ""
```

**读取多个变量：**
```bash
# 读取多个变量
read -p "请输入姓名和年龄: " name age
echo "姓名: $name, 年龄: $age"

# 读取数组
read -a arr -p "请输入多个值: "
echo "数组元素: ${arr[0]}, ${arr[1]}, ${arr[2]}"
```

### 7.10 Shell函数_系统函数

**常用系统函数：**
```bash
# basename：获取文件名
basename /path/to/file.txt
basename /path/to/file.txt .txt   # 去掉后缀

# dirname：获取目录名
dirname /path/to/file.txt

# date：获取日期时间
date +%Y-%m-%d
date +%H:%M:%S
date +%s                          # 时间戳

# sleep：延迟执行
sleep 5                           # 延迟5秒

# exit：退出脚本
exit 0                            # 正常退出
exit 1                            # 异常退出
```

**字符串处理函数：**
```bash
# 获取字符串长度
str="hello"
echo ${#str}                      # 输出5

# 字符串截取
str="hello world"
echo ${str:0:5}                   # 从第0位开始取5个字符
echo ${str:6}                     # 从第6位开始取到末尾

# 字符串替换
str="hello world"
echo ${str/world/universe}        # 替换第一个匹配
echo ${str//l/x}                  # 替换所有匹配

# 字符串删除
str="hello world"
echo ${str#*l}                    # 删除从开头到第一个l
echo ${str##*l}                   # 删除从开头到最后一个l
echo ${str%l*}                    # 删除从最后一个l到末尾
echo ${str%%l*}                   # 删除从第一个l到末尾
```

### 7.11 Shell函数_自定义函数

**定义函数：**
```bash
# 基本格式
function greet {
    echo "Hello, World!"
}

# 简化格式
greet() {
    echo "Hello, World!"
}

# 调用函数
greet
```

**带参数函数：**
```bash
# 带参数的函数
add() {
    sum=$(( $1 + $2 ))
    echo "和为: $sum"
}

# 调用函数
add 10 5
```

**带返回值函数：**
```bash
# 返回值函数
add() {
    return $(( $1 + $2 ))
}

# 调用并获取返回值
add 10 5
echo "返回值: $?"
```

**函数变量作用域：**
```bash
# 全局变量（默认）
var="global"

test_func() {
    var="local"
    echo "函数内: $var"
}

test_func
echo "函数外: $var"

# 使用local声明局部变量
test_func2() {
    local var="local"
    echo "函数内: $var"
}

test_func2
echo "函数外: $var"
```

**递归函数：**
```bash
# 阶乘函数
factorial() {
    if [ $1 -eq 1 ]; then
        echo 1
    else
        local temp=$(( $1 - 1 ))
        local result=$(factorial $temp)
        echo $(( $1 * result ))
    fi
}

# 调用递归函数
factorial 5
```

### 7.12 Shell综合应用案例_归档文件

**归档脚本示例：**
```bash
#!/bin/bash

# 归档目录
SOURCE_DIR="/var/log"
# 归档目标目录
DEST_DIR="/backup"
# 归档文件名（包含日期）
FILENAME="logs_$(date +%Y%m%d_%H%M%S).tar.gz"

# 创建目标目录
mkdir -p $DEST_DIR

# 执行归档
tar -zcvf $DEST_DIR/$FILENAME $SOURCE_DIR

# 检查归档是否成功
if [ $? -eq 0 ]; then
    echo "归档成功: $DEST_DIR/$FILENAME"
    # 显示归档文件大小
    ls -lh $DEST_DIR/$FILENAME
else
    echo "归档失败"
    exit 1
fi
```

**脚本使用：**
```bash
# 赋予执行权限
chmod +x archive.sh

# 执行脚本
./archive.sh

# 查看归档文件
ls -la /backup/
```

### 7.13 Shell综合应用案例_定时归档文件

**定时归档脚本：**
```bash
#!/bin/bash

# 配置参数
SOURCE_DIR="/var/log"
DEST_DIR="/backup/logs"
RETENTION_DAYS=30

# 创建目标目录
mkdir -p $DEST_DIR

# 执行归档
FILENAME="logs_$(date +%Y%m%d_%H%M%S).tar.gz"
tar -zcf $DEST_DIR/$FILENAME $SOURCE_DIR &>/dev/null

# 检查归档结果
if [ $? -eq 0 ]; then
    echo "$(date '+%Y-%m-%d %H:%M:%S') - 归档成功: $FILENAME" >> /var/log/archive.log
else
    echo "$(date '+%Y-%m-%d %H:%M:%S') - 归档失败" >> /var/log/archive.log
    exit 1
fi

# 删除过期归档（保留最近30天）
find $DEST_DIR -name "*.tar.gz" -mtime +$RETENTION_DAYS -delete
echo "$(date '+%Y-%m-%d %H:%M:%S') - 已清理过期归档" >> /var/log/archive.log
```

**配置定时任务：**
```bash
# 编辑定时任务
crontab -e

# 添加定时任务（每天凌晨2点执行）
0 2 * * * /usr/bin/archive_script.sh
```

### 7.14 Shell_正则表达式

**基础正则表达式：**
```bash
# 匹配以a开头的行
grep "^a" file.txt

# 匹配以z结尾的行
grep "z$" file.txt

# 匹配包含任意字符的行
grep "." file.txt

# 匹配包含abc或adc的行
grep "a[bd]c" file.txt

# 匹配包含数字的行
grep "[0-9]" file.txt

# 匹配不包含数字的行
grep "[^0-9]" file.txt

# 匹配0个或多个a
grep "a*" file.txt

# 匹配1个或多个a
grep "a\+" file.txt

# 匹配0个或1个a
grep "a\?" file.txt

# 精确匹配单词
grep "\<word\>" file.txt
```

**扩展正则表达式：**
```bash
# 使用扩展正则表达式
grep -E "pattern" file.txt

# 匹配abc或def
grep -E "abc|def" file.txt

# 匹配重复字符
grep -E "a{2,3}" file.txt         # 匹配2-3个a
grep -E "a{2,}" file.txt          # 匹配2个及以上a
grep -E "a{2}" file.txt           # 精确匹配2个a

# 分组匹配
grep -E "(ab)+" file.txt          # 匹配ab重复1次及以上

# 非贪婪匹配（最短匹配）
grep -E "a.*?b" file.txt
```

**正则表达式常用元字符：**
| 元字符 | 说明 |
|--------|------|
| `.` | 匹配任意单个字符 |
| `^` | 匹配行首 |
| `$` | 匹配行尾 |
| `*` | 匹配0个或多个前一个字符 |
| `+` | 匹配1个或多个前一个字符 |
| `?` | 匹配0个或1个前一个字符 |
| `[]` | 匹配括号内任意一个字符 |
| `[^]` | 匹配不在括号内的字符 |
| `()` | 分组 |
| `\|` | 或 |
| `{n,m}` | 匹配n到m个前一个字符 |

### 7.15 Shell文本处理工具_cut

**cut命令：**
```bash
# 按列提取（默认分隔符Tab）
cut -f 1,3 file.txt

# 指定分隔符
cut -d ":" -f 1,6 /etc/passwd

# 按字节提取
cut -c 1-5 file.txt

# 按字符提取（支持多字节字符）
cut -c 1-5 file.txt

# 提取第1-3列
cut -d "," -f 1-3 data.csv
```

**cut命令选项：**
| 选项 | 说明 |
|------|------|
| `-f` | 指定列号 |
| `-d` | 指定分隔符 |
| `-c` | 指定字符位置 |
| `-b` | 指定字节位置 |
| `-s` | 不显示不含分隔符的行 |

**cut命令示例：**
```bash
# 提取用户名
cut -d ":" -f 1 /etc/passwd

# 提取用户名和主目录
cut -d ":" -f 1,6 /etc/passwd

# 提取每行前10个字符
cut -c 1-10 file.txt
```

### 7.16 Shell文本处理工具_awk

**awk基础用法：**
```bash
# 打印所有行
awk '{print}' file.txt

# 打印指定列
awk '{print $1, $3}' file.txt

# 指定分隔符
awk -F ":" '{print $1, $6}' /etc/passwd

# 打印行号
awk '{print NR, $0}' file.txt

# 打印最后一列
awk '{print $NF}' file.txt
```

**awk条件处理：**
```bash
# 条件筛选
awk '$1 > 100 {print}' data.txt

# 多条件筛选
awk '$1 > 100 && $2 == "yes" {print}' data.txt

# 匹配字符串
awk '/error/ {print}' log.txt

# 不匹配字符串
awk '!/error/ {print}' log.txt
```

**awk内置变量：**
```bash
# 使用内置变量
awk '{print NR, NF, $0}' file.txt

# NR：行号
# NF：字段数
# $0：整行内容
# $1-$n：第1到第n个字段
# FS：字段分隔符（默认空格）
# OFS：输出字段分隔符（默认空格）
# RS：记录分隔符（默认换行）
```

**awk计算功能：**
```bash
# 计算求和
awk '{sum += $1} END {print sum}' data.txt

# 计算平均值
awk '{sum += $1; count++} END {print sum/count}' data.txt

# 计算最大值
awk 'BEGIN {max=0} {if($1>max) max=$1} END {print max}' data.txt

# 统计行数
awk 'END {print NR}' file.txt
```

**awk流程控制：**
```bash
# if语句
awk '{if($1 > 100) print "大于100"; else print "小于等于100"}' data.txt

# for循环
awk '{for(i=1; i<=NF; i++) print $i}' file.txt

# while循环
awk '{i=1; while(i<=NF) {print $i; i++}}' file.txt
```

**awk内置函数：**
```bash
# 字符串函数
awk '{print length($0)}' file.txt       # 字符串长度
awk '{print substr($0,1,10)}' file.txt  # 字符串截取
awk '{print index($0,"pattern")}' file.txt # 查找位置
awk '{print toupper($0)}' file.txt      # 转大写
awk '{print tolower($0)}' file.txt      # 转小写

# 数学函数
awk '{print sqrt($1)}' data.txt         # 平方根
awk '{print log($1)}' data.txt          # 自然对数
awk '{print exp($1)}' data.txt          # 指数
awk '{print int($1)}' data.txt          # 取整
```

**awk综合示例：**
```bash
# 统计日志中各状态码的数量
awk '{count[$9]++} END {for(code in count) print code, count[code]}' access.log

# 计算文件大小并格式化输出
ls -la | awk '{sum += $5} END {print "总大小: " sum/1024/1024 " MB"}'

# 提取IP地址并去重
awk '{print $1}' access.log | sort | uniq -c | sort -rn

# 生成报表
awk -F "," '{printf "%-20s %10d %10.2f\n", $1, $2, $3}' data.csv
```

---

## 附录

### A.1 常用快捷键

| 快捷键 | 说明 |
|--------|------|
| `Ctrl+C` | 中断当前命令 |
| `Ctrl+D` | 结束输入（EOF） |
| `Ctrl+Z` | 挂起当前进程 |
| `Ctrl+L` | 清屏 |
| `Ctrl+A` | 光标移到行首 |
| `Ctrl+E` | 光标移到行尾 |
| `Ctrl+U` | 删除光标到行首 |
| `Ctrl+K` | 删除光标到行尾 |
| `Tab` | 自动补全 |
| `↑/↓` | 上下翻阅命令历史 |

### A.2 常用命令速查

| 命令 | 说明 |
|------|------|
| `ls` | 列出文件 |
| `cd` | 切换目录 |
| `pwd` | 显示当前目录 |
| `mkdir` | 创建目录 |
| `rm` | 删除文件/目录 |
| `cp` | 复制文件/目录 |
| `mv` | 移动/重命名 |
| `cat` | 查看文件内容 |
| `vim` | 编辑文件 |
| `grep` | 搜索内容 |
| `find` | 查找文件 |
| `tar` | 打包压缩 |
| `chmod` | 修改权限 |
| `chown` | 修改所有者 |
| `ps` | 查看进程 |
| `top` | 实时监控进程 |
| `kill` | 终止进程 |
| `systemctl` | 管理服务 |
| `yum` | 包管理 |

---

> **文档版本**：v1.0  
> **最后更新**：2024年  
> **适用版本**：CentOS 7/8/9