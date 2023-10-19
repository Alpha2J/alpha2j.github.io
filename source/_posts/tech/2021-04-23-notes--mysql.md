---
title: MySQL知识点总结
date: 2021-04-23
tags: 
  - notes
  - mysql
categories: technology
keywords: 'basic mysql'
---

# 一、基础概念

1. 数据库和表：一个MySQL实例可能存在许多数据库（database），一个数据库可能存在许多张表（table）。
2. `AUTO_INCREMENT`：该关键字是用于修饰表的列的，在每个行添加到数据表中的时候，MySQL可以自动地为每个行分配下一个可用的编号，而不用在添加一行时手动地分配唯一值。
3. `schema`：在MySQL中`schema`可以看作是`database`的同义词。
4. `DDL, DML, DCL`：
    - `DDL`：全称`data definition language`。主要是用于定义或改变表结构，数据类型等。主要命令有：`CREATE`，`ALTER`，`DROP`。
    - `DML`：全称`data manipulation language`。主要用于对数据表记录进行操作。主要命令有：`SELECT`，`UPDATE`，`INSERT`，`DELETE`。
    - `DCL`：全称`data control language`。主要用于设置或更改数据库用户或角色权限的语句。主要命令有：`GRANT`，`DENY`，`REVOKE`等。
5. 注释：在MySQL中，有三种风格的注释：`#`（单行注释），`-`（单行注释, 后面至少需要跟一个空格）， `/* */`（多行注释）。
6. [字符集](https://www.cnblogs.com/geaozhang/p/6724393.html#mysqlyuzifuji)

# 二、SQL操作

## 2.1 用户操作

用户应该对他们需要的数据具有适当的访问权，即不能多也不能少。在日常工作中，决不能使用root，应该创建一系列的账号，**有的用于管理，有的用于供用户使用，有的供开发人员使用**等等。MySQL用户账号和信息存储在名为`mysql`的MySQL数据库的`user`表中，可以访问这张表进行相应的操作，但是一般不需要。

```sql
# 查看当前数据库的所有用户
SELECT user FROM user;
# 创建用户账号, 并设置密码为123456
CREATE USER alpha IDENTIFIED BY '123456';
# 重命名用户
RENAME USER alpha TO jeb;
# 删除用户账号及相关权限
DROP USER jeb;

# 显示jeb用户的所有权限
SHOW GRANTS FOR jeb;
# 表示对于任何一台主机, jeb用户对任何数据库的任何表都没有权限.
# USAGE表示没有权限, *.*标识任何数据库的任何数据表, `%`表示任何主机.
# GRANT USAGE ON *.* TO `jeb`@`%`

# 授予jeb用户对test_database数据库上所有表的SELECT权限.
GRANT SELECT ON test_database.* TO jeb;
# 撤销jeb用户对test_database数据库上所有表的SELECT权限.
REVOKE SELECT ON test_database.* FROM jeb;

# GRANT ALL 表示赋予所有操作权限, REVOKE ALL 表示撤销所有操作权限.
# 对于test_database数据库的所有表, 赋予所有操作权限给jeb用户.
GRANT ALL ON test_database.* TO jeb;
# 对于test_database数据库的所有表, 移除所有给jeb用户的操作权限.
REVOKE ALL ON test_database.* FROM jeb;

# 使用SET PASSWORD更新用户口令, 新口令必须传递到PASSWORD ()函数进行加密.
SET PASSWORD FOR jeb = PASSWORD ('newpassword');
# 使用SET PASSWORD来设置自己的口令.
SET PASSWORD = PASSWORD ('newpassword');
```

## 2.2 数据库操作

```sql
# ----创建数据库----
CREATE DATABASE test_database;

# ----删除数据库----
DROP DATABASE test_database;

# ----查看数据库----
# 查看所有数据库
SHOW DATABASES;
# 查看当前使用的数据库
SELECT DATABASE();
# 查看创建数据库的语句
SHOW CREATE DATABASE test_database;

# ----连接数据库----
# 刚连接到MySQL数据库的时候是没有任何数据库被打开供我们使用的, 在执行任何数据库操作之前需要选择数据库
# 使用test_database这个数据库
USE test_database;
```

## 2.3 数据表操作

### 2.3.1 表结构操作

```sql
# ----创建表----
CREATE TABLE test_table
(
  id   INT NOT NULL PRIMARY KEY AUTO_INCREMENT COMMENT '自增非空主键',
  col1 INT NOT NULL DEFAULT 1 COMMENT '默认值',
  col2 VARCHAR(255),
  UNIQUE KEY ('col2')
) COMMENT '这是表注释';

# ----删除表----
DROP TABLE test_table;

# ----查看表信息----
# 查看所有数据表
SHOW TABLES;

# 查看数据表的列信息, 这三种方式都是等价的
SHOW COLUMNS FROM test_table;
DESC test_table;
DESCRIBE test_table;

# 查看创建数据表的语句
SHOW CREATE TABLE test_table;

# ----修改表结构----
# 表改名
ALTER TABLE test_table RENAME TO new_test_table;

# 增加表数据列
ALTER TABLE test_table
  ADD col3 VARCHAR(255) NOT NULL;

# 删除表数据列
ALTER TABLE test_table
  DROP col3;

# 修改数据表列定义
ALTER TABLE test_table
  MODIFY col1 VARCHAR(255) NOT NULL;

# 数据列改名
ALTER TABLE test_table
  CHANGE COLUMN col3 new_col3 VARCHAR(255);
```

### 2.3.2 表数据修改操作

```sql
# ----插入数据----
# 普通插入
INSERT INTO test_table(col1, col2, col3)
VALUES ('col1', 'col2', 'col3');

# 插入检索出来的数据
INSERT INTO test_table(col1, col2)
SELECT col1, col2
FROM test_table2;

# 将一个表的内容插入到一个新表
CREATE TABLE new_table AS
SELECT *
FROM test_table;

# ----更新数据----
UPDATE test_table
SET col1 = 1
WHERE id = 1;

# 如果不带上WHERE, 那么这个表中所有记录都会被更新掉
UPDATE test_table
SET col1 = 2;

# ----删除数据----
# 删除表的所有记录. 非常危险的操作, 会删除所有记录.
DELETE
FROM test_table;

# 删除符合条件的记录
DELETE
FROM test_table
WHERE id = 1;
```

### 2.3.3 表数据查询操作

```sql
# 查找单个列
SELECT col1
FROM test_table;

# 查找多个列
SELECT col1, col2
FROM test_table;

# 查找所有列(性能问题, 最好不要这样用)
SELECT *
FROM test_table;

# 查找列, 并为它重命名
SELECT col1 AS renamedCol1
FROM test_table;
```

### 2.3.3.1 DISTINCT

如果多行数据相同，那么`DISTINCT`关键字只会返回他们之间的一行。如果数据行是由多个列组合而成，那么两行数据要每列的值都相同才认为这两行数据是相同的，才会只返回它们之间的一行。

```sql
# col1: 1
# col1: 1
# col2: 2
# 第一和第二行数据相同, 所以会返回两行数据: 第一行1, 第二行2
SELECT DISTINCT col1
FROM test_table;

# col1: 1, col2: 1
# col1: 1, col2: 2
# col1: 1, col2: 2
# 这样第一和第二行数据不同, 第二和第三行数据相同, 所以会返回两行数据.
SELECT DISTINCT col1, col2
FROM test_table;
```

### 2.3.3.2 LIMIT

限制返回的数据行数。如果有两个参数：第一个参数为起始行（下标从0开始)），第二个参数为返回的总行数。如果只有一个参数：从第0行开始，返回参数指定的行数。如果数据行数小于指定行数，则返回尽可能多。

