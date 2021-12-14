---
title: sql server知识归纳总结
date: 2021-10-28 15:35:23
layout: 2021-10-28 15:35:23
id: 0024
tags:
 - sqlserver
---


Sql Server 知识点总结：
* 索引（INDEX）
* 连接（JOIN）
* 联合（UNION）
* 键（KEY）

<!--more-->

### 索引
* 聚集索引
  1. 聚集索引存储记录为连续
  2. 一个表只能存在一个聚集索引

* 非聚集索引
  1. 存储记录非连续
  2. 一个表可以有一个或多个

 索引管理的sql脚本
 ```sql
 --创建索引
 CREATE [UNIQUE][CLUSTERED|NONCLUSTERED] INDEX index_name
 ON {table_name | view_name} [WITH [index_property[,....n]]]

 --删除索引
 DROP INDEX table_name.index_name[,table_name.index_name]

 --显示索引信息，使用系统存储过程sp_helpindex
 exec sp_helpindex table_name;
 ```

<strong style="color:red">注意：</strong>
* 主键是一个约束(Constraint),依附在一个索引上，可以是聚集的也可以是非聚集的。
* 如果没有指定表的主键是聚集索引，可能表格还是以堆的方式管理，而不是B-树的方式管理，效率低下


哪些情况适合创建索引


动作描述   | 是否应该使用聚集索引  | 是否应该使用非聚集索引 
:---:|:---:|:---:
外键列 | 是  | 是  
主键列 | 是  | 是  
列经常被分组排序（order by) | 是  | 是
返回某范围内的数据 | 是  | 否
小数目的不同值 | 是  | 否
大数目的不同值 | 否  | 是
频繁更新的列 | 否  | 是
频繁修改索引列 | 否  | 是
一个或极少不同值 | 否  | 否


建立索引原则：
* 主键列一定要定义索引
* 外键列一定要定义索引
* 经常查询的数据列，最好定义索引
* 对于需要在指定范围内的快速频繁查询的数据列
* 经常用在WHERE子句的列
* 经常出现在ORDER BY,GROUP BY,DISTINCT后面的字段，建立索引。若建立的符合索引，索引字段顺序需要和这些关键字后面的字段顺序一致，否则索引不会被使用
* 查询中很少涉及的列，重复值比较多的列不要建立索引
* 定义为text, image和bit类型的列不要建立索引
* 经常存取的列避免建立索引
* 限制表中的索引数目。存在大量更新操作的表，建索引数目一般不超过3个，最多不要超过5个。索引虽然提高访问速度，但太多索引会影响数据更新操作。
* 对于复合索引，按照字段在查询条件中的出现的频度建立索引。在复合索引中，记录首先按照第一个字段排序。对于在第一个字段上取值相同的记录，系统再按照第二个字段的取值排序，以此类推。因此只有复合索引的第一个字段出现在查询条件中，该索引才可能被使用,因此将应用频度高的字段，放置在复合索引的前面，会使系统最大可能地使用此索引，发挥索引的作用。

一般的：
* 有大量重复值、且经常有范围查询（between, >,< ，>=,< =）和order by、group by发生的列，可考虑建立群集索引；

* 经常同时存取多列，且每列都含有重复值可考虑建立组合索引；

* 组合索引要尽量使关键查询形成索引覆盖，其前导列一定是使用最频繁的列。

### 表连接
为了得到完整结果，需要从两个或更多的表中获取结果，此时需要执行join操作。具体可分为
* JOIN：如果表中有至少一个匹配，则返回行(INNER JOIN等同)
* LEFT JOIN：即使右表中没有匹配，也从左表返回所有的行
* RIGHT JOIN：即使左表中没有匹配，也从右表返回所有行
* FULL JOIN：只要其中一个表中存在匹配，就返回行
部分数据库会将LEFT JOIN成为LEFT OUTTER JOIN，RIGHT JOIN成为RIGHT OUTTER JOIN。


### 联合查询
UNION操作符用于合并两个或多个SELECT的结果集。SELECT语句必须拥有相同数量的列，且对应字段的数据类型相似，列的顺序须相同。

UNION会去除SELECT结果集中的重复内容
UNION ALL几乎等同于UNION，但SELECT结果集会全部返回（不去重）

```sql
 SELECT EmpName FROM Employees_USA
 union [all]
 SELECT EmpName FROM Employees_China
```


### 键
PRIMARY KEY 约束唯一标识表中的每条记录。必须包含唯一的值，不能包含NULL值，每个表应该有且只有一个主键。
```sql
CREATE TABLE Persons
(
Id_P int NOT NULL PRIMARY KEY,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)

--命名 PRIMARY KEY 约束，以及为多个列定义 PRIMARY KEY 约束
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CONSTRAINT pk_PersonID PRIMARY KEY (Id_P,LastName)
)

--已创建的表中添加主键
ALTER TABLE Persons
ADD PRIMARY KEY (Id_P)
--已创建的表中添加多列主键约束
ALTER TABLE Persons
ADD CONSTRAINT pk_PersonID PRIMARY KEY (Id_P,LastName)
--删除主键约束
ALTER TABLE Persons
DROP CONSTRAINT pk_PersonID
```

