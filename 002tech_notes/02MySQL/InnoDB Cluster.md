# InnoDB Cluster
官方文档：
[Chapter 21 InnoDB Cluster](https://dev.mysql.com/doc/refman/8.0/en/mysql-innodb-cluster-introduction.html)
[Chapter 7 MySQL InnoDB Cluster](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-innodb-cluster.html)

## 1 InnoDB Cluster 要求
1. 满足组复制的要求：[组复制要求](https://dev.mysql.com/doc/refman/8.0/en/group-replication-requirements.html)
2. 组复制的表必须是 InnoDB
3. 不能有其他的组复制通道 #unknown 
4. Performance Schema 必须开启
5. 需要 python 
```shell
/usr/bin/env python
```

## 2 InnoDB Cluster 已知限制
1. **MySQL Shell 8.0.27 无法管理在 MySQL Server 8.0.25 上运行的 InnoDB 集群。在使用 Shell 8.0.27 前，要将 MySQL Server 升级到 26 以上。这个问题在 MySQL Shell 8.0.28 中将会被修复。**
2. 


## 3 使用docker 搭建 InnoDB Cluster

这篇文章详细讲述了使用 docker 搭建 InnoDB Cluster 的步骤。文章提到要先搭建一组 mysql 组复制镜像。
> https://github.com/wagnerjfr/mysql-group-replication-docker
> https://github.com/wagnerjfr/mysql-innodb-cluster


