根据powernode老杜Mysql课程整理（上方是代码，下方无序列表是解释）



# MySql-DQL笔记
# 一、单表查询

## 1 简单查询

```sql
# 查询员工姓名和年薪（月薪*12）
select ename, month_sal * 12 as year_sal from emp;
select ename, month_sal * 12 'year_sal' from emp;

select * from emp;
```

 - as 可以给列取别名, as 可以省略。
 - MySql可以用**双引号**也可以用**单引号**，但其他数据库不识别双引号。所以建议用**单引号**。
 - 列可以参与运算（month_sal * 12）
 - 实际开发不建议使用 select *
	
---
## 2 条件查询
```sql
select 字段, ...
from 表名
where 条件;
# 先执行from， 再where， 最后select

select sal from emp where ename = 'smith';
select ename from emp where sal <> 3000;
```

 - 先执行from， 再where， 最后select
 - 字符串用单引号括起来。
 - 查询的字符串也**不区分**大小写
 - 相等用`=`而不是`==`
 - 关于**不等**，最初的数据库标准是用`<>`，后来大部分数据库也支持了`!=`。

### 2.1 between... and...
```sql
select ename from emp where sal >= 1100 and sal <= 3000;
select ename from emp where sal between 1100 and 3000; 	#闭区间

# 用在字符串上，左闭右开。
# 会出现a，b，c开头的字符串。
select ename from emp where ename between 'A' and 'D'; 
```


 - 关于between ... and ... 
 	- 用在数字上是闭区间。
 	- **==一定左边数字小，右边数字大==**。
 	- 除了用在数字上，也可以用在字符串上。


### 2.2 NULL
```sql
select ename from emp where comm = null;	#错误方式
select ename from emp where comm is null;
select ename from emp where comm is not null;
```
- `null`不是值，不能用`=`衡量，要用`is null`或`is not null`


### 2.3 in & not in
```sql
select * from emp where job in('SALESMAN', 'MANAGER');
select * from emp where sal in(1000, 5000);
```
- **in 用圆括号。**
- in 等同于or。
- in 后面的值不是区间,是具体的值。

## 3 模糊查询 like?
```sql
# 找出所有名字中带 o 的人
select * from emp where ename like '%o%';

# 找出名字中有下划线的
select * from emp where ename like '%\_%';

```
- `%`代表任意多个字符，`_`代表一个任意字符
- `\`用来转义。

---
## 4 排序 order by
```sql
# 默认升序排列
select ename, sal from EMP order by sal;
select ename, sal from EMP order by sal desc;

# 按照第二列(sal)排序，(不鲁棒，程序中不许这样写)
select ename, sal from EMP order by 2;

# 按工资的 降序排列，当工资相同时按名字升序排列
select ename, sal from EMP order by sal desc, ename asc;
```
- `asc `升序、`desc`降序
-  缺省`asc `、`desc`时默认升序。
- 多字段同时排序，**前面的字段起的作用大**(只有前面的字段相等时，后面的字段才起作用)。
```sql
select ename, sal as salary			# 3
from EMP							# 1
where sal > 2000					# 2
order by salary;					# 4
```
- `where`中必须用 sal，`order`中可以用 sal 或salary。体现了执行顺序。


---
## 5 分组查询
>sql执行顺序
```sql
select..	5
from ..		1
where ..	2
group by ..	3
having ..	4
order by ..	6
```
### 5.1 分组函数（多行处理函数，聚合函数）: 输入多行，输出一行

函数| 作用
:-------- | :-----
`count(...)`|取得记录数
`sum(...)`|求和
`avg(...)`|取平均
`max(...)`|取最大的数
`min(...)`|取最小的数

- **所有分组函数嗾使对“某一组”数据进行操作的**。
- 分组函数一共5个。
- 多行处理函数: 输入多行，输出一行。
- **分组函数自动忽`NULL`**

```sql
# ERROR 1111 (HY000): Invalid use of group function
select ename, sal from EMP where sal > avg(sal);
```
- 分组函数不可直接出现在where子句中。**原因：分组函数在执行group by之后才会执行，group by在where之后执行。**


### 5.2 group by
- group by：按照某个字段或者某些字段进行分组。

```sql
select max(sal) from EMP group by job;
```
- 分组函数通常和group by联合使用。
- 分组函数在执行group by之后才会执行。

```sql
select ename, job, max(sal) from EMP group by job;
```
- 以上在低版本MySQL上可以执行，但查出来的`ename`字段没有意义。
- 在高版本MySQL和Oracle中会报错。
-  **当一条语句中有`group by`试，select后面只能跟 ==分组函数== 和 ==参与分组的字段==**

```sql
# 联合分组
# 找出每个部门 不同工作岗位的最高薪资
select deptno, job, max(sal)
from EMP
group by deptno, job;
```

### 5.3 having
- having：对分组后的数据再次过滤。
```sql
# 找出每个部门的最高薪资,要求显示薪资大于2500的数据