```sql
# 从第4行开始(下标从0开始算), 返回最多10行数据.
SELECT *
FROM test_table
LIMIT 3, 10;

# 从第0行开始(包括第0行), 返回最多4行数据.
SELECT col1
FROM test_table
LIMIT 4;
```

### 2.3.3.3 排序

可以使用`ORDER BY`来对返回的数据进行排序。`ASC`升序，`DESC`降序。

```sql
# 升序
SELECT col1
FROM test_table
ORDER BY col1 ASC;

# 降序
SELECT col1
FROM test_table
ORDER BY col1 DESC;
```

### 2.3.3.4 过滤

使用`WHERE`子句来对数据进行过滤。可以用`AND`或者`OR`来进行多个连接操作，也可以使用`()`来决定优先级。`IN`操作符用于匹配一组值，`NOT`用于否定一个条件。对于`NULL`列的过滤还可以使用`IS NULL`或者`IS NOT NULL`。

```sql
# 使用WHERE来进行过滤
SELECT col1
FROM test_table
WHERE col1 = 'hello world';

# 相当于SELECT col1 FROM test_table WHERE (col1 = '1' AND col2 = '2') OR col3 = '3';
SELECT col1
FROM test_table
WHERE col1 = '1' AND col2 = '2'
   OR col3 = '3';

# 使用IN
SELECT col1
FROM test_table
WHERE col1 IN ('1', '111', '1111');

# 使用NOT
SELECT col1
FROM test_table
WHERE col1 NOT IN ('1', '111', '1111');

# 使用IS NULL
SELECT col1
FROM test_table
WHERE col1 IS NULL;

# 使用 IS NOT NULL
SELECT col1
FROM test_table
WHERE col1 IS NOT NULL;
```

`WHERE`子句可用的操作符有：

| 操作符 | 说明 |
| --- | --- |
| = | 等于 |
| < | 小于 |
| > | 大于 |
| <>, != | 不等于 |
| <=, !> | 小于等于 |
| >=, !< | 大于等于 |
| BETWEEN | 在两个值之间 |
| IS NULL | 为NULL值 |

### 2.3.3.5 通配符

可以在`WHERE`过滤子句中使用`LIKE`来进行通配符匹配：

- `%`：匹配0个或多个字符。
- `_`：匹配一个字符。
    
    ```sql
    # 匹配列的值有>=1个字符的, 且第一个字符是1开头
    SELECT col1
    FROM test_table
    WHERE col1 LIKE '1%';
    
    # 匹配列的值只有2个字符的, 且第一个字符是1开头
    SELECT col1
    FROM test_table
    WHERE col1 LIKE '1_';
    ```
    

### 2.3.3.6 分组

