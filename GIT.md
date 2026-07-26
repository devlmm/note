# Git技术手册

   **阅读指南**：由浅入深，每个知识点均配有精简示例。建议按顺序阅读，环环相扣。

   ---
   
## 目录

1. [Git基础](#1-git基础)
   - [1.1 下载安装](#11-下载安装)
   - [1.2 环境配置](#12-环境配置)
   - [1.3 初始化本地仓库](#13-初始化本地仓库)
   - [1.4 文件的两种状态](#14-文件的两种状态)
   - [1.5 文件加入暂存区](#15-文件加入暂存区)
   - [1.6 文件提交与删除](#16-文件提交与删除)
   - [1.7 文件添加至忽略列表](#17-文件添加至忽略列表)
   - [1.8 日志记录操作](#18-日志记录操作)
   - [1.9 比较文件差异](#19-比较文件差异)
   - [1.10 还原文件](#110-还原文件)

2. [Git远程仓库](#2-git远程仓库)
   - [2.1 常见的Git托管平台](#21-常见的git托管平台)
   - [2.2 创建远程仓库](#22-创建远程仓库)
   - [2.3 远程仓库操作](#23-远程仓库操作)
   - [2.4 推送、抓取和拉取](#24-推送抓取和拉取)
   - [2.5 多人协作冲突问题](#25-多人协作冲突问题)
   - [2.6 SSH协议推送](#26-ssh协议推送)

3. [Git分支](#3-git分支)
   - [3.1 使用分支的原因](#31-使用分支的原因)
   - [3.2 创建与切换分支](#32-创建与切换分支)
   - [3.3 合并分支](#33-合并分支)
   - [3.4 删除分支](#34-删除分支)
   - [3.5 合并分支冲突问题](#35-合并分支冲突问题)

4. [Git标签](#4-git标签)
   - [4.1 标签概念](#41-标签概念)
   - [4.2 创建标签](#42-创建标签)
   - [4.3 检出与删除标签](#43-检出与删除标签)

5. [Git工作流](#5-git工作流)
   - [5.1 Git Flow工作流](#51-git-flow工作流)
   - [5.2 GitHub Flow工作流](#52-github-flow工作流)
   - [5.3 Git Commit Message规范](#53-git-commit-message规范)
   - [5.4 Dev-Sit-Prd企业级工作流](#54-dev-sit-prd企业级工作流)

6. [在IDEA中使用Git](#6-在idea中使用git)
   - [6.1 配置Git环境](#61-配置git环境)
   - [6.2 Git基本操作](#62-git基本操作)
   - [6.3 Git分支操作](#63-git分支操作)
   - [6.4 解决冲突](#64-解决冲突)

7. [Git进阶技巧](#7-git进阶技巧)
   - [7.1 交互式变基（Rebase）](#71-交互式变基rebase)
   - [7.2 临时存储（Stash）](#72-临时存储stash)
   - [7.3 挑选提交（Cherry-Pick）](#73-挑选提交cherry-pick)
   - [7.4 二分查找（Bisect）](#74-二分查找bisect)
   - [7.5 Git钩子（Hooks）](#75-git钩子hooks)

---

## 1 Git基础

### 1.1 下载安装

Git是一个分布式版本控制系统，支持Windows、macOS和Linux。

**Windows安装：**
```bash
# 从官网下载安装程序
# https://git-scm.com/download/win
# 安装时建议勾选"Add Git to PATH"
```

**验证安装：**
```bash
git --version  # 查看Git版本
```

**实用技巧：**
- 建议使用默认安装路径，避免中文路径导致问题
- 安装完成后重启终端使PATH生效

---

### 1.2 环境配置

首次使用Git需要配置用户名和邮箱，用于标识提交者身份。

**全局配置：**
```bash
# 配置用户名
git config --global user.name "Your Name"

# 配置邮箱
git config --global user.email "your.email@example.com"

# 查看所有配置
git config --list
```

**局部配置（针对单个仓库）：**
```bash
# 进入仓库目录后执行，仅对当前仓库生效
git config user.name "Another Name"
```

**配置编辑器：**
```bash
# 设置默认编辑器为VS Code
git config --global core.editor "code --wait"
```

---

### 1.3 初始化本地仓库

将普通文件夹转换为Git仓库，开始版本控制。

**初始化仓库：**
```bash
# 创建项目目录并进入
mkdir my-project && cd my-project

# 初始化Git仓库
git init
```

**初始化已有项目：**
```bash
# 进入已有项目目录
cd existing-project

# 初始化Git仓库
git init
```

**验证仓库状态：**
```bash
git status  # 查看当前仓库状态
```

**实用技巧：**
- `.git`目录存储所有版本信息，不要手动修改
- 初始化后会创建一个默认的`master`或`main`分支

---

### 1.4 文件的两种状态

Git管理的文件有两种基本状态：

| 状态 | 说明 | 特点 |
|------|------|------|
| **已跟踪** | Git已经知道的文件 | 会被版本控制，可提交 |
| **未跟踪** | Git不知道的新文件 | 不会被版本控制 |

**查看文件状态：**
```bash
git status  # 详细显示状态
git status -s  # 简洁显示状态
```

**状态标识含义：**
```bash
# 未跟踪文件（Untracked）
?? README.md

# 已修改未暂存（Modified）
M  src/main.java

# 已暂存待提交（Staged）
A  src/utils.java
```

---

### 1.5 文件加入暂存区

暂存区（Staging Area）是提交前的缓冲区，用于组织本次提交的内容。

**添加单个文件：**
```bash
git add README.md  # 将README.md加入暂存区
```

**添加多个文件：**
```bash
git add file1.java file2.java  # 添加多个指定文件
```

**添加所有文件：**
```bash
git add .  # 添加当前目录下所有文件（推荐）
git add -A  # 添加所有变更（包括删除的文件）
```

**查看暂存区内容：**
```bash
git diff --cached  # 查看暂存区与上次提交的差异
```

**从暂存区移除：**
```bash
git restore --staged README.md  # 将文件移出暂存区
```

---

### 1.6 文件提交与删除

提交（Commit）是将暂存区的内容保存为一个版本快照。

**提交变更：**
```bash
# 提交暂存区内容，-m后跟提交信息
git commit -m "feat: 添加用户登录功能"
```

**详细提交：**
```bash
# 打开编辑器编写提交信息
git commit
```

**提交规范（Conventional Commits）：**
```bash
# 格式：<type>(<scope>): <description>
git commit -m "feat(user): 添加用户注册接口"
git commit -m "fix(auth): 修复token过期问题"
git commit -m "docs(readme): 更新项目说明文档"
```

**删除文件：**
```bash
# 删除并提交到暂存区
git rm unwanted-file.txt

# 删除目录（递归）
git rm -r old-directory

# 仅从Git中删除，保留本地文件
git rm --cached keep-local.txt
```

---

### 1.7 文件添加至忽略列表

`.gitignore`文件用于指定不需要被Git跟踪的文件或目录。

**创建.gitignore文件：**
```bash
touch .gitignore
```

**常用忽略规则：**
```gitignore
# 忽略所有.class文件
*.class

# 忽略target目录
target/

# 忽略IDE配置
.idea/
.vscode/
*.iml

# 忽略操作系统文件
.DS_Store
Thumbs.db

# 忽略日志文件
*.log

# 忽略构建工具目录
node_modules/
vendor/
```

**递归忽略子目录：**
```gitignore
# 忽略所有目录下的temp文件夹
**/temp/

# 忽略特定目录下的特定文件
src/test/resources/*.xml
```

**实用技巧：**
- `.gitignore`文件需要提交到仓库中
- 使用`git status --ignored`查看被忽略的文件

---

### 1.8 日志记录操作

日志（Log）记录了所有提交历史，便于追溯和理解项目演进。

**查看完整日志：**
```bash
git log  # 显示所有提交记录
```

**简洁日志：**
```bash
git log --oneline  # 每行显示一个提交，包含简短哈希和信息
```

**图形化日志：**
```bash
git log --graph --oneline  # 带分支图的简洁日志
git log --graph --oneline --all  # 显示所有分支
```

**查看指定数量的日志：**
```bash
git log -5  # 最近5条提交
```

**查看详细变更：**
```bash
git log -p  # 显示提交的详细diff
```

**查看特定文件的变更历史：**
```bash
git log --follow -- README.md  # 跟踪文件重命名
```

---

### 1.9 比较文件差异

Diff命令用于查看文件在不同状态下的差异。

**比较工作区与暂存区：**
```bash
git diff  # 显示未暂存的变更
```

**比较暂存区与上次提交：**
```bash
git diff --cached  # 显示已暂存但未提交的变更
```

**比较两个提交：**
```bash
git diff commit1 commit2  # 比较两个提交之间的差异
```

**比较特定文件：**
```bash
git diff README.md  # 仅查看README.md的差异
```

**实用技巧：**
- `diff`显示的是"补丁"格式，`+`表示新增，`-`表示删除
- 使用`git difftool`可以用图形化工具查看差异

---

### 1.10 还原文件

还原（Restore）用于撤销工作区或暂存区的变更。

**还原工作区文件：**
```bash
git restore README.md  # 将文件恢复到暂存区状态
```

**还原到上次提交状态：**
```bash
git restore --staged README.md  # 取消暂存
git restore README.md  # 还原工作区
```

**重置到指定提交：**
```bash
# 软重置：保留工作区和暂存区
git reset --soft HEAD~1

# 混合重置：保留工作区，清空暂存区（默认）
git reset HEAD~1

# 硬重置：丢弃所有变更（慎用）
git reset --hard HEAD~1
```

**找回已删除的提交：**
```bash
git reflog  # 查看引用日志
git reset --hard <commit-hash>  # 恢复到指定提交
```

**实用技巧：**
- `HEAD`表示当前提交，`HEAD~1`表示上一次提交
- `git restore`是Git 2.23+新增命令，替代`git checkout`的部分功能

---

## 2 Git远程仓库

### 2.1 常见的Git托管平台

| 平台 | 特点 | 适用场景 |
|------|------|----------|
| **GitHub** | 全球最大，开源友好 | 开源项目、个人学习 |
| **Gitee（码云）** | 国内访问快，中文支持好 | 国内项目、企业内部 |
| **GitLab** | 可私有化部署，功能强大 | 企业级项目管理 |
| **Bitbucket** | Atlassian生态，免费私有仓库 | 团队协作、JIRA集成 |

---

### 2.2 创建远程仓库

以Gitee为例，创建远程仓库的步骤：

1. 登录Gitee账号
2. 点击"新建仓库"
3. 填写仓库信息：
   - 仓库名称（必填）
   - 仓库描述（可选）
   - 是否开源（公开/私有）
4. 点击"创建"

**获取远程仓库地址：**
```bash
# HTTPS地址（需要输入账号密码）
https://gitee.com/username/repo-name.git

# SSH地址（需要配置SSH密钥）
git@gitee.com:username/repo-name.git
```

---

### 2.3 远程仓库操作

**查看已配置的远程仓库：**
```bash
git remote -v  # 显示远程仓库别名和URL
```

**添加远程仓库：**
```bash
# 格式：git remote add <别名> <URL>
git remote add origin https://gitee.com/username/repo-name.git
```

**修改远程仓库地址：**
```bash
git remote set-url origin https://gitee.com/new-username/repo-name.git
```

**删除远程仓库：**
```bash
git remote remove origin
```

**获取远程仓库信息：**
```bash
git remote show origin  # 显示远程仓库详细信息
```

---

### 2.4 推送、抓取和拉取

**推送（Push）：**将本地提交上传到远程仓库
```bash
# 推送指定分支
git push origin master

# 设置上游跟踪（第一次推送时使用）
git push -u origin master

# 推送所有分支
git push --all
```

**抓取（Fetch）：**获取远程仓库更新，但不合并
```bash
git fetch origin  # 获取所有远程分支更新
git fetch origin master  # 获取指定分支更新
```

**拉取（Pull）：**获取远程更新并合并到本地
```bash
git pull origin master  # 拉取并合并指定分支
```

**实用技巧：**
- `git pull` = `git fetch` + `git merge`
- 多人协作时，建议先`fetch`查看变更，再决定是否`merge`

---

### 2.5 多人协作冲突问题

当多人修改同一文件的同一部分时，会产生冲突。

**冲突示例：**
```bash
# 拉取时遇到冲突
git pull origin master

# 冲突标记会出现在文件中
<<<<<<< HEAD
我的修改内容
=======
远程仓库的修改内容
>>>>>>> origin/master
```

**解决冲突步骤：**
```bash
# 1. 查看冲突文件
git status  # 显示未合并的路径

# 2. 手动编辑文件，解决冲突
# 删除冲突标记，保留需要的内容

# 3. 将解决后的文件加入暂存区
git add conflicted-file.java

# 4. 完成合并提交
git commit
```

**实用技巧：**
- 冲突解决后，提交信息默认包含"Merge branch..."
- 使用`git mergetool`可以调用图形化工具解决冲突

---

### 2.6 SSH协议推送

SSH协议无需每次输入密码，更安全便捷。

**生成SSH密钥：**
```bash
# 生成密钥对，一路回车使用默认设置
ssh-keygen -t ed25519 -C "your.email@example.com"

# 启动SSH代理并添加密钥
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

**查看公钥：**
```bash
cat ~/.ssh/id_ed25519.pub  # 复制公钥内容
```

**配置到Gitee/GitHub：**
1. 登录平台
2. 进入个人设置 → SSH公钥
3. 粘贴公钥内容并保存

**测试连接：**
```bash
ssh -T git@gitee.com  # 测试Gitee连接
ssh -T git@github.com  # 测试GitHub连接
```

**使用SSH地址：**
```bash
git remote set-url origin git@gitee.com:username/repo-name.git
git push origin master  # 无需输入密码
```

---

## 3 Git分支

### 3.1 使用分支的原因

分支（Branch）是Git最强大的功能之一，支持并行开发。

**分支的优势：**
- **隔离开发**：不同功能在独立分支开发，互不影响
- **安全测试**：新功能在分支中测试，不影响主分支稳定性
- **敏捷迭代**：支持快速创建和合并，适应敏捷开发流程
- **多人协作**：每人一个分支，最后合并到主分支

**分支模型示意：**
```
master ────────────────────────●──●──●
       \           \           /
        feature-a───●──●──●───/
                    \
                     bugfix──●
```

---

### 3.2 创建与切换分支

**创建分支：**
```bash
git branch feature-login  # 创建分支但不切换
```

**创建并切换分支：**
```bash
git checkout -b feature-login  # 创建并切换到新分支
```

**切换分支：**
```bash
git checkout master  # 切换到master分支
```

**查看所有分支：**
```bash
git branch  # 查看本地分支，*表示当前分支
git branch -a  # 查看所有分支（包括远程）
```

**查看分支详情：**
```bash
git show feature-login  # 显示分支最后一次提交
```

---

### 3.3 合并分支

合并（Merge）将一个分支的变更整合到另一个分支。

**合并流程：**
```bash
# 1. 切换到目标分支
git checkout master

# 2. 合并源分支
git merge feature-login
```

**合并策略：**
```bash
# Fast-forward合并（线性历史）
git merge --ff feature-login

# 强制创建合并提交（保留分支历史）
git merge --no-ff feature-login
```

**查看合并结果：**
```bash
git log --graph --oneline  # 查看分支合并图
```

---

### 3.4 删除分支

**删除已合并的分支：**
```bash
git branch -d feature-login  # 删除本地分支
```

**强制删除未合并的分支：**
```bash
git branch -D feature-login  # 强制删除（慎用）
```

**删除远程分支：**
```bash
git push origin --delete feature-login
```

---

### 3.5 合并分支冲突问题

分支合并时可能产生冲突，解决方法与拉取冲突类似。

**冲突场景：**
```bash
# 合并时遇到冲突
git merge feature-login

# 查看冲突文件
git status
```

**解决步骤：**
```bash
# 1. 查看冲突内容
git diff  # 显示冲突区域

# 2. 手动编辑解决冲突
# 移除<<<<<<<、=======、>>>>>>>标记

# 3. 暂存解决后的文件
git add conflicted-file.java

# 4. 完成合并
git commit
```

**实用技巧：**
- 合并冲突后，使用`git merge --abort`可以取消合并
- 推荐在合并前先`git pull`确保本地分支是最新的

---

## 4 Git标签

### 4.1 标签概念

标签（Tag）用于标记重要的版本节点，如发布版本。

**标签与分支的区别：**
| 特性 | 标签 | 分支 |
|------|------|------|
| 状态 | 静态快照 | 动态指针 |
| 用途 | 标记版本 | 开发迭代 |
| 变更 | 不可移动 | 可移动 |

**常用标签类型：**
- **轻量标签（Lightweight）**：简单的引用，记录提交哈希
- **附注标签（Annotated）**：包含详细信息，推荐使用

---

### 4.2 创建标签

**创建轻量标签：**
```bash
git tag v1.0.0  # 基于当前提交创建
```

**创建附注标签：**
```bash
# 创建带信息的附注标签
git tag -a v1.0.0 -m "版本1.0.0发布"

# 为历史提交创建标签
git tag -a v1.0.0 <commit-hash>
```

**查看标签：**
```bash
git tag  # 列出所有标签
git show v1.0.0  # 查看标签详情
```

---

### 4.3 检出与删除标签

**检出标签：**
```bash
# 创建临时分支并检出标签
git checkout -b release-v1.0.0 v1.0.0
```

**推送标签到远程：**
```bash
git push origin v1.0.0  # 推送单个标签
git push origin --tags  # 推送所有标签
```

**删除本地标签：**
```bash
git tag -d v1.0.0
```

**删除远程标签：**
```bash
git push origin :v1.0.0
```

---

## 5 Git工作流

### 5.1 Git Flow工作流

Git Flow是经典的分支管理模型，适用于大型项目。

**分支类型：**
| 分支 | 用途 | 生命周期 |
|------|------|----------|
| **main/master** | 生产环境代码 | 永久存在 |
| **develop** | 开发集成分支 | 永久存在 |
| **feature** | 功能开发 | 临时，完成后合并到develop |
| **release** | 发布准备 | 临时，完成后合并到main和develop |
| **hotfix** | 紧急修复 | 临时，完成后合并到main和develop |

**工作流程：**
```bash
# 1. 从develop创建功能分支
git checkout -b feature/user-auth develop

# 2. 开发完成后合并到develop
git checkout develop
git merge --no-ff feature/user-auth
git branch -d feature/user-auth

# 3. 创建发布分支
git checkout -b release/v1.0.0 develop

# 4. 发布完成后合并到main和develop
git checkout main
git merge --no-ff release/v1.0.0
git tag -a v1.0.0 -m "版本1.0.0"

git checkout develop
git merge --no-ff release/v1.0.0

# 5. 紧急修复
git checkout -b hotfix/v1.0.1 main
# 修复bug后
git checkout main
git merge --no-ff hotfix/v1.0.1
git tag -a v1.0.1 -m "紧急修复v1.0.1"

git checkout develop
git merge --no-ff hotfix/v1.0.1
```

---

### 5.2 GitHub Flow工作流

GitHub Flow是简化的工作流，适用于敏捷开发。

**核心原则：**
1. 主分支`main`始终是可部署的
2. 新功能创建新分支（从`main`创建）
3. 提交后创建Pull Request
4. 代码审查通过后合并到`main`
5. 合并后立即部署

**工作流程：**
```bash
# 1. 确保main分支是最新的
git checkout main
git pull origin main

# 2. 创建功能分支
git checkout -b feature/new-feature

# 3. 开发并提交
git add .
git commit -m "feat: 添加新功能"

# 4. 推送到远程
git push -u origin feature/new-feature

# 5. 创建Pull Request（在GitHub界面操作）

# 6. 审查通过后合并（在GitHub界面操作）

# 7. 删除本地分支
git checkout main
git pull origin main
git branch -d feature/new-feature
```

---

### 5.3 Git Commit Message规范

企业级项目中，统一的提交信息规范是团队协作的基础。采用**Conventional Commits**规范，便于自动化生成CHANGELOG、版本管理和代码审查。

**提交信息格式：**
```bash
<type>(<scope>): <description>

<body>

<footer>
```

**Type类型说明：**
| Type | 说明 | 示例 |
|------|------|------|
| **feat** | 新增功能 | feat(user): 添加用户注册接口 |
| **fix** | 修复bug | fix(auth): 修复token过期后无法刷新 |
| **docs** | 文档更新 | docs(readme): 更新API接口说明 |
| **style** | 代码格式调整（不影响功能） | style(utils): 格式化代码缩进 |
| **refactor** | 重构（既不新增功能也不修复bug） | refactor(service): 优化查询逻辑 |
| **test** | 测试相关 | test(user): 添加用户登录单元测试 |
| **chore** | 构建/工具更新 | chore(build): 更新Maven依赖版本 |
| **perf** | 性能优化 | perf(db): 优化数据库查询索引 |
| **revert** | 回滚提交 | revert: 撤销"feat: 添加支付功能" |

**Scope说明：**
- 表示修改的模块或范围
- 如：user、auth、order、payment、api等
- 可选，如不明确可省略

**企业真实案例：**

**案例1：功能开发**
```bash
feat(order): 实现订单创建功能

- 创建订单接口 /api/orders
- 实现库存扣减逻辑
- 添加订单状态流转
- 关联用户ID和收货地址
```

**案例2：Bug修复**
```bash
fix(payment): 修复微信支付回调签名验证失败

原因：微信SDK升级后签名算法变更，旧版验签逻辑不兼容
修复：升级wechat-pay-sdk至3.0版本，更新验签方法

Closes: #123
```

**案例3：代码重构**
```bash
refactor(user): 重构用户服务层代码

- 将用户查询逻辑抽取为UserQueryService
- 统一异常处理，移除重复try-catch
- 优化DTO转换，使用MapStruct替代手动转换
- 代码覆盖率提升至85%
```

**案例4：紧急修复（Hotfix）**
```bash
fix(order): 紧急修复订单超时未取消问题

紧急修复：订单超过30分钟未支付应自动取消，之前定时任务未正确执行
修复方案：
1. 修复定时任务cron表达式错误
2. 添加手动触发取消接口
3. 增加日志记录便于排查

Urgent: 生产环境已受影响，需立即部署
```

**案例5：文档更新**
```bash
docs(api): 更新用户模块API文档

- 补充用户注册接口返回字段说明
- 添加分页查询参数示例
- 修正错误的接口路径
```

**案例6：回滚提交**
```bash
revert: feat(payment): 添加支付宝支付功能

原因：支付宝支付接口联调未完成，上线后导致支付失败
回滚至提交：a1b2c3d4e5f6
```

**提交信息编写原则：**
1. **简洁明了**：标题不超过50个字符，清晰表达变更内容
2. **使用中文**：便于团队协作和代码审查（根据团队约定）
3. **描述原因**：body部分说明修改的原因和实现细节
4. **关联任务**：footer部分关联Issue编号或需求ID
5. **避免模糊**：不用"修改代码"、"更新功能"等笼统描述

**自动化校验（配合commit-msg钩子）：**
```bash
#!/bin/bash
# commit-msg钩子脚本

COMMIT_MSG=$(cat "$1")

# 检查标题格式
if ! echo "$COMMIT_MSG" | head -1 | grep -qE "^(feat|fix|docs|style|refactor|test|chore|perf|revert)\([a-z0-9_-]+\): "; then
  echo "❌ Commit message格式错误！"
  echo "✅ 正确格式：feat(module): 描述内容"
  echo "📋 Type可选值：feat|fix|docs|style|refactor|test|chore|perf|revert"
  exit 1
fi

# 检查标题长度
TITLE_LENGTH=$(echo "$COMMIT_MSG" | head -1 | wc -c)
if [ $TITLE_LENGTH -gt 50 ]; then
  echo "❌ 标题超过50个字符！当前长度：$TITLE_LENGTH"
  exit 1
fi
```

---

### 5.4 Dev-Sit-Prd企业级工作流

这是一套完整的企业级开发流程，覆盖从需求开发到生产发布的全生命周期。

**分支命名规范：**
| 分支类型 | 命名格式 | 示例 |
|----------|----------|------|
| 主分支 | main/master | main |
| 开发分支 | develop | develop |
| 功能分支 | feature/{模块}-{功能} | feature/user-login |
| 测试分支 | sit | sit |
| 预发布分支 | release/{版本号} | release/v1.0.0 |
| 紧急修复 | hotfix/{版本号} | hotfix/v1.0.1 |

**完整工作流程：**

#### 阶段一：项目初始化

**架构师创建项目基础：**
```bash
# 1. 创建远程仓库（在Git平台操作）
# 2. 克隆到本地
git clone git@gitlab.example.com:project/repo.git
cd repo

# 3. 架构师提交基础架构代码
git add .
git commit -m "chore(project): 初始化项目架构"
git push origin main

# 4. 打初始标签
git tag -a v0.1.0 -m "项目初始化完成"
git push origin v0.1.0

# 5. 从main拉取develop分支
git checkout -b develop
git push origin develop
```

#### 阶段二：功能开发（Dev阶段）

**程序员1开发用户登录功能：**
```bash
# 1. 切换到develop并拉取最新代码
git checkout develop
git pull origin develop

# 2. 创建功能分支
git checkout -b feature/user-login

# 3. 开发并提交（多次迭代）
git add .
git commit -m "feat(user): 实现用户登录接口"
git add .
git commit -m "fix(user): 修复密码加密逻辑"
git add .
git commit -m "test(user): 添加登录单元测试"
```

**程序员2开发订单功能：**
```bash
git checkout develop
git pull origin develop
git checkout -b feature/order-create
git add .
git commit -m "feat(order): 实现订单创建功能"
git add .
git commit -m "fix(order): 修复库存扣减并发问题"
```

**自测完成后合并到develop：**
```bash
# 程序员1合并登录功能
git checkout develop
git pull origin develop
git merge --no-ff feature/user-login
git push origin develop
git branch -d feature/user-login

# 程序员2合并订单功能
git checkout develop
git pull origin develop
git merge --no-ff feature/order-create
git push origin develop
git branch -d feature/order-create
```

#### 阶段三：测试环境（Sit阶段）

**测试人员创建SIT分支：**
```bash
git checkout develop
git pull origin develop
git checkout -b sit
git push origin sit
```

**测试发现Bug，程序员修复：**
```bash
# 程序员从develop拉取bug修复分支
git checkout develop
git pull origin develop
git checkout -b feature/bugfix-login-error

# 修复bug
git add .
git commit -m "fix(user): 修复登录失败返回错误码"

# 合并到develop
git checkout develop
git merge --no-ff feature/bugfix-login-error
git push origin develop
git branch -d feature/bugfix-login-error
```

**测试将develop改动合并到sit：**
```bash
git checkout sit
git pull origin sit
git merge --no-ff develop
git push origin sit
```

**重复以上流程，直到测试通过。**

#### 阶段四：预发布（Release阶段）

**创建Release分支：**
```bash
git checkout develop
git pull origin develop
git checkout -b release/v1.0.0
git push origin release/v1.0.0
```

**合并SIT测试结果到Release：**
```bash
git checkout release/v1.0.0
git pull origin release/v1.0.0
git merge --no-ff sit
git push origin release/v1.0.0
```

**预发布环境测试：**
- 测试人员在预发布环境进行最终验证
- 发现问题→打回开发→修复后合并到develop和release
- 测试通过→进入生产发布

#### 阶段五：生产发布（Prd阶段）

**合并到main并打标签：**
```bash
git checkout main
git pull origin main
git merge --no-ff release/v1.0.0
git push origin main

# 打发布标签
git tag -a v1.0.0 -m "正式发布版本1.0.0"
git push origin v1.0.0
```

**同步到develop：**
```bash
git checkout develop
git pull origin develop
git merge --no-ff release/v1.0.0
git push origin develop
```

**删除临时分支：**
```bash
git branch -d release/v1.0.0
git push origin --delete release/v1.0.0
git push origin --delete sit
```

#### 阶段六：紧急修复（Hotfix）

**生产环境发现Bug，需要紧急修复：**
```bash
# 1. 从main拉取hotfix分支
git checkout main
git pull origin main
git checkout -b hotfix/v1.0.1

# 2. 修复bug
git add .
git commit -m "fix(order): 紧急修复订单支付失败问题"

# 3. 测试验证后合并到main
git checkout main
git merge --no-ff hotfix/v1.0.1
git push origin main

# 4. 打紧急修复标签
git tag -a v1.0.1 -m "紧急修复v1.0.1"
git push origin v1.0.1

# 5. 同步到develop
git checkout develop
git pull origin develop
git merge --no-ff hotfix/v1.0.1
git push origin develop

# 6. 删除hotfix分支
git branch -d hotfix/v1.0.1
git push origin --delete hotfix/v1.0.1
```

**工作流流程图：**
```
main ────────────────────────────────────────●─────────●──●
       \                                     │         │  │
        \   develop ──●──●──●──●──●─────────┼─────────┼──┘
         \      \      /  /  /  /  /         │         │
          \     feat1─●  /  /  /  /          │         │
           \          \ /  /  /  /           │         │
            \       feat2─●  /  /            │         │
             \             /  /              │         │
              \    sit ───●──●              /         │
               \                            /         │
                \   release/v1.0.0 ────────●          │
                 \                                     │
                  \                        hotfix/v1.0.1
                   \                               │
                    └───────────────────────────────┘
```

**关键规则总结：**
| 规则 | 说明 |
|------|------|
| **分支保护** | main分支设置保护，禁止直接推送，必须通过PR/MR合并 |
| **代码审查** | 功能分支合并到develop前必须经过Code Review |
| **CI/CD集成** | 提交自动触发构建和测试，失败则不能合并 |
| **标签管理** | 每次发布必须打版本标签，便于追溯和回滚 |
| **文档同步** | 发布前更新CHANGELOG和版本说明文档 |
| **回滚策略** | 保留历史标签，出现问题可快速回滚到上一版本 |

---

## 6 在IDEA中使用Git

### 6.1 配置Git环境

**配置步骤：**
1. 打开IDEA → File → Settings
2. 找到Version Control → Git
3. 在Path to Git executable中选择Git安装路径
4. 点击Test按钮验证配置

**配置界面：**
```
Settings > Version Control > Git
  ├── Path to Git executable: C:\Program Files\Git\bin\git.exe
  ├── Test (验证配置)
  └── SSH executable: Native
```

---

### 6.2 Git基本操作

**初始化仓库：**
```
右键项目 → Git → Initialize Repository
```

**添加文件：**
```
右键文件 → Git → Add
```

**提交变更：**
```
Ctrl+K (或右键 → Git → Commit)
填写提交信息 → Commit
```

**查看状态：**
```
底部状态栏 → Git标签页
或 View → Tool Windows → Version Control
```

**查看日志：**
```
右键项目 → Git → Show History
```

---

### 6.3 Git分支操作

**创建分支：**
```
底部状态栏 → 点击分支名称 → New Branch
输入分支名称 → Create
```

**切换分支：**
```
底部状态栏 → 点击分支名称 → 选择目标分支
```

**合并分支：**
```
Git标签页 → Branches → 右键目标分支 → Merge into Current
```

**删除分支：**
```
Git标签页 → Branches → 右键分支 → Delete
```

---

### 6.4 解决冲突

**冲突提示：**
```
IDEA会自动检测冲突，显示冲突文件列表
冲突文件会标记为红色
```

**解决步骤：**
```
1. 打开冲突文件
2. IDEA会显示三个版本：
   - Left：本地版本
   - Right：远程版本
   - Merged：合并后版本

3. 选择保留的内容：
   - Accept Left：保留本地
   - Accept Right：保留远程
   - Accept Both：两者都保留

4. 点击Apply应用更改

5. 右键文件 → Git → Add

6. 提交合并结果
```

**实用技巧：**
- 使用`Resolve Conflicts`对话框可视化解决冲突
- 冲突解决后，记得提交合并结果

---

## 7 Git进阶技巧

### 7.1 交互式变基（Rebase）

变基（Rebase）将当前分支的提交"重新播放"到目标分支之上，产生线性历史。

**基本变基：**
```bash
# 将feature分支变基到main分支
git checkout feature
git rebase main
```

**交互式变基：**
```bash
# 编辑最近3个提交
git rebase -i HEAD~3
```

**交互式变基命令：**
```bash
# 变基编辑界面（Vim格式）
pick 1a2b3c4 feat: 添加登录功能
pick 5d6e7f8 fix: 修复密码验证
pick 9g0h1i2 docs: 更新文档

# 常用命令：
# pick：保留提交
# reword：修改提交信息
# edit：修改提交内容
# squash：合并到上一个提交
# drop：删除提交
```

**Rebase vs Merge：**
| 特性 | Rebase | Merge |
|------|--------|-------|
| 历史 | 线性 | 保留分支结构 |
| 冲突 | 逐个解决 | 一次性解决 |
| 场景 | 个人特性分支 | 公共分支合并 |

**实用技巧：**
- 不要对已推送的公共分支执行rebase
- 使用`git rebase --abort`取消变基
- 使用`git rebase --continue`继续解决冲突后的变基

---

### 7.2 临时存储（Stash）

Stash用于临时保存工作区变更，便于切换分支或处理紧急任务。

**保存工作区：**
```bash
git stash  # 保存当前工作区和暂存区
```

**保存并添加备注：**
```bash
git stash push -m "wip: 用户登录功能"
```

**查看stash列表：**
```bash
git stash list  # 显示所有保存的stash
```

**恢复stash：**
```bash
git stash pop  # 恢复最近的stash并删除
git stash apply  # 恢复stash但不删除
```

**恢复指定stash：**
```bash
git stash pop stash@{0}  # 恢复第一个stash
```

**查看stash内容：**
```bash
git stash show  # 显示最近stash的差异
git stash show -p  # 显示详细差异
```

**删除stash：**
```bash
git stash drop  # 删除最近的stash
git stash clear  # 删除所有stash
```

---

### 7.3 挑选提交（Cherry-Pick）

Cherry-Pick将指定提交复制到当前分支。

**挑选单个提交：**
```bash
git cherry-pick <commit-hash>
```

**挑选多个提交：**
```bash
git cherry-pick commit1 commit2  # 挑选多个提交
```

**挑选连续提交范围：**
```bash
git cherry-pick A..B  # 挑选A到B之间的提交（不含A）
```

**处理冲突：**
```bash
# cherry-pick遇到冲突时
git cherry-pick --continue  # 解决冲突后继续
git cherry-pick --abort  # 取消操作
```

**实用场景：**
- 将bug修复从feature分支复制到main分支
- 提取特定功能到另一个分支

---

### 7.4 二分查找（Bisect）

Bisect用于快速定位引入bug的提交。

**开始二分查找：**
```bash
git bisect start  # 开始二分查找
git bisect bad  # 标记当前提交为有bug
git bisect good v1.0.0  # 标记已知无bug的提交
```

**测试当前提交：**
```bash
# Git会自动切换到中间提交，运行测试
npm test  # 或其他测试命令
git bisect good  # 如果测试通过
git bisect bad  # 如果测试失败
```

**完成查找：**
```bash
git bisect reset  # 结束二分查找，恢复到原分支
```

**自动化测试：**
```bash
# 自动运行测试脚本
git bisect run npm test
```

**实用技巧：**
- 二分查找的时间复杂度为O(log n)
- 需要标记一个已知好的提交和一个已知坏的提交

---

### 7.5 Git钩子（Hooks）

钩子（Hooks）是在特定Git事件触发时执行的脚本。

**钩子位置：**
```bash
.git/hooks/  # 钩子脚本存放目录
```

**常用钩子：**
| 钩子 | 触发时机 | 用途 |
|------|----------|------|
| `pre-commit` | 提交前 | 代码检查、格式化 |
| `commit-msg` | 提交信息校验 | 强制提交规范 |
| `pre-push` | 推送前 | 运行测试、检查 |
| `post-receive` | 服务端接收后 | 部署、通知 |

**创建pre-commit钩子：**
```bash
#!/bin/bash
# 代码格式检查
echo "Running code format check..."
npm run lint
if [ $? -ne 0 ]; then
  echo "Lint failed, aborting commit."
  exit 1
fi
```

**创建commit-msg钩子：**
```bash
#!/bin/bash
# 检查提交信息是否符合规范
COMMIT_MSG=$(cat "$1")
if ! echo "$COMMIT_MSG" | grep -qE "^(feat|fix|docs|style|refactor|test|chore)\(.*\): "; then
  echo "Commit message format error."
  echo "Expected: feat(scope): description"
  exit 1
fi
```

**启用钩子：**
```bash
chmod +x .git/hooks/pre-commit
chmod +x .git/hooks/commit-msg
```

**共享钩子：**
```bash
# 将钩子放入项目目录
mkdir -p .githooks
mv .git/hooks/pre-commit .githooks/

# 配置Git使用自定义钩子目录
git config core.hooksPath .githooks
```
