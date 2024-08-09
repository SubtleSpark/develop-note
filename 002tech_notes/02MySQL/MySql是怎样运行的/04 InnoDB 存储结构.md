![[Excalidraw/InnoDB存储结构 2024-08-08 22.52.11.excalidraw|InnoDB存储结构 2024-08-08 22.52.11.excalidraw]]
## 1 基本认识
Page 是 MySQL 处理数据的基本单位
- 系统变量 innodb_page_size 设置了一个 page 的大小
- 默认 16284 即 16KB
- 只有mysql 初始化时可以修改，之后就再也不能改了

## 2 InnoDB 行格式
### 2.1 record_format_demo 数据定义
```sql
CREATE TABLE t (
    c1 VARCHAR(10),
    c2 VARCHAR(10) NOT NULL,
    c3 CHAR(10),
    c4 VARCHAR(10)
) CHARSET=ascii ROW_FORMAT=COMPACT;

/*
插入数据

+------+-----+------+------+
| c1   | c2  | c3   | c4   |
+------+-----+------+------+
| aaaa | bbb | cc   | d    |
| eeee | fff | NULL | NULL |
+------+-----+------+------+

*/
```

### 2.2 COMOACT 行格式
#### 2.2.1 变长字段列表
类似 `varchar(512)`、`TEXT`、`BLOB` 这种变长类型。对应的字段存储的字节数是不固定的。存储时，需要将这些字段占用的字节数也存起来。
COMOACT 行格式中，各变长字段的字节数存储在头部，形成一个变长字段长度列表，并且按照列的顺序**逆序存放**。
> 每个长度的存储格式类似 [packed_integer](https://dev.mysql.com/doc/dev/mysql-server/latest/classmysql_1_1binlog_1_1event_1_1Binary__log__event.html#packed_integer) ，也是变长，但具体规则不同。
> 当字段的最大可能的长度（如C1：`10 * 2 = 20`）

#### NULL 值列表

