# 基于 spring 配置多数据源
## 1 spring 注入数据源原理





![[attachments/spring 多数据源原理.svg]]
### 1.1 
当调用 orm 框架 crud 的 API 时，会从 spring 容器中取得注入的 DataSource 对象，然后调用这个对象的 getConnection() 方法获得连接。通过这个连接执行CRUD。

