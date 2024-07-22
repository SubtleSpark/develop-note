## 1 参考资料

| 来源方           | 文档地址                                                                                          | 备注  |     |
| ------------- | --------------------------------------------------------------------------------------------- | --- | --- |
| mysql develop | https://dev.mysql.com/doc/dev/mysql-server/latest/page_protocol_replication_binlog_event.html |     |     |
|               |                                                                                               |     |     |
| mysql server  | https://dev.mysql.com/doc/refman/8.0/en/mysqlbinlog.html                                      |     |     |
| mysql server  | https://dev.mysql.com/doc/refman/8.4/en/mysqlbinlog-row-events.html                           |     |     |
|               |                                                                                               |     |     |
| 第三方           | https://www.cnblogs.com/softidea/p/12624778.html                                              |     |     |

## 2 部分字段说明
### 2.1 table map event
https://dev.mysql.com/doc/dev/mysql-server/latest/classmysql_1_1binlog_1_1event_1_1Table__map__event.html

根据文档中 Detailed Description 作出如下示意图。固定长度部分  8 字节，剩下字节如图所示。

m_null_bits:  四舍五入到最接近的整数字节数，例如 13 个字段时，占用 2 byte ，但只有前 13 bit 有效
TVL format: Type, Length, Value(TLV) format. Type 占用 1 个字节。Length 是 packed integer。Value 占用 Length 字节。
packed integer: [packed integer 描述](https://dev.mysql.com/doc/dev/mysql-server/latest/classmysql_1_1binlog_1_1event_1_1Binary__log__event.html#packed_integer)

```
The buffer layout for fixed data part is as follows:
+-----------------------------------+
| table_id | Reserved for future use|
+-----------------------------------+
|   6B     |          2B            |
+-----------------------------------+

The buffer layout for variable data part is as follows:
+------------------------------------------------------------------------------+
| db len| db name | table len| table name | no of cols     | array of col types|
+------------------------------------------------------------------------------+
|   1B  |  db len |    1B    | table len  | packed integer |     1B * cols     |
+------------------------------------------------------------------------------+

+-------------------------------------------------------------------------------------------+
| metadata len   | metadata block | m_null_bits                  | optional metadata fields |
+-------------------------------------------------------------------------------------------+
| packed integer | metadata len   | (1b * cols) round up to byte |     TLV format           |
+-------------------------------------------------------------------------------------------+
```


#### 2.1.1 table map 中 metadata 部分字段只有在 binlog_row_matadata=FULL 时才有值。
![[attachments/Pasted image 20240721183235.png]]

### 2.2 增删改 binlog
参考 :  https://dev.mysql.com/doc/dev/mysql-server/latest/classmysql_1_1binlog_1_1event_1_1Rows__event.html





## 3 other
https://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA==&mid=2651747096&idx=1&sn=e561ce9254f69c3fcd04c73b44e56e69&chksm=bd12aa558a65234332298e1d611da408ec45afb2d08ecd22078a600379525e75b2fa13149fb0&mpshare=1&scene=23&srcid=1116o1gBWoNf47Rv1E1ngYir#rd

文件中:              0x 8594 ad59
大小端处理后:    0x 59ad 9485
转换10进制:       1504547973
对应时间:           2017-9-5  01:59:33

