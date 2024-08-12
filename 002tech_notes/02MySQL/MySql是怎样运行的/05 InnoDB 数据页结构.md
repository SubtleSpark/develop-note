## 数据页中字段说明

|          名称          | 中文名       | Byte | 简单描述         |
| :------------------: | :-------- | ---: | :----------- |
|    `File Header`     | 文件头部      |   38 | 页的一些通用信息     |
|    `Page Header`     | 页面头部      |   56 | 数据页专有的一些信息   |
| `Infimum + Supremum` | 最小记录和最大记录 |   26 | 两个虚拟的行记录     |
|    `User Records`    | 用户记录      |  不确定 | 实际存储的行记录内容   |
|     `Free Space`     | 空闲空间      |  不确定 | 页中尚未使用的空间    |
|   `Page Directory`   | 页面目录      |  不确定 | 页中的某些记录的相对位置 |
|    `File Trailer`    | 文件尾部      |    8 | 校验页是否完整      |


## page demo 数据定义
```sql
CREATE TABLE page_demo(
    c1 INT,
    c2 INT,
    c3 VARCHAR(10000),
    PRIMARY KEY (c1)
) CHARSET=ascii ROW_FORMAT=Compact;

INSERT INTO page_demo
VALUES (1, 100, 'aaaa'),
       (2, 200, 'bbbb'),
       (3, 300, 'cccc'),
       (4, 400, 'dddd');

DELETE FROM page_demo WHERE c1 = 2;

```


