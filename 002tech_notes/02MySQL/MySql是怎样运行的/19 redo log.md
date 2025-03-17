# MySQL Redo Log 详解

## 1. 为什么需要 redo log

- Buffer Pool 的设计是为了提高读写效率，但会带来数据丢失的风险
- 事务提交后，所修改的页面可能仍在 Buffer Pool 中，未刷新到磁盘
- 系统崩溃可能导致内存数据丢失
- Redo Log 通过先写日志再修改页面的方式（WAL），来保证数据的持久性

## 2. redo log 的基本概念

### 2.1 redo log 的格式
- 物理日志：记录对页面的物理修改
- 格式：表空间号 + 页号 + 偏移量 + 修改数据
- Mini-Transaction (mtr)：一个最小的逻辑操作单位
  - 一个mtr可以包含多条redo日志
  - 具有原子性：要么全部完成，要么全部不完成

### 2.2 redo log 的写入过程
1. 事务开始时，分配一个 LSN（Log Sequence Number）
2. 修改Buffer Pool中的页面
3. 生成redo日志，写入redo log buffer
4. 根据innodb_flush_log_at_trx_commit参数决定写入时机：
   - 0：每秒写入磁盘，可能丢失1秒数据
   - 1：事务提交时写入磁盘（默认）
   - 2：事务提交时写入操作系统缓存

## 3. redo log 的存储结构

### 3.1 redo log block
- 大小固定为512字节
- 包含头部、尾部和数据部分
- 头部12字节，尾部8字节，数据部分492字节

### 3.2 redo log file
- 默认存在ib_logfile0和ib_logfile1两个文件
- 形成环形结构，循环写入
- 由参数innodb_log_file_size控制单个文件大小
- 由参数innodb_log_files_in_group控制文件个数

## 4. checkpoint机制

### 4.1 为什么需要checkpoint
- redo log空间有限，需要循环使用
- 需要确保覆盖的redo log对应的脏页已经刷新到磁盘

### 4.2 checkpoint的触发条件
1. Sharp Checkpoint：关闭数据库时
2. Fuzzy Checkpoint：
   - Master Thread Checkpoint：主线程周期性触发
   - FLUSH_LRU_LIST Checkpoint：LRU列表可用页面不足
   - Async/Sync Flush Checkpoint：redo log空间不足
   - Dirty Page too much Checkpoint：脏页太多

## 5. redo log的恢复机制

### 5.1 崩溃恢复流程
1. 从最后一个checkpoint开始扫描redo log
2. 重放redo log，将页面恢复到崩溃前的状态
3. 对于已经刷新到磁盘的页面，忽略相应的redo log

### 5.2 恢复策略
- 并行恢复：多个线程同时应用redo log
- 预读取：将需要修改的页面提前读入内存
- 跳过已经刷新的页面

## 6. 性能优化建议

1. 合理配置redo log文件大小
2. 适当调整innodb_flush_log_at_trx_commit参数
3. 注意监控checkpoint位置和redo log使用情况
4. 避免产生大量的redo log：
   - 合理控制事务大小
   - 避免频繁的小事务
   - 适当批量处理数据
