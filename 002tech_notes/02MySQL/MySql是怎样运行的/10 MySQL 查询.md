## 单表访问方法



| 访问方法  | 描述 |
| ----- | ---- | 
| const | 通过 **主键** or **唯一索引** 与常数等值比较，如 select 1 from table where primary_key=1 |
| ref | 通过 **非唯一索引** 与常数等值比较，形成单点扫描区间，如 select 1 from table where key=1 |




在没有开启 MRR （Multi-Range Read）的情况下，

## 连接查询



## 查询成本


