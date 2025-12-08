

## 蓝山工作室运维组第七节课——MySQL

### 什么是数据库

> 我们已经学习了Linux、python基础、计算机网络、docker、docker swarm等等，学习了这些我们基本可以去github上面找一个项目部署实战了，但还差一个环节，就是数据库。每一个产品服务背后都离不开数据库，所以知道如何操作管理数据库，对运维来说是至关重要的。

数据库可以理解为一个**专门存放、管理数据的电子系统**。它类似一个功能强大的“信息仓库”，能把大量数据（例如用户信息、商品、订单、成绩等）**结构化（以一种特定的格式）地保存起来**，并且支持**快速查询、更新和删除**。相比手写记录或文件存储，数据库更安全、更高效，能保证多人同时操作时数据不冲突、不丢失。网站、APP、公司系统等几乎所有软件都依赖数据库来运作。简单来说，数据库就是现代软件世界中**用来存数据并让你随时能查到的核心工具**。

如果把 MySQL 水平定义成 1~10 级，下面是我对各种级别水平的理解。

| 级别 | 描述                                                    |
| ---- | ------------------------------------------------------- |
| 0    | 完全不了解                                              |
| 1    | 基本安装、配置、使用                                    |
| 2    | 熟悉常用配置项，能根据实际情况修改                      |
| 3    | 熟练定位 db 性能瓶颈，并能作出对应改进                  |
| 4    | 精通 MySQL 运行机制和体系架构，能从源码解析定位问题根源 |
| 5    | 精通业务程序、MySQL 源码，高效解决各种 db 相关问题      |
| 6~10 | 境界太高，理解不了                                      |

