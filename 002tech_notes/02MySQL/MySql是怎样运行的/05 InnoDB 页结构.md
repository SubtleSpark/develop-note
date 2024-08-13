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

## Infimum、 Supremum、 User Records
- Infimum、 Supremum 是 innodb 自动添加的伪记录。
- next record 串联的链表会跳过被删除的记录。
- 被删除的记录会被标记为删除，但是不会立即释放空间。
- 所有被删除掉的记录都会组成一个所谓的垃圾链表，在这个链表中的记录占用的空间称之为所谓的可重用空间，之后如果有新记录插入到表中的话，可能把这些被删除的记录占用的存储空间覆盖掉。


## Page Directory（页目录）
### 结构
Page Directory 是一个数组，数组中的每个元素称为一个槽(slot)。
- 将页中所有记录按主键顺序排列后，可以将连续的几个记录组成一组。
- 每个槽存储的是每组中**最大的记录**的**真实数据部分**，相对于页的起始位置的偏移量。


### 页目录记录规则
Page Directory 和 用户记录组成了一个跳表。遵循以下的规则，随着用户记录的增长而变化。
- 对于最小记录所在的分组只能有  *1* 条记录，最大记录所在的分组拥有的记录条数只能在 *1~8* 条之间，剩下的分组中记录的条数范围只能在是 *4~8* 条之间。所以分组是按照下面的步骤进行的
- 初始情况下一个数据页里只有最小记录和最大记录两条记录，它们分属于两个分组。
- 之后每插入一条记录，都会从`页目录`中找到主键值比本记录的主键值大并且差值最小的槽，然后把该槽对应的记录的`n_owned`值加1，表示本组内又添加了一条记录，直到该组中的记录数等于8个。
- 在一个组中的记录数等于8个后再插入一条记录时，会将组中的记录拆分成两个组，一个组中4条记录，另一个5条记录。这个过程会在`页目录`中新增一个`槽`来记录这个新增分组中最大的那条记录的偏移量。


## Page Header

| 名称                  | 占用空间大小 | 描述                                                                 |
| :------------------ | :----: | :----------------------------------------------------------------- |
| `PAGE_N_DIR_SLOTS`  | `2`字节  | 在页目录中的槽数量                                                          |
| `PAGE_HEAP_TOP`     | `2`字节  | 还未使用的空间最小地址，也就是说从该地址之后就是`Free Space`                               |
| `PAGE_N_HEAP`       | `2`字节  | 本页中的记录的数量（包括最小和最大记录以及标记为删除的记录）                                     |
| `PAGE_FREE`         | `2`字节  | 第一个已经标记为删除的记录地址（各个已删除的记录通过`next_record`也会组成一个单链表，这个单链表中的记录可以被重新利用） |
| `PAGE_GARBAGE`      | `2`字节  | 已删除记录占用的字节数                                                        |
| `PAGE_LAST_INSERT`  | `2`字节  | 最后插入记录的位置                                                          |
| `PAGE_DIRECTION`    | `2`字节  | 记录插入的方向                                                            |
| `PAGE_N_DIRECTION`  | `2`字节  | 一个方向连续插入的记录数量                                                      |
| `PAGE_N_RECS`       | `2`字节  | 该页中记录的数量（不包括最小和最大记录以及被标记为删除的记录）                                    |
| `PAGE_MAX_TRX_ID`   | `8`字节  | 修改当前页的最大事务ID，该值仅在二级索引中定义                                           |
| `PAGE_LEVEL`        | `2`字节  | 当前页在B+树中所处的层级                                                      |
| `PAGE_INDEX_ID`     | `8`字节  | 索引ID，表示当前页属于哪个索引                                                   |
| `PAGE_BTR_SEG_LEAF` | `10`字节 | B+树叶子段的头部信息，仅在B+树的Root页定义                                          |
| `PAGE_BTR_SEG_TOP`  | `10`字节 | B+树非叶子段的头部信息，仅在B+树的Root页定义                                         |

**PAGE_DIRECTION**
假如新插入的一条记录的主键值比上一条记录的主键值大，我们说这条记录的插入方向是右边，反之则是左边。用来表示最后一条记录插入方向的状态就是`PAGE_DIRECTION`

**PAGE_N_DIRECTION**
假设连续几次插入新记录的方向都是一致的，`InnoDB`会把沿着同一个方向插入记录的条数记下来，这个条数就用`PAGE_N_DIRECTION`这个状态表示。当然，如果最后一条记录的插入方向改变了的话，这个状态的值会被清零重新统计。


## File Header（文件头部）

| 名称                                 | 占用空间大小 | 描述                                          |
| :--------------------------------- | :----: | :------------------------------------------ |
| `FIL_PAGE_SPACE_OR_CHKSUM`         | `4`字节  | 页的校验和（checksum值）                            |
| `FIL_PAGE_OFFSET`                  | `4`字节  | 页号。`InnoDB`通过页号来可以唯一定位一个`页`。                |
| `FIL_PAGE_PREV`                    | `4`字节  | 上一个页的页号                                     |
| `FIL_PAGE_NEXT`                    | `4`字节  | 下一个页的页号                                     |
| `FIL_PAGE_LSN`                     | `8`字节  | 页面被最后修改时对应的日志序列位置（英文名是：Log Sequence Number） |
| `FIL_PAGE_TYPE`                    | `2`字节  | 该页的类型                                       |
| `FIL_PAGE_FILE_FLUSH_LSN`          | `8`字节  | 仅在系统表空间的一个页中定义，代表文件至少被刷新到了对应的LSN值           |
| `FIL_PAGE_ARCH_LOG_NO_OR_SPACE_ID` | `4`字节  | 页属于哪个表空间                                    |

### FIL_PAGE_TYPE 枚举

| 类型名称                      |  十六进制  | 描述                |
| :------------------------ | :----: | :---------------- |
| `FIL_PAGE_TYPE_ALLOCATED` | 0x0000 | 最新分配，还没使用         |
| `FIL_PAGE_UNDO_LOG`       | 0x0002 | Undo日志页           |
| `FIL_PAGE_INODE`          | 0x0003 | 段信息节点             |
| `FIL_PAGE_IBUF_FREE_LIST` | 0x0004 | Insert Buffer空闲列表 |
| `FIL_PAGE_IBUF_BITMAP`    | 0x0005 | Insert Buffer位图   |
| `FIL_PAGE_TYPE_SYS`       | 0x0006 | 系统页               |
| `FIL_PAGE_TYPE_TRX_SYS`   | 0x0007 | 事务系统数据            |
| `FIL_PAGE_TYPE_FSP_HDR`   | 0x0008 | 表空间头部信息           |
| `FIL_PAGE_TYPE_XDES`      | 0x0009 | 扩展描述页             |
| `FIL_PAGE_TYPE_BLOB`      | 0x000A | BLOB页             |
| `FIL_PAGE_INDEX`          | 0x45BF | 索引页，也就是我们所说的`数据页` |
