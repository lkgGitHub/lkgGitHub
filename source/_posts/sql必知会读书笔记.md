---
sqtitle: sql必知会读书笔记
date: 2019-07-24 11:53:46
tags: sql
categories: 数据库
---
1. sql是什么

   SQL （发音为字母 S-Q-L 或 sequel ）是结构化查询语言（ Structured Query Language ）的缩写。 SQL 是一种专门用来与数据库沟通的语言。

2. 为什么

   设计 SQL 的目的是很好地完成一项任务 —— **提供一种从数据库中读写数据的简单有效的方法。**
   
   标准 SQL 由 ANSI 标准委员会管理，从而称为 ANSI SQL 。


# 1. 数据库基础
## 概念：
1. 数据库（database）
保存有组织的数据的容器
2. 表（table）
表是一种结构化的文件，可用来存储某种特定类型的数据。
3. 模式（schema）
关于数据库和表的布局及特性的信息。这些特性定义了数据在表中如何存储，包含存储什么样的数据，数据如何分解，各部分信息如何命名等信息。
4. 列（ column ）
表中的一个字段。所有表都是由一个或多个列组成的。
5. 数据类型
所允许的数据的类型。每个表列都有相应的数据类型，它限制（或允许）该列中存储的数据。

**注意：数据类型兼容**
数据类型及其名称是 SQL 不兼容的一个主要原因。虽然大多数基本数据类型得到了一致的支持，但许多高级的数据类型却没有。更糟的是，
偶然会有相同的数据类型在不同的 DBMS 中具有不同的名称。

6. 行（row）

   表中的一个记录。表中的数据是按行存储的，所保存的每个记录存储在自己的行内。

7. 主键（ primary key ）
	一列（或一组列），其值能够唯一标识表中每一行。

	**主键的条件：**

   > 1. 任意两行都不具有相同的主键值；
   > 2. 每一行都必须具有一个主键值（主键列不允许NULL值）；
   > 3. 主键列中的值不允许修改或更新；
   > 4. 主键值不能重用（如果某行从表中删除，它的主键不能赋给以后的新行）。

   

# 2. 检索数据

提示：

>1. **结束 SQL 语句**。多条 SQL 语句必须以分号（；）分隔。多数 DBMS 不需要在单条 SQL 语句后加分号。
>
>2. **SQL 语句不区分大小写**，因此SELECT与select是相同的。
>
>3. **使用空格**。在处理 SQL 语句时，其中所有空格都被忽略。
>4. **当心逗号**。在选择多个列时，一定要在列名之间加上逗号，但最后一个列名后不加。如果在最后一个列名后加了逗号，将出现错误

注释：

单行：“#” 开头

多行：/*      */



#  3. 排序检索数据

**ORDER BY 子句的位置**
在指定一条ORDER BY子句时，应该保证它是SELECT语句中**最后一条子句**。如果它不是最后的子句，将会出现错误消息

##   按多个列排序

下面的代码检索 3 个列，并按其中两个列对结果进行排序 —— 首先按价格，然后按名称排序。

```sql
SELECT prod_id, prod_price, prod_name
FROM Products
ORDER BY prod_price, prod_name;
```

 ## 按列位置排序

```sql
SELECT prod_id, prod_price, prod_name
FROM Products
ORDER BY 2, 3;
```

ORDER BY 2，3表示先按prod_price，再按prod_name进行排序。

## 指定排序方向

DESC

DESC关键字只应用到直接位于其前面的列名。

警告：在多个列上降序排序
如果想在多个列上进行降序排序，必须对每一列指定DESC关键字。

# 4. 过滤数据

##  WHERE 子句

在SELECT语句中，数据根据WHERE子句中指定的搜索条件进行过滤。WHERE子句在表名（FROM子句）之后给出，



警告： WHERE 子句的位置
在同时使用ORDER BY和WHERE子句时，应该让ORDER BY位于WHERE之后，否则将会产生错误



##  WHERE 子句操作符

| 操作符  | 说 明              |
| ------- | ------------------ |
| BETWEEN | 在指定的两个值之间 |
| IS NULL | 为 NULL 值         |
| < >     | 不等于             |



```sql
SELECT prod_name, prod_price
FROM Products
WHERE (vend_id = 'DLL01' OR vend_id = ‘BRS01’)
AND prod_price >= 10;
```

这条SELECT语句与前一条的唯一差别是，将前两个条件用圆括号括了起来。因为圆括号具有比AND或OR操作符更高的求值顺序，所以 DBMS 首先
过滤圆括号内的OR条件。这时， SQL 语句变成了选择由供应商DLL01或BRS01制造的且价格在 10 美元及以上的所有产品，这正是我们希望的结
果。

# 关键字

| 关键字               | 作用                 | 示例 |
| -------------------- | -------------------- | ---- |
| select               |                      |      |
| from                 |                      |      |
| DISTINCT    distinct |                      |      |
| ORDER BY             |                      |      |
| DESC （DESCENDING）  | 降序                 |      |
| ASC（ASCENDING）     | 升序（升序是默认的） |      |
| WHERE                | 指定搜索条件进行过滤 |      |



1. **distinct**

   它指示数据库只返回不同的值。

   > eg：SELECT DISTINCT vend_id FROM Products;

   **警告：不能部分使用 DISTINCT**
   DISTINCT关键字作用于所有的列，不仅仅是跟在其后的那一列。例如，你指定SELECT DISTINCT vend_id, prod_price，除非指定的两列完
   全相同，否则所有的行都会被检索出来。

   