将查询出来的数据进行分组。规则是`GROUP BY`参数的值相等的分为一组。

- 当`GROUP BY`后面有多个参数时，需要每个参数都相同才算是同一分组。
- 可以对同一分组的数据使用汇总函数进行处理，比如`COUNT()`。
- 使用`HAVING`来对组内的数据进行过滤。

```sql
# GROUP BY后面接两个参数, 当col1和col2相同时才算一个分组
SELECT col1, col2, col3
FROM test_table
GROUP BY col1, col2;

# 对同一分组使用聚合函数, 这里COUNT(*)计算的是分组每个分组有多少个元素
SELECT COUNT(*), col1, col2, col3
FROM test_table
GROUP BY col1, col2;

# 使用HAVING过滤分组
# 根据col1, col2进行分组后, 过滤掉分组中col1字段长度小于1的
# 并按照col1字段进行降序排序
SELECT col1, col2, col3
FROM test_table
GROUP BY col1, col2
HAVING LENGTH(col1) > 2
ORDER BY col1 DESC;
```

### 2.3.3.7 组合查询

使用`UNION`或`UNION ALL`来组合两个查询，即将后面查询的结果拼接到前面的查询的结果上。注意：

- 使用`UNION`会去除相同的行。
- 使用`UNION ALL`则不会去除相同行。

```sql
# 这里最多只会返回两行
# 因为前面col1=1的最多一行, 后面col2=2的最多一行
SELECT col1
FROM test_table
WHERE col1 = '1'
UNION
SELECT col2
FROM test_table
WHERE col2 = '2';

# 会返回col=1的所有行数和col2=1的所有行数的总数
SELECT col1
FROM test_table
WHERE col1 = '1'
UNION ALL
SELECT col2
FROM test_table
WHERE col2 = '1';
```

### 2.3.3.8 连接

使用`JOIN...ON`来对表进行连接。有内连接，自连接，外连接。

```sql
# 内连接又称等值连接, 使用INNER JOIN关键字
SELECT A.value, B.value
FROM tablea AS A
       INNER JOIN tableb AS B
                  ON A.key = B.key;
# 可以不明确使用 INNER JOIN, 而使用普通查询并在 WHERE 中将两个表中要连接的列用等值方法连接起来.
# 如果没有条件语句的话返回的是笛卡尔积
SELECT A.value, B.value
FROM tablea AS A,
     tableb AS B
WHERE A.key = B.key;

# 自连接也是内连接的一种, 知识连接的表是自身而已
SELECT e1.name
FROM employee AS e1
       INNER JOIN employee AS e2
                  ON e1.department = e2.department
                    AND e2.name = "Jim";

# 外连接, 有左外连接和右外连接. 左外连接就是保留左表没有关联的行.
# 右外连接就是保留右表没有关联的行.
SELECT Customers.cust_id, Orders.order_num
FROM Customers
       LEFT OUTER JOIN Orders
                       ON Customers.cust_id = Orders.cust_id;
```

### 2.3.4 数据表约束操作

约束是表中的数据的限制条件，在设计表的时候加入约束就是为了保证表中记录的完整和有效。MySQL中约束有：非空约束，唯一性约束，主键约束，外键约束。

```sql
# ----非空约束----
# 非空约束的列不能插入为空.
CREATE TABLE notnull_constraint
(
  id   INT(10),
  name VARCHAR(255) NOT NULL
);

# 非空约束的列不能为空, 否则报错.
# INSERT INTO notnull_constraint(id, name) VALUES (1, null);

# ----唯一约束----
# 唯一约束要求同一列不能有相同的数据.
CREATE TABLE unique_constraint
(
  id   INT(10),
  name VARCHAR(255) UNIQUE
);
INSERT INTO unique_constraint(id, name)
VALUES (1, 'hello');

# 唯一约束不能有相同值
# INSERT INTO unique_constraint(id, name)
# VALUES (2, 'hello');

# 如果创建唯一约束的时候指定两个列, 那么要两个列都相同才算不唯一.
# 这里创建了组合唯一约束, 所以要id和name都相同才算不唯一
# 1 hello
# 2 hello
# 上面两个是可以插入成功的.
CREATE TABLE unique_constraint1
(
  id   INT(10),
  name VARCHAR(255),
  UNIQUE (id, name)
);
INSERT INTO unique_constraint1(id, name)
VALUES (1, 'hello');
INSERT INTO unique_constraint1(id, name)
VALUES (2, 'hello');

# 可以给约束起名字, 方便后续删除约束.
CREATE TABLE unique_constraint2
(
  id   INT(10),
  name VARCHAR(255),
  CONSTRAINT id_name_unique UNIQUE (id, name)
);

# ----主键约束----
# 根据数据库的第二范式, 表设计的时候应该有主键. **为表字段添加主键约束后, 字段不能重复也不能为空, 相当于NOT NULL UNIQUE**. 主键值是当行数据的唯一标识, 两行数据由于主键值不同, 也被认为是不同的记录.
# 单一列主键
CREATE TABLE primary_constraint
(
  id   INT(10) NOT NULL PRIMARY KEY,
  name VARCHAR(255)
);

# 多列主键
# 如果创建主键约束的时候指定多个列, 那么多个列的值相同时才被认为是同一条记录.
CREATE TABLE primary_constraint1
(
  id   INT(10),
  name VARCHAR(255),
  PRIMARY KEY (id, name)
);
INSERT INTO primary_constraint1(id, name)
VALUES (1, 'a');
# 再次执行会失败, 因为primary key相当于not null unique
# INSERT INTO primary_constraint1(id, name)
# VALUES (1, 'a');
# 和唯一约束一样, 可以为它起名.
CREATE TABLE primary_constraint2
(
  id   INT(10),
  name VARCHAR(255),
  CONSTRAINT id_name_primary PRIMARY KEY (id, name)
);

# ----外键约束----
# 外键引用另一张表的字段.
CREATE TABLE class
(
  id   INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(255)
);

CREATE TABLE student
(
  id       INT PRIMARY KEY AUTO_INCREMENT,
  name     VARCHAR(255),
  class_id INT,
  CONSTRAINT FOREIGN KEY (class_id) REFERENCES class (id)
);
```

