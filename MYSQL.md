# MySQL 核心技术手册

> **阅读指南**：由浅入深，每个知识点均配有精简示例。建议按顺序阅读，环环相扣。

## 目录

- [**第一部分：MySQL 架构与核心概念**](#第一部分mysql-架构与核心概念)
  - [1.1 整体架构分层](#11-整体架构分层)
  - [1.2 一条 SQL 的执行路径](#12-一条-sql-的执行路径)
  - [1.3 InnoDB 内存结构与磁盘结构](#13-innodb-内存结构与磁盘结构)
  - [1.4 存储引擎对比](#14-存储引擎对比)
- [**第二部分：基础查询与数据操作（DML）**](#第二部分基础查询与数据操作dml)
  - [2.1 SELECT 查询骨架](#21-select-查询骨架)
  - [2.2 WHERE 过滤大全](#22-where-过滤大全)
  - [2.3 ORDER BY 排序细节](#23-order-by-排序细节)
  - [2.4 GROUP BY + 聚合函数（五大金刚）](#24-group-by--聚合函数五大金刚)
  - [2.5 HAVING vs WHERE](#25-having-vs-where)
  - [2.6 JOIN 家族（七种 JOIN）](#26-join-家族七种-join)
  - [2.7 子查询（Subquery）](#27-子查询subquery)
  - [2.8 集合操作](#28-集合操作)
  - [2.9 DML 三剑客](#29-dml-三剑客)
  - [2.10 UPSERT（键冲突处理）](#210-upsert键冲突处理)
- [**第三部分：数据库对象设计与结构（DDL）**](#第三部分数据库对象设计与结构ddl)
  - [3.1 整数字段选型](#31-整数字段选型)
  - [3.2 字符串选型](#32-字符串选型)
  - [3.3 日期时间选型](#33-日期时间选型)
  - [3.4 精确小数（金融计算必看）](#34-精确小数金融计算必看)
  - [3.5 六大约束](#35-六大约束)
  - [3.6 DDL 操作速查](#36-ddl-操作速查)
  - [3.7 表分区（大表优化利器）](#37-表分区大表优化利器)
  - [3.8 定时事件](#38-定时事件)
- [**第四部分：编程与流程控制**](#第四部分编程与流程控制)
  - [4.1 变量体系](#41-变量体系)
  - [4.2 流程控制](#42-流程控制)
  - [4.3 游标（逐行处理）](#43-游标逐行处理)
  - [4.4 触发器](#44-触发器)
  - [4.5 存储过程 vs 函数](#45-存储过程-vs-函数)
- [**第五部分：事务、MVCC 与锁机制**](#第五部分事务mvcc-与锁机制)
  - [5.1 事务控制（ACID）](#51-事务控制acid)
  - [5.2 MVCC（多版本并发控制）](#52-mvcc多版本并发控制)
  - [5.3 Redo Log 与 Undo Log](#53-redo-log-与-undo-log)
  - [5.4 锁机制（并发核心）](#54-锁机制并发核心)
  - [5.5 乐观锁（应用层实现）](#55-乐观锁应用层实现)
  - [5.6 表级锁](#56-表级锁)
- [**第六部分：索引原理与优化**](#第六部分索引原理与优化)
  - [6.1 索引类型总览](#61-索引类型总览)
  - [6.2 B+Tree 结构与原理](#62-btree-结构与原理)
  - [6.3 复合索引与最左前缀原则](#63-复合索引与最左前缀原则)
  - [6.4 索引失效场景（避坑指南）](#64-索引失效场景避坑指南)
  - [6.5 覆盖索引（无需回表）](#65-覆盖索引无需回表)
  - [6.6 前缀索引](#66-前缀索引)
  - [6.7 索引下推（ICP，MySQL 5.6+）](#67-索引下推icp-mysql-56)
- [**第七部分：性能分析与调优**](#第七部分性能分析与调优)
  - [7.1 EXPLAIN 详解](#71-explain-详解)
  - [7.2 慢查询定位与优化](#72-慢查询定位与优化)
  - [7.3 查询优化器提示（Hint）](#73-查询优化器提示hint)
  - [7.4 参数调优速查](#74-参数调优速查)
- [**第八部分：权限与安全管理**](#第八部分权限与安全管理)
  - [8.1 用户与权限管理](#81-用户与权限管理)
  - [8.2 权限层级](#82-权限层级)
  - [8.3 安全最佳实践](#83-安全最佳实践)
  - [8.4 SQL 注入防护](#84-sql-注入防护)
- [**第九部分：备份与恢复**](#第九部分备份与恢复)
  - [9.1 逻辑备份：mysqldump](#91-逻辑备份mysqldump)
  - [9.2 物理备份：Xtrabackup](#92-物理备份xtrabackup)
  - [9.3 Binlog 恢复（时间点恢复 PITR）](#93-binlog-恢复时间点恢复-pitr)
  - [9.4 备份策略建议](#94-备份策略建议)
- [**第十部分：主从复制与高可用**](#第十部分主从复制与高可用)
  - [10.1 主从复制原理](#101-主从复制原理)
  - [10.2 搭建主从（核心步骤）](#102-搭建主从核心步骤)
  - [10.3 GTID 复制（MySQL 5.6+）](#103-gtid-复制mysql-56)
  - [10.4 常见复制架构](#104-常见复制架构)
  - [10.5 主从延迟处理](#105-主从延迟处理)
- [**第十一部分：MySQL 8.0+ 高级特性**](#第十一部分mysql-80-高级特性)
  - [11.1 窗口函数（Window Functions）](#111-窗口函数window-functions)
  - [11.2 CTE（公共表表达式）](#112-cte公共表表达式)
  - [11.3 JSON 数据类型](#113-json-数据类型)
  - [11.4 其他 8.0 新特性](#114-其他-80-新特性)
- [**第十二部分：数据导入导出**](#第十二部分数据导入导出)
  - [12.1 SELECT INTO OUTFILE（导出）](#121-select-into-outfile导出)
  - [12.2 LOAD DATA INFILE（导入，比INSERT快10~100倍）](#122-load-data-infile导入比insert快10100倍)
  - [12.3 mysqlimport 命令行工具](#123-mysqlimport-命令行工具)
- [**第十三部分：监控与故障排查**](#第十三部分监控与故障排查)
  - [13.1 连接与会话监控](#131-连接与会话监控)
  - [13.2 锁监控](#132-锁监控)
  - [13.3 性能监控关键指标](#133-性能监控关键指标)
  - [13.4 常见故障排查速查](#134-常见故障排查速查)
  - [13.5 常用运维命令速查](#135-常用运维命令速查)
- [**附录：最佳实践与规范**](#附录最佳实践与规范)
  - [A.1 建表规范](#a1-建表规范)
  - [A.2 反模式（千万别这么做）](#a2-反模式千万别这么做)
  - [A.3 范式与反范式](#a3-范式与反范式)

---

## 第一部分：MySQL 架构与核心概念

### 1.1 整体架构分层

```
┌─────────────────────────────────┐
│  连接层：连接池、认证、线程复用   │
├─────────────────────────────────┤
│  服务层：SQL解析、优化器、查询缓存 │
├─────────────────────────────────┤
│  引擎层：InnoDB / MyISAM / Memory │  ← 可插拔存储引擎
├─────────────────────────────────┤
│  存储层：文件系统、日志、数据文件   │
└─────────────────────────────────┘
```

### 1.2 一条 SQL 的执行路径

```sql
-- 以这条查询为例，跟踪全链路
SELECT * FROM users WHERE id = 100;
```


| 阶段    | 组件                | 做什么                               |
| ------- | ------------------- | ------------------------------------ |
| ① 连接 | 连接器              | 建立TCP连接、身份认证、权限读取      |
| ② 缓存 | 查询缓存（8.0移除） | 以SQL为key查找结果（表有更新即失效） |
| ③ 解析 | 分析器              | 词法分析→语法分析→生成解析树       |
| ④ 优化 | 优化器              | 选择索引、决定JOIN顺序、重写SQL      |
| ⑤ 执行 | 执行器              | 调用引擎接口逐行读取/过滤/返回       |
| ⑥ 存储 | InnoDB引擎          | Buffer Pool查找→磁盘读取→返回      |

### 1.3 InnoDB 内存结构与磁盘结构

```
内存（In-Memory）：
  Buffer Pool      ← 缓存数据页（核心，占内存80%）
  Change Buffer    ← 缓存二级索引变更（减少随机IO）
  Adaptive Hash    ← 热点页的哈希索引（自动）
  Log Buffer       ← redo log缓冲（16MB，每s或事务提交刷盘）

磁盘（On-Disk）：
  表空间文件 (*.ibd)        ← 每表独立/系统共享
  redo log (ib_logfile0/1)  ← 物理日志，崩溃恢复
  undo log (回滚段)         ← 逻辑日志，事务回滚+MVCC
  binlog (binlog.000xxx)    ← Server层逻辑日志，主从复制
  doublewrite buffer        ← 防止页断裂（16K页写一半断电）
```

```sql
-- 查看 InnoDB 核心状态
SHOW ENGINE INNODB STATUS\G
-- 查看 Buffer Pool 命中率
SHOW STATUS LIKE 'Innodb_buffer_pool_read%';
-- 命中率 = 1 - (Innodb_buffer_pool_reads / Innodb_buffer_pool_read_requests)
```

### 1.4 存储引擎对比


| 特性         | InnoDB（默认） |      MyISAM      |     Memory     |
| ------------ | :------------: | :--------------: | :------------: |
| 事务（ACID） |       ✅       |        ❌        |       ❌       |
| 行级锁       |       ✅       |    ❌（表锁）    |   ❌（表锁）   |
| 外键         |       ✅       |        ❌        |       ❌       |
| MVCC         |       ✅       |        ❌        |       ❌       |
| 崩溃恢复     | ✅（redo log） |   ❌（修复表）   | ❌（重启丢失） |
| 数据存储     |     表空间     |   .MYD + .MYI   |      内存      |
| 适用场景     | 99%的OLTP场景 | 日志表、只读计数 |  临时表、缓存  |

```sql
-- 查看所有引擎
SHOW ENGINES;
-- 建表指定引擎
CREATE TABLE log_archive (
    id INT,
    content TEXT
) ENGINE=MyISAM;
-- 查看表使用的引擎
SELECT TABLE_NAME, ENGINE FROM information_schema.TABLES 
WHERE TABLE_SCHEMA = 'test_db';
```

---

## 第二部分：基础查询与数据操作（DML）

### 2.1 SELECT 查询骨架

```sql
-- 完整执行顺序（不是书写顺序！）
-- FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
SELECT DISTINCT department, COUNT(*) AS cnt   -- 5. 选列+聚合
FROM employees                                 -- 1. 读取表
WHERE salary > 5000                            -- 2. 行过滤
GROUP BY department                            -- 3. 分组
HAVING cnt > 3                                 -- 4. 分组后过滤
ORDER BY cnt DESC                              -- 6. 排序
LIMIT 10 OFFSET 0;                             -- 7. 分页
```

```sql
-- DISTINCT：对后面所有列组合去重
SELECT DISTINCT city, gender FROM users;

-- 表达式与常量
SELECT name, salary * 1.2 AS annual_bonus, 'CNY' AS currency FROM employees;
```

### 2.2 WHERE 过滤大全

```sql
-- 比较运算符
SELECT * FROM products WHERE price >= 100 AND price <= 500;
SELECT * FROM products WHERE price BETWEEN 100 AND 500;  -- 等价

-- 模糊匹配：% 任意多个字符，_ 单个字符
SELECT * FROM users WHERE name LIKE '张%';     -- 张开头
SELECT * FROM users WHERE name LIKE '张_';     -- 张+单个字（如"张三"）

-- 集合匹配
SELECT * FROM orders WHERE status IN ('paid', 'shipped', 'done');
SELECT * FROM orders WHERE status NOT IN ('cancelled');

-- NULL 判断（= NULL 永远为假！）
SELECT * FROM users WHERE email IS NULL;
SELECT * FROM users WHERE email IS NOT NULL;

-- 安全等于：可同时匹配 NULL 和普通值（少用）
SELECT * FROM users WHERE email <=> NULL;
```

### 2.3 ORDER BY 排序细节

```sql
-- 升序 ASC（默认）、降序 DESC
SELECT name, salary FROM employees ORDER BY salary DESC;
-- 多字段排序：先按部门升序，同部门内按薪资降序
SELECT dept, name, salary FROM employees ORDER BY dept ASC, salary DESC;
-- 可按列位置排序（不推荐，可读性差）
SELECT name, salary FROM employees ORDER BY 2 DESC;  -- 按第2列
```

### 2.4 GROUP BY + 聚合函数（五大金刚）

```sql
-- 聚合函数均忽略 NULL（COUNT(*) 除外）
SELECT 
    department,
    COUNT(*) AS total_emp,                          -- 总行数（含NULL）
    COUNT(email) AS has_email,                      -- email非NULL行数
    COUNT(DISTINCT city) AS city_cnt,               -- 去重城市数
    SUM(salary) AS total_cost,                      -- 求和
    AVG(salary) AS avg_salary,                      -- 平均（忽略NULL）
    MAX(hire_date) AS latest_hire,                  -- 最大（日期也可）
    MIN(salary) AS min_salary                       -- 最小
FROM employees
GROUP BY department;
-- 注意：SELECT中非聚合列必须出现在GROUP BY中
```

```sql
-- GROUP BY 配合 WITH ROLLUP：增加一行汇总
SELECT department, SUM(salary) 
FROM employees 
GROUP BY department WITH ROLLUP;  -- 最后一行 department=NULL 为总计
```

### 2.5 HAVING vs WHERE

```sql
-- WHERE：分组前过滤行，不能用聚合函数
-- HAVING：分组后过滤组，可以用聚合函数
SELECT department, AVG(salary) AS avg_sal
FROM employees
WHERE hire_date > '2020-01-01'          -- 先筛选2020年后入职
GROUP BY department
HAVING avg_sal > 10000;                 -- 再看哪些部门均薪>10000
```

### 2.6 JOIN 家族（七种 JOIN）

```sql
-- 建两张测试表
-- users: id=1(张三), id=2(李四)
-- orders: uid=1(订单A), uid=1(订单B), uid=3(订单C)

-- ① INNER JOIN：取交集（只返回有匹配的行）
SELECT u.name, o.order_name
FROM users u
INNER JOIN orders o ON u.id = o.uid;   -- 张三+A, 张三+B（李四无订单，不返回）

-- ② LEFT JOIN：保留左表全部行
SELECT u.name, o.order_name
FROM users u
LEFT JOIN orders o ON u.id = o.uid;    -- 张三+A,张三+B,李四+NULL

-- ③ RIGHT JOIN：保留右表全部行
SELECT u.name, o.order_name
FROM users u
RIGHT JOIN orders o ON u.id = o.uid;   -- 张三+A,张三+B,NULL+C

-- ④ FULL OUTER JOIN（MySQL 不支持，用 UNION 模拟）
SELECT u.name, o.order_name FROM users u LEFT JOIN orders o ON u.id = o.uid
UNION
SELECT u.name, o.order_name FROM users u RIGHT JOIN orders o ON u.id = o.uid;

-- ⑤ LEFT JOIN ... IS NULL：左表独有（只在users不在orders）
SELECT u.* FROM users u 
LEFT JOIN orders o ON u.id = o.uid 
WHERE o.uid IS NULL;                    -- 李四

-- ⑥ RIGHT JOIN ... IS NULL：右表独有
-- ⑦ CROSS JOIN：笛卡尔积（慎用！左表行数×右表行数）
SELECT u.name, o.order_name FROM users u CROSS JOIN orders o;
```

```sql
-- 多表连接（n个表至少需要 n-1 个JOIN条件）
SELECT u.name, o.order_name, p.product_name
FROM users u
INNER JOIN orders o ON u.id = o.uid
INNER JOIN products p ON o.product_id = p.id;
```

### 2.7 子查询（Subquery）

```sql
-- 标量子查询：返回单个值（可在SELECT/WHERE/HAVING中使用）
SELECT name, (SELECT MAX(salary) FROM employees) AS top_salary FROM employees;

-- 行子查询：返回一行多列
SELECT * FROM employees WHERE (department, salary) = (
    SELECT department, MAX(salary) FROM employees
);

-- 列子查询（IN / NOT IN）
SELECT * FROM employees WHERE department IN (
    SELECT department FROM departments WHERE location = '北京'
);

-- 表子查询（派生表，必须给别名）
SELECT dept, avg_sal FROM (
    SELECT department AS dept, AVG(salary) AS avg_sal 
    FROM employees GROUP BY department
) AS dept_stats WHERE avg_sal > 10000;
```

```sql
-- EXISTS：判断子查询是否有结果（比IN高效，查到就停）
SELECT * FROM employees e
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.uid = e.id
);

-- 关联子查询：内查询引用外查询的列（逐行执行，慎用！）
SELECT name, salary, (SELECT AVG(salary) FROM employees e2 
    WHERE e2.department = e1.department) AS dept_avg
FROM employees e1;
```

### 2.8 集合操作

```sql
-- UNION：并集+去重；UNION ALL：直接合并
SELECT name FROM china_users
UNION ALL                           -- ALL保留重复，比UNION快很多
SELECT name FROM us_users;

-- INTERSECT 模拟（交集：同时在两个表的数据）
SELECT DISTINCT a.name FROM china_users a
INNER JOIN us_users b ON a.name = b.name;

-- EXCEPT 模拟（差集：只在第一个表不在第二个表）
SELECT a.name FROM china_users a
LEFT JOIN us_users b ON a.name = b.name
WHERE b.name IS NULL;
```

### 2.9 DML 三剑客

```sql
-- INSERT：单行/多行
INSERT INTO users (name, age, city) VALUES ('张三', 25, '北京');
INSERT INTO users (name, age, city) VALUES 
    ('李四', 30, '上海'), 
    ('王五', 28, '深圳');
-- 从其他表插入
INSERT INTO users_archive SELECT * FROM users WHERE status = 'inactive';
```

```sql
-- UPDATE：务必带WHERE！否则全表更新
UPDATE users SET age = 26, city = '杭州' WHERE id = 1;
-- 多表更新
UPDATE users u INNER JOIN orders o ON u.id = o.uid 
SET u.order_count = u.order_count + 1 WHERE o.status = 'completed';
```

```sql
-- DELETE：务必带WHERE！全表删除用 TRUNCATE
DELETE FROM users WHERE id = 1;
-- 多表删除
DELETE u FROM users u 
LEFT JOIN orders o ON u.id = o.uid 
WHERE o.id IS NULL;  -- 删除没有订单的用户
```

```sql
-- TRUNCATE vs DELETE
TRUNCATE TABLE temp_log;    -- DDL，无法回滚，重置自增ID，快
DELETE FROM temp_log;       -- DML，可回滚，逐行删，慢，不重置自增
```

### 2.10 UPSERT（键冲突处理）

```sql
-- 方案1：ON DUPLICATE KEY UPDATE（主键/唯一键冲突时更新）
INSERT INTO users (id, name, score) VALUES (1, '张三', 90)
ON DUPLICATE KEY UPDATE name = '张三', score = score + 90;  -- 累加分数

-- 方案2：REPLACE INTO（先删后插，不推荐：会丢失未指定的列值）
REPLACE INTO users (id, name) VALUES (1, '张三新名');

-- 方案3（推荐）：先UPDATE，再判断affected_rows，若=0再INSERT
-- 应用层实现：UPDATE users SET score=90 WHERE id=1; 若 rows_affected=0 则 INSERT
```

---

## 第三部分：数据库对象设计与结构（DDL）

### 3.1 整数字段选型


| 类型      | 字节 | 有符号范围             | 无符号范围      |
| --------- | :--: | ---------------------- | --------------- |
| TINYINT   |  1  | -128 ~ 127             | 0 ~ 255         |
| SMALLINT  |  2  | -32768 ~ 32767         | 0 ~ 65535       |
| MEDIUMINT |  3  | -838万 ~ 838万         | 0 ~ 1677万      |
| INT       |  4  | -21亿 ~ 21亿           | 0 ~ 42亿        |
| BIGINT    |  8  | -9×10¹⁸ ~ 9×10¹⁸ | 0 ~ 1.8×10¹⁹ |

```sql
-- 最佳实践
CREATE TABLE users (
    id         BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,  -- 主键统一用BIGINT
    age        TINYINT UNSIGNED,      -- 年龄0~255够用
    status     TINYINT DEFAULT 0,     -- 状态用TINYINT替代ENUM
    PRIMARY KEY (id)
) ENGINE=InnoDB;
-- UNSIGNED 不自减：SELECT age - 1 FROM users WHERE age=0 → 语法报错
-- 解决：CAST(age AS SIGNED) - 1
```

### 3.2 字符串选型

```sql
-- CHAR(N)：固定长度，读取快，适合定长数据（MD5、手机号、身份证号）
-- VARCHAR(N)：变长，省空间，需要额外1~2字节存长度
CREATE TABLE example_str (
    mobile    CHAR(11),               -- 手机号固定11位
    name      VARCHAR(50),            -- 姓名变长
    intro     TEXT,                   -- 大文本（有性能问题，见下）
    content   MEDIUMTEXT,             -- 16MB大文本
    KEY idx_mobile (mobile)           -- CHAR在索引中更紧凑
);
```

```sql
-- VARCHAR 长度陷阱：N 是字符数，不是字节数！
-- UTF8MB4下，1字符=1~4字节，VARCHAR(255) → 最多 255×4+1 = 1021字节

-- TEXT 性能问题：用临时表时会落盘（磁盘），尽量用 VARCHAR 替代
-- 如需对TEXT字段索引，必须指定前缀长度
ALTER TABLE articles ADD INDEX idx_intro (intro(20));  -- 只索引前20字符
```

### 3.3 日期时间选型


| 类型      | 范围       | 时区 | 存储 | 适用                       |
| --------- | ---------- | :--: | ---- | -------------------------- |
| DATE      | 1000~9999  |  ❌  | 3B   | 生日                       |
| DATETIME  | 1000~9999  |  ❌  | 5~8B | 业务时间（推荐）           |
| TIMESTAMP | 1970~2038  |  ✅  | 4B   | 自动更新时间（2038问题！） |
| TIME      | -838h~838h |  ❌  | 3B   | 时长                       |

```sql
CREATE TABLE events (
    created_at  DATETIME DEFAULT CURRENT_TIMESTAMP,             -- 创建时间
    updated_at  DATETIME DEFAULT CURRENT_TIMESTAMP 
                ON UPDATE CURRENT_TIMESTAMP,                    -- 自动更新
    birth_date  DATE,
    PRIMARY KEY (id)
) ENGINE=InnoDB;
-- 关键原则：业务时间存DATETIME（不受时区影响），别用TIMESTAMP
```

### 3.4 精确小数（金融计算必看）

```sql
-- FLOAT/DOUBLE：二进制近似存储，存0.1实际是0.0999999... 禁止用于钱！
-- DECIMAL(M,D)：字符串存储，精确。M=总位数，D=小数位
CREATE TABLE finance (
    amount     DECIMAL(18,2),   -- 最大 ¥9999万亿.99，够用
    rate       DECIMAL(5,4),    -- 如 0.0536
    tax        DECIMAL(10,2)
) ENGINE=InnoDB;

-- 对比测试
SELECT 0.1 + 0.2;                          -- 0.30000000000000004
SELECT CAST(0.1 AS DECIMAL(2,1)) + CAST(0.2 AS DECIMAL(2,1)); -- 0.3 ✓
```

### 3.5 六大约束

```sql
CREATE TABLE products (
    id        BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    sku       VARCHAR(32)  NOT NULL,
    name      VARCHAR(100) NOT NULL DEFAULT '',
    price     DECIMAL(10,2) NOT NULL DEFAULT 0.00,
    stock     INT NOT NULL DEFAULT 0,
    status    TINYINT NOT NULL DEFAULT 1,

    -- ① PRIMARY KEY：唯一+非空+聚簇索引（每表只能一个）
    PRIMARY KEY (id),

    -- ② UNIQUE：唯一（可多个NULL），自动创建索引
    UNIQUE KEY uk_sku (sku),

    -- ③ FOREIGN KEY：引用完整性（线上慎用，有锁开销）
    -- CONSTRAINT fk_category FOREIGN KEY (category_id) 
    --     REFERENCES categories(id) ON DELETE SET NULL,

    -- ④ CHECK（MySQL 8.0+真正生效）
    CONSTRAINT chk_price CHECK (price > 0),
    CONSTRAINT chk_stock CHECK (stock >= 0),

    -- ⑤ NOT NULL：尽量给所有列加，优化器可利用此信息
    -- ⑥ DEFAULT：与NOT NULL配合
    INDEX idx_status (status)
) ENGINE=InnoDB;
```

### 3.6 DDL 操作速查

```sql
-- 数据库
CREATE DATABASE shop CHARSET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER DATABASE shop CHARSET utf8mb4;
DROP DATABASE shop;  -- 危险！三思！

-- 表结构变更（InnoDB大部分ALTER会锁表/重建表，线上慎用）
ALTER TABLE users ADD COLUMN phone VARCHAR(20) AFTER name;
ALTER TABLE users MODIFY COLUMN phone VARCHAR(30) NOT NULL;
ALTER TABLE users CHANGE COLUMN phone mobile VARCHAR(20);  -- 改名+改类型
ALTER TABLE users DROP COLUMN phone;
ALTER TABLE users RENAME TO customers;
DROP TABLE temp_data;  -- 直接删除，不进回收站！

-- 索引
CREATE INDEX idx_city ON users(city);
CREATE INDEX idx_city_age ON users(city, age);      -- 复合索引
DROP INDEX idx_city ON users;
```

```sql
-- 视图：保存查询的虚拟表（安全隔离/简化复杂查询）
CREATE VIEW v_active_users AS 
SELECT id, name, city FROM users WHERE status = 1;
-- 查看视图定义
SHOW CREATE VIEW v_active_users;
-- 部分视图可更新（单表、无聚合、无DISTINCT）
UPDATE v_active_users SET city = '上海' WHERE id = 1;
DROP VIEW v_active_users;
```

### 3.7 表分区（大表优化利器）

```sql
-- 范围分区（最常用）：按年份分区
CREATE TABLE order_log (
    id BIGINT NOT NULL,
    order_date DATE NOT NULL,
    amount DECIMAL(10,2),
    PRIMARY KEY (id, order_date)   -- ⚠️ 分区键必须包含在所有主键/唯一键中！
) ENGINE=InnoDB
PARTITION BY RANGE (YEAR(order_date)) (
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- 管理分区
ALTER TABLE order_log ADD PARTITION (PARTITION p2025 VALUES LESS THAN (2026));
ALTER TABLE order_log DROP PARTITION p2022;         -- 瞬间清空2022数据
ALTER TABLE order_log TRUNCATE PARTITION p2023;     -- 瞬间清空2023数据

-- 分区失效场景：WHERE条件未使用分区键
SELECT * FROM order_log WHERE id = 100;              -- 扫描所有分区！
SELECT * FROM order_log WHERE order_date = '2024-06-01'; -- 只扫p2024 ✓
```

### 3.8 定时事件

```sql
-- 前提：开启事件调度器
SET GLOBAL event_scheduler = ON;
SHOW VARIABLES LIKE 'event_scheduler';

-- 创建定时任务：每天凌晨3点清理过期日志
CREATE EVENT evt_clean_old_logs
ON SCHEDULE EVERY 1 DAY STARTS '2026-06-10 03:00:00'
DO DELETE FROM access_log WHERE created_at < DATE_SUB(NOW(), INTERVAL 90 DAY);

-- 查看/关闭/启用
SHOW EVENTS;
ALTER EVENT evt_clean_old_logs DISABLE;
ALTER EVENT evt_clean_old_logs ENABLE;
DROP EVENT evt_clean_old_logs;
```

---

## 第四部分：编程与流程控制

### 4.1 变量体系

```sql
-- ① 用户变量（@var）：会话级，无需声明，弱类型
SET @total := 0;  -- := 用于SET外的赋值
SELECT @total := SUM(amount) FROM orders;  -- 结果赋给变量
SELECT @total;  -- 查看

-- ② 局部变量（DECLARE）：仅在存储过程/函数内
DELIMITER $$
CREATE PROCEDURE demo_var()
BEGIN
    DECLARE cnt INT DEFAULT 0;            -- 声明+默认值
    DECLARE done INT DEFAULT FALSE;
    SELECT COUNT(*) INTO cnt FROM users;   -- 将查询结果赋值
    SELECT cnt;
END$$
DELIMITER ;
```

### 4.2 流程控制

```sql
DELIMITER $$
CREATE PROCEDURE demo_flow(IN score INT)
BEGIN
    -- IF-ELSEIF-ELSE
    IF score >= 90 THEN
        SELECT '优秀';
    ELSEIF score >= 60 THEN
        SELECT '及格';
    ELSE
        SELECT '不及格';
    END IF;

    -- CASE（两种写法）
    CASE score                                 -- 简单CASE
        WHEN 100 THEN SELECT '满分';
        WHEN 0   THEN SELECT '零分';
        ELSE SELECT '其他';
    END CASE;

    CASE                                        -- 搜索CASE
        WHEN score >= 90 THEN SELECT 'A';
        WHEN score >= 80 THEN SELECT 'B';
        ELSE SELECT 'C';
    END CASE;
END$$
DELIMITER ;
```

```sql
DELIMITER $$
CREATE PROCEDURE demo_loop()
BEGIN
    DECLARE i INT DEFAULT 0;

    -- WHILE（先判断后执行，可能0次）
    WHILE i < 5 DO
        SET i = i + 1;
        SELECT i;
    END WHILE;

    -- REPEAT（先执行后判断，至少1次）
    SET i = 0;
    REPEAT
        SET i = i + 1;
        SELECT i;
    UNTIL i >= 5 END REPEAT;

    -- LOOP + LEAVE（无限循环+跳出）
    SET i = 0;
    my_loop: LOOP
        SET i = i + 1;
        IF i > 5 THEN LEAVE my_loop; END IF;
        SELECT i;
    END LOOP my_loop;
END$$
DELIMITER ;
```

### 4.3 游标（逐行处理）

```sql
DELIMITER $$
CREATE PROCEDURE demo_cursor()
BEGIN
    DECLARE v_name VARCHAR(50);
    DECLARE v_salary DECIMAL(10,2);
    DECLARE done INT DEFAULT FALSE;

    -- 声明游标
    DECLARE cur CURSOR FOR SELECT name, salary FROM employees;
    -- 异常处理器：游标取完时 NOT FOUND
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

    OPEN cur;
    read_loop: LOOP
        FETCH cur INTO v_name, v_salary;
        IF done THEN LEAVE read_loop; END IF;
        -- 逐行处理逻辑
        INSERT INTO salary_log (name, salary) VALUES (v_name, v_salary);
    END LOOP;
    CLOSE cur;
END$$
DELIMITER ;
```

### 3.4 触发器

```sql
-- 记录更新日志（审计）
DELIMITER $$
CREATE TRIGGER trg_users_update
    AFTER UPDATE ON users
    FOR EACH ROW
BEGIN
    -- NEW：更新后的行，OLD：更新前的行
    INSERT INTO users_audit (user_id, old_name, new_name, changed_at)
    VALUES (NEW.id, OLD.name, NEW.name, NOW());
END$$
DELIMITER ;

-- 注意：① 触发器不能返回结果集 ② 内部不能使用 COMMIT/ROLLBACK
--        ③ 慎用过多触发器，降低写入性能且增加调试难度
```

### 3.5 存储过程 vs 函数

```sql
-- 函数：必须有返回值，可在SQL中直接调用
DELIMITER $$
CREATE FUNCTION calc_tax(amount DECIMAL(10,2)) 
RETURNS DECIMAL(10,2)
DETERMINISTIC            -- 相同输入必相同输出（优化器可利用）
READS SQL DATA
BEGIN
    RETURN amount * 0.06;
END$$
DELIMITER ;

SELECT name, calc_tax(salary) AS tax FROM employees;  -- 直接在查询中用
```


| 对比      | 存储过程        | 函数          |
| --------- | --------------- | ------------- |
| 返回值    | 0~多个(OUT参数) | 必须有1个     |
| SQL中使用 | CALL proc()     | SELECT func() |
| 事务控制  | ✅ 可以         | ❌ 不可以     |
| 适用场景  | 复杂业务逻辑    | 简单计算/转换 |

---

## 第五部分：事务、MVCC 与锁机制

### 5.1 事务控制（ACID）

```sql
-- 显式事务
START TRANSACTION;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;  -- 或 ROLLBACK;

-- 保存点（部分回滚）
START TRANSACTION;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    SAVEPOINT sp1;
    UPDATE accounts SET balance = balance + 100 WHERE id = 999;  -- 可能失败
    -- ROLLBACK TO SAVEPOINT sp1;  -- 只回滚到保存点
    -- RELEASE SAVEPOINT sp1;      -- 释放保存点
COMMIT;
```

```sql
-- 四种隔离级别
SELECT @@transaction_isolation;                         -- 查看当前级别（默认REPEATABLE-READ）
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;  -- 设置当前会话级别

-- 脏读演示（READ UNCOMMITTED级别）
-- 事务A：START TX; UPDATE users SET age=99 WHERE id=1; -- 未提交
-- 事务B(READ UNCOMMITTED)：SELECT age FROM users WHERE id=1; -- 看到99（脏读！）

-- 不可重复读（READ COMMITTED级别）
-- 事务A：SELECT age FROM users WHERE id=1; → 25
-- 事务B：UPDATE users SET age=99 WHERE id=1; COMMIT;
-- 事务A：SELECT age FROM users WHERE id=1; → 99（不可重复读！）

-- 幻读（REPEATABLE READ级别也有，但InnoDB用间隙锁缓解）
-- 事务A：SELECT * FROM users WHERE age BETWEEN 20 AND 30; -- 10条
-- 事务B：INSERT INTO users (age) VALUES (25); COMMIT;
-- 事务A：SELECT * FROM users WHERE age BETWEEN 20 AND 30; -- 仍是10条（快照读）
-- 事务A：UPDATE users SET name='X' WHERE age BETWEEN 20 AND 30; -- 影响11行！（当前读出现幻读）
```

### 5.2 MVCC（多版本并发控制）

```
核心原理：每条行记录隐式包含3个字段
  DB_TRX_ID  ← 最近修改本行的事务ID（6字节）
  DB_ROLL_PTR ← 指向undo log的回滚指针（7字节）
  DB_ROW_ID  ← 隐式自增行ID（6字节，无主键时使用）

ReadView（快照读时的可见性判断）：
  - m_ids：生成ReadView时所有活跃事务ID集合
  - min_trx_id：m_ids最小值
  - max_trx_id：下一个尚未分配的事务ID
  - creator_trx_id：生成此ReadView的事务ID

可见性规则（对读到的每一行判断）：
  ① trx_id == creator_trx_id → 可见（自己改的）
  ② trx_id < min_trx_id       → 可见（修改已提交）
  ③ trx_id >= max_trx_id      → 不可见（未来事务）
  ④ trx_id ∈ m_ids            → 不可见（活跃事务未提交）
  否则                          → 可见
  若不可见 → 沿DB_ROLL_PTR找undo log链中符合条件的版本
```

```sql
-- 快照读 vs 当前读
-- 快照读：普通SELECT，基于MVCC读某个版本的数据（不加锁）
SELECT * FROM users WHERE id = 1;

-- 当前读：读取最新已提交版本并加锁
SELECT * FROM users WHERE id = 1 FOR UPDATE;       -- 排他锁
SELECT * FROM users WHERE id = 1 LOCK IN SHARE MODE; -- 共享锁（8.0+）
SELECT * FROM users WHERE id = 1 FOR SHARE;          -- 同上（语法糖）
```

### 5.3 Redo Log 与 Undo Log

```sql
-- Redo Log（物理日志）：保证持久性(D)，崩溃恢复用
--   - 循环写，ib_logfile0 + ib_logfile1
--   - WAL（Write-Ahead Log）：先写日志再写磁盘
--   - 事务提交时 fsync 刷盘（innodb_flush_log_at_trx_commit=1 最安全）

-- Undo Log（逻辑日志）：保证原子性(A) + 隔离性(I)，回滚+MVCC用
--   - INSERT对应的undo：删除该行
--   - UPDATE对应的undo：恢复旧值
--   - DELETE对应的undo：恢复删除行
--   - 位于共享表空间或独立undo表空间

-- 查看Redo Log大小
SHOW VARIABLES LIKE 'innodb_log_file_size';       -- 默认48MB（建议调大至1~2GB）
SHOW VARIABLES LIKE 'innodb_log_files_in_group';  -- 默认2个文件
```

### 5.4 锁机制（并发核心）

```sql
-- ====== 行级锁 ======
-- ① 共享锁（S锁）：读锁，允许其他事务读，不允许写
SELECT * FROM users WHERE id = 1 FOR SHARE;       -- 显式加S锁

-- ② 排他锁（X锁）：写锁，不允许其他事务读写
SELECT * FROM users WHERE id = 1 FOR UPDATE;       -- 显式加X锁
UPDATE users SET age=18 WHERE id=1;                -- 隐式加X锁

-- S锁与X锁兼容矩阵：S+S✅  S+X❌  X+X❌

-- ====== 意向锁（表级，InnoDB自动加）======
-- IS锁：事务想在某些行加S锁时，先在表上加IS锁
-- IX锁：事务想在某些行加X锁时，先在表上加IX锁
-- 作用：判断表级锁能否获取（如LOCK TABLES时快速判断）

-- ====== 间隙锁（Gap Lock）：防止幻读 ======
-- RR级别下，对索引间隙加锁，阻止INSERT
-- 例：表中有 id=5 和 id=10
SELECT * FROM users WHERE id BETWEEN 5 AND 10 FOR UPDATE;
-- 锁住：(5,10)间隙 + id=5 + id=10，其他事务无法在(5,10)间插入

-- ====== 死锁检测与排查 ======
SHOW ENGINE INNODB STATUS\G  -- 查看最近死锁信息（LATEST DETECTED DEADLOCK部分）

-- 避免死锁的最佳实践：
-- ① 所有事务按相同顺序访问表和行（如都先操作账户1再操作账户2）
-- ② 尽量缩短事务（不要在事务中做外部调用/用户交互）
-- ③ 为相关列建索引（减少锁范围）
```

### 5.5 乐观锁（应用层实现）

```sql
-- 通过版本号实现乐观锁
-- 建表时加上 version 字段
ALTER TABLE products ADD COLUMN version INT NOT NULL DEFAULT 0;

-- 更新时带版本号判断
UPDATE products SET stock = stock - 1, version = version + 1 
WHERE id = 1 AND version = @old_version;  -- @old_version 是读取时的版本号

-- 若 affected_rows = 0 → 版本已变，重试或提示用户
-- 适用：读多写少、冲突概率低的场景
```

### 5.6 表级锁

```sql
-- 显式表锁（MyISAM常用，InnoDB尽量不用）
LOCK TABLES users READ;      -- 读锁：其他连接可读，不可写
LOCK TABLES users WRITE;     -- 写锁：其他连接不可读不可写
UNLOCK TABLES;               -- 释放所有表锁

-- 元数据锁（MDL）：MySQL自动管理，DDL与DML互斥
-- 线上大表加字段会被正在执行的DML阻塞，解决方案：
-- ① 使用 pt-online-schema-change（无锁DDL工具）
-- ② 设置 lock_wait_timeout 避免无限等待
```

---

## 第六部分：索引原理与优化

### 6.1 索引类型总览


| 索引类型        | 存储结构 | 特点                  | 适用场景        |
| --------------- | -------- | --------------------- | --------------- |
| B+Tree          | B+树     | 有序、范围查询快      | 99%场景（默认） |
| Hash            | 哈希表   | =查询O(1)，不支持范围 | Memory引擎      |
| Full-Text       | 倒排索引 | 全文搜索              | 文章内容搜索    |
| Spatial(R-Tree) | R树      | 地理位置查询          | GIS地理数据     |

```sql
-- 索引分类（按逻辑用途）
-- ① 主键索引（聚簇索引）：叶子节点存整行数据
-- ② 二级索引（辅助索引）：叶子节点存主键值，回表查数据
-- ③ 唯一索引：UNIQUE约束自动创建
-- ④ 复合索引：多列组合
-- ⑤ 覆盖索引：查询列全在索引中，不需要回表（最理想）
-- ⑥ 前缀索引：只索引字段前N字符（省空间）
```

### 6.2 B+Tree 结构与原理

```
              [30 | 70]                    ← 非叶子节点（只存键值+指针）
             /    |    \
      [5|12]    [35|50]    [75|90]         ← 非叶子节点
      /  |  \    /  |  \    /  |  \
    [1] [8] [15] ...                       ← 叶子节点（存完整数据/主键值）
     ↑________双向链表_______↑              ← 支持范围查询
```

```sql
-- 页（Page）：InnoDB最小存储单元，默认16KB
-- 高度3的B+Tree可存约2千万行数据（假设每行1KB）
-- 这就是为什么 4层以上B+Tree要警惕的原因
```

### 6.3 复合索引与最左前缀原则

```sql
-- 创建复合索引 idx(a, b, c)，相当于创建了三个索引：
-- ① idx(a)         ← a列可用索引
-- ② idx(a, b)      ← a+b可用索引
-- ③ idx(a, b, c)   ← a+b+c可用索引

-- 不能利用索引的查询（跳过/断开了最左列）：
-- WHERE b = 1       ← 没有a，索引失效
-- WHERE a = 1 AND c = 2  ← 只有a能用，c不行（中间跳过了b）

CREATE INDEX idx_dept_salary ON employees(department, salary);

-- ✅ 能用索引
SELECT * FROM employees WHERE department = 'IT';           -- 用到department
SELECT * FROM employees WHERE department = 'IT' 
         ORDER BY salary;                                  -- 用到department + salary排序
SELECT * FROM employees WHERE department = 'IT' 
         AND salary > 10000;                               -- 两列都用上

-- ❌ 索引失效
SELECT * FROM employees WHERE salary > 10000;              -- 缺department
SELECT * FROM employees WHERE department LIKE '%IT%';      -- 左模糊，索引失效
```

### 6.4 索引失效场景（避坑指南）

```sql
-- ① 在索引列上使用函数
-- ❌ SELECT * FROM orders WHERE DATE(created_at) = '2026-01-01';
-- ✅ SELECT * FROM orders WHERE created_at >= '2026-01-01' AND created_at < '2026-01-02';

-- ② 隐式类型转换（字段是VARCHAR，传INT会触发转换）
-- ❌ SELECT * FROM users WHERE phone = 13800138000;   -- phone是VARCHAR
-- ✅ SELECT * FROM users WHERE phone = '13800138000';

-- ③ LIKE 左模糊
-- ❌ SELECT * FROM products WHERE name LIKE '%手机%';
-- ✅ SELECT * FROM products WHERE name LIKE '手机%';  -- 右模糊可用索引

-- ④ OR 连接不同列
-- ❌ SELECT * FROM users WHERE name = '张三' OR age = 25;  -- 可能全表扫描
-- ✅ SELECT * FROM users WHERE name = '张三' 
--    UNION SELECT * FROM users WHERE age = 25 AND name <> '张三';

-- ⑤ != / <> / NOT IN 大概率不走索引（优化器判断回表成本后可能放弃）
-- ⑥ IS NOT NULL 不一定走索引（取决于NULL值分布）

-- 验证：EXPLAIN SELECT * FROM users WHERE phone = '13800138000';
```

### 6.5 覆盖索引（无需回表）

```sql
-- 概念：查询所需列全部在索引中，不需要回表查聚簇索引
-- 这是最理想的查询场景

CREATE INDEX idx_dept_salary_name ON employees(department, salary, name);

-- 覆盖索引：所有列都在 idx_dept_salary_name 中
EXPLAIN SELECT department, salary, name 
FROM employees WHERE department = 'IT';
-- Extra: Using index  ← 表示覆盖索引，不需要回表

-- 非覆盖索引：SELECT * 需要回表
EXPLAIN SELECT * FROM employees WHERE department = 'IT';
-- Extra: Using index condition  ← 需要回表（通过主键查完整行）
```

### 6.6 前缀索引

```sql
-- 对长VARCHAR/TEXT字段索引前N字符，节省空间
ALTER TABLE articles ADD INDEX idx_title_prefix (title(20));

-- 选择合适的前缀长度：让区分度接近完整列
-- 查看完整列区分度
SELECT COUNT(DISTINCT title) / COUNT(*) FROM articles;      -- 假设0.95
-- 查看前缀20的区分度
SELECT COUNT(DISTINCT LEFT(title, 20)) / COUNT(*) FROM articles; -- 假设0.92
-- 在空间和区分度间平衡，一般前缀长度在10~30之间
```

### 6.7 索引下推（ICP，MySQL 5.6+）

```sql
-- 概念：把WHERE中部分过滤下推到引擎层，减少回表次数
-- 无ICP：引擎逐条读二级索引→回表取完整行→Server层过滤
-- 有ICP：引擎层直接对索引中的列做过滤→减少回表

CREATE INDEX idx_name_age ON users(name, age);
-- 查询中 age 在索引里，可下推到引擎过滤
EXPLAIN SELECT * FROM users WHERE name LIKE '张%' AND age = 25;
-- Extra: Using index condition  ← 表示使用了ICP
```

---

## 第七部分：性能分析与调优

### 7.1 EXPLAIN 详解

```sql
EXPLAIN SELECT * FROM orders WHERE user_id = 100;
-- 或使用更详细的格式
EXPLAIN FORMAT=JSON SELECT * FROM orders WHERE user_id = 100;
```


| 列名              | 含义                                        | 重点关注               |
| ----------------- | ------------------------------------------- | ---------------------- |
| **id**            | 执行顺序（同id从上到下，大id优先）          | 复杂嵌套查询           |
| **select_type**   | 查询类型（SIMPLE/PRIMARY/SUBQUERY/DERIVED） | DERIVED=派生表(临时表) |
| **type**          | 访问类型（见下表）                          | 目标：range及以上      |
| **possible_keys** | 可能用到的索引                              | 为NULL→没索引可用     |
| **key**           | 实际用的索引                                | 为NULL→走了全表扫描   |
| **rows**          | 预估扫描行数                                | 越小越好               |
| **filtered**      | 按条件过滤后剩余行百分比                    | rows×filtered=实际    |
| **Extra**         | 额外信息                                    | 见下表                 |

**type 性能排序（从好到差）：**

```
system > const > eq_ref > ref > range > index > ALL
         主键=   唯一键=  非唯一  范围   全索引   全表
```

**Extra 关键信息：**


| Extra值               | 含义               | 优化建议                 |
| --------------------- | ------------------ | ------------------------ |
| Using index           | 覆盖索引，无需回表 | ✅ 最佳状态              |
| Using index condition | ICP优化            | ✅ 较好                  |
| Using where           | Server层过滤       | 可接受                   |
| Using filesort        | 文件排序           | ❌ 优化ORDER BY列加索引  |
| Using temporary       | 使用临时表         | ❌ 优化GROUP BY/DISTINCT |
| Using join buffer     | JOIN使用了buffer   | ❌ 优化JOIN条件加索引    |

```sql
-- 快速查看表状态
SHOW TABLE STATUS LIKE 'orders'\G
-- 查看索引使用情况
SHOW INDEX FROM orders;
-- 查看索引区分度（Cardinality越接近行数越好）
```

### 7.2 慢查询定位与优化

```sql
-- ① 开启慢查询日志
SET GLOBAL slow_query_log = ON;
SET GLOBAL long_query_time = 1;          -- 超过1秒记录
SET GLOBAL log_queries_not_using_indexes = ON;  -- 记录未用索引的查询
SHOW VARIABLES LIKE 'slow_query%';

-- ② 查看慢查询（或用 pt-query-digest 分析慢日志文件）
-- pt-query-digest /var/log/mysql/slow.log  ← 分析工具
```

```sql
-- ③ 经典 ORDER BY + LIMIT 翻页优化
-- ❌ 深分页（OFFSET很大时须先跳过大量行）
SELECT * FROM users ORDER BY id LIMIT 1000000, 10;  -- 需扫描1000010行

-- ✅ 方案1：延迟关联（只取id再回表）
SELECT * FROM users u
INNER JOIN (SELECT id FROM users ORDER BY id LIMIT 1000000, 10) t
ON u.id = t.id;

-- ✅ 方案2：游标分页（记住上次的id）
SELECT * FROM users WHERE id > 1000000 ORDER BY id LIMIT 10;
-- 前提：id连续递增且无删除
```

```sql
-- ④ COUNT(*) 优化
-- MyISAM：COUNT(*) O(1)（有行计数器）
-- InnoDB：COUNT(*) 需扫描（MVCC导致）
-- 近似计数方案：
SELECT TABLE_ROWS FROM information_schema.TABLES 
WHERE TABLE_NAME = 'users';                    -- 估算值，不精确

-- 大表精确计数：维护计数表
CREATE TABLE counters (table_name VARCHAR(50), cnt BIGINT);
-- 用触发器在INSERT/DELETE时 ±1 更新计数表
```

### 7.3 查询优化器提示（Hint）

```sql
-- 强制使用/忽略索引
SELECT * FROM users FORCE INDEX(idx_name) WHERE name = '张三';
SELECT * FROM users USE INDEX(idx_name) WHERE name = '张三';   -- 建议
SELECT * FROM users IGNORE INDEX(idx_name) WHERE name = '张三'; -- 忽略

-- 调整JOIN驱动表顺序
SELECT /*+ JOIN_ORDER(u, o) */ u.name, o.amount 
FROM orders o JOIN users u ON o.uid = u.id;  -- 指定users为驱动表

-- 一般让优化器自己选，只在确认优化器选错时使用Hint
```

### 7.4 参数调优速查

```sql
-- 查看关键配置
SHOW VARIABLES LIKE '%innodb_buffer_pool_size%';  -- 建议设为物理内存的50%~80%
SHOW VARIABLES LIKE '%innodb_log_file_size%';     -- 建议1~2GB
SHOW VARIABLES LIKE '%innodb_flush_log_at_trx_commit%';  -- 1=最安全,2=高性能
SHOW VARIABLES LIKE '%sync_binlog%';               -- 1=最安全，0=高性能
SHOW VARIABLES LIKE '%max_connections%';           -- 根据业务调整
SHOW VARIABLES LIKE '%innodb_io_capacity%';       -- SSD设为2000~5000
```

---

## 第八部分：权限与安全管理

### 8.1 用户与权限管理

```sql
-- 创建用户
CREATE USER 'app'@'%' IDENTIFIED BY 'StrongPass123!';            -- 任意主机
CREATE USER 'app'@'192.168.1.%' IDENTIFIED BY 'StrongPass123!';  -- 限定C段
CREATE USER 'readonly'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'xxx';

-- 授权
GRANT SELECT, INSERT, UPDATE ON shop.* TO 'app'@'%';             -- 数据库级
GRANT SELECT ON shop.products TO 'readonly'@'localhost';          -- 表级
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost';               -- 全局（DBA）
GRANT EXECUTE ON PROCEDURE shop.calc_tax TO 'app'@'%';           -- 存储过程
FLUSH PRIVILEGES;                                                  -- 刷新权限

-- 查看权限
SHOW GRANTS FOR 'app'@'%';
SELECT * FROM mysql.user WHERE User='app'\G

-- 回收权限
REVOKE INSERT ON shop.* FROM 'app'@'%';
-- 删除用户
DROP USER 'app'@'%';
```

### 8.2 权限层级

```sql
-- 全局级    GRANT ... ON *.*           → mysql.user表
-- 数据库级  GRANT ... ON db.*         → mysql.db表
-- 表级      GRANT ... ON db.table     → mysql.tables_priv
-- 列级      GRANT SELECT(col) ON ...  → 不常用
-- 存储过程  GRANT EXECUTE ON PROCEDURE db.proc

-- 权限叠加：所有层级的权限取并集（只增不减）
-- 查看完整权限树
SELECT * FROM mysql.user WHERE User='app'\G
SELECT * FROM mysql.db WHERE User='app'\G
```

### 8.3 安全最佳实践

```sql
-- ① 开启安全更新模式（防止无WHERE的UPDATE/DELETE）
SET sql_safe_updates = 1;
-- 此时不带 WHERE/LIMIT 的 UPDATE/DELETE 会报错
-- 在[mysqld]配置中加 sql_safe_updates=1 持久化

-- ② 删除前先用SELECT验证
SELECT * FROM orders WHERE status = 'cancelled';  -- 先确认
-- 确认无误后
DELETE FROM orders WHERE status = 'cancelled';

-- ③ 生产环境最小权限原则
--    应用账号：只给SELECT/INSERT/UPDATE/DELETE，不给DDL/DROP
--    只读账号：只给SELECT
--    DBA账号：限制来源IP，使用强密码

-- ④ 密码策略
SHOW VARIABLES LIKE 'validate_password%';  -- 密码强度插件
-- validate_password.length=12, validate_password.mixed_case_count=1 等
```

### 8.4 SQL 注入防护

```sql
-- ❌ 危险：拼接SQL
-- $sql = "SELECT * FROM users WHERE name = '$input'";
-- 输入 ' OR '1'='1  →  SELECT * FROM users WHERE name = '' OR '1'='1'

-- ✅ 使用参数化查询（Prepared Statement）
PREPARE stmt FROM 'SELECT * FROM users WHERE name = ?';
SET @name = '张三';
EXECUTE stmt USING @name;
DEALLOCATE PREPARE stmt;

-- 应用层必须使用参数化查询！不要相信任何用户输入！
```

---

## 第九部分：备份与恢复

### 9.1 逻辑备份：mysqldump

```sql
-- 备份单表
mysqldump -u root -p shop users > users_bak.sql

-- 备份单库（含结构和数据）
mysqldump -u root -p --databases shop > shop_bak.sql

-- 备份全部数据库
mysqldump -u root -p --all-databases --routines --triggers --events > all_bak.sql

-- 常用选项
-- --single-transaction  InnoDB一致性备份（不锁表）
-- --quick               逐行导出（大表不会撑爆内存）
-- --no-data             只导出结构
-- --where="id > 1000"   按条件导出
-- --routines            含存储过程
-- --triggers            含触发器
```

```sql
-- 恢复
mysql -u root -p shop < shop_bak.sql
-- 或进入mysql后
source /path/to/backup.sql;
```

### 9.2 物理备份：Xtrabackup

```sql
-- 全量备份（不锁表，在线热备）
xtrabackup --backup --target-dir=/backup/full --user=root --password=xxx

-- 增量备份（基于上次全量/增量的LSN）
xtrabackup --backup --target-dir=/backup/inc1 \
    --incremental-basedir=/backup/full --user=root --password=xxx

-- 准备（恢复前必须apply log，使数据一致）
xtrabackup --prepare --target-dir=/backup/full
xtrabackup --prepare --target-dir=/backup/full \
    --incremental-dir=/backup/inc1   -- 应用增量到全量

-- 恢复（关闭mysqld后）
xtrabackup --copy-back --target-dir=/backup/full
chown -R mysql:mysql /var/lib/mysql
```

### 9.3 Binlog 恢复（时间点恢复 PITR）

```sql
-- 查看当前binlog状态
SHOW MASTER STATUS;  -- 记录当前文件和位置
SHOW BINARY LOGS;    -- 所有binlog文件列表

-- 解析binlog
mysqlbinlog --start-datetime="2026-06-09 10:00:00" \
            --stop-datetime="2026-06-09 10:05:00" \
            binlog.000001 > recovery.sql

-- 按位置恢复
mysqlbinlog --start-position=1000 --stop-position=2000 binlog.000001 | mysql -u root -p
```

### 9.4 备份策略建议

```
全量备份：每周日凌晨2点（Xtrabackup物理备份）
增量备份：周一到周六凌晨2点（Xtrabackup增量）
Binlog备份：实时（5分钟一次同步到远程）

恢复流程：
  ① 恢复最近全量备份
  ② 依次恢复增量备份
  ③ 恢复全量/增量之后的所有binlog到指定时间点

⚠️ 备份不验证 = 没有备份！定期做恢复演练！
```

---

## 第十部分：主从复制与高可用

### 10.1 主从复制原理

```
三步走：
  ① Master binlog dump线程：把binlog日志发给Slave
  ② Slave IO线程：接收binlog并写入relay log（中继日志）
  ③ Slave SQL线程：从relay log读取并执行，完成同步

延迟原因：
  - Slave单线程执行（5.6及以前）→ 升级5.7+并行复制
  - 大事务（bulk insert/delete）→ 拆小
  - Slave机器性能差
  - 网络延迟
```

### 10.2 搭建主从（核心步骤）

```sql
-- Master配置（my.cnf）
-- [mysqld]
-- server-id = 1
-- log-bin = mysql-bin
-- binlog_format = ROW           ← 推荐ROW格式
-- binlog_row_image = FULL       ← FULL最安全，MINIMAL省空间

-- Master创建复制账号
CREATE USER 'repl'@'%' IDENTIFIED BY 'ReplPass123!';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';

-- 获取Master起始位置（锁定后导出，或用Xtrabackup）
FLUSH TABLES WITH READ LOCK;
SHOW MASTER STATUS;  -- 记录 File 和 Position
UNLOCK TABLES;
```

```sql
-- Slave配置
-- [mysqld]
-- server-id = 2                      ← 必须不同于Master
-- relay-log = relay-bin
-- read_only = ON                     ← 从库只读
-- log_slave_updates = ON            ← 级联复制时需要

-- Slave启动复制
CHANGE MASTER TO
    MASTER_HOST = '192.168.1.100',
    MASTER_PORT = 3306,
    MASTER_USER = 'repl',
    MASTER_PASSWORD = 'ReplPass123!',
    MASTER_LOG_FILE = 'mysql-bin.000001',  -- 对应SHOW MASTER STATUS的File
    MASTER_LOG_POS = 1234;                  -- 对应Position

START SLAVE;
SHOW SLAVE STATUS\G  -- 重点看 Slave_IO_Running 和 Slave_SQL_Running
```

### 10.3 GTID 复制（MySQL 5.6+）

```sql
-- 传统复制依赖（File, Position），主从切换需重新指向位置
-- GTID：全局事务ID，格式 server_uuid:transaction_id
-- 优点：主从切换自动找位置，无需手动指定

-- Master配置
-- gtid_mode = ON
-- enforce_gtid_consistency = ON

-- Slave用GTID方式启动
CHANGE MASTER TO MASTER_AUTO_POSITION = 1;

-- 查看GTID执行情况
SELECT @@server_uuid;
SHOW VARIABLES LIKE 'gtid_executed';
```

### 10.4 常见复制架构

```
① 一主一从（最简单）
   [Master] ────> [Slave]
   写           读/备份

② 一主多从（读写分离）
           ┌──> [Slave1] 读
   [Master]┼──> [Slave2] 读
           └──> [Slave3] 备份
   中间件：Atlas / Proxysql / ShardingSphere-JDBC

③ 双主（互为主从，慎用！）
   [Master1] <──> [Master2]
   问题：自增ID冲突、数据不一致、脑裂
   解决：auto_increment_increment=2, auto_increment_offset分离

④ MGR（MySQL Group Replication，8.0官方高可用）
   多主/单主模式，基于Paxos协议自动选主
```

### 10.5 主从延迟处理

```sql
-- 查看延迟秒数
SHOW SLAVE STATUS\G  -- Seconds_Behind_Master

-- 延迟原因及解决方案
-- ① 并行复制（5.7+）
-- slave_parallel_type = LOGICAL_CLOCK
-- slave_parallel_workers = 4          ← 设为核心数1~2倍

-- ② 临时跳过错误（谨慎！）
-- STOP SLAVE; SET GLOBAL sql_slave_skip_counter = 1; START SLAVE;

-- ③ 从库加索引加速查询不一致场景（从库可以有不同索引）
```

---

## 第十一部分：MySQL 8.0+ 高级特性

### 11.1 窗口函数（Window Functions）

```sql
-- 窗口函数 = 聚合函数 + OVER()子句，不改变行数
-- 语法：函数名([参数]) OVER ([PARTITION BY 列] [ORDER BY 列] [窗口子句])

-- ① ROW_NUMBER / RANK / DENSE_RANK
SELECT name, department, salary,
    ROW_NUMBER()  OVER (PARTITION BY department ORDER BY salary DESC) AS rn,
    RANK()        OVER (PARTITION BY department ORDER BY salary DESC) AS rk,
    DENSE_RANK()  OVER (PARTITION BY department ORDER BY salary DESC) AS drk
FROM employees;
-- rn: 1,2,3,4...（不重复）
-- rk: 1,1,3,4...（并列跳号）
-- drk:1,1,2,3...（并列不跳号）

-- ② 累计/移动平均
SELECT date, amount,
    SUM(amount)  OVER (ORDER BY date) AS cumulative_total,
    AVG(amount)  OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS ma_7days
FROM daily_sales;

-- ③ LAG/LEAD（上下行对比）
SELECT date, amount,
    LAG(amount, 1)  OVER (ORDER BY date) AS prev_day,
    LEAD(amount, 1) OVER (ORDER BY date) AS next_day,
    amount - LAG(amount, 1) OVER (ORDER BY date) AS diff
FROM daily_sales;
```

### 11.2 CTE（公共表表达式）

```sql
-- 普通CTE：比子查询可读性更好
WITH dept_avg AS (
    SELECT department, AVG(salary) AS avg_sal
    FROM employees GROUP BY department
)
SELECT e.name, e.salary, d.avg_sal
FROM employees e
JOIN dept_avg d ON e.department = d.department
WHERE e.salary > d.avg_sal;  -- 找出薪资高于部门平均的员工

-- 递归CTE：处理树形结构（如组织架构、分类树）
WITH RECURSIVE org_tree AS (
    -- 锚点成员（根节点）
    SELECT id, name, parent_id, 1 AS level FROM dept WHERE parent_id IS NULL
    UNION ALL
    -- 递归成员
    SELECT d.id, d.name, d.parent_id, t.level + 1
    FROM dept d
    INNER JOIN org_tree t ON d.parent_id = t.id
)
SELECT * FROM org_tree ORDER BY level, id;
-- MySQL 默认递归深度限制为1000层，可调：SET SESSION cte_max_recursion_depth = 2000;
```

### 11.3 JSON 数据类型

```sql
-- 创建JSON列
CREATE TABLE config (
    id INT PRIMARY KEY,
    settings JSON
);

-- 插入
INSERT INTO config VALUES (1, '{"theme":"dark","lang":"zh","notify":{"email":true,"sms":false}}');

-- 查询
SELECT settings->'$.theme' AS theme FROM config;       -- 返回 "dark"（带引号的JSON）
SELECT settings->>'$.theme' AS theme FROM config;      -- 返回 dark（纯文本）
SELECT settings->>'$.notify.email' AS email FROM config;

-- 更新
UPDATE config SET settings = JSON_SET(settings, '$.theme', 'light') WHERE id = 1;

-- JSON 数组
SELECT JSON_ARRAY('red', 'blue', 'green');      -- 创建数组
SELECT JSON_CONTAINS('["a","b","c"]', '"b"');   -- 是否包含 → 1
SELECT JSON_LENGTH('["a","b","c"]');             -- 长度 → 3

-- 在JSON列上建索引（虚拟列+索引）
ALTER TABLE config ADD COLUMN theme VARCHAR(20) 
    GENERATED ALWAYS AS (settings->>'$.theme') VIRTUAL;
CREATE INDEX idx_theme ON config(theme);
```

### 11.4 其他 8.0 新特性

```sql
-- ① 降序索引（InnoDB真正支持）
CREATE INDEX idx_id_desc ON users(id DESC);

-- ② 不可见索引（测试删除索引的效果，不真正删除）
ALTER TABLE users ALTER INDEX idx_name INVISIBLE;
-- 查看优化器是否使用（用于评估索引是否真的需要）
-- VISIBLE 恢复

-- ③ 原子 DDL（CREATE/ALTER/DROP TABLE要么全成功要么全回滚）
-- 不再出现"删了一半"的尴尬局面

-- ④ 克隆插件（物理克隆整个实例，比mysqldump快得多）
-- INSTALL PLUGIN clone SONAME 'mysql_clone.so';
-- CLONE LOCAL DATA DIRECTORY = '/data/clone';

-- ⑤ 默认字符集改为 utf8mb4（终于！告别latin1坑）
SHOW VARIABLES LIKE 'character_set_server';
```

---

## 第十二部分：数据导入导出

### 12.1 SELECT INTO OUTFILE（导出）

```sql
-- 导出CSV（需 secure_file_priv 不为NULL）
SELECT * FROM orders 
INTO OUTFILE '/var/lib/mysql-files/orders.csv'
FIELDS TERMINATED BY ','         -- 列分隔符
OPTIONALLY ENCLOSED BY '"'       -- 字符串用引号包裹
LINES TERMINATED BY '\n';        -- 行分隔符

-- 查看允许的导出目录
SHOW VARIABLES LIKE 'secure_file_priv';
```

### 12.2 LOAD DATA INFILE（导入，比INSERT快10~100倍）

```sql
-- 导入CSV
LOAD DATA INFILE '/var/lib/mysql-files/orders.csv'
INTO TABLE orders
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 LINES                    -- 跳过标题行
(id, name, @amount, @created_at)  -- 列映射，@变量暂存
SET amount = @amount,
    created_at = STR_TO_DATE(@created_at, '%Y-%m-%d %H:%i:%s');  -- 格式转换

-- 速度对比：INSERT逐行 1000行/秒 vs LOAD DATA 100000行/秒
```

### 12.3 mysqlimport 命令行工具

```bash
# mysqlimport 是 LOAD DATA INFILE 的命令行版本
# 文件名 = 表名！
mysqlimport -u root -p --local \
    --fields-terminated-by=',' \
    --lines-terminated-by='\n' \
    shop /path/to/orders.csv
```

---

## 第十三部分：监控与故障排查

### 13.1 连接与会话监控

```sql
-- 当前所有连接
SHOW PROCESSLIST;                         -- 简单版
SELECT * FROM information_schema.PROCESSLIST;  -- 完整版

-- 查看连接数
SHOW VARIABLES LIKE 'max_connections';
SHOW STATUS LIKE 'Threads_connected';      -- 当前连接数
SHOW STATUS LIKE 'Threads_running';        -- 活跃（非Sleep）连接数
-- 连接数使用率 > 80% 需要关注！

-- 杀死连接
KILL 123;     -- 温和（等待事务结束）
KILL QUERY 123;  -- 只杀当前查询，保留连接

-- 查看长时间未提交的事务（连接池常见问题）
SELECT * FROM information_schema.INNODB_TRX 
WHERE TIMESTAMPDIFF(SECOND, trx_started, NOW()) > 30;
```

### 13.2 锁监控

```sql
-- 查看当前锁等待
SELECT * FROM performance_schema.data_lock_waits\G  -- 8.0
-- 或 MySQL 5.7
SELECT * FROM information_schema.INNODB_LOCK_WAITS\G

-- 查看所有锁信息
SELECT * FROM performance_schema.data_locks;        -- 8.0

-- 谁锁了谁？
SELECT 
    r.trx_id AS waiting_trx,
    r.trx_mysql_thread_id AS waiting_thread,
    b.trx_id AS blocking_trx,
    b.trx_mysql_thread_id AS blocking_thread
FROM information_schema.INNODB_LOCK_WAITS w
JOIN information_schema.INNODB_TRX r ON w.requesting_trx_id = r.trx_id
JOIN information_schema.INNODB_TRX b ON w.blocking_trx_id = b.trx_id;
```

### 13.3 性能监控关键指标

```sql
-- InnoDB Buffer Pool 命中率（目标 > 99%）
SHOW STATUS LIKE 'Innodb_buffer_pool_read%';
-- 1 - (reads / requests) = 命中率

-- 表锁等待
SHOW STATUS LIKE 'Table_locks_waited';

-- 临时表（磁盘）
SHOW STATUS LIKE 'Created_tmp_disk_tables';
SHOW STATUS LIKE 'Created_tmp_tables';
-- Created_tmp_disk_tables / Created_tmp_tables > 25% 需要优化

-- 排序（磁盘）
SHOW STATUS LIKE 'Sort_merge_passes';  -- 值越大排序越慢

-- 连接错误
SHOW STATUS LIKE 'Aborted_connects';

-- QPS / TPS（需定时采集计算）
SHOW STATUS LIKE 'Questions';    -- 总查询数
SHOW STATUS LIKE 'Com_commit';   -- 总提交数
SHOW STATUS LIKE 'Com_rollback'; -- 总回滚数
```

### 13.4 常见故障排查速查

```sql
-- ① CPU 高
--    看 processlist 中 Time 大的查询 → EXPLAIN 优化
--    检查是否有死循环的存储过程

-- ② IO 高
--    检查 Buffer Pool 是否太小（SHOW STATUS LIKE 'Innodb_buffer_pool_reads'）
--    查看是否有大事务/大表的 DDL

-- ③ 连接数满
--    检查 max_connections，检查是否有连接泄露
--    查看 Sleep 连接，设置 wait_timeout / interactive_timeout 回收

-- ④ 主从延迟大
--    检查大事务（SHOW BINLOG EVENTS）
--    开启并行复制（slave_parallel_workers）

-- ⑤ 死锁频繁
--    SHOW ENGINE INNODB STATUS\G  查看最近死锁
--    缩短事务、统一加锁顺序

-- ⑥ Can't connect to MySQL server
--    ping 检查网络，telnet 3306 检查端口
--    检查 bind-address / skip-networking 配置
--    检查防火墙和 max_connections
```

### 13.5 常用运维命令速查

```sql
-- 查看版本
SELECT VERSION();

-- 查看MySQL数据目录
SELECT @@datadir;

-- 查看错误日志位置
SHOW VARIABLES LIKE 'log_error';

-- 表空间大小
SELECT table_name, 
       ROUND(data_length / 1024 / 1024, 2) AS data_mb,
       ROUND(index_length / 1024 / 1024, 2) AS index_mb
FROM information_schema.TABLES 
WHERE table_schema = 'shop'
ORDER BY (data_length + index_length) DESC;

-- 碎片整理（暂停写入期间操作）
OPTIMIZE TABLE orders;  -- 等价 ALTER TABLE ... ENGINE=InnoDB

-- 分析表（更新索引统计信息）
ANALYZE TABLE users;

-- 检查表
CHECK TABLE users;

-- 修复表（MyISAM用，InnoDB会自动崩溃恢复）
REPAIR TABLE users;
```

---

## 附录：设计规范与最佳实践

### A.1 建表规范

```sql
-- 推荐建表模板
CREATE TABLE tb_order (
    id              BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键',
    order_no        VARCHAR(32) NOT NULL COMMENT '订单号',
    user_id         BIGINT UNSIGNED NOT NULL COMMENT '用户ID',
    amount          DECIMAL(18,2) NOT NULL DEFAULT 0.00 COMMENT '金额',
    status          TINYINT NOT NULL DEFAULT 0 COMMENT '状态:0-待支付,1-已支付,2-已取消',
    created_at      DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at      DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (id),
    UNIQUE KEY uk_order_no (order_no),
    KEY idx_user_status (user_id, status),
    KEY idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='订单表';
```

### A.2 反模式（千万别这么做）


| 反模式                 | 为什么不行                 | 正确做法              |
| ---------------------- | -------------------------- | --------------------- |
| 用FLOAT/DOUBLE存钱     | 精度丢失                   | DECIMAL               |
| SELECT *               | 回表+无法覆盖索引+额外带宽 | 只选需要的列          |
| VARCHAR(65535)         | 行溢出，性能极差           | TEXT或拆表            |
| 在WHERE中使用函数      | 索引失效                   | 改写条件              |
| 无WHERE的UPDATE/DELETE | 全表事故                   | sql_safe_updates=1    |
| 用ENUM                 | 修改成本高                 | TINYINT+代码映射      |
| 存储图片/大文件        | 备份巨大，查询慢           | 存URL，文件放对象存储 |
| 大量重复索引           | 浪费空间+降低写入性能      | 定期清理冗余索引      |
| 不在测试环境验证就上线 | 执行计划可能不同           | 测试环境预演          |

### A.3 范式与反范式

```
三范式（3NF）：
  1NF：列不可再分（原子性）
  2NF：非主键列完全依赖主键（消除部分依赖）
  3NF：非主键列只依赖主键（消除传递依赖）

反范式化：故意冗余数据，减少JOIN，提高查询性能
  例：orders表冗余存 user_name, user_phone（虽违反3NF，但避免JOIN users）

原则：先满足3NF设计，再根据性能需求反范式化
```

---
