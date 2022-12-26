# mysql 备份恢复
## 1 官网示例
[MySQL :: MySQL 8.0 Reference Manual :: 4.6.9.3 Using mysqlbinlog to Back Up Binary Log Files](https://dev.mysql.com/doc/refman/8.0/en/mysqlbinlog-backup.html)

## 2 mysqldump + mysqlbinlog for Backup and Restore



## 3 开发 162 上的数据库配置

![[attachments/Pasted image 20220518103019.png|400]]
### 3.1 log-bin
![[attachments/Pasted image 20220518104147.png|500]]

### 3.2 log-bin-index
index文件内容是 log-bin 配置 + .000337 等序列号
![[attachments/Pasted image 20220518104209.png|500]]

### 3.3 binlog-expire-logs-seconds
超过这个时间点binlog会自动删除。
删除的时机：数据库启动时，或 binlog flush 时。
默认一个月，上面配置的7天。

### 3.4 sync_binlog
0 = 不同步写binlog， 依赖操作系统刷新到磁盘上。
1 = 事务提交前就会同步写 binlog 。恢复数据时，未提交的事物处于准备状态，可以回滚，保证不丢失任何已提交的事物。数据最安全，性能损耗最大。
n = n 次提交写一次 binlog。n越高性能损耗越小，数据越不安全。

>Controls how often the MySQL server synchronizes the binary log to disk.
>
> -   [`sync_binlog=0`](dfile:///Users/jun/Library/Application%20Support/Dash/DocSets/MySQL/MySQL.docset/Contents/Resources/Documents/replication.html#sysvar_sync_binlog): Disables synchronization of the binary log to disk by the MySQL server. Instead, the MySQL server relies on the operating system to flush the binary log to disk from time to time as it does for any other file. This setting provides the best performance, but in the event of a power failure or operating system crash, it is possible that the server has committed transactions that have not been synchronized to the binary log.
> -   [`sync_binlog=1`](dfile:///Users/jun/Library/Application%20Support/Dash/DocSets/MySQL/MySQL.docset/Contents/Resources/Documents/replication.html#sysvar_sync_binlog): Enables synchronization of the binary log to disk before transactions are committed. This is the safest setting but can have a negative impact on performance due to the increased number of disk writes. In the event of a power failure or operating system crash, transactions that are missing from the binary log are only in a prepared state. This permits the automatic recovery routine to roll back the transactions, which guarantees that no transaction is lost from the binary log.
> -   [``sync_binlog=_`N`_``](dfile:///Users/jun/Library/Application%20Support/Dash/DocSets/MySQL/MySQL.docset/Contents/Resources/Documents/replication.html#sysvar_sync_binlog), where _`N`_ is a value other than 0 or 1: The binary log is synchronized to disk after `N` binary log commit groups have been collected. In the event of a power failure or operating system crash, it is possible that the server has committed transactions that have not been flushed to the binary log. This setting can have a negative impact on performance due to the increased number of disk writes. A higher value improves performance, but with an increased risk of data loss.

## 4 binlog 恢复成 sql
```shell
mysqlbinlog --base64-output=DECODE-ROWS -v mysql-bin.000003
```

## 5 binlog 闪回
### 5.1 闪回工具
[美团MyFlash](https://tech.meituan.com/2017/11/17/mysql-flashback.html)





