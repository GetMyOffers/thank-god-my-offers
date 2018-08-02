## 参考
- 《SQL必知必会》
- [基础语法](http://www.w3school.com.cn/sql/index.asp)
- [范式](https://www.zhihu.com/question/24696366)
- [索引](http://www.cnblogs.com/king1302217/archive/2010/05/11/1732250.html)
- [数据库面试题](https://zhuanlan.zhihu.com/p/23713529)


## 概念
DMBS数据库管理系统

RDMBS关系型数据库管理系统

SQL对大小写不敏感，SQL（结构化查询语言）包括DML（数据操纵语言)，DDL(数据定义语言)


## 基础语法
#### SELECT
从表中选取数据存储在结果表中
```
select 列名称 from 表名称
select * from 表名称    //*表示选取所有的列
```
#### DINSTICT
用于返回唯一不同的值。

```
SELECT DISTINCT 列名称 FROM 表名称
```
#### WHERE子句
如需有条件地从表中选取数据，可将 WHERE 子句添加到 SELECT 语句。

```
SELECT 列名称 FROM 表名称 WHERE 列 运算符 值
```
可使用的运算符有比较运算符，BETWEEN AND（范围），LIKE（模式）。

#### AND & OR
用于基于一个以上的条件对记录进行过滤。
```
SELECT * FROM Persons WHERE (FirstName='Thomas' OR FirstName='William') AND LastName='Carter'
```
当一起使用时，AND优先级比OR高。

#### ORDER BY

用于对结果集排序，根据指定的列和顺序。

默认升序排列，降序排列使用DESC。

```
SELECT Company, OrderNumber FROM Orders ORDER BY Company DESC, OrderNumber ASC
```

#### INSERT INTO
向表格中插入新的行

```
INSERT INTO 表名称 VALUES (值1, 值2,....)

INSERT INTO table_name (列1, 列2,...) VALUES (值1, 值2,....)
```

#### UPDATE 
用于修改表中的数据

```
UPDATE 表名称 SET 列名称 = 新值 WHERE 列名称 = 某值
```
#### DELETE
DELETE 语句用于删除表中的行。

```
删除某行
DELETE FROM 表名称 WHERE 列名称 = 值

删除所有行(表结构，属性，索引仍完整)
DELETE FROM table_name
或者：
DELETE * FROM table_name
```

## 高级教程
#### LIKE
`LIKE`用于所有指定的匹配模式

`NOT LIKE`表示不匹配指定的模式。
```
SELECT column_name(s)
FROM table_name
WHERE column_name LIKE pattern
```
通配符：
- `%`表示任意字符，包括零个。
- `_`表示一个字符。
- `[charset]`字符列中的任意单个字符
- `[^charlist]`或者
`[!charlist]`
不在字符列中的任何单一字符

#### IS [NOT] NULL
`NULL`表示空值，只能用`IS NULL`或者`IS NOT NULL`判断
#### IN
在`WHERE`子句中规定多个值

```
SELECT column_name(s)
FROM table_name
WHERE column_name IN (value1,value2,...)
```
#### BETWEEN
选取两个值之间的范围。值类型可以是数值，文本，日期。

```
SELECT column_name(s)
FROM table_name
WHERE column_name
BETWEEN value1 AND value2
```
#### 指定别名Alias
可为列名称和表名称指定别名

```
表的 SQL Alias 语法，有的数据库不加括号
SELECT column_name(s)
FROM table_name
AS alias_name

列的 SQL Alias 语法
SELECT column_name AS alias_name
FROM table_name

```
#### JOIN
用于连接两个表，通常使用主键。
- `INNER JOIN`同`JOIN`，至少存在一个匹配，即两表均有的行。
- `LEFT JOIN`，返回左表所有的行。(有些数据库中用`LEFT OUTER JOIN`).
- `RIGHT JOIN`，类似`LEFT JOIN`.
- `FULL JOIN`，只要某个表有匹配就返回行。

注意：后跟 `on`和`where`的用法
```
引用两个表
SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
FROM Persons, Orders
WHERE Persons.Id_P = Orders.Id_P 

使用关键字JOIN
SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
FROM Persons
INNER JOIN Orders
ON Persons.Id_P = Orders.Id_P
ORDER BY Persons.LastName
```

#### UNION
合并多个SELECT语句的结果集。
注意，SELECT语句之间的列数和数据类型应保持一致。另外，UNION 结果集中的列名总是等于 UNION 中第一个 SELECT 语句中的列名
```
SQL UNION 语法
SELECT column_name(s) FROM table_name1
UNION
SELECT column_name(s) FROM table_name2
注释：默认地，UNION 操作符选取不同的值。如果允许重复的值，请使用 UNION ALL。

SQL UNION ALL 语法
SELECT column_name(s) FROM table_name1
UNION ALL
SELECT column_name(s) FROM table_name2
```

#### SELECT　INTO
从一个表中选取数据插入另一个表中，常用备份表。后可跟WHERE子句或者JOIN连接表。

```
您可以把所有的列插入新表：
SELECT *
INTO new_table_name [IN externaldatabase] 
FROM old_tablename

或者只把希望的列插入新表：
SELECT column_name(s)
INTO new_table_name [IN externaldatabase] 
FROM old_tablename
```
#### CREATE DATABASE
创建数据库

```
CREATE DATABASE database_name
```
#### CREATE TABLE
创建表，数据类型参考不同的数据库，后可跟约束。
```
CREATE TABLE 表名称
(
列名称1 数据类型,
列名称2 数据类型,
列名称3 数据类型,
....
)
```

#### SQL约束Constraints
可以在创建表时规定约束（通过` CREATE TABLE` 语句），或者在表创建之后也可以（通过` ALTER TABLE `语句）。有些可用于单列，有的可用于多列。
- `NOT NULL` 强制字段始终包含值
- `UNIQUE` 唯一标识表中的每条记录，可有多个
- `PRIMARY KEY` 主键即`NOT NULL`+`UNIQUE`，只能有一个
- `FOREIGN KEY`，外键

    ```
    CREATE TABLE Orders
    (
    Id_P int,
    FOREIGN KEY (Id_P) REFERENCES Persons(Id_P)
    )
    ```
- `CHECK`约束，限制范围
   
    ```
    CREATE TABLE Persons
    (
    CONSTRAINT chk_Person CHECK (Id_P>0 AND City='Sandnes')
    )
    ```
- DEFAULT用于向列中插入默认值
     
    ```
    CREATE TABLE Persons
    (
    City varchar(255) DEFAULT 'Sandnes'
    )
    ```
#### CREATE INDEX
创建索引，加速查询。

```
允许使用重复值
CREATE INDEX index_name
ON table_name (column_name)

唯一的索引
CREATE UNIQUE INDEX index_name
ON table_name (column_name)

示范：在一个表上多个列建立索引，第一个降序
CREATE INDEX PersonIndex
ON Person (LastName DESC, FirstName)
```
#### DROP
删除索引，表，数据库

```
DROP INDEX/TABLE/DATABASE 名称
```

#### ALTER TABLE
增加，修改，删除列。

```
ALTER TABLE table_name
ADD column_name datatype

ALTER TABLE table_name 
DROP COLUMN column_name
注释：某些数据库系统不允许这种在数据库表中删除列的方式 (DROP COLUMN column_name)。


ALTER TABLE table_name
ALTER COLUMN column_name datatype
```
#### AUTO INCREMENT
在新记录插入表中时生成一个唯一的数字。

```
P_Id int NOT NULL AUTO_INCREMENT
默认从1开始，每次递增1

ALTER TABLE Persons AUTO_INCREMENT=100
改变起始值
```

#### VIEW
虚表，或可视化的表。

```
CREATE VIEW view_name AS
SELECT column_name(s)
FROM table_name
WHERE condition
```


## 函数
#### 聚合函数
注意，只能在SELECT或HAVING下使用
- AVG
- SUM
- COUNT
- MIN
- MAX