FOREIGN KEY 一个表中的FOREIGN KEY指向另一个表的PRIMARY KEY。FOREIGN KEY约束用于预防破坏表之间连接的动作；也能防止非法数据插入外键列，因为它必须是它指向的那个表中的值之一。
```sql
CREATE TABLE Orders
(
Id_O int NOT NULL PRIMARY KEY,
OrderNo int NOT NULL,
Id_P int FOREIGN KEY REFERENCES Persons(Id_P)
)


--需要命名 FOREIGN KEY 约束，以及为多个列定义 FOREIGN KEY 约束
CREATE TABLE Orders
(
Id_O int NOT NULL,
OrderNo int NOT NULL,
Id_P int,
PRIMARY KEY (Id_O),
CONSTRAINT fk_PerOrders FOREIGN KEY (Id_P)
REFERENCES Persons(Id_P)
)
```


---

### Northwind数据库示例查询
SQL Server2000的时候，安装包中带了SampleDb，微软引入了一个Northwind的商贸公司数据作为示例。这个数据库作为示例讲解具有一定价值，但后期版本中重新引入了一个新的AdventureWorks示例数据库。Northwind在网上搜索一番，官方已经找不到下载连接了。我这里提供一个创建的[脚本下载连接](https://gitee.com/gxwang/codes/ad3te8no1iyf49bk2wvmc24)，脚本文件较大，gitee上需要下载。下载后直接使用txt或其他文本工具打开即可。下面是一些常用的查询题目。
```sql
--1.查询订购日期在1996年7月1日至1996年7月15日之间的订单的订购日期、订单ID、客户ID和雇员ID等字段的值
select OrderDate, OrderId, CustomerId, EmployeeId from Orders
where OrderDate between '1996-07-01' and '1996-07-15'

--2.--查询“Northwind”示例数据库中供应商的ID、公司名称、地区、城市和电话字段的值。条件是“地区等于Western”并且“联系人头衔等于Sales Representative”。
select SupplierID, CompanyName, Region, City, Phone 
FROM
Suppliers WHERE Region = 'Western' and ContactTitle = 'Sales Representative'


--6.查询“10248”和“10254”号订单的订单ID、运货商的公司名称、订单上所订购的产品的名称
SELECT A.OrderId, B.CompanyName as ShipperCompanyName, D.ProductName 
FROM Orders as A 
JOIN Shippers as B ON A.ShipVia = B.ShipperID
JOIN [Order Details] as C ON C.OrderId =A.OrderId
JOIN Products as D on D.ProductID = C.ProductID 
where A.OrderID = 10248 OR A.OrderID = 10254


--10.查询单价介于10至30元的所有产品的产品ID、产品名称和库存量
SELECT ProductID, ProductName, UnitsInStock
FROM Products
WHERE UnitPrice BETWEEN 10 AND 30


--11.--查询 单价大于20元  的所有 产品 的 ‘产品名称’、‘单价’以及‘供应商的公司名称’、‘电话’
SELECT A.ProductName as '产品名称', A.UnitPrice as '单价', B.CompanyName as '供应商的公司名称', B.Phone as '电话'
FROM Products as A JOIN Suppliers as B on A.SupplierId = B.SupplierId
WHERE A.UnitPrice > 20


--14.按 运货商公司名称，统计 1997年 由各个运货商承运的 '订单的总数量'
SELECT COUNT(A.OrderId) as '订单总量', B.CompanyName as '货运商' FROM Orders as A
join Shippers as B on A.ShipVia = B.ShipperID
WHERE DATEPART(YEAR, A.OrderDate) = '1997'
-- WHERE YEAR(A.OrderDate) = '1997'
GROUP BY B.CompanyName


--15.统计 1997年上半年 的 每份订单 上所订购的 产品 的 总'数量'

SELECT A.OrderID as '订单号', SUM(A.Quantity) as '产品总数量' FROM [Order Details] as A
join Orders as B on A.OrderID = B.OrderID
WHERE B.OrderDate < '19970701'
GROUP BY (A.OrderID)


--16.统计 各类产品 的 平均价格
SELECT B.CategoryName as '商品类别', AVG(A.UnitPrice) as '均价' FROM Products as A
JOIN Categories as B on A.CategoryID = B.CategoryID
GROUP BY B.CategoryName


-- 查看指定表的索引信息,使用系统存储过程 sp_helpindex
Exec sp_helpindex Orders;

```

