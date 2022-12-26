# 使用docker 搭建 InnoDB Cluster

这篇文章详细讲述了使用 docker 搭建 InnoDB Cluster 的步骤。文章提到要先搭建一组 mysql 组复制镜像。
> https://github.com/wagnerjfr/mysql-group-replication-docker
> https://github.com/wagnerjfr/mysql-innodb-cluster


## 1 MySQL Group Replication with Docker MySQL Images



```shell
docker run -d -p 13306:3306 --privileged=true -e MYSQL_ROOT_PASSWORD=123456 --name t_mysql_1 mysql:8.0.28 -v //E/docker_containers/t_mysql_1/config:/etc/mysql/conf.d -v //E/docker_containers/t_mysql_1/data:/var/lib/mysql 

```


