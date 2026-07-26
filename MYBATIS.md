# MyBatis 核心技术手册

   **阅读指南**：由浅入深，每个知识点均配有精简示例。建议按顺序阅读，环环相扣。

   ---

## 目录

### 1. 基础入门
- [1.1 MyBatis概述](#11-mybatis概述)
- [1.2 环境搭建](#12-环境搭建)
- [1.3 持久层接口和映射文件](#13-持久层接口和映射文件)
- [1.4 测试持久层接口方法](#14-测试持久层接口方法)
- [1.5 MyBatis核心对象和工作流程](#15-mybatis核心对象和工作流程)

### 2. 核心配置
- [2.1 properties属性配置](#21-properties属性配置)
- [2.2 typeAliases类型别名](#22-typealiases类型别名)
- [2.3 environments环境配置](#23-environments环境配置)
- [2.4 mappers映射器注册](#24-mappers映射器注册)

### 3. 映射文件
- [3.1 resultMap结果映射](#31-resultmap结果映射)
- [3.2 sql&include片段复用](#32-sqlinclude片段复用)
- [3.3 特殊字符处理](#33-特殊字符处理)
- [3.4 增删改查基础操作](#34-增删改查基础操作)

### 4. 动态SQL
- [4.1 if条件判断](#41-if条件判断)
- [4.2 where&set标签](#42-whereset标签)
- [4.3 choose&when&otherwise](#43-choosewhenotherwise)
- [4.4 foreach遍历](#44-foreach遍历)

### 5. 关联查询
- [5.1 一对一关联查询](#51-一对一关联查询)
- [5.2 一对多关联查询](#52-一对多关联查询)
- [5.3 多对多关联查询](#53-多对多关联查询)
- [5.4 分解式查询与延迟加载](#54-分解式查询与延迟加载)

### 6. 缓存机制
- [6.1 一级缓存](#61-一级缓存)
- [6.2 清除一级缓存](#62-清除一级缓存)
- [6.3 二级缓存](#63-二级缓存)

### 7. 注解开发
- [7.1 注解开发基础](#71-注解开发基础)
- [7.2 注解增删改查](#72-注解增删改查)
- [7.3 注解动态SQL](#73-注解动态sql)
- [7.4 注解自定义映射](#74-注解自定义映射)

### 8. 高级特性
- [8.1 插件机制(Interceptor)](#81-插件机制interceptor)
- [8.2 TypeHandler类型处理器](#82-typehandler类型处理器)
- [8.3 批量操作优化](#83-批量操作优化)
- [8.4 分页插件PageHelper](#84-分页插件pagehelper)

### 9. 生态集成
- [9.1 MyBatis与Spring集成](#91-mybatis与spring集成)
- [9.2 MyBatis与SpringBoot集成](#92-mybatis与springboot集成)
- [9.3 事务管理](#93-事务管理)

### 10. 最佳实践
- [10.1 性能优化](#101-性能优化)
- [10.2 常见问题排查](#102-常见问题排查)
- [10.3 MyBatisGenerator代码生成](#103-mybatisgenerator代码生成)

---

## 1. 基础入门

### 1.1 MyBatis概述

**MyBatis** 是一款优秀的持久层框架，它支持定制化SQL、存储过程以及高级映射。MyBatis避免了几乎所有的JDBC代码和手动设置参数以及获取结果集。

**核心优势：**
- 简化数据访问层开发
- SQL与Java代码分离，便于维护
- 强大的映射能力，支持复杂关联关系
- 灵活的动态SQL
- 内置缓存机制

```java
// 传统JDBC vs MyBatis对比
// JDBC需要手动管理连接、Statement、结果集
Connection conn = DriverManager.getConnection(url, user, pwd);
PreparedStatement ps = conn.prepareStatement("SELECT * FROM user WHERE id = ?");
ps.setInt(1, 1);
ResultSet rs = ps.executeQuery();
// ... 手动映射结果集

// MyBatis只需定义接口和SQL，框架自动完成其余工作
User user = userMapper.findById(1);
```

---

### 1.2 环境搭建

**步骤：**
1. 添加Maven依赖
2. 配置MyBatis核心文件 `mybatis-config.xml`
3. 创建实体类(POJO)
4. 创建Mapper接口和映射文件

```xml
<!-- pom.xml 添加MyBatis依赖 -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.13</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.33</version>
</dependency>
```

```xml
<!-- mybatis-config.xml 核心配置文件 -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 环境配置 -->
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?serverTimezone=UTC"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
    <!-- 映射器注册 -->
    <mappers>
        <mapper resource="mapper/UserMapper.xml"/>
    </mappers>
</configuration>
```

---

### 1.3 持久层接口和映射文件

**Mapper接口定义：**

```java
// UserMapper.java - 持久层接口
public interface UserMapper {
    // 根据id查询用户
    User findById(Integer id);
    // 查询所有用户
    List<User> findAll();
}
```

**映射文件定义：**

```xml
<!-- UserMapper.xml - SQL映射文件 -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace必须与Mapper接口全限定名一致 -->
<mapper namespace="com.example.mapper.UserMapper">
    
    <!-- 查询语句，id必须与接口方法名一致 -->
    <select id="findById" parameterType="Integer" resultType="com.example.entity.User">
        SELECT id, username, password, email FROM user WHERE id = #{id}
    </select>
    
    <select id="findAll" resultType="com.example.entity.User">
        SELECT id, username, password, email FROM user
    </select>
</mapper>
```

**实体类定义：**

```java
// User.java - 实体类
public class User {
    private Integer id;
    private String username;
    private String password;
    private String email;
    
    // 构造方法、getter、setter、toString省略
}
```

---

### 1.4 测试持久层接口方法

**方式一：使用SqlSession直接调用**

```java
public class MyBatisTest {
    public static void main(String[] args) throws IOException {
        // 1. 读取配置文件
        InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
        
        // 2. 创建SqlSessionFactory
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        
        // 3. 获取SqlSession
        try (SqlSession sqlSession = sqlSessionFactory.openSession()) {
            // 4. 获取Mapper接口代理对象
            UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
            
            // 5. 调用方法执行SQL
            User user = userMapper.findById(1);
            System.out.println(user);
            
            List<User> users = userMapper.findAll();
            users.forEach(System.out::println);
        }
    }
}
```

**方式二：使用JUnit测试**

```java
@Before
public void setUp() throws IOException {
    InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
    sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
}

@Test
public void testFindById() {
    try (SqlSession sqlSession = sqlSessionFactory.openSession()) {
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        User user = mapper.findById(1);
        assertNotNull(user);
    }
}
```

---

### 1.5 MyBatis核心对象和工作流程

**核心对象：**

| 对象 | 作用 | 生命周期 |
|------|------|----------|
| `SqlSessionFactoryBuilder` | 构建SqlSessionFactory | 方法级 |
| `SqlSessionFactory` | 创建SqlSession | 应用级 |
| `SqlSession` | 执行SQL、管理事务 | 请求级 |
| `Mapper` | 接口代理对象 | 请求级 |

**工作流程：**

```
1. 读取mybatis-config.xml配置文件
          ↓
2. SqlSessionFactoryBuilder构建SqlSessionFactory
          ↓
3. SqlSessionFactory创建SqlSession
          ↓
4. SqlSession获取Mapper接口代理对象
          ↓
5. 调用Mapper方法，框架解析映射文件
          ↓
6. 执行SQL，自动映射结果集到实体对象
          ↓
7. 返回结果给调用者
```

**底层原理：JDK动态代理**

```java
// MyBatis通过JDK动态代理创建Mapper实现类
// 核心代码原理：
MapperProxyFactory<T> factory = new MapperProxyFactory<>(mapperInterface);
T mapperInstance = (T) Proxy.newProxyInstance(
    mapperInterface.getClassLoader(),
    new Class[]{mapperInterface},
    new MapperProxy<>(sqlSession, mapperInterface, methodCache)
);
```

---

## 2. 核心配置

### 2.1 properties属性配置

**外部配置文件方式：**

```properties
# db.properties
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis?serverTimezone=UTC
jdbc.username=root
jdbc.password=123456
```

```xml
<!-- mybatis-config.xml -->
<configuration>
    <!-- 加载外部属性文件 -->
    <properties resource="db.properties"/>
    
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <!-- 使用${}引用属性值 -->
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```

**优先级：**
1. method参数传递
2. properties元素内部定义
3. 外部属性文件
4. MyBatis内置默认值

---

### 2.2 typeAliases类型别名

**方式一：单个类型别名**

```xml
<configuration>
    <typeAliases>
        <!-- 为全限定名创建别名 -->
        <typeAlias type="com.example.entity.User" alias="User"/>
        <typeAlias type="com.example.entity.Order" alias="Order"/>
    </typeAliases>
</configuration>
```

**方式二：包扫描**

```xml
<configuration>
    <typeAliases>
        <!-- 扫描整个包，别名为类名首字母小写 -->
        <package name="com.example.entity"/>
    </typeAliases>
</configuration>
```

**使用别名：**

```xml
<!-- 映射文件中使用别名代替全限定名 -->
<select id="findById" parameterType="Integer" resultType="User">
    SELECT * FROM user WHERE id = #{id}
</select>
```

**内置别名：**

| 别名 | Java类型 |
|------|----------|
| `int` | `Integer` |
| `long` | `Long` |
| `string` | `String` |
| `list` | `List` |
| `map` | `Map` |

---

### 2.3 environments环境配置

**多环境配置：**

```xml
<configuration>
    <environments default="development">
        <!-- 开发环境 -->
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.dev.url}"/>
                <property name="username" value="${jdbc.dev.username}"/>
                <property name="password" value="${jdbc.dev.password}"/>
            </dataSource>
        </environment>
        
        <!-- 测试环境 -->
        <environment id="test">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.test.url}"/>
                <property name="username" value="${jdbc.test.username}"/>
                <property name="password" value="${jdbc.test.password}"/>
            </dataSource>
        </environment>
        
        <!-- 生产环境 -->
        <environment id="production">
            <transactionManager type="MANAGED"/>
            <dataSource type="JNDI">
                <property name="data_source" value="java:comp/env/jdbc/mybatis"/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```

**transactionManager类型：**
- `JDBC`: 手动提交事务
- `MANAGED`: 交由容器管理事务

**dataSource类型：**
- `UNPOOLED`: 每次请求创建新连接
- `POOLED`: 连接池复用
- `JNDI`: 从JNDI数据源获取

---

### 2.4 mappers映射器注册

**方式一：resource方式**

```xml
<mappers>
    <mapper resource="mapper/UserMapper.xml"/>
    <mapper resource="mapper/OrderMapper.xml"/>
</mappers>
```

**方式二：class方式（注解开发）**

```xml
<mappers>
    <mapper class="com.example.mapper.UserMapper"/>
    <mapper class="com.example.mapper.OrderMapper"/>
</mappers>
```

**方式三：包扫描**

```xml
<mappers>
    <!-- 扫描整个包下的所有Mapper接口 -->
    <package name="com.example.mapper"/>
</mappers>
```

**注意事项：**
- 使用class方式时，接口和映射文件必须同名且在同一目录
- 使用package方式时，接口和映射文件必须同名且在同一目录

---

## 3. 映射文件

### 3.1 resultMap结果映射

**场景：字段名与属性名不一致时使用**

```java
// User.java - 属性名
private Integer userId;
private String userName;
```

```xml
<!-- UserMapper.xml -->
<resultMap id="UserResultMap" type="User">
    <!-- id表示主键映射 -->
    <id property="userId" column="id"/>
    <!-- result表示普通字段映射 -->
    <result property="userName" column="username"/>
    <result property="password" column="password"/>
    <result property="email" column="email"/>
</resultMap>

<select id="findById" parameterType="Integer" resultMap="UserResultMap">
    SELECT id, username, password, email FROM user WHERE id = #{id}
</select>
```

**复杂映射：**

```xml
<resultMap id="OrderResultMap" type="Order">
    <id property="orderId" column="order_id"/>
    <result property="orderNo" column="order_no"/>
    <result property="totalAmount" column="total_amount"/>
    <!-- 一对一关联 -->
    <association property="user" javaType="User">
        <id property="userId" column="user_id"/>
        <result property="userName" column="username"/>
    </association>
</resultMap>
```

---

### 3.2 sql&include片段复用

**抽取公共SQL片段：**

```xml
<!-- UserMapper.xml -->
<!-- 定义SQL片段 -->
<sql id="userColumns">
    id, username, password, email, create_time, update_time
</sql>

<sql id="userTable">
    `user`
</sql>

<!-- 使用include引用片段 -->
<select id="findById" parameterType="Integer" resultType="User">
    SELECT <include refid="userColumns"/>
    FROM <include refid="userTable"/>
    WHERE id = #{id}
</select>

<select id="findAll" resultType="User">
    SELECT <include refid="userColumns"/>
    FROM <include refid="userTable"/>
</select>
```

**带参数的SQL片段：**

```xml
<sql id="dynamicColumns">
    <if test="includePassword != null and includePassword">
        , password
    </if>
    <if test="includeEmail != null and includeEmail">
        , email
    </if>
</sql>

<select id="findByIdSelective" resultType="User">
    SELECT id, username <include refid="dynamicColumns"/>
    FROM user WHERE id = #{id}
</select>
```

---

### 3.3 特殊字符处理

**方式一：使用XML实体**

```xml
<select id="findByAgeRange" resultType="User">
    SELECT * FROM user 
    WHERE age &gt;= #{minAge} AND age &lt;= #{maxAge}
</select>
```

**方式二：使用CDATA块**

```xml
<select id="findByAgeRange" resultType="User">
    <![CDATA[
        SELECT * FROM user 
        WHERE age >= #{minAge} AND age <= #{maxAge}
    ]]>
</select>
```

**常用XML实体：**

| 字符 | XML实体 | 含义 |
|------|---------|------|
| `<` | `&lt;` | 小于 |
| `>` | `&gt;` | 大于 |
| `&` | `&amp;` | 和 |
| `"` | `&quot;` | 双引号 |
| `'` | `&apos;` | 单引号 |

---

### 3.4 增删改查基础操作

**新增(INSERT)：**

```xml
<insert id="insert" parameterType="User">
    INSERT INTO user (username, password, email)
    VALUES (#{username}, #{password}, #{email})
</insert>
```

```java
// Java调用
int rows = userMapper.insert(user);
sqlSession.commit(); // 增删改必须提交事务
```

**主键回填：**

```xml
<insert id="insert" parameterType="User" useGeneratedKeys="true" keyProperty="id">
    INSERT INTO user (username, password, email)
    VALUES (#{username}, #{password}, #{email})
</insert>
```

```java
// 插入后自动获取主键值
userMapper.insert(user);
System.out.println(user.getId()); // 已自动回填
```

**修改(UPDATE)：**

```xml
<update id="update" parameterType="User">
    UPDATE user 
    SET username = #{username}, password = #{password}, email = #{email}
    WHERE id = #{id}
</update>
```

**删除(DELETE)：**

```xml
<delete id="deleteById" parameterType="Integer">
    DELETE FROM user WHERE id = #{id}
</delete>
```

**查询(SELECT)：**

```xml
<!-- 根据id查询单个 -->
<select id="findById" parameterType="Integer" resultType="User">
    SELECT * FROM user WHERE id = #{id}
</select>

<!-- 查询所有 -->
<select id="findAll" resultType="User">
    SELECT * FROM user
</select>

<!-- 模糊查询 -->
<select id="findByUsername" parameterType="String" resultType="User">
    SELECT * FROM user WHERE username LIKE '%${value}%'
</select>
```

**#{}与${}的区别：**

| 方式 | 特点 | 安全性 |
|------|------|--------|
| `#{}` | 预编译，参数占位符 | 安全，防止SQL注入 |
| `${}` | 字符串拼接，直接替换 | 不安全，需谨慎使用 |

---

## 4. 动态SQL

### 4.1 if条件判断

**场景：根据条件动态拼接SQL**

```xml
<!-- UserMapper.xml -->
<select id="findByCondition" parameterType="User" resultType="User">
    SELECT * FROM user WHERE 1=1
    <!-- 条件判断：username不为空则拼接 -->
    <if test="username != null and username != ''">
        AND username LIKE '%${username}%'
    </if>
    <!-- 条件判断：email不为空则拼接 -->
    <if test="email != null and email != ''">
        AND email = #{email}
    </if>
</select>
```

```java
// 调用示例：只传username
User condition = new User();
condition.setUsername("张三");
List<User> users = userMapper.findByCondition(condition);
```

**注意：**
- `test`属性支持OGNL表达式
- 常用判断：`!= null`、`!= ''`、`size > 0`

---

### 4.2 where&set标签

**where标签：自动处理AND/OR前缀**

```xml
<select id="findByCondition" parameterType="User" resultType="User">
    SELECT * FROM user
    <where>
        <if test="username != null and username != ''">
            AND username LIKE '%${username}%'
        </if>
        <if test="email != null and email != ''">
            AND email = #{email}
        </if>
    </where>
</select>
```

**set标签：自动处理逗号**

```xml
<update id="updateSelective" parameterType="User">
    UPDATE user
    <set>
        <if test="username != null and username != ''">
            username = #{username},
        </if>
        <if test="password != null and password != ''">
            password = #{password},
        </if>
        <if test="email != null and email != ''">
            email = #{email},
        </if>
    </set>
    WHERE id = #{id}
</update>
```

---

### 4.3 choose&when&otherwise

**场景：多选一，类似switch-case**

```xml
<select id="findByPriority" parameterType="User" resultType="User">
    SELECT * FROM user
    <where>
        <choose>
            <!-- 优先级最高 -->
            <when test="id != null">
                AND id = #{id}
            </when>
            <!-- 次优先级 -->
            <when test="username != null and username != ''">
                AND username = #{username}
            </when>
            <!-- 默认条件 -->
            <otherwise>
                AND status = 1
            </otherwise>
        </choose>
    </where>
</select>
```

---

### 4.4 foreach遍历

**遍历数组：**

```xml
<select id="findByIds" parameterType="int[]" resultType="User">
    SELECT * FROM user WHERE id IN
    <foreach collection="array" item="id" open="(" separator="," close=")">
        #{id}
    </foreach>
</select>
```

**遍历List：**

```xml
<select id="findByIdList" parameterType="List" resultType="User">
    SELECT * FROM user WHERE id IN
    <foreach collection="list" item="id" open="(" separator="," close=")">
        #{id}
    </foreach>
</select>
```

**遍历Map：**

```xml
<select id="findByMap" parameterType="Map" resultType="User">
    SELECT * FROM user WHERE 1=1
    <foreach collection="entrySet" item="entry" separator="AND">
        ${entry.key} = #{entry.value}
    </foreach>
</select>
```

**批量插入：**

```xml
<insert id="batchInsert" parameterType="List">
    INSERT INTO user (username, password, email) VALUES
    <foreach collection="list" item="user" separator=",">
        (#{user.username}, #{user.password}, #{user.email})
    </foreach>
</insert>
```

**foreach属性说明：**

| 属性 | 作用 |
|------|------|
| `collection` | 集合类型：array/list/entrySet |
| `item` | 循环变量名 |
| `index` | 索引名（可选） |
| `open` | 开始符号 |
| `close` | 结束符号 |
| `separator` | 元素分隔符 |

---

## 5. 关联查询

### 5.1 一对一关联查询

**场景：订单与用户的关系（一个订单属于一个用户）**

**方式一：嵌套结果映射（JOIN查询）**

```xml
<!-- OrderMapper.xml -->
<resultMap id="OrderResultMap" type="Order">
    <id property="orderId" column="order_id"/>
    <result property="orderNo" column="order_no"/>
    <result property="totalAmount" column="total_amount"/>
    <!-- 一对一关联 -->
    <association property="user" javaType="User">
        <id property="userId" column="user_id"/>
        <result property="userName" column="username"/>
        <result property="email" column="email"/>
    </association>
</resultMap>

<select id="findOrderWithUser" parameterType="Integer" resultMap="OrderResultMap">
    SELECT o.order_id, o.order_no, o.total_amount,
           u.id as user_id, u.username, u.email
    FROM `order` o
    JOIN `user` u ON o.user_id = u.id
    WHERE o.order_id = #{orderId}
</select>
```

**方式二：嵌套查询（分步查询）**

```xml
<resultMap id="OrderResultMap" type="Order">
    <id property="orderId" column="order_id"/>
    <result property="orderNo" column="order_no"/>
    <!-- column传递参数给findById方法 -->
    <association property="user" column="user_id" 
                 select="com.example.mapper.UserMapper.findById"/>
</resultMap>

<select id="findOrderWithUser" parameterType="Integer" resultMap="OrderResultMap">
    SELECT order_id, order_no, total_amount, user_id FROM `order` WHERE order_id = #{orderId}
</select>
```

---

### 5.2 一对多关联查询

**场景：用户与订单的关系（一个用户有多个订单）**

```xml
<!-- UserMapper.xml -->
<resultMap id="UserWithOrdersResultMap" type="User">
    <id property="userId" column="id"/>
    <result property="userName" column="username"/>
    <!-- 一对多关联 -->
    <collection property="orders" ofType="Order">
        <id property="orderId" column="order_id"/>
        <result property="orderNo" column="order_no"/>
        <result property="totalAmount" column="total_amount"/>
    </collection>
</resultMap>

<select id="findUserWithOrders" parameterType="Integer" resultMap="UserWithOrdersResultMap">
    SELECT u.id, u.username, u.email,
           o.order_id, o.order_no, o.total_amount
    FROM `user` u
    LEFT JOIN `order` o ON u.id = o.user_id
    WHERE u.id = #{userId}
</select>
```

**分步查询方式：**

```xml
<resultMap id="UserWithOrdersResultMap" type="User">
    <id property="userId" column="id"/>
    <result property="userName" column="username"/>
    <!-- column传递参数给findOrdersByUserId方法 -->
    <collection property="orders" column="id" 
                select="com.example.mapper.OrderMapper.findOrdersByUserId"/>
</resultMap>

<select id="findUserWithOrders" parameterType="Integer" resultMap="UserWithOrdersResultMap">
    SELECT id, username, email FROM `user` WHERE id = #{userId}
</select>
```

---

### 5.3 多对多关联查询

**场景：用户与角色的关系（一个用户有多个角色，一个角色被多个用户拥有）**

**中间表：user_role(user_id, role_id)**

```xml
<!-- UserMapper.xml -->
<resultMap id="UserWithRolesResultMap" type="User">
    <id property="userId" column="user_id"/>
    <result property="userName" column="username"/>
    <!-- 多对多关联 -->
    <collection property="roles" ofType="Role">
        <id property="roleId" column="role_id"/>
        <result property="roleName" column="role_name"/>
    </collection>
</resultMap>

<select id="findUserWithRoles" parameterType="Integer" resultMap="UserWithRolesResultMap">
    SELECT u.id as user_id, u.username,
           r.id as role_id, r.role_name
    FROM `user` u
    JOIN user_role ur ON u.id = ur.user_id
    JOIN role r ON ur.role_id = r.id
    WHERE u.id = #{userId}
</select>
```

---

### 5.4 分解式查询与延迟加载

**延迟加载（懒加载）：**

```xml
<!-- mybatis-config.xml 开启全局延迟加载 -->
<settings>
    <setting name="lazyLoadingEnabled" value="true"/>
    <setting name="aggressiveLazyLoading" value="false"/>
</settings>
```

**分步查询+延迟加载：**

```xml
<resultMap id="OrderResultMap" type="Order">
    <id property="orderId" column="order_id"/>
    <result property="orderNo" column="order_no"/>
    <!-- fetchType="lazy" 延迟加载 -->
    <association property="user" column="user_id" 
                 select="com.example.mapper.UserMapper.findById"
                 fetchType="lazy"/>
</resultMap>
```

**按需加载：**

```java
Order order = orderMapper.findById(1);
// 此时只查询订单，未查询用户
System.out.println(order.getOrderNo());

// 访问user属性时才触发关联查询
System.out.println(order.getUser().getUsername());
```

**延迟加载优缺点：**

| 优点 | 缺点 |
|------|------|
| 按需加载，减少不必要查询 | N+1问题（需要配置合理） |
| 提高首次查询性能 | 代码可读性降低 |

---

## 6. 缓存机制

### 6.1 一级缓存

**一级缓存：SqlSession级别，默认开启**

```java
try (SqlSession sqlSession = sqlSessionFactory.openSession()) {
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    
    // 第一次查询，执行SQL
    User user1 = mapper.findById(1);
    
    // 第二次查询相同id，从缓存获取，不执行SQL
    User user2 = mapper.findById(1);
    
    // 同一个对象引用
    System.out.println(user1 == user2); // true
}
```

**一级缓存工作原理：**

```
1. 查询时先检查SqlSession缓存
2. 缓存命中则直接返回
3. 缓存未命中则执行SQL，结果存入缓存
4. SqlSession关闭时缓存失效
```

---

### 6.2 清除一级缓存

**方式一：执行增删改操作自动清除**

```java
try (SqlSession sqlSession = sqlSessionFactory.openSession()) {
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    
    User user1 = mapper.findById(1);
    mapper.update(user); // 增删改操作清除缓存
    User user2 = mapper.findById(1); // 重新执行SQL
    
    System.out.println(user1 == user2); // false
}
```

**方式二：手动清除**

```java
sqlSession.clearCache(); // 清除当前SqlSession缓存
```

**方式三：关闭SqlSession**

```java
sqlSession.close(); // 缓存随SqlSession销毁
```

---

### 6.3 二级缓存

**二级缓存：Mapper级别，跨SqlSession共享**

**步骤：**

1. **全局开启二级缓存：**

```xml
<!-- mybatis-config.xml -->
<settings>
    <setting name="cacheEnabled" value="true"/>
</settings>
```

2. **Mapper配置缓存：**

```xml
<!-- UserMapper.xml -->
<!-- 开启当前Mapper的二级缓存 -->
<cache/>

<!-- 或自定义缓存配置 -->
<cache type="org.mybatis.caches.ehcache.EhcacheCache"
       eviction="LRU"
       flushInterval="60000"
       size="512"
       readOnly="true"/>
```

3. **实体类实现Serializable接口：**

```java
public class User implements Serializable {
    // ...
}
```

**使用示例：**

```java
// SqlSession1查询并缓存
try (SqlSession sqlSession1 = sqlSessionFactory.openSession()) {
    UserMapper mapper1 = sqlSession1.getMapper(UserMapper.class);
    User user1 = mapper1.findById(1);
    sqlSession1.commit(); // 必须提交才能写入二级缓存
}

// SqlSession2从二级缓存获取
try (SqlSession sqlSession2 = sqlSessionFactory.openSession()) {
    UserMapper mapper2 = sqlSession2.getMapper(UserMapper.class);
    User user2 = mapper2.findById(1); // 从二级缓存获取
    
    System.out.println(user1 == user2); // false，但内容相同
}
```

**缓存配置参数：**

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `type` | 缓存实现类 | PerpetualCache |
| `eviction` | 回收策略 | LRU |
| `flushInterval` | 刷新间隔(ms) | 不刷新 |
| `size` | 缓存数量 | 1024 |
| `readOnly` | 是否只读 | false |

---

## 7. 注解开发

### 7.1 注解开发基础

**无需XML映射文件，直接在接口上使用注解：**

```java
// UserMapper.java - 注解方式
public interface UserMapper {
    
    @Select("SELECT id, username, password, email FROM user WHERE id = #{id}")
    User findById(Integer id);
    
    @Select("SELECT * FROM user")
    List<User> findAll();
}
```

**mybatis-config.xml配置：**

```xml
<mappers>
    <mapper class="com.example.mapper.UserMapper"/>
</mappers>
```

---

### 7.2 注解增删改查

**新增：**

```java
@Insert("INSERT INTO user (username, password, email) VALUES (#{username}, #{password}, #{email})")
@Options(useGeneratedKeys = true, keyProperty = "id")
int insert(User user);
```

**修改：**

```java
@Update("UPDATE user SET username = #{username}, password = #{password}, email = #{email} WHERE id = #{id}")
int update(User user);
```

**删除：**

```java
@Delete("DELETE FROM user WHERE id = #{id}")
int deleteById(Integer id);
```

**查询：**

```java
@Select("SELECT * FROM user WHERE username LIKE '%${username}%'")
List<User> findByUsername(String username);
```

---

### 7.3 注解动态SQL

**使用@Provider注解：**

```java
public interface UserMapper {
    
    @SelectProvider(type = UserSqlProvider.class, method = "findByCondition")
    List<User> findByCondition(User user);
}

// SQL提供类
public class UserSqlProvider {
    public String findByCondition(User user) {
        StringBuilder sql = new StringBuilder("SELECT * FROM user WHERE 1=1");
        if (user.getUsername() != null && !user.getUsername().isEmpty()) {
            sql.append(" AND username LIKE '%").append(user.getUsername()).append("%'");
        }
        if (user.getEmail() != null && !user.getEmail().isEmpty()) {
            sql.append(" AND email = #{email}");
        }
        return sql.toString();
    }
}
```

**使用Script注解（3.5+）：**

```java
@Select("<script>" +
        "SELECT * FROM user WHERE 1=1" +
        "<if test='username != null'>AND username LIKE '%${username}%'</if>" +
        "</script>")
List<User> findByCondition(User user);
```

---

### 7.4 注解自定义映射

**使用@Results和@Result：**

```java
@Select("SELECT id, username, password, email FROM user WHERE id = #{id}")
@Results({
    @Result(property = "userId", column = "id"),
    @Result(property = "userName", column = "username"),
    @Result(property = "password", column = "password"),
    @Result(property = "email", column = "email")
})
User findById(Integer id);
```

**一对一关联：**

```java
@Select("SELECT order_id, order_no, total_amount, user_id FROM `order` WHERE order_id = #{orderId}")
@Results({
    @Result(property = "orderId", column = "order_id"),
    @Result(property = "orderNo", column = "order_no"),
    @Result(property = "user", column = "user_id", 
            one = @One(select = "com.example.mapper.UserMapper.findById"))
})
Order findOrderWithUser(Integer orderId);
```

**一对多关联：**

```java
@Select("SELECT id, username, email FROM `user` WHERE id = #{userId}")
@Results({
    @Result(property = "userId", column = "id"),
    @Result(property = "userName", column = "username"),
    @Result(property = "orders", column = "id", 
            many = @Many(select = "com.example.mapper.OrderMapper.findOrdersByUserId"))
})
User findUserWithOrders(Integer userId);
```

---

## 8. 高级特性

### 8.1 插件机制(Interceptor)

**自定义插件：**

```java
@Intercepts({@Signature(
    type = Executor.class,
    method = "update",
    args = {MappedStatement.class, Object.class}
)})
public class MyInterceptor implements Interceptor {
    
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        // 执行前逻辑
        System.out.println("Before SQL execution");
        
        // 执行原方法
        Object result = invocation.proceed();
        
        // 执行后逻辑
        System.out.println("After SQL execution");
        
        return result;
    }
    
    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }
    
    @Override
    public void setProperties(Properties properties) {
        // 读取配置属性
    }
}
```

**注册插件：**

```xml
<!-- mybatis-config.xml -->
<plugins>
    <plugin interceptor="com.example.interceptor.MyInterceptor">
        <property name="key" value="value"/>
    </plugin>
</plugins>
```

**可拦截的对象：**

| 对象 | 说明 |
|------|------|
| `Executor` | 执行器，处理CRUD操作 |
| `StatementHandler` | SQL语句处理器 |
| `ParameterHandler` | 参数处理器 |
| `ResultSetHandler` | 结果集处理器 |

---

### 8.2 TypeHandler类型处理器

**自定义TypeHandler：**

```java
// 将枚举类型与数据库字符串映射
public class GenderTypeHandler extends BaseTypeHandler<Gender> {
    
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, 
                                     Gender parameter, JdbcType jdbcType) throws SQLException {
        ps.setString(i, parameter.name());
    }
    
    @Override
    public Gender getNullableResult(ResultSet rs, String columnName) throws SQLException {
        String value = rs.getString(columnName);
        return value != null ? Gender.valueOf(value) : null;
    }
    
    @Override
    public Gender getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        String value = rs.getString(columnIndex);
        return value != null ? Gender.valueOf(value) : null;
    }
    
    @Override
    public Gender getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        String value = cs.getString(columnIndex);
        return value != null ? Gender.valueOf(value) : null;
    }
}
```

**注册TypeHandler：**

```xml
<!-- mybatis-config.xml -->
<typeHandlers>
    <typeHandler handler="com.example.typehandler.GenderTypeHandler"/>
</typeHandlers>
```

**映射文件中使用：**

```xml
<resultMap id="UserResultMap" type="User">
    <result property="gender" column="gender" typeHandler="com.example.typehandler.GenderTypeHandler"/>
</resultMap>
```

---

### 8.3 批量操作优化

**方式一：使用foreach批量插入**

```xml
<insert id="batchInsert" parameterType="List">
    INSERT INTO user (username, password, email) VALUES
    <foreach collection="list" item="user" separator=",">
        (#{user.username}, #{user.password}, #{user.email})
    </foreach>
</insert>
```

**方式二：使用ExecutorType.BATCH**

```java
try (SqlSession sqlSession = sqlSessionFactory.openSession(ExecutorType.BATCH)) {
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    
    for (User user : userList) {
        mapper.insert(user);
    }
    
    sqlSession.commit();
}
```

**方式三：分批提交**

```java
int batchSize = 1000;
try (SqlSession sqlSession = sqlSessionFactory.openSession(ExecutorType.BATCH)) {
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    
    for (int i = 0; i < userList.size(); i++) {
        mapper.insert(userList.get(i));
        
        // 每1000条提交一次
        if ((i + 1) % batchSize == 0) {
            sqlSession.commit();
            sqlSession.clearCache();
        }
    }
    
    sqlSession.commit();
}
```

---

### 8.4 分页插件PageHelper

**添加依赖：**

```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>5.3.2</version>
</dependency>
```

**配置插件：**

```xml
<!-- mybatis-config.xml -->
<plugins>
    <plugin interceptor="com.github.pagehelper.PageInterceptor">
        <property name="helperDialect" value="mysql"/>
        <property name="reasonable" value="true"/>
    </plugin>
</plugins>
```

**使用方式：**

```java
// 设置分页参数
PageHelper.startPage(1, 10); // 第1页，每页10条

// 执行查询，PageHelper自动添加分页SQL
List<User> users = userMapper.findAll();

// 获取分页信息
PageInfo<User> pageInfo = new PageInfo<>(users);
System.out.println("总记录数：" + pageInfo.getTotal());
System.out.println("总页数：" + pageInfo.getPages());
System.out.println("当前页：" + pageInfo.getPageNum());
```

**分页信息：**

| 属性 | 说明 |
|------|------|
| `pageNum` | 当前页码 |
| `pageSize` | 每页大小 |
| `total` | 总记录数 |
| `pages` | 总页数 |
| `list` | 当前页数据 |

---

## 9. 生态集成

### 9.1 MyBatis与Spring集成

**添加依赖：**

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>2.0.7</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.3.20</version>
</dependency>
```

**Spring配置文件：**

```xml
<!-- applicationContext.xml -->
<!-- 数据源 -->
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
    <property name="username" value="root"/>
    <property name="password" value="123456"/>
</bean>

<!-- SqlSessionFactory -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <property name="configLocation" value="classpath:mybatis-config.xml"/>
    <property name="mapperLocations" value="classpath:mapper/*.xml"/>
</bean>

<!-- Mapper扫描器 -->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.example.mapper"/>
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
</bean>

<!-- 事务管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>

<!-- 开启事务注解 -->
<tx:annotation-driven transaction-manager="transactionManager"/>
```

**Service层使用：**

```java
@Service
@Transactional
public class UserServiceImpl implements UserService {
    
    @Autowired
    private UserMapper userMapper;
    
    @Override
    public User findById(Integer id) {
        return userMapper.findById(id);
    }
    
    @Override
    public void save(User user) {
        userMapper.insert(user);
    }
}
```

---

### 9.2 MyBatis与SpringBoot集成

**添加依赖：**

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.2.2</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

**application.yml配置：**

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mybatis?serverTimezone=UTC
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver

mybatis:
  config-location: classpath:mybatis-config.xml
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: com.example.entity
```

**启动类配置：**

```java
@SpringBootApplication
@MapperScan("com.example.mapper") // 扫描Mapper接口
public class MyBatisApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyBatisApplication.class, args);
    }
}
```

**使用方式：**

```java
@RestController
@RequestMapping("/users")
public class UserController {
    
    @Autowired
    private UserMapper userMapper;
    
    @GetMapping("/{id}")
    public User findById(@PathVariable Integer id) {
        return userMapper.findById(id);
    }
    
    @PostMapping
    public void save(@RequestBody User user) {
        userMapper.insert(user);
    }
}
```

---

### 9.3 事务管理

**声明式事务：**

```java
@Service
@Transactional // 类级别事务
public class UserServiceImpl implements UserService {
    
    @Autowired
    private UserMapper userMapper;
    
    @Autowired
    private OrderMapper orderMapper;
    
    @Override
    @Transactional(propagation = Propagation.REQUIRED)
    public void createUserWithOrder(User user, Order order) {
        userMapper.insert(user);
        order.setUserId(user.getId());
        orderMapper.insert(order);
    }
    
    @Override
    @Transactional(readOnly = true)
    public User findById(Integer id) {
        return userMapper.findById(id);
    }
}
```

**事务传播行为：**

| 传播行为 | 说明 |
|----------|------|
| `REQUIRED` | 如果当前没有事务，创建新事务；如果已有事务，加入该事务 |
| `SUPPORTS` | 如果当前有事务，加入该事务；否则以非事务方式执行 |
| `MANDATORY` | 如果当前有事务，加入该事务；否则抛出异常 |
| `REQUIRES_NEW` | 创建新事务，如果当前有事务则挂起 |
| `NOT_SUPPORTED` | 以非事务方式执行，如果当前有事务则挂起 |
| `NEVER` | 以非事务方式执行，如果当前有事务则抛出异常 |

---

## 10. 最佳实践

### 10.1 性能优化

**1. 使用连接池：**

```xml
<dataSource type="POOLED">
    <property name="driver" value="${jdbc.driver}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
    <!-- 连接池配置 -->
    <property name="poolMaximumActiveConnections" value="20"/>
    <property name="poolMaximumIdleConnections" value="5"/>
</dataSource>
```

**2. 合理使用缓存：**

```xml
<!-- 开启二级缓存 -->
<cache eviction="LRU" flushInterval="60000" size="512"/>
```

**3. 避免N+1查询问题：**

```xml
<!-- 使用JOIN查询代替分步查询 -->
<select id="findUserWithOrders" resultMap="UserWithOrdersResultMap">
    SELECT u.*, o.* FROM `user` u JOIN `order` o ON u.id = o.user_id
    WHERE u.id = #{userId}
</select>
```

**4. 使用批量操作：**

```java
// 使用BATCH模式
try (SqlSession sqlSession = sqlSessionFactory.openSession(ExecutorType.BATCH)) {
    for (User user : list) {
        mapper.insert(user);
    }
    sqlSession.commit();
}
```

**5. 合理使用fetchSize：**

```xml
<select id="findAll" resultType="User" fetchSize="100">
    SELECT * FROM user
</select>
```

---

### 10.2 常见问题排查

**1. SQL注入：**

```xml
<!-- 错误：使用${}拼接 -->
<select id="findByUsername" resultType="User">
    SELECT * FROM user WHERE username = '${username}'
</select>

<!-- 正确：使用#{}预编译 -->
<select id="findByUsername" resultType="User">
    SELECT * FROM user WHERE username = #{username}
</select>
```

**2. 参数绑定错误：**

```xml
<!-- 错误：参数名与方法参数不一致 -->
<select id="findByUser" parameterType="User" resultType="User">
    SELECT * FROM user WHERE username = #{name}
</select>

<!-- 正确：参数名必须与User属性名一致 -->
<select id="findByUser" parameterType="User" resultType="User">
    SELECT * FROM user WHERE username = #{username}
</select>
```

**3. 映射文件找不到：**

```xml
<!-- 错误：路径问题 -->
<mapper resource="UserMapper.xml"/>

<!-- 正确：使用完整路径 -->
<mapper resource="mapper/UserMapper.xml"/>
```

**4. 日志配置：**

```xml
<!-- mybatis-config.xml 开启SQL日志 -->
<settings>
    <setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
```

---

### 10.3 MyBatisGenerator代码生成

**添加依赖：**

```xml
<dependency>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-core</artifactId>
    <version>1.4.1</version>
</dependency>
```

**配置文件：**

```xml
<!-- generatorConfig.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <context id="DB2Tables" targetRuntime="MyBatis3">
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/mybatis"
                        userId="root"
                        password="123456"/>
        
        <javaModelGenerator targetPackage="com.example.entity" 
                            targetProject="src/main/java"/>
        
        <sqlMapGenerator targetPackage="mapper" 
                         targetProject="src/main/resources"/>
        
        <javaClientGenerator type="XMLMAPPER" 
                             targetPackage="com.example.mapper" 
                             targetProject="src/main/java"/>
        
        <table tableName="user" domainObjectName="User"/>
        <table tableName="order" domainObjectName="Order"/>
    </context>
</generatorConfiguration>
```

**执行生成：**

```java
List<String> warnings = new ArrayList<>();
File configFile = new File("generatorConfig.xml");
ConfigurationParser cp = new ConfigurationParser(warnings);
Configuration config = cp.parseConfiguration(configFile);
DefaultShellCallback callback = new DefaultShellCallback(true);
MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
myBatisGenerator.generate(null);
```

**生成内容：**

| 文件类型 | 说明 |
|----------|------|
| `User.java` | 实体类 |
| `UserMapper.java` | Mapper接口 |
| `UserMapper.xml` | 映射文件 |
| `UserExample.java` | 查询条件封装类 |

---

## 附录

### A. MyBatis生命周期管理

```
SqlSessionFactoryBuilder → SqlSessionFactory → SqlSession → Mapper
    (方法级)              (应用级)           (请求级)      (请求级)
```

### B. 常用OGNL表达式

| 表达式 | 说明 |
|--------|------|
| `username != null` | 判断是否为空 |
| `username != ''` | 判断是否为空字符串 |
| `list.size > 0` | 判断集合是否为空 |
| `map.containsKey('key')` | 判断Map是否包含key |

### C. 常用配置项

```xml
<settings>
    <setting name="cacheEnabled" value="true"/>
    <setting name="lazyLoadingEnabled" value="true"/>
    <setting name="logImpl" value="STDOUT_LOGGING"/>
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

### D. 版本兼容性

| MyBatis版本 | 最低Java版本 | 推荐Spring版本 |
|-------------|-------------|---------------|
| 3.5.x | Java 8 | Spring 5.x |
| 3.4.x | Java 6 | Spring 4.x |

---

*文档版本：v1.0*  
*更新时间：2026-07-18*