# 第一步:找出每个部门的最高薪资
select max(sal),deptno from emp group by deptno;

# 第二部：找出薪资大于2900的（效率低）
select max(sal),deptno from EMP group by deptno having max(sal) > 2900;

# 更好写法where提前过滤
select max(sal),deptno from emp where sal > 2900 group by deptno;


# 找出每个部门的平均薪资,要求显示薪资大于2000的数据
select deptno, avg(sal) avg_sal from EMP group by deptno having avg_sal > 2000;

```
- 建议能用`where`过滤的尽量使用`where`提前过滤


## 6 单行处理函数：输入一行，输出一行
函数 | 作用
:-------- | :-----
`ifnull(可能为空的字段, 被当作什么值)` | 处理空值

```sql
# 如果comm为NULL，yearsal 就为 NULL
select ename, (sal + comm) * 12 as yearsal from EMP;

select ename, (sal + ifnull(comm, 0)) * 12 as yearsal from EMP;

```
- 单行处理函数：输入一行，输出一行。
- **表达式中只要有`NULL`运算结果就为`NULL`**（所有数据库都是这样规定）。

## 7 结果去重`distinct`
```sql
SELECT DISTINCT JOB FROM emp;			#　正确语法

SELECT ENAME, DISTINCT JOB FROM emp;	# 错误，`distinct`只能出现在所有字段的最前面

SELECT DISTINCT ENAME, JOB FROM emp;	# 后面所有字段联合起来去重
```
 - `distinct`只能出现在所有字段的最前面，两列数据的条数要一样。
 - `distinct`加在最前面，后面所有字段联合起来去重。

# 二、连接查询
- 内连接
	- 等值连接
	- 非等值连接
	- 自连接
- 外连接
	- 左连接
	- 右连接
- 全连接（很少用）

---
## 8 内连接
### 8.1 等值连接
```sql
# SQL92写法，太老不用
select ename, dname from emp, dept where emp.deptno = dept.deptno;

# SQL99 常用
select e.ename, d.dname
from emp e
inner join dept d
on e.deptno = d.deptno;
```
- 99语法比92语法的优势：SQL结构更清晰，表的连接条件和where的过滤条件分离了。
- `inner`是默认的，可以省略，带着可读性更好。

### 8.2 非等值连接
```sql
# 找出每个员工的工资等级,要求显示员工名、工资、工资等级
select e.ename, e.sal, s.grade
from emp e
join salgrade s
on e.sal between s.losal and s.hisal;
```
- 连接条件不是等式

### 8.3 自连接
```sql
# 找出每个员工的上级领导，显示员工名对应的领导名
select a.ename '员工', b.ename '领导' from emp a join emp b on a.mgr = b.empno;
```
- 一张表当两张表，自己连接自己。

---
## 9 外连接
### 9.1 什么是外连接，和内连接有什么区别?
内连接：
假设A和B表进行连接，使用内连接的话，凡是A表和B表能够匹配上的记录查询出来，这就是内连接B两张表**没有主副之分**，两张表是平等的。

外连接：
假设A和B表进行连接，使用外连接的话,AB两张表中有一张表是主表，一张表是副表，主要查询主表中数据，捎带着查询副表，当副表中的数据没有和主表中的数据匹配上，副表**自动模拟出NULL**与之匹配。
### 9.2 左外连接&右外连接
```sql
select a.ename '员工', b.ename '领导' from emp a left join emp b on a.mgr = b.empno;