## 2.4 事务操作

基本术语：

- 事务(transaction)：指一组SQL语句。
- 回退(rollback)：回退事务，撤销事务中SQL的执行。
- 提交(commit)：提交事务，提交事务中SQL的执行。
- 保留点(savepoint)：事务中设置的临时占位符(placeholder)，可以对它发布回退(与回退整个事务处理不同)。

**MySQL中事务默认是隐式提交，每执行一条语句就把这条语句当成一个事务然后进行提交。**当出现`START TRANSACTION`语句时，会关闭隐式提交，当`COMMIT`或`ROLLBACK`语句执行后，事务会自动关闭，重新恢复隐式提交。通过语句`SET AUTOCOMMIT = 0;`可以关闭自动提交，`SET AUTOCOMMIT = 1;`重新打开自动提交。重点：

- `AUTOCOMMIT`是针对每个连接，而不是针对整个服务器的。也就是说，一个连接关闭了自动提交，那么其他连接是不会被关闭的。
- 关闭自动提交后要使用`COMMIT;`将事务做出的改变提交上去，否则连接关闭后改变没办法持久化到数据库。

```sql
# 开启事务
START TRANSACTION;
# ...做一些操作
SAVEPOINT delete1;
# ...做另一些操作

# 回滚到delete1这个保留点
ROLLBACK TO delete1;
# ...最后做一些操作

# 提交事务
COMMIT;
```

## 2.5 索引操作

正确创建并使用索引的话能使数据检索效率显著提高，在MySQL中索引默认是使用B+树结构来存储的。MySQL中索引分为4种：主键索引，唯一索引，全文索引和普通索引。

- 主键索引：使用`PRIMARY KEY`创建主键约束时会自动创建主键索引。
- 唯一索引：使用`UNIQUE`对列创建唯一约束时会自动创建唯一索引。
- 全文索引：
    
    ```sql
    CREATE TABLE fulltextTest
    (
      `id`      int(11) AUTO_INCREMENT PRIMARY KEY,
      `content` text,
      FULLTEXT (content)
    );
    ```
    
- 普通索引：`CREATE INDEX student_firstname_index ON student (firstname);`
    
    ```sql
    # 创建索引
    # 直接创建索引, 指向表的列
    CREATE INDEX student_firstname_index ON student (firstname);
    
    # 修改表创建索引
    ALTER TABLE student
      ADD INDEX student_lastname_index (lastname);
    
    # 建表的时候创建索引
    CREATE TABLE student
    (
      id        INT AUTO_INCREMENT PRIMARY KEY,
      firstname VARCHAR(255),
      lastname  VARCHAR(255),
      INDEX student_lastname_index (lastname)
    );
    
    # 删除索引
    DROP INDEX index_name ON table_name;
    ```
    

## 2.6 其他操作

### 2.6.1 视图

视图是虚拟的表，本身不包含数据，也就不能对其进行索引操作。对视图的操作和对普通表的操作一样。视图具有如下好处：

- 简化复杂SQL操作，比如复杂的连接。
- 只使用实际表的一部分数据。
- 通过只给用户访问视图的权限，保证数据的安全性。
- 更改数据格式和表示

```sql
CREATE VIEW myview AS
SELECT Concat(col1, col2) AS concat_col, col3 * col4 AS compute_col
FROM mytable
WHERE col5 = val;
```

### 2.6.2 存储过程

存储过程可以看成是对一系列SQL操作的批处理。注意：

- 命令行中创建存储过程需要自定义结束符，因为命令行中默认结束符是`;`，和存储过程中的语句分隔符`;`冲突。
- 参数有三种：输入参数`in`，输出参数`out`和输入输出参数`inout`。
- 变量赋值用`SELECT INTO`语句或者用`SET`
- 声明变量使用`DECLARE`

```sql
# 重新定义命令行结束符
# delimiter //

CREATE PROCEDURE myProcedure(OUT ret INT)
BEGIN
  # 声明变量
  DECLARE y INT;
  # 用SET为y赋值
  SET y = 0;
  # 用SELECT...INTO为y赋值
  SELECT SUM(col1)
  FROM myTable INTO y;
  SELECT y * y INTO ret;
END;
```

### 2.6.3 游标

