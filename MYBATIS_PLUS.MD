# MyBatis-Plus 核心技术手册

   **阅读指南**：由浅入深，每个知识点均配有精简示例与注释。建议按顺序阅读，环环相扣。

   ---

## 目录

### 1. 基础入门
- [1.1 MyBatisPlus概述](#11-mybatisplus概述)
- [1.2 SpringBoot集成MyBatisPlus](#12-springboot集成mybatisplus)

### 2. CRUD核心操作
- [2.1 添加操作](#21-添加操作)
- [2.2 相关注解](#22-相关注解)
- [2.3 修改操作](#23-修改操作)
- [2.4 删除操作](#24-删除操作)
- [2.5 查询操作](#25-查询操作)
- [2.6 条件构造器](#26-条件构造器)
- [2.7 分页查询](#27-分页查询)
- [2.8 全局配置](#28-全局配置)

### 3. ActiveRecord模式
- [3.1 ActiveRecord概念](#31-activerecord概念)
- [3.2 ActiveRecord增删改查](#32-activerecord增删改查)

### 4. 插件机制
- [4.1 插件概述](#41-插件概述)
- [4.2 分页插件与防止全表更新删除插件](#42-分页插件与防止全表更新删除插件)
- [4.3 乐观锁插件概念](#43-乐观锁插件概念)
- [4.4 乐观锁插件使用](#44-乐观锁插件使用)

### 5. 逻辑删除
- [5.1 逻辑删除概念](#51-逻辑删除概念)
- [5.2 逻辑删除使用](#52-逻辑删除使用)

### 6. 扩展功能
- [6.1 自动填充](#61-自动填充)
- [6.2 SQL注入器](#62-sql注入器)
- [6.3 代码生成器](#63-代码生成器)

---

## 1. 基础入门

### 1.1 MyBatisPlus概述

**MyBatis-Plus**（简称MP）是一个MyBatis的增强工具，在MyBatis的基础上**只做增强不做改变**，为简化开发、提高效率而生。

**核心特性：**
- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响
- **损耗小**：启动即注入基本CRUD，性能基本无损耗
- **强大的CRUD操作**：内置通用Mapper、Service，少量配置即可实现单表大部分CRUD
- **Lambda表达式**：通过Lambda编写各类查询条件，无需担心字段写错
- **支持主键自动生成**：支持多达4种主键策略（含分布式唯一ID生成器）
- **支持ActiveRecord模式**：实体类只需继承Model即可拥有CRUD能力
- **内置代码生成器**：采用代码或Maven插件快速生成各层代码
- **内置分页插件**：基于MyBatis物理分页，开发者无需关心分页操作
- **内置性能分析插件**：可输出SQL语句及执行时间

**架构层次：**

```
┌─────────────────────────────────────────┐
│  Spring Boot / Spring                   │  ← 容器层
├─────────────────────────────────────────┤
│  MyBatis-Plus (扩展增强)                │  ← 增强层
│   ├── 核心: CRUD/条件构造器/分页        │
│   ├── 扩展: ActiveRecord/逻辑删除/填充  │
│   └── 插件: 分页/乐观锁/防全表更新      │
├─────────────────────────────────────────┤
│  MyBatis (SQL映射框架)                  │  ← 基础层
├─────────────────────────────────────────┤
│  JDBC → 数据库                          │  ← 持久层
└─────────────────────────────────────────┘
```

**与MyBatis对比：**

| 对比项 | MyBatis | MyBatis-Plus |
|--------|---------|--------------|
| 单表CRUD | 手写XML/注解 | 内置通用方法 |
| 条件构造 | XML动态SQL | Wrapper链式API |
| 分页 | 需借助PageHelper | 内置物理分页插件 |
| 代码生成 | MBG（功能单一） | 强大灵活的生成器 |
| 主键策略 | 手动配置 | 内置雪花算法等 |

---

### 1.2 SpringBoot集成MyBatisPlus

**集成步骤：**
1. 引入 `mybatis-plus-boot-starter` 依赖（**不要同时引入mybatis**，避免冲突）
2. 配置数据源与MP参数
3. 创建实体类、Mapper接口（继承 `BaseMapper`）
4. 启动类添加 `@MapperScan` 扫描Mapper包

```xml
<!-- pom.xml：MP的SpringBoot启动器，已包含MyBatis依赖 -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.3.1</version>
</dependency>
```

```yaml
# application.yml：数据源与MP核心配置
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mp?useUnicode=true&characterEncoding=utf-8
    username: root
    password: 123456
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl  # 打印SQL日志
    map-underscore-to-camel-case: true                      # 驼峰映射
  global-config:
    db-config:
      id-type: assign_id        # 全局主键策略:雪花算法
      table-prefix: t_          # 表名前缀,实体类User自动映射t_user
```

```java
// 实体类：与表 t_user 映射
@Data
@TableName("t_user")  // 显式指定表名（不配table-prefix时使用）
public class User {
    @TableId(type = IdType.AUTO)  // 主键,采用数据库自增
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

```java
// Mapper接口：继承BaseMapper即拥有单表CRUD能力,无需编写XML
@Mapper
public interface UserMapper extends BaseMapper<User> {
    // BaseMapper已提供insert/deleteById/updateById/selectById等方法
}
```

```java
// 启动类：扫描Mapper接口所在包
@MapperScan("com.example.mapper")
@SpringBootApplication
public class MpApplication { public static void main(String[] a){ SpringApplication.run(MpApplication.class,a);} }
```

> **原理**：MP启动时通过 `MybatisPlusAutoConfiguration` 注册 `SqlSessionFactory`，并将 `BaseMapper` 的方法解析为 `MappedStatement` 注入到MyBatis配置中，使接口方法与SQL绑定。

---

## 2. CRUD核心操作

### 2.1 添加操作

**核心方法**：`BaseMapper#insert(T entity)`，返回受影响行数；插入后主键会自动回写到实体对象。

```java
// 1. 基本插入：返回受影响行数,主键自动回写
User u = new User();
u.setName("张三"); u.setAge(20); u.setEmail("zs@xx.com");
int rows = userMapper.insert(u);     // rows=1
System.out.println(u.getId());       // 主键已回填

// 2. 主键策略由@TableId的type决定:
//    AUTO(数据库自增) / ASSIGN_ID(雪花算法,默认) / ASSIGN_UUID / INPUT(自行输入)
```

```java
// 3. IService批量插入（继承ServiceImpl后可直接用）
userService.saveBatch(userList);        // 批量插入,默认1000条一批
userService.saveBatch(userList, 500);   // 指定每批500条
```

> **注意**：MP默认只插入非null字段（`FieldStrategy.NOT_NULL`），null字段不会出现在SQL中，可避免覆盖默认值。

---

### 2.2 相关注解

**实体类常用注解：**

| 注解 | 作用 | 示例 |
|------|------|------|
| `@TableName` | 指定表名 | `@TableName("t_user")` |
| `@TableId` | 标记主键及策略 | `@TableId(type=IdType.AUTO)` |
| `@TableField` | 字段映射/策略 | `@TableField("user_name")` |
| `@TableLogic` | 逻辑删除标记 | `@TableLogic` |

```java
@Data
@TableName("t_user")                       // 类映射表t_user
public class User {
    @TableId(type = IdType.AUTO)            // 主键,数据库自增
    private Long id;

    @TableField("user_name")                // 字段名映射(驼峰可省略)
    private String name;

    @TableField(exist = false)              // 非数据库字段,不参与CRUD
    private String tempField;

    @TableField(insertStrategy = FieldStrategy.NEVER)  // 插入时不参与(用DB默认值)
    private LocalDateTime createTime;

    @TableField(fill = FieldFill.INSERT)    // 插入时自动填充(配合MetaObjectHandler)
    private LocalDateTime gmtCreate;

    @Version                                // 乐观锁版本号(配合乐观锁插件)
    private Integer version;

    @TableLogic                             // 逻辑删除字段(配合逻辑删除配置)
    private Integer deleted;
}
```

> **IdType策略详解**：
> - `AUTO`：数据库自增，依赖DB
> - `NONE`：未设置，跟随全局
> - `INPUT`：需自行set主键
> - `ASSIGN_ID`：雪花算法分布式ID（默认），19位long
> - `ASSIGN_UUID`：分配UUID，String类型

---

### 2.3 修改操作

```java
// 1. 根据ID更新:updateById依据主键更新非null字段
User u = new User();
u.setId(1L); u.setName("李四");
userMapper.updateById(u);
// SQL: UPDATE t_user SET name=? WHERE id=?

// 2. 条件更新:用UpdateWrapper灵活构造条件
UpdateWrapper<User> uw = new UpdateWrapper<>();
uw.eq("age", 20).set("name", "王五");   // WHERE age=20 SET name='王五'
userMapper.update(null, uw);

// 3. Lambda方式:避免字段名硬编码写错
LambdaUpdateWrapper<User> luw = Wrappers.lambdaUpdate(User.class);
luw.gt(User::getAge, 18).set(User::getName, "成年");
userMapper.update(null, luw);
```

> **关键区别**：`updateById` 仅更新非null字段（部分更新）；如需将字段更新为null，需用 `UpdateWrapper.set("field", null)` 或 `@TableField(updateStrategy = FieldStrategy.IGNORED)`。

---

### 2.4 删除操作

```java
// 1. 根据ID删除
userMapper.deleteById(1L);

// 2. 批量删除
userMapper.deleteBatchIds(Arrays.asList(2L, 3L, 4L));

// 3. 根据Map条件删除(等值条件)
Map<String,Object> map = new HashMap<>();
map.put("name","张三"); map.put("age",20);
userMapper.deleteByMap(map);   // AND 关系

// 4. 条件删除(Lambda)
LambdaQueryWrapper<User> w = Wrappers.lambdaQuery(User.class);
w.lt(User::getAge, 18);
userMapper.delete(w);
```

> **物理删除 vs 逻辑删除**：上述方法默认执行物理DELETE。若启用了逻辑删除（见第5章），DELETE语句会自动转为UPDATE设置删除标识字段，且查询自动过滤已删除数据。

---

### 2.5 查询操作

```java
// 1. 根据ID查询
User u = userMapper.selectById(1L);

// 2. 批量查询
List<User> list = userMapper.selectBatchIds(Arrays.asList(1L, 2L, 3L));

// 3. 查询单条(结果多条会抛TooManyResultsException)
LambdaQueryWrapper<User> w1 = Wrappers.lambdaQuery(User.class);
w1.eq(User::getName, "张三");
User one = userMapper.selectOne(w1);

// 4. 条件查询列表
List<User> users = userMapper.selectList(
    Wrappers.<User>lambdaQuery().gt(User::getAge, 18)
);

// 5. 查询数量
long count = userMapper.selectCount(
    Wrappers.<User>lambdaQuery().ge(User::getAge, 18)
);

// 6. 查询所有
List<User> all = userMapper.selectList(null);
```

> **推荐使用Lambda**：`User::getName` 编译期检查字段名，重构安全；字符串 `"name"` 易写错且不易发现。

---

### 2.6 条件构造器

**Wrapper体系**：`AbstractWrapper` → `QueryWrapper`/`UpdateWrapper`，对应Lambda版本为 `LambdaQueryWrapper`/`LambdaUpdateWrapper`。

```java
// 1. 比较条件: eq/ne/gt/ge/lt/le/between/like/in
LambdaQueryWrapper<User> w = Wrappers.lambdaQuery(User.class);
w.eq(User::getName, "张三")        // name = '张三'
 .ne(User::getAge, 20)            // age <> 20
 .gt(User::getAge, 18)            // age > 18
 .between(User::getAge, 18, 30)   // age BETWEEN 18 AND 30
 .like(User::getName, "张")       // name LIKE '%张%'
 .likeRight(User::getName, "张")  // name LIKE '张%'
 .in(User::getId, Arrays.asList(1L,2L))
 .isNotNull(User::getEmail);      // email IS NOT NULL
```

```java
// 2. 条件拼接:or / nested / apply(原生SQL)
w.and(q -> q.gt(User::getAge, 18).lt(User::getAge, 30))  // (age>18 AND age<30)
 .or(q -> q.isNull(User::getEmail))                       // OR email IS NULL
 .apply("date(create_time) = {0}", "2026-08-07");         // 原生条件,参数化防注入
```

```java
// 3. 排序/分组/选择字段
LambdaQueryWrapper<User> qw = Wrappers.lambdaQuery(User.class);
qw.select(User::getId, User::getName)  // 只查指定列
  .orderByDesc(User::getAge)           // ORDER BY age DESC
  .groupBy(User::getName);             // GROUP BY name
```

```java
// 4. condition参数:动态拼装(为true才生效,简化if判断)
String keyword = request.getParameter("name");
LambdaQueryWrapper<User> w = Wrappers.lambdaQuery(User.class);
w.like(StringUtils.isNotBlank(keyword), User::getName, keyword);  // 关键字非空才加条件
List<User> list = userMapper.selectList(w);
```

> **高频技巧**：`condition` 形参是MP的精髓，让动态查询无需手写大量if-else，链式即可完成。

---

### 2.7 分页查询

**注意**：必须先配置分页插件（见[4.2](#42-分页插件与防止全表更新删除插件)），否则 `selectPage` 不会执行 `LIMIT`，而是查全表。

```java
// 1. 基本分页:setCurrent页码,setSize每页条数
Page<User> page = new Page<>(1, 10);   // 第1页,每页10条
Page<User> result = userMapper.selectPage(page, null);

result.getRecords();   // 当前页数据List
result.getTotal();     // 总记录数
result.getPages();     // 总页数
result.getCurrent();   // 当前页码
```

```java
// 2. 条件分页:配合Wrapper
Page<User> page = new Page<>(2, 5);
LambdaQueryWrapper<User> w = Wrappers.lambdaQuery(User.class);
w.gt(User::getAge, 18).orderByDesc(User::getId);
Page<User> r = userMapper.selectPage(page, w);
```

```java
// 3. 自定义SQL分页:Mapper方法第一个参数必须为IPage
// Mapper接口:
// IPage<UserVO> selectUserVo(Page<UserVO> page, @Param("ew") Wrapper<User> wrapper);
// XML中: SELECT u.*, d.name AS deptName FROM t_user u LEFT JOIN t_dept d ON u.dept_id=d.id
//        ${ew.customSqlSegment}   ← 自动拼接WHERE条件
```

> **原理**：分页插件拦截 `Executor#query`，先执行 `SELECT COUNT(*)` 再改写SQL追加 `LIMIT ?,?`，是真正的物理分页。

---

### 2.8 全局配置

**目的**：将通用规则统一到配置文件，减少实体类重复注解。

```yaml
mybatis-plus:
  global-config:
    banner: false                  # 关闭启动banner
    db-config:
      id-type: assign_id           # 全局主键策略
      table-prefix: t_             # 全局表名前缀
      table-underline: true        # 驼峰转下划线
      logic-delete-field: deleted  # 全局逻辑删除字段
      logic-delete-value: 1        # 已删除值
      logic-not-delete-value: 0    # 未删除值
      insert-strategy: not_null    # 全局插入策略
      update-strategy: not_null    # 全局更新策略
  configuration:
    map-underscore-to-camel-case: true
    cache-enabled: false           # 关闭一级缓存
```

**策略优先级**：实体注解 `@TableId`/`@TableField` > 全局配置 > 默认值。

```java
// 示例:配置table-prefix=t_后,以下实体自动映射t_user,无需@TableName
@Data
public class User {           // 类名User → 表t_user
    private Long id;          // id默认雪花算法
    private String userName;  // userName → user_name列
}
```

---

## 3. ActiveRecord模式

### 3.1 ActiveRecord概念

**ActiveRecord（活动记录）** 是一种领域模型模式：一个对象既承载业务数据，又自带持久化方法（save/delete等），使实体类本身成为数据访问入口。

**MP中的实现**：实体类继承 `Model<T>` 后即可直接调用CRUD方法，无需通过Mapper。底层仍委托给 `BaseMapper`，要求对应的Mapper Bean必须存在于Spring容器中。

```java
// 标准用法：实体类继承Model<T>
@Data
@TableName("t_user")
public class User extends Model<User> {   // 继承Model获得CRUD能力
    @TableId(type = IdType.AUTO)
    private Long id;
    private String name;
    private Integer age;
}
```

```java
// 对应Mapper仍需定义（AR底层依赖它注入到容器）
@Mapper
public interface UserMapper extends BaseMapper<User> { }
```

> **适用场景**：领域驱动的中小型项目，对象操作即数据库操作，编码流畅。**不适用**于复杂多表关联业务，仍应走Service/Mapper。

---

### 3.2 ActiveRecord增删改查

```java
// 1. 新增：实体对象直接insert,返回boolean
User u = new User();
u.setName("AR用户").setAge(25);
boolean ok = u.insert();        // 等价于 userMapper.insert(u)

// 2. 查询：通过主键或条件
User one = u.selectById(1L);    // 等价于 userMapper.selectById(1L)

// 3. 修改：updateById
u.setId(1L); u.setName("改名");
boolean updated = u.updateById();

// 4. 删除：deleteById/deleteById(主键值)
boolean deleted = u.deleteById();    // 依据实体主键删除
boolean deleted2 = u.deleteById(2L); // 直接传主键
```

```java
// 5. AR也支持条件构造器(底层仍调用BaseMapper)
List<User> list = new User().selectAll();   // 查全部
// 通过selectList传Wrapper:
LambdaQueryWrapper<User> w = Wrappers.lambdaQuery(User.class);
w.gt(User::getAge, 18);
List<User> adults = new User().selectList(w);
```

> **本质**：`Model<T>` 内部通过 Spring 工具类获取对应的 `BaseMapper` Bean 并委托调用，AR只是API的语法糖，性能与Mapper一致。

---

## 4. 插件机制

### 4.1 插件概述

**MP插件基于MyBatis的 `Interceptor` 接口实现**，通过拦截 `Executor`、`StatementHandler`、`ParameterHandler`、`ResultSetHandler` 四大核心对象的方法，对SQL进行改写或增强。

**MP内置插件：**

| 插件 | 拦截对象 | 作用 |
|------|----------|------|
| `PaginationInnerInterceptor` | Executor.query | 物理分页 |
| `OptimisticLockerInnerInterceptor` | Executor.update | 乐观锁版本控制 |
| `BlockAttackInnerInterceptor` | Executor.update/delete | 阻止全表更新/删除 |
| `IllegalSQLInnerInterceptor` | StatementHandler | SQL安全规范校验 |
| `DataPermissionInterceptor` | StatementHandler | 数据权限控制 |

```java
// 插件统一注册：通过MybatisPlusInterceptor聚合多个内部插件
@Configuration
public class MybatisPlusConfig {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        // 顺序建议: 分页 → 乐观锁 → 防全表(后加的先执行)
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        interceptor.addInnerInterceptor(new BlockAttackInnerInterceptor());
        return interceptor;
    }
}
```

> **关键**：3.4.0+版本必须使用 `MybatisPlusInterceptor` 聚合，旧版 `@Bean PaginationInterceptor` 已废弃。

---

### 4.2 分页插件与防止全表更新删除插件

**分页插件 `PaginationInnerInterceptor`**：

```java
// 1. 注册插件(指定数据库类型,影响LIMIT语法)
interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
```

```java
// 2. 使用:不调用selectPage也可手动拼接(原始方式)
Page<User> page = new Page<>(1, 10);
// 优化:不需要总数时,关闭count提升性能
Page<User> fastPage = new Page<>(1, 10, false);
userMapper.selectPage(fastPage, null);   // 不执行SELECT COUNT(*)
```

```java
// 3. 自定义count优化:当原SQL过于复杂,可指定单独的count SQL
// 在Mapper方法上:@Select指定countSql,MP自动识别
```

**防止全表更新删除插件 `BlockAttackInnerInterceptor`**：

```java
// 注册后,update/delete没有WHERE条件会直接抛异常
interceptor.addInnerInterceptor(new BlockAttackInnerInterceptor());
```

```java
// 危险操作被拦截:
userMapper.delete(null);   // 抛 ProhibitionOfWorkingException
userMapper.update(null, new UpdateWrapper<>());  // 全表更新也被阻止

// 正确做法:必须带条件
userMapper.delete(Wrappers.<User>lambdaQuery().eq(User::getAge, 0));
```

> **生产必装**：该插件能有效防止 `delete from t_user` 这类灾难性误操作，强烈建议在生产环境开启。

---

### 4.3 乐观锁插件概念

**乐观锁**：假设冲突很少发生，更新时检查版本号是否变化，而非加锁阻塞。适合读多写少场景。

**实现原理**：

```
1. 表中增加version字段，初始值=0
2. 读取记录时: SELECT id,name,version FROM t_user WHERE id=1   → version=0
3. 更新时带上version条件:
   UPDATE t_user SET name=?, version=version+1 
   WHERE id=? AND version=0     ← 期望版本必须匹配
4. 若期间被他人修改,version已变化,affectedRows=0,更新失败
```

**对比悲观锁**：

| 维度 | 乐观锁 | 悲观锁 |
|------|--------|--------|
| 思路 | 先读后校验 | 先加锁再操作 |
| 并发性 | 高 | 低 |
| 适用 | 读多写少 | 写多冲突频繁 |
| 实现 | version字段 | `SELECT ... FOR UPDATE` |

> **要求**：实体必须有 `@Version` 标注的字段，且数据库表存在对应列；插件只对 `updateById(entity)`、`update(entity,wrapper)` 生效。

---

### 4.4 乐观锁插件使用

```java
// 1. 注册插件
interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
```

```java
// 2. 实体类添加version字段
@Data
@TableName("t_user")
public class User {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String name;
    @Version                  // 乐观锁版本字段
    private Integer version;
}
```

```java
// 3. 标准乐观锁更新流程:先查后改
User u = userMapper.selectById(1L);   // 查询时拿到version=0
u.setName("新名字");
int rows = userMapper.updateById(u);  // 自动追加AND version=0
// SQL: UPDATE t_user SET name=?, version=1 WHERE id=1 AND version=0

if (rows == 0) {
    // 版本已变,需重试或提示用户
    throw new RuntimeException("数据已被他人修改,请刷新后重试");
}
```

> **执行细节**：
> - 插件拦截 `update` 操作，从实体取出 `@Version` 字段值作为条件，并将新版本设为 `old+1`
> - **必须先查出实体再更新**，不能new一个对象直接updateById（version为null无法作为条件）
> - 乐观锁与逻辑删除可共存，插件会自动组合条件

---

## 5. 逻辑删除

### 5.1 逻辑删除概念

**逻辑删除（软删除）**：并非真正从表中删除数据，而是通过标记字段（如 `deleted=1`）表示已删除。查询时自动过滤已删除数据。

**适用场景**：
- 需要数据回溯、审计、恢复（订单、用户、日志）
- 外键关联复杂，物理删除会破坏参照完整性
- 法规要求保留数据（如交易记录）

**对比物理删除**：

| 维度 | 逻辑删除 | 物理删除 |
|------|----------|----------|
| 存储 | 保留，仅标记 | 真正删除 |
| 查询 | 自动过滤 | 无影响 |
| 性能 | 数据增长，需索引 | 表保持小 |
| 可恢复 | 可恢复 | 不可恢复 |

**MP实现要点**：
- 实体字段加 `@TableLogic`
- `deleteById` 等方法自动变为 `UPDATE ... SET deleted=1`
- 所有查询自动追加 `WHERE deleted=0`
- 唯一索引需包含 `deleted` 字段，否则同一条数据无法多次"删除-重建"

---

### 5.2 逻辑删除使用

```yaml
# 方式一:全局配置(推荐,统一管理)
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: deleted    # 全局逻辑删除字段名
      logic-delete-value: 1          # 删除值
      logic-not-delete-value: 0      # 未删除值
```

```java
// 方式二:实体注解(优先级高于全局配置)
@Data
public class User {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String name;
    @TableLogic                  // 标记逻辑删除字段
    private Integer deleted;     // 0未删除,1已删除
}
```

```java
// 1. 删除:实际执行UPDATE,而非DELETE
userMapper.deleteById(1L);
// SQL: UPDATE t_user SET deleted=1 WHERE id=1 AND deleted=0
```

```java
// 2. 查询:自动过滤已删除数据
List<User> list = userMapper.selectList(null);
// SQL: SELECT * FROM t_user WHERE deleted=0
```

```java
// 3. 需要查询已删除数据:绕过逻辑删除过滤
// 方式1: 自定义XML中不使用MP条件构造器
List<User> all = userMapper.selectAllIncludeDeleted();   // 自定义SQL

// 方式2: 临时关闭(3.5.0+)
List<User> r = InterceptorIgnoreHelper.execute(IgnoreStrategy.builder().logicDelete(true).build(),
    () -> userMapper.selectList(null));
```

> **常见坑**：
> - 唯一索引冲突：删除后再插入同主键业务数据会失败 → 唯一索引需含 `deleted` 字段
> - `update` 操作不会自动追加 `deleted=0`，需在Wrapper中手动添加，否则可能更新到已删除数据

---

## 6. 扩展功能

### 6.1 自动填充

**场景**：每张表都有 `create_time`、`update_time`、`create_by` 等审计字段，每次插入/更新都要手动set，繁琐易漏。MP提供 `MetaObjectHandler` 在SQL执行前自动填充。

```java
// 1. 实体标注填充策略
@Data
@TableName("t_user")
public class User {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String name;
    @TableField(fill = FieldFill.INSERT)         // 插入时填充
    private LocalDateTime createTime;
    @TableField(fill = FieldFill.INSERT_UPDATE)  // 插入和更新时填充
    private LocalDateTime updateTime;
    @TableField(fill = FieldFill.INSERT)
    private String createBy;
}
```

```java
// 2. 实现MetaObjectHandler:注册为Spring Bean
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        // strictInsertFill:字段存在才填充(避免覆盖非null值)
        this.strictInsertFill(metaObject, "createTime", LocalDateTime.class, LocalDateTime.now());
        this.strictInsertFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
        this.strictInsertFill(metaObject, "createBy", String.class, currentUserName());
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        // 更新时只刷新updateTime
        this.strictUpdateFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
    }

    private String currentUserName() {
        return Optional.ofNullable(SecurityContextHolder.getContext().getAuthentication())
                .map(Authentication::getName).orElse("system");
    }
}
```

```java
// 3. 使用时无需手动set时间字段
User u = new User();
u.setName("自动填充");
userMapper.insert(u);   // createTime/updateTime/createBy 自动赋值
```

> **填充时机**：在 `MybatisDefaultParameterHandler` 设置参数前调用，填充值进入实体对象后参与SQL拼接。`strictInsertFill` 会检查字段是否已赋值，避免覆盖传入值。

---

### 6.2 SQL注入器

**问题**：`BaseMapper` 提供的方法是全局通用的，无法直接满足"按业务规则自定义"的通用方法。SQL注入器允许向 `BaseMapper` **批量注入自定义通用方法**，所有Mapper共享。

**典型场景**：所有Mapper都需要 `deleteAll`、`insertBatchSomeColumn`、`updateByCondition` 等MP未提供的方法。

```java
// 1. 定义通用方法:继承AbstractMethod,实现injectMappedStatement
public class DeleteAll extends AbstractMethod {
    @Override
    public MappedStatement injectMappedStatement(Class<?> mapperClass, Class<?> modelClass,
                                                TableInfo tableInfo) {
        // 自定义SQL:物理删除全表(注意:生产慎用,仅作演示)
        String sql = "DELETE FROM " + tableInfo.getTableName();
        SqlSource sqlSource = languageDriver.createSqlSource(configuration, sql, modelClass);
        return addDeleteMappedStatement(mapperClass, "deleteAll", sqlSource);
    }
}
```

```java
// 2. 自定义注入器:注册所有自定义方法
public class MySqlInjector extends DefaultSqlInjector {
    @Override
    public List<AbstractMethod> getMethodList(Class<?> mapperClass, TableInfo tableInfo) {
        List<AbstractMethod> list = super.getMethodList(mapperClass, tableInfo);  // 保留原有方法
        list.add(new DeleteAll());            // 追加自定义方法
        return list;
    }
}
```

```java
// 3. 注册注入器为Bean
@Configuration
public class MpConfig {
    @Bean
    public ISqlInjector sqlInjector() { return new MySqlInjector(); }
}
```

```java
// 4. 定义公共Mapper基类,所有需要该方法的Mapper继承它
public interface MyBaseMapper<T> extends BaseMapper<T> {
    int deleteAll();   // 由注入器自动生成SQL
}

@Mapper
public interface UserMapper extends MyBaseMapper<User> { }
// 使用: userMapper.deleteAll();
```

> **MP内置可注入方法**：`InsertBatchSomeColumn`、`LogicDeleteByIdWithFill`、`AlwaysUpdateSomeColumnById` 等，位于 `com.baomidou.mybatisplus.extension.injector.methods` 包，可直接复用。

---

### 6.3 代码生成器

**新版（3.5.3+）代码生成器**采用构建器风格，灵活强大，可生成 Entity/Mapper/Service/Controller 各层代码及XML。

```java
// 1. 引入依赖
// mybatis-plus-generator(代码生成核心) + 模板引擎(freemarker/velocity)
```

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.5.3.1</version>
</dependency>
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.32</version>
</dependency>
```

```java
// 2. 快速生成:FastAutoGenerator链式API
FastAutoGenerator.create(
        "jdbc:mysql://localhost:3306/mp", "root", "123456")
    .globalConfig(builder -> builder
            .author("dev")
            .outputDir(System.getProperty("user.dir") + "/src/main/java")
            .enableSwagger()                    // 生成Swagger注解
            .dateType(DateType.TIME_PACK))      // 使用LocalDateTime
    .packageConfig(builder -> builder
            .parent("com.example")
            .moduleName("system")
            .pathInfo(Collections.singletonMap(
                OutputFile.xml,                 // Mapper XML输出路径
                System.getProperty("user.dir") + "/src/main/resources/mapper/system")))
    .strategyConfig(builder -> builder
            .addInclude("t_user", "t_dept")     // 需生成的表名
            .addTablePrefix("t_")               // 去除表前缀生成类名
            .entityBuilder()
                .enableLombok()                 // 使用Lombok
                .enableTableFieldAnnotation()   // 字段加@TableField
                .logicDeleteColumnName("deleted")
                .versionColumnName("version")
                .controllerBuilder().enableRestStyle()
                .mapperBuilder().enableMapperXml())
    .templateEngine(new FreemarkerTemplateEngine())  // 指定模板引擎
    .execute();   // 执行生成
```

```java
// 3. 生成的目录结构示例
// com.example.system
//   ├── controller/UserController.java
//   ├── service/UserService.java  / impl/UserServiceImpl.java
//   ├── mapper/UserMapper.java
//   └── entity/User.java
// resources/mapper/system/UserMapper.xml
```

> **进阶技巧**：
> - **自定义模板**：覆盖 `templates/entity.java.ftl`，在 `templateConfig` 中指定路径，可定制生成内容
> - **字段类型映射**：通过 `dbConfig` 的 `typeMap` 自定义DB类型到Java类型的映射
> - **多数据源**：`DataSourceConfig` 支持配置 `typeConvert`、`tableNameConvert` 实现命名风格转换
> - **Maven插件方式**：`mybatis-plus-generator` 也提供maven插件，避免写Java代码

---

## 附录：MyBatis-Plus知识脉络图

```
MyBatis-Plus
├── 基础: 集成SpringBoot / 实体注解 / BaseMapper
├── CRUD: insert/update/delete/select + Wrapper条件构造
│   ├── 条件构造器: QueryWrapper / LambdaQueryWrapper
│   ├── 分页查询: Page + PaginationInnerInterceptor
│   └── 全局配置: application.yml统一策略
├── 模式: ActiveRecord (Model<T>) —— 实体即DAO
├── 插件 (MybatisPlusInterceptor聚合)
│   ├── PaginationInnerInterceptor  —— 物理分页
│   ├── OptimisticLockerInnerInterceptor —— 乐观锁
│   └── BlockAttackInnerInterceptor —— 防全表操作
├── 逻辑删除: @TableLogic + 全局配置,查询自动过滤
└── 扩展
    ├── MetaObjectHandler —— 自动填充审计字段
    ├── ISqlInjector —— 注入自定义通用Mapper方法
    └── FastAutoGenerator —— 一键生成各层代码
```

**学习路径建议**：基础集成 → CRUD熟练 → Wrapper精通 → 插件配置 → 逻辑删除/自动填充 → SQL注入器/代码生成器。每一步都建立在前一步之上，循序渐进。
