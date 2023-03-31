# 一、DDL(create ,drop,alter)
## `create`
```sql
create table 表名(
	字段名1 数据类型,
	字段名2 数据类型,
	字段名3 数据类型,
	....
);
```

- 表名在数据库当中一般建议以：t_或者tbl_开始。

### 常见类型
常用类型 | 描述
:---- | :----
**`int`** | 整数型(java中的int)
**`bigint`** | 长整型(java中的long)
**`float`** | 浮点型(java中的float double)
**`char`** | **定长**字符串(String)
**`varchar`** | **可变长**字符串(StringBuffer/StringBuilder)
**`date`** | 日期类型 （对应Java中的java.sql.Date类型。1986-10-23
`BLOB` | 二进制大对象（存储图片、视频等流媒体信息） Binary Large OBject （对应java中的Object）
`CLOB` | 字符大对象（存储较大文本，比如，可以存储4G的字符串。） Character Large OBject（对应java中的Object）

### char和varchar怎么选择？
- 当某个字段中的数据长度不发生改变的时候，是定长的，例如：性别、生日等都是采用char。
- 当一个字段的数据长度不确定，例如：简介、姓名等都是采用varchar。

### 表的复制
```sql
create table 表名 as select语句;
create table dept1 as select * from dept;
```
- 将查询结果当做表创建出来。

## `drop`
```sql
drop table 表名; // 这个通用。
drop table if exists 表名; // oracle不支持这种写法。
```

## `alter`
- 实际开发中很少发生。
- 使用工具完成较为稳妥，也足够应用了。


---
# 二、DML(insert,delete,update)
## `insert`
```sql
insert into 表名(字段名1,字段名2,字段名3,....) values(值1,值2,值3,....)
```
- 字段的数量和值的数量相同，并且数据类型要对应相同。
- 是`values`
```sql
insert into t_student values(1,'jack','0','gaosan2ban','1986-10-23');
```
- 字段可以省略不写，但是后面的value对数量和顺序都有要求。
### 一次插入多行数据
```sql
insert into t_student
	(no,name,sex,classno,birth) 
values
	(3,'rose','1','gaosi2ban','1952-12-14'),
	(4,'laotie','1','gaosi2ban','1955-12-14');
```
- 逗号隔开，括号括起来。
###  将查询结果插入到一张表中
```sql
insert into dept1 select * from dept;
```
- 查询出来的字段要能对上。

## `delete` & `truncate`
```sql
delete from 表名 where 条件;

# 怎么删除大表中的数据？（重点）
truncate table 表名; // 表被截断，不可回滚。永久丢失。
```
- `delete`删除效率较低，但较为安全。
- 用`truncate`再三小心，表被截断，不可回滚，永久丢失。并且**自增列归零**。

## `update`
```sql
update 表名 set 字段名1=值1,字段名2=值2... where 条件;
update dept1 set loc = 'SHANGHAI', dname = 'RENSHIBU' where deptno = 10;
```
- `set`多个字段之间是**逗号**，不是and。
- 不加`where`更新所有记录。

---
# 三、约束
1. 非空约束(`not null`)：约束的字段不能为NULL
2. 唯一约束(`unique`)：约束的字段不能重复，可以为空
3. 主键约束(`primary key`)：约束的字段**既不能为NULL**，**也不能重复**（简称PK）
4. 外键约束(`foreign key`)：...（简称FK）
5. 检查约束(`check`)：注意Oracle数据库有check约束，但是mysql没有，目前mysql不支持该约束。
## 1. 非空约束 (`not null`)
```sql
drop table if exists t_user;
create table t_user(
	id int,
	username varchar(255) not null,
	password varchar(255)
);
insert into t_user(id,password) values(1,'123');
ERROR 1364 (HY000): Field 'username' doesn't have a default value
```
* 注意：not null约束只有列级约束。没有表级约束。
---
## 2. 唯一性约束(`unique`)
	
