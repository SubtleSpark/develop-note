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
![[attachments/Pasted image 20240721183235.png]]
https://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA==&mid=2651747096&idx=1&sn=e561ce9254f69c3fcd04c73b44e56e69&chksm=bd12aa558a65234332298e1d611da408ec45afb2d08ecd22078a600379525e75b2fa13149fb0&mpshare=1&scene=23&srcid=1116o1gBWoNf47Rv1E1ngYir#rd

文件中:              0x 8594 ad59
大小端处理后:    0x 59ad 9485
转换10进制:       1504547973
对应时间:           2017-9-5  01:59:33