在存储过程中使用游标可以对一个结果集进行移动遍历。游标主要用于交互式应用，其中用户需要对数据集中的任意行进行浏览和修改。使用游标的四个步骤：

- 声明游标，这个过程没有实际检索出数据。
- 打开游标。
- 取出数据。
- 关闭游标。

```sql
DELIMITER //
CREATE PROCEDURE myProcedure(OUT ret INT)
BEGIN
  DECLARE done BOOLEAN DEFAULT 0;

  DECLARE myCursor CURSOR FOR
    SELECT col1 FROM myTable;
  # 定义了一个 continue handler, 当 sqlstate '02000' 这个条件出现时, 会执行 set done = 1
  DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done = 1;

  OPEN myCursor;

  REPEAT
    FETCH myCursor INTO ret;
    SELECT ret;
  UNTIL done END REPEAT;

  CLOSE myCursor;
END //
DELIMITER ;
```

### 2.6.4 触发器

触发器会在某个表执行以下语句时自动执行：`DELETE`，`INSERT`，`UPDATE`。触发器必须指定在语句执行之前还是之后执行。之前执行用`BEFORE`，之后执行用`AFTER`。`BEFORE`一般用于数据验证和净化，`AFTER`用于审计跟踪，将修改记录到另一张表中。

```sql
CREATE TRIGGER myTrigger
  AFTER INSERT
  ON myTable
  FOR EACH ROW SELECT NEW.col into @result;

SELECT @result; -- 获取结果
```

## 2.7 帮助命令

```sql
help 'select';
```

# 三、数据类型

## 3.1 整型

| 类型 | 存储(byte) | 最小值 | 最大值 |
| --- | --- | --- | --- |
| TINYINT | 1 | -27 = -128 | 27-1 = 127 |
| SMALLINT | 2 |  |  |
| MEDIUMINT | 3 |  |  |
| INT | 4 | -231 = -2147483648 | 231-1 = 2147483647 |
| BIGINT | 8 |  |  |

一般情况下，能满足要求的存储空间越小越好。**`INT(11)`中的数字只是规定了交互工具显示字符的个数，对于存储和计算来说是没有意义的。**

## 3.2 浮点数

`FLOAT`和`DOUBLE`为浮点类型，`DECIMAL`为高精度小数类型。CPU 原生支持浮点运算，但是不支持`DECIMAl`类型的计算，因此`DECIMAL`的计算比浮点类型需要更高的代价。三种类型都可以指定列宽，如`DECIMAL(18, 9)`表示总共18位，取9位存储小数部分，剩下9位存储整数部分。

## 3.3 字符串

主要有`CHAR`和`VARCHAR`。

- `CHAR`：定长，效率高，一般用于固定长度的表单提交数据存储。如身份证号，手机号等。`CHAR(M)`列的长度是固定的，M取值0-255。如果定义`CHAR(10)`，存入字符串“abc”的时候，长度不变，后面的7个位置用空格填充，当取出数据的时候会将后面填充的空格删去。
- `VARCHAR`：不定长，效率偏低。`VARCHAR(M)`列的长度为可变长的，M取值0-65535。如果定义`VARCHAR(10)`，存入字符串“abc”的时候，长度变为3，后面不会使用空格来填充。

## 3.4 时间和日期

- `DATETIME`：保存从1001年到9999年的日期和时间，精度为秒，使用8字节的存储空间。
- `TIMESTAMP`：保存从1970年1月1日午夜以来的描述，使用4个字节，只能表示从1970年到2038年。

# 四、索引和锁

## 4.1 索引

1. 索引分类：
    - 物理存储：聚簇索引（叶子节点上有全部数据）、非聚簇索引
    - 功能：主键索引、唯一索引、普通索引（单列、组合）、全文索引
    - 底层实现：B树、B+树、哈希
2. 索引失效的情况：
    - 数据类型不匹配：如果在查询条件中使用的数据类型与索引列的数据类型不匹配，索引将失效。例如，如果索引列是整数，但查询条件使用了字符串，索引将无法生效。
    - 函数或表达式的使用：当在查询条件中使用了函数、表达式或计算字段时，索引通常会失效。
    - 使用通配符前缀：如果在查询条件中使用通配符前缀，如**`LIKE '%value%'`**，索引将无法加速这种查询。
    - 索引列上使用了不适当的函数：某些函数可以使索引失效，例如**`LOWER()`**、**`UPPER()`**等函数，因为它们改变了索引列的值，导致无法使用索引。
    - 不使用索引列：如果查询中不包括索引列，索引将不会生效。索引是用于加速特定列的查询，如果查询中没有包括这些列，索引不会发挥作用。
    - 针对小表的查询：对于非常小的表，MySQL可能会选择全表扫描而不是使用索引，因为索引的开销可能超过全表扫描的开销。
    - 复合索引顺序不合理：如果复合索引的列顺序与查询条件不匹配，索引可能会失效。