* 唯一约束修饰的字段具有唯一性，不能重复。但可以为NULL。
* 案例：给某一列添加`unique`【列级约束】
```sql
	drop table if exists t_user;
	create table t_user(
		id int,
		username varchar(255) unique  // 列级约束
	);
	insert into t_user values(1,'zhangsan');
	insert into t_user values(2,'zhangsan');
	ERROR 1062 (23000): Duplicate entry 'zhangsan' for key 'username'
```
* 案例：给两个列或者多个列添加`unique`【表级约束】
```sql
	drop table if exists t_user;
	create table t_user(
		id int, 
		usercode varchar(255),
		username varchar(255),
		unique(usercode,username) // 多个字段联合起来添加1个约束unique 【表级约束】
	);

	insert into t_user values(1,'111','zs');
	insert into t_user values(2,'111','ls');
	insert into t_user values(3,'222','zs');
	select * from t_user;
	insert into t_user values(4,'111','zs');
	ERROR 1062 (23000): Duplicate entry '111-zs' for key 'usercode'

	drop table if exists t_user;
	create table t_user(
		id int, 
		usercode varchar(255) unique,
		username varchar(255) unique
	);
	insert into t_user values(1,'111','zs');
	insert into t_user values(2,'111','ls');
	ERROR 1062 (23000): Duplicate entry '111' for key 'usercode'
```
---
## 3. 主键约束(`primary key`)
* 一张表的主键约束只能有1个。（必须记住）
### 添加主键
####  列级约束添加主键
```sql
create table t_user(
	id int primary key,  // 列级约束
	username varchar(255),
	email varchar(255)
);
```

#### 使用表级约束方式定义主键：
```sql
create table t_user(
	id int,
	username varchar(255),
	primary key(id)
);

# 以下内容是演示以下复合主键，不需要掌握：
create table t_user(
	id int,
	username varchar(255),
	password varchar(255),
	primary key(id,username) # 复合主键
);
insert .......
```

### 主键相关的术语？
>主键约束 : primary key
>主键字段 : id字段添加primary key之后，id叫做主键字段
>主键值 : id字段中的每一个值都是主键值。
	
### 主键有什么作用？
> - 主键的作用：主键值是这行记录在这张表当中的**唯一标识**。（就像一个人的身份证号码一样。）
> - 索引

### 主键的分类？

> **根据主键字段的字段数量来划分：**
> - **单一主键**（推荐的，常用的。）
> - **复合主键**(多个字段联合起来添加一个主键约束)（复合主键不建议使用，因为复合主键违背三范式中的第二范式。）

> **根据主键性质来划分：** 
> - **逻辑主键（代理主键）**：主键值最好就是一个和业务**没有任何关系**的自然数。（这种方式是推荐的）
> - **业务主键（自然主键）**：主键值和系统的业务挂钩，例如：拿着银行卡的卡号做主键。（不推荐用）

- **最好不要拿着和业务挂钩的字段作为主键**。因为以后的业务一旦发生改变的时候，主键值可能也需要随着发生变化。

### mysql提供主键值自增`auto_increment`：（非常重要。）
```sql
drop table if exists t_user;
create table t_user(
	id int primary key auto_increment, // id字段自动维护一个自增的数字，从1开始，以1递增。
	username varchar(255)
);
```
-	提示：Oracle当中也提供了一个自增机制，叫做：序列（sequence）对象。
---
## 4. 外键约束(`foreign key`)：重难点
* 外键值可以为NULL
* 外键字段引用其他表的某个字段的时候，被引用的字段必须是主键吗？
		注意：被引用的字段不一定是主键，但至少具有**unique**约束。
### 顺序要求：
>	1. 删除数据的时候，先删除子表，再删除父表。
>	2. 添加数据的时候，先添加父表，在添加子表。
>	3. 创建表的时候，先创建父表，再创建子表。
>	4. 删除表的时候，先删除子表，在删除父表。

### 创建外键约束
```sql
create table t_class(
	cno int,
	cname varchar(255),
	primary key(cno)
);

create table t_student(
	sno int,
	sname varchar(255),
	classno int,
	primary key(sno),
	foreign key(classno) references t_class(cno)
);
```