# 事务（Transaction）

## 1 3.1、什么是事务？
一个事务是一个完整的业务逻辑单元，不可再分。

比如：银行账户转账，从A账户向B账户转账10000.需要执行两条update语句：
```sql
update t_act set balance = balance - 10000 where actno = 'act-001';
update t_act set balance = balance + 10000 where actno = 'act-002';
```
以上两条DML语句必须同时成功，或者同时失败，不允许出现一条成功，一条失败。

要想保证以上的两条DML语句同时成功或者同时失败，那么就需要使用数据库的“事务机制”。

## 2 3.2、和事务相关的语句只有：DML语句。（insert delete update）
为什么？因为它们这三个语句都是和数据库表当中的“数据”相关的。
事务的存在是为了保证数据的完整性，安全性。

## 3 3.3、假设所有的业务都能使用1条DML语句搞定，还需要事务机制吗？
不需要事务。
但实际情况不是这样的，通常一个“事务【业务】”需要多条DML语句共同联合完成。

## 4 3.4、事务的特性（ACID）
事务包括四大特性：ACID
A: 原子性：事务是最小的工作单元，不可再分。
C: 一致性：事务必须保证多条DML语句同时成功或者同时失败。
I：隔离性：事务A与事务B之间具有隔离。
D：持久性：持久性说的是最终数据必须持久化到硬盘文件中，事务才算成功的结束。

## 5 3.5、事务隔离级别
事务隔离性存在隔离级别，理论上隔离级别包括4个：

### 5.1 第一级别：读未提交（read uncommitted）
对方事务还没有提交，我们当前事务可以读取到对方未提交的数据。
读未提交存在脏读（Dirty Read）现象：表示读到了脏的数据。

### 5.2 第二级别：读已提交（read committed）
对方事务提交之后的数据我方可以读取到。
这种隔离级别解决了: 脏读现象没有了。
读已提交存在的问题是：不可重复读（同一事物中，两次读取同一行，数据可能不一致）。

### 5.3 第三级别：可重复读（repeatable read）
这种隔离级别解决了：不可重复读问题。
这种隔离级别存在的问题是：读取到的数据是幻象。

### 5.4 第四级别：序列化读/串行化读（serializable） 
解决了所有问题。
效率低。需要事务排队。

### 5.5 不同数据库的隔离级别
oracle数据库默认的隔离级别是：读已提交。
mysql数据库默认的隔离级别是：可重复读。
	
## 6 3.6、演示事务
* mysql事务默认情况下是自动提交的。
	（什么是自动提交？只要执行任意一条DML语句则提交一次。）怎么关闭自动提交？start transaction;

* 准备表：
```sql
drop table if exists t_user;
create table t_user(
	id int primary key auto_increment,
	username varchar(255)
);
```
* 演示：mysql中的事务是支持自动提交的，只要执行一条DML，则提交一次。
```sql
insert into t_user(username) values('zs');
Query OK, 1 row affected (0.03 sec)

mysql> select * from t_user;
+----+----------+
| id | username |
+----+----------+
|  1 | zs       |
+----+----------+
1 row in set (0.00 sec)

mysql> rollback;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from t_user;
+----+----------+
| id | username |
+----+----------+
|  1 | zs       |
+----+----------+
1 row in set (0.00 sec)
```
* 演示：使用start transaction;关闭自动提交机制。
```sql
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> insert into t_user(username) values('lisi');
Query OK, 1 row affected (0.00 sec)

mysql> select * from t_user;
+----+----------+
| id | username |
+----+----------+
|  1 | zs       |
|  2 | lisi     |
+----+----------+
2 rows in set (0.00 sec)

mysql> insert into t_user(username) values('wangwu');
Query OK, 1 row affected (0.00 sec)

mysql> select * from t_user;
+----+----------+
| id | username |
+----+----------+
|  1 | zs       |
|  2 | lisi     |
|  3 | wangwu   |
+----+----------+
3 rows in set (0.00 sec)

mysql> rollback;
Query OK, 0 rows affected (0.02 sec)

mysql> select * from t_user;
+----+----------+
| id | username |
+----+----------+
|  1 | zs       |
+----+----------+
1 row in set (0.00 sec)
--------------------------------------------------------------------
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> insert into t_user(username) values('wangwu');
Query OK, 1 row affected (0.00 sec)

mysql> insert into t_user(username) values('rose');
Query OK, 1 row affected (0.00 sec)

mysql> insert into t_user(username) values('jack');
Query OK, 1 row affected (0.00 sec)

mysql> select * from t_user;
+----+----------+
| id | username |
+----+----------+
|  1 | zs       |
|  4 | wangwu   |
|  5 | rose     |
|  6 | jack     |
+----+----------+
4 rows in set (0.00 sec)

mysql> commit;
Query OK, 0 rows affected (0.04 sec)

mysql> select * from t_user;
+----+----------+
| id | username |
+----+----------+
|  1 | zs       |
|  4 | wangwu   |
|  5 | rose     |
|  6 | jack     |
+----+----------+
4 rows in set (0.00 sec)

mysql> rollback;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from t_user;
+----+----------+
| id | username |
+----+----------+
|  1 | zs       |
|  4 | wangwu   |
|  5 | rose     |
|  6 | jack     |
+----+----------+
4 rows in set (0.00 sec)
```
* 演示两个事务，假如隔离级别
```sql
演示第1级别：读未提交
	set global transaction isolation level read uncommitted;
演示第2级别：读已提交
	set global transaction isolation level read committed;
演示第3级别：可重复读
	set global transaction isolation level repeatable read;
演示第4级别：序列化
	...
```
* mysql远程登录：mysql -h192.168.151.18 -uroot -p444