3. explain索引分析工具：
    - 是MySQL提供的一个用于分析查询执行计划的关键字，它可以帮助你了解MySQL是如何执行查询的，包括查询中是否使用了索引、执行的顺序、扫描的行数等重要信息。**`EXPLAIN`**的输出结果提供了有关查询性能和索引使用的宝贵信息，可以帮助你优化查询。
    - 执行结果中，比较重要的列有：
        - type：表示MySQL在表中查找行的方式。这是一个关键的列，它可以告诉你是否使用了索引。常见的值包括**`ALL`**（全表扫描）、**`index`**（索引扫描）、**`range`**（范围扫描）、**`ref`**（基于索引的连接）等。
        - possible_keys**：** 表示MySQL可能使用的索引的名称列表。
        - key**：** 表示MySQL实际使用的索引的名称。如果为NULL，表示没有使用索引。
        - ref**：** 显示了在索引中使用的列或常数值，用于查找行。
        - rows**：** 表示MySQL估计需要扫描的行数。
4. 其他重点：
    - 聚簇索引：叶子节点有全部数据。
    - 二级索引：和聚簇索引一样，但是非叶子节点存的是索引列的值，而叶子节点存的是索引列的值和主键值，因此还需要回表。
    - 联合索引：多个索引比较，先按第一个索引排序，如果第一个索引相同，则按第二个排以此类推。
    - 索引的适用条件：只为用于搜索、排序或分组的列创建索引。

## 4.2 锁

1. 全局锁：
    - 解释：全局锁是最强大的锁，用于锁住整个MySQL实例，以防止其他会话对数据库执行任何操作。通常用于备份和维护操作。
    - 使用方式：
        - 获取全局锁：**`FLUSH TABLES WITH READ LOCK;`**
        - 释放全局锁：**`UNLOCK TABLES;`**
2. 表锁：
    - 解释：表锁用于锁住整个表，以防止其他会话对该表执行写操作。通常用于MyISAM等不支持行级锁的存储引擎。
    - 使用方式：
        - 获取表锁（共享读锁）：**`LOCK TABLES table_name READ;`**
        - 获取表锁（排他写锁）：**`LOCK TABLES table_name WRITE;`**
        - 释放表锁：**`UNLOCK TABLES;`**
3. 行锁：
    - 解释：行锁用于锁住表中的单个行，以允许多个会话并发地访问不同行而不互相干扰。通常用于支持事务的存储引擎（如InnoDB）。
    - 使用方式：
        - 开启事务：**`START TRANSACTION;`**
        - 获取行级锁（共享锁）：**`SELECT * FROM table_name WHERE condition FOR SHARE;`**
        - 获取行级锁（排他锁）：**`SELECT * FROM table_name WHERE condition FOR UPDATE;`**
        - 释放行级锁：不需要显式释放，它们会在事务结束时自动释放
    - 行锁和事务的关系：
        - 无论对一行数据加的是读锁还是写锁，其他任何事务都要等这个锁释放了才能对数据进行更改，但不影响查询
        - 对数据做变更，会自动加锁
        - 两个事务同时修改行r1，如果事务A先执行update语句，则事务A先hold住了行r1上的锁，事务B的update语句要等事务A提交了释放锁之后才能执行下否，否则一直阻塞。

# 五、应用

## 5.1 分库分表

1. 是什么？
    - 分库分表是一种数据库架构设计方法，通常用于处理大规模数据和高并发访问的情况。它的主要目的是将数据库中的数据划分为多个数据库或表，以提高数据库性能、可扩展性和负载均衡。分库分表通常用于关系型数据库管理系统（RDBMS）中。
2. 有哪些概念？
    - 分库（Sharding）：分库是将整个数据库拆分为多个独立的数据库实例。每个数据库实例通常存储部分数据。这个过程可以根据一定的规则或数据分布来完成，以确保数据均匀地分布在不同的数据库中。
    - 分表（Partitioning）：分表是将单个数据库表拆分为多个表。每个表通常包含数据的一部分，例如按时间范围、分片键或其他规则进行拆分。这有助于降低单个表的数据量，提高查询性能。
3. 切分方式：
    - 只分表
    - 分表又分库
    - 只分库不分表（有必要吗）
4. 垂直切分和水平切分的场景？
    - 垂直切分：
        - 数据模块化：当数据库中的数据可以根据功能、模块或实体的逻辑进行划分时，垂直切分是一种常见的策略。例如，用户信息、订单信息、产品信息等可以分别存储在不同的表或数据库中。
        - 查询模式不同：如果不同数据实体具有不同的查询模式和访问频率，可以将其分开以优化性能。例如，某些表经常用于读操作，而其他表主要用于写操作。
        - 数据隔离：某些数据需要额外的安全性或隔离，例如涉及个人身份信息（PII）的数据。垂直切分可以帮助实现更强的访问控制。
        - 性能优化：某些数据表可能包含大量列，但应用程序只需要使用其中的一部分。通过将不常用的列分离到另一个表中，可以提高查询性能。
    - 水平切分：
        - 大规模数据：当数据库中的数据量非常大，单个表的数据过多，导致性能下降时，水平切分可以将数据拆分成多个分片，以分散负载。
        - 可扩展性：水平切分是一种增加系统容量和性能的有效方式。随着数据的增长，可以轻松地添加更多的分片以支持更多数据。
        - 负载均衡：当应用程序需要处理高并发或大规模的查询请求时，水平切分可以帮助实现负载均衡，确保查询请求分散到不同的分片上。

## 5.2 主从复制