[数据库其实就那么回事](https://www.bilibili.com/video/BV1Hh411C7Up/?spm_id_from=333.337.search-card.all.click&vd_source=a57b854dceb9e43274d666a1811d0403)(后面是广告不用看，看前面科普就好)

[什么是MySQL](https://www.bilibili.com/video/BV1p5qhYsE4f/?spm_id_from=333.337.search-card.all.click&vd_source=a57b854dceb9e43274d666a1811d0403)

[一小时MySQL教程](https://www.bilibili.com/video/BV1AX4y147tA?spm_id_from=333.788.videopod.sections&vd_source=a57b854dceb9e43274d666a1811d0403&p=7)

[黑马程序员 MySQL数据库入门到精通，从mysql安装到mysql高级、mysql优化全囊括_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Kr4y1i7ru/?vd_source=e81cb97597d16e12fed88e4e65b8eab5)

### 数据库的分类

> 数据库的分类方式有很多很多，这里简单分为关系型数据库和非关系型数据库两种，有兴趣的同学可以自行了解其他分类

##### 关系型数据库

1. 用**表格（行列）**存数据，结构固定

2. 用 **SQL** 查询，能做复杂操作

3. 支持 **ACID 事务**，数据一致性强

4. 适合订单、用户、财务等 **复杂业务场景**  

   ***如MySQL，PostgreSQL***

##### 非关系型数据库

1. 数据结构**灵活**（键值、文档、列族等）

2. 查询简单，没有统一语言

3. 性能高、扩展容易，多为**最终一致性**

4. 常用于缓存、日志、海量数据等 **高并发场景**

   ***如Redis，MongoDB***

### MySQL数据库

#### 部署

可以直接部署在本机上或者使用Docker部署。容器部署数据库已是大势所趋，更加推荐使用Docker。

**直接安装**：[MySQL官网](https://dev.mysql.com/downloads/mysql/)

[MySQL安装、配置与卸载教程（Windows版）](https://juejin.cn/post/7337186171210170383)

**使用Docker安装**：

如何安装Docker，看这里：

```bash
docker pull mysql:8.0
docker run --name mysql \
  -e MYSQL_ROOT_PASSWORD=123456 \   # 设置root密码
  -p 3306:3306 \                    # 端口映射（主机3306 → 容器3306，方便本地连接）
  -v mysql-data:/var/lib/mysql \    # 数据持久化（避免容器删除后数据丢失）
  -d mysql:8.0 
```

进入容器

```bash
docker exec -it mysql /bin/bash
```

**登陆**

```bash
mysql -u root -p
```

然后输入密码即可



#### 介绍

MySQL是一种**关系型数据库**，关系型数据库的数据都是以**数据表**的形式进行存储的

> 看起来和excel差不多

```sql
+--------------+-----------------+------+-----+---------+----------------+
| Field        | Type            | Null | Key | Default | Extra          |
+--------------+-----------------+------+-----+---------+----------------+
| id           | bigint unsigned | NO   | PRI | NULL    | auto_increment |
| created_at   | datetime(3)     | YES  |     | NULL    |                |
| updated_at   | datetime(3)     | YES  |     | NULL    |                |
| nick_name    | varchar(42)     | YES  |     | NULL    |                |
| gender       | varchar(12)     | YES  |     | NULL    |                |
| password     | varchar(258)    | YES  |     | NULL    |                |
| avatar       | varchar(256)    | YES  |     | NULL    |                |
| introduction | longtext        | YES  |     | NULL    |                |
| email        | varchar(128)    | YES  |     | NULL    |                |
| qq           | bigint          | YES  |     | NULL    |                |
| tel          | varchar(18)     | YES  |     | NULL    |                |
| birthday     | varchar(128)    | YES  |     | NULL    |                |
| role         | tinyint         | YES  |     | 1       |                |
| user_name    | varchar(42)     | YES  |     | NULL    |                |
+--------------+-----------------+------+-----+---------+----------------+
```

上图就是一个user表，接下来我们简单来解析一下

1. **Field**（字段名）

- **含义**：表示表中列的名称，也就是数据表中每一列的名字。

2. **Type**（数据类型）

- **含义**：表示该字段的数据类型及其长度或精度。MySQL支持各种数据类型，如整数（INT）、浮动小数（FLOAT）、字符串（VARCHAR）、日期时间（DATETIME）等。详细内容可参考下表。

| **类别**         | **数据类型**          | **说明**                             | **范围/长度**                                                |
| ---------------- | --------------------- | ------------------------------------ | ------------------------------------------------------------ |
| **数字类型**     | `TINYINT`             | 存储小范围的整数                     | 有符号：-128 到 127；无符号：0 到 255                        |
|                  | `SMALLINT`            | 存储较小范围的整数                   | 有符号：-32,768 到 32,767；无符号：0 到 65,535               |
|                  | `MEDIUMINT`           | 存储中等范围的整数                   | 有符号：-8,388,608 到 8,388,607；无符号：0 到 16,777,215     |
|                  | `INT` / `INTEGER`     | 存储普通范围的整数                   | 有符号：-2,147,483,648 到 2,147,483,647；无符号：0 到 4,294,967,295 |
|                  | `BIGINT`              | 存储大范围的整数                     | 有符号：-9,223,372,036,854,775,808 到 9,223,372,036,854,775,807；无符号：0 到 18,446,744,073,709,551,615 |
|                  | `FLOAT`               | 存储单精度浮点数                     | 4 字节，精度通常为 7 位数字                                  |
|                  | `DOUBLE`              | 存储双精度浮点数                     | 8 字节，精度通常为 15 位数字                                 |
|                  | `DECIMAL` / `NUMERIC` | 存储定点数（高精度）                 | 定义时指定精度和小数位数，例如 `DECIMAL(10,2)`，表示最多 10 位，2 位小数 |
| **字符串类型**   | `CHAR`                | 固定长度字符串                       | 1 到 255 字符                                                |
|                  | `VARCHAR`             | 可变长度字符串                       | 1 到 65,535 字符                                             |
|                  | `TEXT`                | 变长文本数据                         | 最多 65,535 字符                                             |
|                  | `TINYTEXT`            | 变长文本数据                         | 最多 255 字符                                                |
|                  | `MEDIUMTEXT`          | 变长文本数据                         | 最多 16,777,215 字符                                         |
|                  | `LONGTEXT`            | 变长文本数据                         | 最多 4,294,967,295 字符                                      |
| **日期时间类型** | `DATE`                | 存储日期                             | `YYYY-MM-DD`，范围：1000-01-01 到 9999-12-31                 |
|                  | `DATETIME`            | 存储日期和时间                       | `YYYY-MM-DD HH:MM:SS`，范围：1000-01-01 00:00:00 到 9999-12-31 23:59:59 |
|                  | `TIMESTAMP`           | 存储时间戳                           | `YYYY-MM-DD HH:MM:SS`，范围：1970-01-01 00:00:01 到 2038-01-19 03:14:07 (UTC) |
|                  | `TIME`                | 存储时间                             | `HH:MM:SS`，范围：`-838:59:59` 到 `838:59:59`                |
|                  | `YEAR`                | 存储年份                             | `YYYY`，范围：1901 到 2155                                   |
| **布尔类型**     | `BOOLEAN` / `BOOL`    | 布尔值                               | 0（`FALSE`）或 1（`TRUE`）                                   |
| **二进制类型**   | `BINARY`              | 固定长度的二进制数据                 | 1 到 255 字节                                                |
|                  | `VARBINARY`           | 可变长度的二进制数据                 | 1 到 65,535 字节                                             |
|                  | `BLOB`                | 二进制大对象                         | 最多 65,535 字节                                             |
|                  | `TINYBLOB`            | 小型二进制大对象                     | 最多 255 字节                                                |
|                  | `MEDIUMBLOB`          | 中型二进制大对象                     | 最多 16,777,215 字节                                         |
|                  | `LONGBLOB`            | 大型二进制大对象                     | 最多 4,294,967,295 字节                                      |
| **JSON 类型**    | `JSON`                | 存储 JSON 格式数据                   | 最多 4GB 数据                                                |
| **集合类型**     | `ENUM`                | 枚举类型（限制为一组预定义值之一）   | 1 到 65,535 个预定义值                                       |
|                  | `SET`                 | 集合类型（可存储多个预定义值的组合） | 1 到 64 个预定义值的组合                                     |

3. **Null**（是否允许为NULL）

- **含义**：表示该字段是否允许存储 `NULL` 值，`NULL` 表示缺失或未知的数据。
- 值：

  - `YES`：字段允许为 `NULL`，即可以没有值。
  - `NO`：字段不允许为 `NULL`，即该列必须有值。

4. **Key**（索引类型）

- **含义**：表示该字段是否作为索引的一部分，并指示索引的类型。MySQL支持多种类型的索引。
- **值**：
  - `PRI`：主键索引（Primary Key）。这是一个唯一且非空的索引，表中每个记录的主键值**必须唯一**。
  - `UNI`：唯一索引（Unique Key）。该字段的值**必须唯一**，但允许 `NULL` 值。
  - `MUL`：多重索引（Multiple）。表示该字段是普通索引的一部分，允许有重复值。
  - 如果该列没有任何索引，则该列显示为空。

5. **Default**（默认值）

- **含义**：表示该字段在没有指定值时使用的默认值。默认值可以是常量，也可以是 `NULL`。

- 示例：

  - 如果你在创建表时定义 `age INT DEFAULT 18`，当插入数据时，如果没有给 `age` 字段指定值，它会自动使用默认值 `18`。
- 如果一个字段没有默认值，`Default` 会显示为 `NULL`，表示没有默认值。

6. **Extra**（附加信息）

- **含义**：提供与字段相关的额外信息，例如是否自动递增（AUTO_INCREMENT）等。
- **值**：
  - `auto_increment`：字段是自动递增的，通常用于主键字段（如自增ID）。
  - `on update CURRENT_TIMESTAMP`：表示当记录被更新时，字段会自动更新为当前时间，通常用于记录最后更新时间的字段。
  - 如果没有额外信息，则该列为空

**插入数据后，表内容示例**（这里去掉了一些字段，看起来直观一些）

```sql
+----+------------+-----------+------------------------+---------+------+---------------------+
| id | user_name  | nick_name |         email          | gender  | role |     created_at      |
+----+------------+-----------+------------------------+---------+------+---------------------+
| 1  | xiaoming   | 小明      | xiaoming@example.com   | male    | 1    | 2025-01-15 10:12:33 |
| 2  | xiaohong   | 小红      | xiaohong@example.com   | female  | 1    | 2025-01-15 11:20:10 |
| 3  | admin      | Admin     | admin@example.com      | male    | 2    | 2025-01-10 08:33:21 |
| 4  | anon       | 匿名用户   | anon@example.com       | NULL    | 1    | 2025-01-17 15:02:55 |
| 5  | sonwwall   | 外城      | waicheng@example.com   | male    | 1    | 2025-01-18 21:33:59 |
+----+------------+-----------+------------------------+---------+------+---------------------+
```

### SQL

#### 介绍

SQL 是一种用于**查询和管理关系型数据库**的语言。它可以用来**增删改查数据**，以及创建或修改表结构。语法接近英文，是 MySQL、PostgreSQL、Oracle 等数据库都支持的标准语言。

#### 语法

[sql教程-菜鸟教程](https://www.runoob.com/sql/sql-tutorial.html)

[MySQL入门到精通-黑马程序员](https://www.bilibili.com/video/BV1Kr4y1i7ru/?spm_id_from=333.337.search-card.all.click&vd_source=a57b854dceb9e43274d666a1811d0403)（可以先看到p57,了解一下基本概要）

**这里展示一些基本用法**

#### **SQL 语句的基本结构**

SQL 语句通常由以下几个组成部分构成：

- **关键字（Keywords）**：如 `SELECT`、`INSERT`、`UPDATE`、`DELETE`、`FROM`、`WHERE` 等。
- **表名或列名**：指定操作的目标表或列。
- **操作符（Operators）**：如 `=`, `>`, `<`, `AND`, `OR`, `BETWEEN`, `IN` 等。
- **常量值**：如数字、字符串（字符串需要用单引号包围）。
- **括号（Parentheses）**：用于组织逻辑结构，例如子查询、函数参数等。

#### **SQL 语句书写规则**

- **区分大小写**：SQL 中的关键字通常不区分大小写，大写小写都随你（如 `SELECT` 和 `select` 是等效的），但某些数据库管理系统（如 PostgreSQL）区分表名和列名的大小写。

- **语句结束符**：SQL 语句需要以**分号**（`;`）结尾，特别是在多条语句执行时。

- **空格和缩进**：为了提高代码的可读性，SQL 语句中的关键词、表名、列名、条件等之间应使用空格分隔。通常推荐将每个关键字大写，且在复杂查询中使用缩进来提高可读性。

- **注释**：

  ```sql
  -- 单行注释
  # 单行注释（mysql独有此注释语法）
  /*
  多行注释
  多行注释
  */
  ```

  ---

  

#### **DDL（数据定义语言）** 语句常见规则

**一些基本查询**

```sql
-- 查看当前用户下所有数据库,一般为root用户
SHOW DATABASES;
-- 选择数据库
USE WeCQPUT;
-- 查看当前选择的数据库
SELECT DATABASE();
-- 查询表的结构
DESC Table_Name;
-- 查询指定表的建表语句
SHOW CREATE TABLE Table_name;
```

**CREATE** 语句

`CREATE` 语句用于创建新的数据库对象（如数据库、表、视图、索引等）。创建对象时需要定义其结构和属性。

- **创建数据库**：用于创建一个新的数据库。

  ```sql
  CREATE DATABASE database_name;
  ```

- **创建表**：用于创建一张新的表，并定义表中的列和数据类型。

  ```sql
  CREATE TABLE employees (
      id INT PRIMARY KEY,
      name VARCHAR(100),
      salary DECIMAL(10, 2)
  );
  ```

- **创建索引**：用于创建索引以提高查询效率。

  ```sql
  CREATE INDEX idx_salary ON employees(salary);
  ```

- **创建视图**：用于创建视图，视图是一个虚拟表，它基于一个或多个表的查询结果。

  ```sql
  CREATE VIEW high_salary_employees AS
  SELECT name, salary FROM employees WHERE salary > 50000;
  ```

**规则**：

- 创建对象时，需要明确指定对象的各项属性（如列名、数据类型、约束条件等）。
- `CREATE` 语句通常是不可逆的，删除对象时需要使用 `DROP` 语句。

**ALTER** 语句

`ALTER` 语句用于修改已存在的数据库对象。例如，修改表的结构，添加、删除、修改列，添加约束等。

- **修改表**：用于修改现有表的结构，如添加、删除、修改列。

  - 添加列：

    ```sql
    ALTER TABLE employees ADD hire_date DATE;
    ```

  - 修改列的数据类型：

    ```sql
    ALTER TABLE employees MODIFY salary DECIMAL(12, 2);
    ```

  - 删除列：

    ```sql
    ALTER TABLE employees DROP COLUMN hire_date;
    ```

- **添加约束**：可以使用 `ALTER` 语句向表中添加新的约束条件，如外键、唯一键等。

  ```sql
  ALTER TABLE employees ADD CONSTRAINT fk_department FOREIGN KEY (department_id) REFERENCES departments(id);
  ```

**规则**：

- `ALTER` 语句能够动态修改现有数据库对象，灵活性较强。
- 修改对象时，需要小心处理已有的数据和结构变化，以避免对现有数据造成影响。

**DROP** 语句

`DROP` 语句用于删除数据库对象，如表、视图、索引等。执行 `DROP` 语句后，数据和对象的定义都会被永久删除。

- **删除表**：

  ```sql
  DROP TABLE employees;
  ```

- **删除视图**：

  ```sql
  DROP VIEW high_salary_employees;
  ```

- **删除索引**：

  ```sql
  DROP INDEX idx_salary;
  ```

**规则**：

- `DROP` 语句会永久删除对象及其内容，操作不可逆。
- 使用 `DROP` 删除表时，表中所有的数据、结构以及相关的约束都会被删除。

**TRUNCATE** 语句

`TRUNCATE` 语句用于删除表中的所有数据，但保留表的结构。与 `DELETE` 语句不同，`TRUNCATE` 语句执行时不逐行删除数据，因此速度更快，但不能在事务中回滚（具体行为视数据库管理系统而定）。

- 删除所有数据：

  ```sql
  truncate Table table_name
  ```

**规则**：

- `TRUNCATE` 语句仅清空数据，不会删除表的结构和约束。
- 不能像 `DELETE` 语句那样逐行删除数据，因此在删除大量数据时执行效率更高。

**RENAME** 语句

`RENAME` 语句用于重命名数据库对象，通常用于修改表名或列名。

- **重命名表**：

  ```sql
  RENAME TABLE employees TO staff;
  ```

- **重命名列**（某些数据库系统支持）：

  ```sql
  ALTER TABLE employees RENAME COLUMN salary TO salary_amount;
  ```

**规则**：

- `RENAME` 语句通常用于修改对象的名称，而不影响其数据和结构。

**COMMENT** 语句

`COMMENT` 语句用于为数据库对象（如表、列、索引等）添加注释，以便于开发人员或数据库管理员理解和使用这些对象。

- 为列添加注释：

  ```sql
  COMMENT ON COLUMN employees.salary IS 'Employee salary, in USD';
  ```

**规则**：

- `COMMENT` 语句不会影响数据的存储，仅为对象添加描述性信息。
- 注释对于文档化数据库结构和提高可维护性非常重要。

#### DML（ 数据处理语言）语句常见规则

1. **SELECT **

- `SELECT` 后面列出查询的列名，如果查询所有列，使用 `*`。

- `FROM` 后指定查询的数据表。

- `WHERE` 用于指定过滤条件。

- 例子：

  ```sql
  SELECT * FROM employees WHERE department = 'HR';
  SELECT name, salary FROM employees WHERE department = 'HR';
  ```

2. **INSERT**

- `INSERT INTO` 用来将数据插入到表中，后面跟表名和列名，`VALUES` 后跟插入的值。

- 插入时，值的顺序和列的顺序必须一致。

- 例子：

  ```sql
  INSERT INTO employees (name, department, salary) VALUES ('John Doe', 'HR', 50000);
  ```

3. **UPDATE**

- `UPDATE` 后指定要更新的表，`SET` 后列出需要更新的列和新值。

- `WHERE` 子句是必须的，否则会更新所有行。

- 例子：

  ```sql
  UPDATE employees SET salary = 55000 WHERE name = 'John Doe';
  ```

4. **DELETE**

- `DELETE FROM` 后指定要删除记录的表。

- `WHERE` 子句用于指定删除的条件。若没有 `WHERE`，则会删除所有记录。

- 例子：

  ```sql
  DELETE FROM employees WHERE id = 101;
  ```

#### **SQL 语句的逻辑顺序**

尽管 SQL 语句的书写顺序是固定的，但执行时的逻辑顺序遵循一定的流程：

1. **FROM**：确定数据来源（表或视图）
2. **WHERE**：根据条件筛选数据
3. **GROUP BY**：对数据进行分组（如按部门分组统计薪资）
4. **HAVING**：过滤分组后的数据（如仅选择薪水大于 50000 的部门）
5. **SELECT**：选择需要显示的列
6. **ORDER BY**：对查询结果进行排序

**例子**：

```sql
SELECT department, AVG(salary)
FROM employees
WHERE salary > 40000
GROUP BY department
HAVING AVG(salary) > 50000
ORDER BY department;
```

**总结起来是以下顺序**

| 关键字   | 描述           | 编写顺序 | 执行顺序 |
| -------- | -------------- | -------- | -------- |
| SELECT   | 字段列表       | ①        | ⑤        |
| FROM     | 表名列表       | ②        | ①        |
| WHERE    | 条件列表       | ③        | ②        |
| GROUP BY | 分组字段列表   | ④        | ③        |
| HAVING   | 分组后条件列表 | ⑤        | ④        |
| ORDER BY | 排序字段列表   | ⑥        | ⑥        |
| LIMIT    | 分页参数       | ⑦        | ⑦        |

#### **常见的约定**

- 字符串与日期：字符串使用单引号`' '`包围；日期常用`YYYY-MM-DD`格式，且也用单引号包围

```sql
SELECT * FROM employees WHERE hire_date = '2024-01-01';
```

+ 在排序查询时，升序（ASC）是默认的

#### **SQL 中的 NULL 值**

- `NULL` 表示缺失或未知的数据。在 SQL 中，`NULL` 不等于任何值，包括零（0）或空字符串（""）。用`IS NULL`或`IS NOT NULL`来检查NULL值

  ```sql
  SELECT * FROM employees WHERE salary IS NULL;
  ```

#### **常用的操作符**

- **比较操作符**：`=`, `>`, `<`, `>=`, `<=`, `<>`（不等于）

- **逻辑操作符**：`AND`, `OR`, `NOT`

- **范围操作符**：`BETWEEN`和`IN`

  ```sql
  SELECT * FROM employees WHERE salary BETWEEN 40000 AND 60000;
  ```

- 模糊匹配：LIKE，通常用于字符串匹配

  ```sql
  SELECT * FROM employees WHERE name LIKE 'J%';  -- 姓名以 J 开头
  ```

#### **数据排序**

- 使用`ORDER BY`对查询结果进行排序，可以指定升序（ASC）或降序（DESC）

  ```sql
  SELECT * FROM employees ORDER BY salary DESC;
  ```

## SQLAlchemy

> 上面介绍了如何使用SQL语句操作数据库，但是在实际写Python代码中一般不会直接写SQL语句，而是通过ORM库进行操作，Python最流行的ORM库之一便是SQLAlchemy

#### 什么是ORM

ORM 全称 **对象关系映射（Object–Relational Mapping）**

简单来说，ORM 让你可以像写普通代码一样访问数据库，不需要频繁手写 SQL。它既提高了开发效率，也减少了 SQL 拼接出错的风险。

#### SQLAlchemy使用

首先，强烈推荐查看SQLAlchemy官方文档[概述 — SQLAlchemy 2.0 文档 - SQLAlchemy 中文](https://docs.sqlalchemy.org.cn/en/20/intro.html)，所有关于SQLAlchemy的用法都在里面提到了，建议多看

**下面给大家一个例子，告诉大家如何使用SQLAlchemy，详细的使用方法一定要看文档！！！**

1. *安装*

```python
pip install sqlalchemy
pip install pymysql  # MySQL驱动
```

2. *连接到数据库*

```python
from sqlalchemy import create_engine, Column, Integer, String, DateTime
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from datetime import datetime

# 创建数据库连接,URL格式：mysql+pymysql://用户名:密码@服务器ip地址:端口/数据库名?charset=utf8mb4
DATABASE_URL = "mysql+pymysql://root:123456@localhost:3306/lanshanteam?charset=utf8mb4"
engine = create_engine(DATABASE_URL, echo=True)  # echo=True会打印SQL语句

# 创建基类
Base = declarative_base()

# 创建Session类
Session = sessionmaker(bind=engine)
```

3. *建立模型*

```python
class Member(Base):
    __tablename__ = 'members'
    
    id = Column(Integer, primary_key=True, autoincrement=True)
    created_at = Column(DateTime, default=datetime.now)
    updated_at = Column(DateTime, default=datetime.now, onupdate=datetime.now)
    name = Column(String(32), nullable=False, comment='成员姓名')
    age = Column(Integer, default=18, comment='年龄')
    department = Column(String(64), index=True, comment='部门名称')
    
    def __repr__(self):
        return f"<Member(id={self.id}, name='{self.name}', age={self.age}, department='{self.department}')>"
```

4. *自动迁移*

```python
# 创建所有表
Base.metadata.create_all(engine)
```

5. *基本增删改查*

   

   ```python
   # 创建Session实例
   session = Session()
   
   # 准备数据
   member1 = Member(name="sz", age=19, department="Ops")
   member2 = Member(name="tt", age=19, department="Ops")
   members = [
       Member(name="grt", age=19, department="Backend-Go"),
       Member(name="wjk", age=18, department="Backend-Go"),
       Member(name="hym", age=19, department="Backend-Go"),
       Member(name="zbw", age=19, department="Backend-Python"),
       Member(name="lhy", age=20, department="Backend-Python"),
       Member(name="lk", age=19, department="Ops"),
   ]
   
   # =======================
   #       增（Create）
   # =======================
   session.add(member1)
   session.add(member2)
   session.add_all(members)
   session.commit()
   print("数据创建完成")
   
   # =======================
   #       查（Read）
   # =======================
   
   # 1. 查第一条记录
   m1 = session.query(Member).first()
   print("First 查询:", m1)
   
   # 2. 根据 ID 查询
   m2 = session.query(Member).filter(Member.id == 3).first()
   print("根据 ID 查询:", m2)
   
   # 3. 条件查询（查 Ops 的成员）
   Ops_members = session.query(Member).filter(Member.department == "Ops").all()
   print("Ops 成员:")
   for m in Ops_members:
       print(f"成员: {m.name}")
   
   # 4. 复杂条件查询
   age_20plus_members = session.query(Member).filter(Member.age >= 20).all()
   print("年龄大于等于20的成员:")
   for m in age_20plus_members:
       print(f"成员: {m.name}")
   
   # =======================
   #       改（Update）
   # =======================
   
   # 1. 更新单个字段
   m1.age = 20
   session.commit()
   
   # 2. 更新多个字段
   session.query(Member).filter(Member.id == m1.id).update({
       "name": "kq-new",
       "department": "Backend-Python"
   })
   session.commit()
   
   print("更新完成")
   
   # =======================
   #       删（Delete）
   # =======================
   
   # 1. 根据 id 删除
   member_to_delete = session.query(Member).filter(Member.id == 2).first()
   if member_to_delete:
       session.delete(member_to_delete)
       session.commit()
   
   # 2. 按条件删除
   ops_members = session.query(Member).filter(Member.department == "Ops").all()
   for member in ops_members:
       session.delete(member)
   session.commit()
   
   # 验证删除
   remaining_ops = session.query(Member).filter(Member.department == "Ops").first()
   if not remaining_ops:
       print("Ops部门的成员已全部删除")
   
   session.close()
   print("删除操作完成")
   ```



### 其他

#### 数据库工具

推荐使用[Navicat](https://www.navicat.com.cn/)

[DataGrip](https://www.jetbrains.com/zh-cn/datagrip/)

实在不想装可以使用Pycharm自带的数据库工具

## 作业

#### lv0

使用docker安装好mysql

#### lv1

聊一下WHERE 和 HAVING 的区别

#### lv2

练习SQL语句，以下是 `employees` 表结构

| 字段名       | 数据类型                               | 约束           | 说明                                      |
| ------------ | -------------------------------------- | -------------- | ----------------------------------------- |
| `id`         | INT                                    | 主键，自动递增 | 员工的唯一标识                            |
| `name`       | VARCHAR(100)                           | NOT NULL       | 员工的名字                                |
| `email`      | VARCHAR(150)                           | 唯一，NOT NULL | 员工的邮箱（唯一性）                      |
| `phone`      | VARCHAR(20)                            | NULL           | 员工的电话号码                            |
| `hire_date`  | DATE                                   | NOT NULL       | 员工的入职日期                            |
| `salary`     | DECIMAL(10, 2)                         | NULL           | 员工的薪资                                |
| `department` | VARCHAR(100)                           | NULL           | 员工的部门                                |
| `manager_id` | INT                                    | 外键，NULL     | 上级经理的员工 `id` (指向 `employees.id`) |
| `status`     | ENUM('active', 'inactive', 'on_leave') | 默认 'active'  | 员工状态                                  |
| `created_at` | TIMESTAMP                              | 默认当前时间   | 记录创建时间                              |
| `updated_at` | TIMESTAMP                              | 默认当前时间   | 记录更新时间                              |

大家先使用sql语句创建这个表。

请写出以下十个问题的SQL语句

1. 查询在过去一年内入职的员工数量
2. 查询每个部门的平均工资
3. 查询有超过10年工龄且工资高于某个值的员工
4. 减少所有 "on_leave" 状态员工百分之10的薪资
5. 查询部门里工资最高的员工
6. 查找曾经有两个不同部门工作的员工
7. 删除所有离职员工的记录
8. 查找至少有一个直接下属的经理
9. 查询在过去 30 天内有变动（如状态变更、薪资变更）的员工信息
10. 查询某个员工的经理和下属信息

部分问题可能需要用到函数、多表查询，**大家尽量做，做多少交多少**

#### lv3(选做)

安装好navicat（或datagrip）,navicat安装包已经发在群里了。

#### lv4(选做)

使用SQLAlchemy完成lv2,也是大家尽量做，能做多少交多少。

---

作业完成后,把截图和源代码提交到邮箱lekai@lanshan.email