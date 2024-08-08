![[Excalidraw/InnoDB存储结构 2024-08-08 22.52.11.excalidraw|InnoDB存储结构 2024-08-08 22.52.11.excalidraw]]
## 基本认识
Page 是 MySQL 处理数据的基本单位
- 系统变量 innodb_page_size 设置了一个 page 的大小
- 默认 16284 即 16KB
- 只有mysql 初始化时可以修改，之后就再也不能改了



## InnoDB 行格式
### COMOACT 行格式
#### 变长字段列表
类似 `varchar(512)`