1. 是什么？
    - MySQL的主从复制是一种数据库复制技术，用于在一个MySQL数据库服务器（主服务器）上的数据更改自动地复制到一个或多个其他MySQL服务器（从服务器）。
2. 为什么要使用？
    - 数据备份和冗余：通过主从复制，可以在从服务器上创建主服务器的完整备份。这提供了数据冗余和紧急情况下的数据恢复选项。如果主服务器发生故障，可以迅速切换到从服务器以保持应用程序的可用性。
    - 负载均衡：通过将读操作分发到从服务器，可以减轻主服务器的负载，提高应用程序的性能。这对于高流量的应用程序非常有用，因为它可以更好地分散查询负载。
    - 高可用性：主从复制提供了一种方式来提高系统的可用性。如果主服务器不可用，可以将流量切换到从服务器，以确保应用程序继续运行。这种故障转移能力对于关键业务系统至关重要。
    - 灾难恢复：如果主服务器所在的数据中心或服务器发生灾难，从服务器可以用于恢复数据。这对于避免数据丢失和快速重建系统非常有帮助。
    - 数据分析和报告：从服务器可以用于执行复杂的数据分析和报告任务，而不会干扰主服务器的正常运行。这允许你执行大型查询，而不会影响应用程序性能。
    - 读写分离：主从复制使读写分离成为可能。通过在从服务器上执行读操作，而在主服务器上执行写操作，可以更好地分配数据库负载，提高应用程序性能。
    - 故障排除和监控：通过监控主从复制状态，你可以更容易地检测数据库问题并进行故障排除。这有助于维护数据库系统的稳定性。
3. 会有哪些问题：
    - 数据一致性问题：主从复制中，可能会出现数据不一致的情况，特别是在复制延迟发生时。应用程序需要处理这种情况，确保数据一致性。
    - 网络问题：网络中断或不稳定的网络连接可能导致主从复制失败或延迟。需要考虑网络稳定性，并在出现问题时采取措施。
    - 复制错误：复制过程中可能会出现错误，如主从服务器的数据不同步或复制线程出现问题。需要监控复制状态并解决这些问题。

## 5.3 读写分离

1. 是什么？
    - 读写分离是一种数据库架构策略，旨在优化数据库系统的性能和可伸缩性。它的核心思想是将对数据库的读操作和写操作分离到不同的服务器或数据库实例上，以便有效地分摊负载、提高查询性能，同时减轻主数据库服务器的压力。这种策略通常用于高流量的应用程序，特别是Web应用程序。
2. 为什么要使用？
    - 提高性能：通过将读操作分散到多个从数据库上，可以提高读操作的性能，从而减轻主数据库的负担。这有助于应对高流量的应用程序。
    - 负载均衡：读写分离可以将流量分发到多个从数据库上，以实现负载均衡。这确保了数据库服务器的负载分散，提高了应用程序的响应性。
    - 高可用性：如果主数据库发生故障，可以快速切换到从数据库以确保应用程序的可用性。这提供了一种灾难恢复和故障切换机制。
    - 减少锁竞争：由于写操作主要发生在主数据库上，从数据库通常不会有太多的写操作，这有助于减少锁竞争和提高并发性能。
    - 数据保护：从数据库可以用于数据备份和报告生成，而不会影响主数据库的性能。
3. 如何实施？
    - 部署主从数据库服务器：首先，在数据库服务器层面，需要设置主数据库和一个或多个从数据库。这可以通过在不同的服务器上安装相同的数据库系统并配置复制来完成。
    - 配置数据库复制：配置主从数据库之间的数据库复制。这包括在主数据库上启用二进制日志（binary log），创建复制账户，以及配置从数据库连接到主数据库并复制数据。
    - 读写分离代理/中间件：引入一个代理或中间件，它将负责路由读操作和写操作到相应的数据库服务器。代理可以是专门设计的软件，也可以是一些成熟的数据库中间件产品，如ProxySQL、MaxScale、HAProxy等。
    - 负载均衡和故障切换：代理/中间件可以实施负载均衡，确保读操作均匀地分布到多个从数据库上，以提高性能和可伸缩性。同时，代理也应具备故障切换机制，当主数据库发生故障时，可以将流量自动切换到另一个可用的主数据库。
    - 应用程序配置：应用程序需要连接到读写分离代理而不是直接连接到数据库服务器。这意味着需要调整应用程序的数据库连接设置，以便应用程序知道如何将读操作和写操作发送到正确的地方。
    - 读写操作路由：代理/中间件会根据请求类型（读或写）和其他规则，将操作路由到主数据库或从数据库。通常，读操作路由到从数据库，写操作路由到主数据库。
    - 监控和维护：建立监控系统以跟踪数据库服务器的状态，包括主服务器和从服务器。监控系统应能够检测到故障和延迟，以及在必要时采取措施，如故障切换。
    - 数据一致性：需要确保数据在主服务器和从服务器之间保持一致。这可能需要处理复制延迟以及在写操作前等待复制完成的策略。
    - 备份和灾难恢复：从服务器可以用于备份和灾难恢复，确保数据的完整性和安全性。
    - 扩展和优化：随着应用程序的发展，可能需要进一步扩展和优化读写分离架构，以应对更大的流量和更多的数据库服务器。

