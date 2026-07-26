# Maven 核心技术手册

  **阅读指南**：由浅入深，每个知识点均配有精简示例。建议按顺序阅读，环环相扣。

---

## 📑 目录

- [**第一部分：Maven 入门与环境配置**](#第一部分maven-入门与环境配置)
  - [1.1 Maven 概述与核心概念](#11-maven-概述与核心概念)
  - [1.2 环境安装与配置](#12-环境安装与配置)
  - [1.3 Maven 仓库类型与配置](#13-maven-仓库类型与配置)
- [**第二部分：Maven 项目结构与生命周期**](#第二部分maven-项目结构与生命周期)
  - [2.1 Maven 项目标准结构](#21-maven-项目标准结构)
  - [2.2 项目生命周期与阶段](#22-项目生命周期与阶段)
  - [2.3 Maven 常用命令](#23-maven-常用命令)
- [**第三部分：POM 文件详解**](#第三部分pom-文件详解)
  - [3.1 POM 文件结构](#31-pom-文件结构)
  - [3.2 依赖管理](#32-依赖管理)
  - [3.3 依赖范围](#33-依赖范围)
  - [3.4 依赖冲突与解决](#34-依赖冲突与解决)
- [**第四部分：Maven 聚合与继承**](#第四部分maven-聚合与继承)
  - [4.1 聚合关系（多模块项目）](#41-聚合关系多模块项目)
  - [4.2 继承关系（父工程）](#42-继承关系父工程)
  - [4.3 聚合案例实战](#43-聚合案例实战)
- [**第五部分：Maven 测试与插件**](#第五部分maven-测试与插件)
  - [5.1 JUnit 测试集成](#51-junit-测试集成)
  - [5.2 Maven 常用插件](#52-maven-常用插件)
  - [5.3 IDEA 中配置 Maven](#53-idea-中配置-maven)
- [**第六部分：高级特性与最佳实践**](#第六部分高级特性与最佳实践)
  - [6.1 依赖传递失效及解决方案](#61-依赖传递失效及解决方案)
  - [6.2 版本锁定与属性管理](#62-版本锁定与属性管理)
  - [6.3 Maven 最佳实践](#63-maven-最佳实践)

---

## 第一部分：Maven 入门与环境配置

### 1.1 Maven 概述与核心概念

```
┌─────────────────────────────────────────────────────────────┐
│                    Maven 核心价值                           │
├─────────────────────────────────────────────────────────────┤
│  ① 依赖管理：自动下载、管理第三方库                          │
│  ② 一键构建：compile → test → package → deploy            │
│  ③ 标准化：统一项目结构，降低学习成本                         │
│  ④ 插件生态：丰富的插件支持各种构建需求                       │
└─────────────────────────────────────────────────────────────┘
```

| 核心概念 | 说明 |
|----------|------|
| **POM** | Project Object Model，项目对象模型，Maven 的核心配置文件 |
| **坐标** | `groupId:artifactId:version`，唯一标识一个依赖 |
| **仓库** | 存放依赖的地方（本地/远程/中央） |
| **生命周期** | 一套标准化的构建流程（clean/default/site） |
| **插件** | 执行具体构建任务的组件 |

### 1.2 环境安装与配置

```bash
# 1. 下载解压：https://maven.apache.org/download.cgi
# 2. 配置环境变量 MAVEN_HOME = D:\apache-maven-3.9.x
# 3. 验证安装
mvn -v  # 显示版本信息即成功
```

```xml
<!-- conf/settings.xml 核心配置 -->
<settings>
    <!-- 本地仓库位置（建议改到非系统盘） -->
    <localRepository>D:\maven\repository</localRepository>
    
    <!-- 镜像配置（加速下载） -->
    <mirrors>
        <mirror>
            <id>aliyunmaven</id>
            <mirrorOf>central</mirrorOf>
            <url>https://maven.aliyun.com/repository/public</url>
        </mirror>
    </mirrors>
</settings>
```

### 1.3 Maven 仓库类型与配置

```
仓库层级关系：
本地仓库 → 镜像仓库 → 中央仓库（repo.maven.apache.org）

本地仓库：本机缓存，首次下载后复用
镜像仓库：中央仓库的镜像（阿里云/华为云等，速度快）
中央仓库：官方仓库，包含绝大多数开源依赖
```

```xml
<!-- 私有仓库配置（企业内部） -->
<repositories>
    <repository>
        <id>company-repo</id>
        <url>http://nexus.company.com/repository/maven-public/</url>
    </repository>
</repositories>
```

---

## 第二部分：Maven 项目结构与生命周期

### 2.1 Maven 项目标准结构

```
my-project/                    # 项目根目录
├── src/
│   ├── main/                  # 主代码
│   │   ├── java/              # Java 源文件
│   │   │   └── com/example/
│   │   └── resources/         # 配置文件（application.properties）
│   └── test/                  # 测试代码
│       ├── java/              # 测试类
│       └── resources/         # 测试配置
└── pom.xml                    # Maven 核心配置文件
```

```bash
# 一键创建标准项目结构
mvn archetype:generate -DgroupId=com.example -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

### 2.2 项目生命周期与阶段

```
三套生命周期（互不干扰）：
① clean：清理之前的构建产物
② default：核心构建流程（最常用）
③ site：生成项目文档
```

| default 生命周期阶段 | 说明 |
|----------------------|------|
| `validate` | 验证项目结构是否正确 |
| `compile` | 编译主代码 |
| `test` | 执行测试 |
| `package` | 打包（jar/war） |
| `install` | 安装到本地仓库 |
| `deploy` | 部署到远程仓库 |

```bash
# 执行生命周期会自动执行所有前置阶段
mvn package  # 自动执行 validate → compile → test → package
```

### 2.3 Maven 常用命令

```bash
mvn clean                    # 清理target目录
mvn compile                  # 编译主代码
mvn test                     # 执行测试
mvn package                  # 打包
mvn install                  # 安装到本地仓库（供其他项目引用）
mvn deploy                   # 部署到远程仓库
mvn clean install            # 清理后重新安装（常用组合）
mvn -DskipTests package      # 打包时跳过测试（加快速度）
mvn dependency:tree          # 查看依赖树（排查冲突）
mvn dependency:analyze       # 分析依赖使用情况
```

---

## 第三部分：POM 文件详解

### 3.1 POM 文件结构

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <!-- 项目坐标（必须） -->
    <groupId>com.example</groupId>      <!-- 组织ID，通常是域名反写 -->
    <artifactId>my-app</artifactId>     <!-- 项目ID -->
    <version>1.0.0-SNAPSHOT</version>   <!-- 版本号 -->
    <packaging>jar</packaging>          <!-- 打包类型：jar/war/pom -->
    
    <name>My App</name>
    <description>A Maven project</description>
    
    <!-- 依赖列表 -->
    <dependencies>
        <!-- 见3.2节 -->
    </dependencies>
</project>
```

### 3.2 依赖管理

```xml
<dependencies>
    <!-- 标准依赖配置 -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13.2</version>
        <scope>test</scope>  <!-- 依赖范围，见3.3节 -->
    </dependency>
    
    <!-- Spring Boot 起步依赖（自动管理版本） -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <!-- 继承parent后无需指定version -->
    </dependency>
</dependencies>
```

### 3.3 依赖范围

| 范围 | 编译 | 测试 | 运行 | 说明 |
|------|------|------|------|------|
| `compile` | ✅ | ✅ | ✅ | 默认，所有阶段有效 |
| `test` | ❌ | ✅ | ❌ | 仅测试时使用（JUnit） |
| `provided` | ✅ | ✅ | ❌ | 运行时由容器提供（Servlet API） |
| `runtime` | ❌ | ✅ | ✅ | 运行时需要（JDBC驱动） |
| `system` | ✅ | ✅ | ❌ | 本地系统依赖（不推荐） |

```xml
<!-- Servlet API：Tomcat已提供，无需打包 -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
</dependency>
```

### 3.4 依赖冲突与解决

```
依赖冲突产生原因：
项目依赖A，A依赖B(1.0)，项目又直接依赖B(2.0)，导致版本冲突
```

| 解决原则 | 说明 |
|----------|------|
| **最短路径优先** | 直接依赖优先于传递依赖 |
| **最先声明优先** | 同一路径下，pom中先声明的优先 |

```xml
<!-- 排除依赖（排除不需要的传递依赖） -->
<dependency>
    <groupId>com.example</groupId>
    <artifactId>problematic-lib</artifactId>
    <exclusions>
        <exclusion>
            <groupId>conflicting-group</groupId>
            <artifactId>conflicting-artifact</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<!-- 锁定版本（强制指定版本） -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>target-lib</artifactId>
            <version>2.0.0</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

---

## 第四部分：Maven 聚合与继承

### 4.1 聚合关系（多模块项目）

```
聚合关系：将多个模块组织在一起，统一构建

my-project/（父工程，packaging=pom）
├── pom.xml              # 聚合配置
├── module-dao/          # 数据访问层
├── module-service/      # 业务逻辑层
└── module-web/          # 表现层（依赖dao和service）
```

```xml
<!-- 父工程 pom.xml：聚合配置 -->
<project>
    <groupId>com.example</groupId>
    <artifactId>my-project</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>  <!-- 聚合工程必须是pom -->
    
    <modules>
        <module>module-dao</module>
        <module>module-service</module>
        <module>module-web</module>
    </modules>
</project>
```

### 4.2 继承关系（父工程）

```
继承关系：子模块继承父工程的配置（依赖、插件等），避免重复配置
```

```xml
<!-- 父工程 pom.xml：供子模块继承 -->
<project>
    <groupId>com.example</groupId>
    <artifactId>my-parent</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>
    
    <!-- 子模块可继承的依赖管理 -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>4.13.2</version>
                <scope>test</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

```xml
<!-- 子模块 pom.xml：继承父工程 -->
<project>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>my-parent</artifactId>
        <version>1.0.0</version>
        <relativePath>../pom.xml</relativePath>  <!-- 父工程路径 -->
    </parent>
    
    <artifactId>module-dao</artifactId>  <!-- 只需指定自身ID -->
    <!-- 可直接使用父工程dependencyManagement中的依赖，无需指定version -->
</project>
```

### 4.3 聚合案例实战

```bash
# 1. 创建父工程（packaging=pom）
mvn archetype:generate -DgroupId=com.example -DartifactId=myproject -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

# 2. 修改父工程pom.xml，添加modules配置
# 3. 在父工程目录下创建子模块
cd myproject
mvn archetype:generate -DgroupId=com.example -DartifactId=dao -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

# 4. 子模块间依赖
# module-web 的 pom.xml 添加对 module-service 的依赖
<dependency>
    <groupId>com.example</groupId>
    <artifactId>service</artifactId>
    <version>1.0.0</version>
</dependency>

# 5. 在父工程目录执行，一键构建所有模块
mvn clean install
```

---

## 第五部分：Maven 测试与插件

### 5.1 JUnit 测试集成

```xml
<!-- 添加JUnit依赖 -->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>
```

```java
// src/test/java/com/example/MyTest.java
import org.junit.Test;
import static org.junit.Assert.assertEquals;

public class MyTest {
    @Test
    public void testAdd() {
        int result = 1 + 1;
        assertEquals(2, result);  // 断言：期望值，实际值
    }
}
```

```bash
# 执行测试
mvn test

# 跳过测试（打包时常用）
mvn package -DskipTests

# 只运行指定测试
mvn test -Dtest=MyTest
```

| 注解 | 说明 |
|------|------|
| `@Test` | 标记测试方法 |
| `@Before` | 每个测试方法执行前运行 |
| `@After` | 每个测试方法执行后运行 |
| `@BeforeClass` | 类加载时运行一次 |
| `@AfterClass` | 类销毁时运行一次 |

### 5.2 Maven 常用插件

```xml
<build>
    <plugins>
        <!-- 编译插件（指定Java版本） -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.1</version>
            <configuration>
                <source>11</source>
                <target>11</target>
                <encoding>UTF-8</encoding>
            </configuration>
        </plugin>
        
        <!-- 打包插件（指定主类） -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <configuration>
                <archive>
                    <manifest>
                        <mainClass>com.example.Main</mainClass>
                    </manifest>
                </archive>
            </configuration>
        </plugin>
        
        <!-- 跳过测试插件配置 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <configuration>
                <skipTests>true</skipTests>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### 5.3 IDEA 中配置 Maven

```
① File → Settings → Build, Execution, Deployment → Build Tools → Maven
② 设置 Maven home directory：选择本地 Maven 安装目录
③ 设置 User settings file：选择 settings.xml
④ 设置 Local repository：选择本地仓库目录
⑤ 勾选 "Override" 生效配置

⑥ 新建 Maven 项目：New → Project → Maven → 选择 archetype
⑦ 已有项目：右键项目 → Maven → Reload Project
```

---

## 第六部分：高级特性与最佳实践

### 6.1 依赖传递失效及解决方案

```
依赖传递失效场景：
① 依赖范围为 test 或 provided
② 排除依赖（exclusion）
③ 版本冲突导致某些传递依赖被覆盖
```

```xml
<!-- 解决方案1：直接声明失效的依赖 -->
<dependency>
    <groupId>missing-group</groupId>
    <artifactId>missing-artifact</artifactId>
    <version>1.0.0</version>
</dependency>

<!-- 解决方案2：检查依赖树 -->
mvn dependency:tree | findstr "missing-artifact"  # Windows
mvn dependency:tree | grep "missing-artifact"     # Linux/Mac
```

### 6.2 版本锁定与属性管理

```xml
<!-- 使用properties统一管理版本 -->
<properties>
    <java.version>11</java.version>
    <spring.version>5.3.20</spring.version>
    <mysql.version>8.0.28</mysql.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql.version}</version>
    </dependency>
</dependencies>
```

```xml
<!-- 使用dependencyManagement统一锁定版本 -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>5.3.20</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 6.3 Maven 最佳实践

```
✅ 推荐：
① 使用依赖管理（dependencyManagement）统一版本
② 使用properties管理版本号，便于维护
③ 优先使用镜像仓库加速下载
④ 合理使用依赖范围（test/provided等）
⑤ 使用mvn dependency:tree排查冲突
⑥ 多模块项目使用聚合+继承

❌ 避免：
① 重复声明相同依赖（使用继承）
② 使用system范围依赖（无法自动下载）
③ 忽略依赖冲突（可能导致运行时错误）
④ 本地仓库路径包含中文或空格
⑤ 直接修改中央仓库依赖的版本
```

```bash
# 常用检查命令
mvn dependency:tree          # 查看完整依赖树
mvn dependency:analyze       # 分析未使用的依赖
mvn validate                 # 验证项目配置
mvn clean install -X         # 调试模式（显示详细日志）
```

---

**总结**：Maven 的核心价值在于**标准化、自动化、依赖管理**。掌握 POM 配置、生命周期、依赖范围是基础，熟练运用聚合继承、依赖冲突解决是进阶能力。