select a.ename '员工', b.ename '领导' from emp b right join emp a on a.mgr = b.empno;
```
- 所有的左外连接都有对应的右外连接的写法。



---
## 10 全连接（很少用）
略...

---
## 11 三张表的连接查询
```sql
# A表和B表先进行表连接,连接之后A表继续和c表进行连接。
...
A
join
B
on
...
join
C
on
...

# 找出每一个员工的部门名称以及工资等级。
select e.ename, d.dname, s.grade
from emp e
left join salgrade s on e.sal between s.losal and s.hisal
left join dept d on d.deptno = e.deptno;

# 找出每一个员工的部门名称、工资等级、以及上级领导
select e.ename '员工', s.grade '薪资等级', b.ename '领导'
from emp e
left join emp b on e.mgr = b.empno
left join salgrade s on e.sal between s.losal and s.hisal;
```
---
# 三、子查询
## 12 where子查询
```sql
# 找出薪资高于平均薪资的人
SELECT * FROM emp WHERE SAL > (SELECT AVG(SAL) FROM emp);
```
- 子查询一定加括号，否则会报错。



## 13 from子查询
```sql
# 找出每个部门平均薪资的水平
SELECT
	DEPTNO,
	s.GRADE 
FROM
	( SELECT DEPTNO, AVG( SAL ) avg_sal FROM emp GROUP BY DEPTNO ) t
JOIN salgrade s ON t.avg_sal BETWEEN s.LOSAL AND s.HISAL;
```
- 每个派生表必须有自己的别名



## 14 select子查询（不常用）
```sql
# 找出每个员工所在的部门名称,要求显示员工名和部门名。
SELECT e.ename,
(SELECT d.dname FROM dept d WHERE d.DEPTNO = e.DEPTNO) dname
FROM emp e;
```

---
# 四、union
```sql
# 查询job是'SALESMAN'或'MANAGER'的员工
# 1
SELECT * FROM emp e WHERE e.JOB in ('SALESMAN', 'MANAGER');
# 2
SELECT * FROM emp e WHERE e.JOB = 'SALESMAN'
UNION
SELECT * FROM emp e WHERE e.JOB = 'MANAGER';

# 能将两个不相干的表中的数据拼接在一起显示，列名是第一个名字（ENAME）
SELECT ENAME FROM emp
UNION
SELECT DNAME FROM dept
```
- 能将两个不相干的表中的数据拼接在一起显示，列名是第一个名字（ENAME）。
- 两个查询的列数要一样。


---
# 五、limit（MySQL特有）
```sql
SELECT ENAME, SAL FROM emp ORDER BY SAL DESC LIMIT 0, 5;
SELECT ENAME, SAL FROM emp ORDER BY SAL DESC LIMIT 5;
```
* 语法机制：limit startIndex, length。
* startIndex表示起始位置，从0开始，0表示第一条数据
length表示取几个
* `limit`后只写一个数字，默认startIndex为0。

## 15 通用的标准分页sql
每页显示`pageSize`条记录：
第`pageNo`页：(`pageNo` - 1) * `pageSize`, `pageSize`

---
# 六、一个完整的DQL语句
```sql
select			5
	...
from			1
	...		
where			2
	...	
group by		3
	...
having			4
	...
order by		6
	...
limit			7
	...;
```