# 六、数据库系统理论

## 6.1 事务

事务指的是满足ACID特性的一组操作，这组操作要么全部成功，要么全部失败。可以通过`commit`提交一个事务，也可以使用`rollback`进行回滚。

## 6.2 事务ACID特性

### 6.2.1 原子性(Atomicity)

原子性是指事务是一个不可分割的工作单位，事务中的操作要么全部成功，要么全部失败。比如在同一个事务中的SQL语句，要么全部执行成功，要么全部执行失败。

### 6.2.2 一致性(Consistency)

事务必须使数据库从一个一致性状态变换到另外一个一致性状态。以转账为例子，A向B转账，假设转账之前这两个用户的钱加起来总共是2000块。那么A向B转账之后，不管这两个账户怎么转，A用户的钱和B用户的钱加起来的总额还是2000，这个就是事务的一致性。

### 6.2.3 隔离性(Isolation)

指的是在多个并发事务同时运行时，每个事务的操作应该被隔离开来，互相不会干扰，从而确保数据的完整性和一致性。

既要达到这么一种效果：对于任意两个并发的事务T1和T2，在事务T1看来，T2要么在T1开始之前已经结束，要么在T1结束之后才开始，这样每个事务都感觉不到有其他事务在并发地执行。

### 6.2.4 持久性(Durability)

**一旦事务提交，则其所做的修改将会永远地保存到数据库中。即使系统发生崩溃，事务执行的结果也不能丢失。**

## 6.3 事务隔离级别

### 6.3.1 未提交读(READ UNCOMMITTED)

事务中的修改，即使是没有提交，对其他事务也是可见的。会导致脏读，不可重复读，幻读。

### 6.3.2 提交读(READ COMMITTED)

事务中的修改，只有在事务提交后才对其他事务可见，也就是说事务T1和T2，T1中的修改只有在事务提交之后才能被T2读取到。**可以避免脏读，但是还是会出现不可重复读和幻读**(因为T2它虽然不能读取T1中修改过的数据，还可能对数据进行修改，所以如果T2对数据进行了修改，然后T1读取了，T2回滚事务，T1再次读取，这时候就会出现不可重复读)。

### 6.3.3 可重复读(REPEATABLE READ)

保证在同一事务中多次读取同样的数据都是一样的，可以阻止脏读和不可重复读，但是还会出现幻读。

### 6.3.4 可串行化(SERIALIZABLE)

最高的隔离级别。所有事务依次逐个执行，这样事务之间就完全不可能产生干扰。可以防止脏读，不可重复读，幻读。这将严重影响程序的性能，通常情况下也不会用到该级别。

## 6.4 并发一致性问题

不同事务隔离级别会出现不同的并发一致性问题

### 6.4.1 丢失修改

T1和T2两个事务都对一个数据进行修改，T1先修改，T2随后修改，T2的修改覆盖了T1的修改。

### 6.4.2 脏读

如果一个事务T1对数据进行了更新，但是事务还没有提交，另一个事务T2读到了这个没有提交的数据，这时事务T1回滚。那么T2读到的就是脏数据。解决办法是将事务的隔离级别调整到`READ_COMMITTED`。

### 6.4.3 不可重复读

在一个事务中，多次读取同一数据，在事务还没有结束的时候由于其他事务对该数据的修改导致多次读取的同一数据是不一样的，这被称为不可重复读。

### 6.4.4 幻影读

T1读取某个范围的数据，T2在这个范围内插入新的数据，T1再次读取这个范围的数据，此时获取的结果和第一次读取的结果不同。

## 6.5 三范式

满足最低要求的范式是第一范式。在第一范式的基础上进一步满足更多规范要求的称为第二范式，其余范式以此类推。**一个数据库的设计如果符合第二范式，一定也符合第一范式。如果符合第三范式，一定也符合第二范式。**

### 6.5.1 第一范式(属性不可分)

字段具有原子性，不可再分。比如姓名字段，如果不用区分姓和名的时候，那么姓和名可以在一个字段。但是如果要区分的时候，他们还在同一个字段，那么这时候就不符合第一范式了。

### 6.5.2 第二范式(需要有主键)

数据表中每个实例或行必须可以被唯一地区分。**通常是为表加上一列，以存储各个实例的标识，这个唯一标识就是主键。**比如学生信息组成学生表，会有一个id列作为主键。可以使用姓名作为主键列吗？不能，因为同名的话就不唯一了。

### 6.5.3 第三范式(消除冗余)

消除冗余，各种信息只在一个地方存储，不存在多个表中。比如大学分了很多系(中文系，英文系…), 每个系都有系编号，系主任，系简介。如果有一张学生表，能不能将系编号，系主任，系简介和放在学生表中和学生信息存在一起？不能，因为这样会冗余。做法是将这些信息放到另一张表中，然后再学生表中使用系编号来引用。

# 七、参考资料

1. 官方文档：https://dev.mysql.com/doc/refman/8.0/en/replication.html
2. mysql是怎样运行的：https://item.jd.com/13009316.html
3. mysql45讲：https://time.geekbang.org/column/intro/139
4. 锁：https://mp.weixin.qq.com/s/b7Qnzh1EIM4wbExwmIkJyA