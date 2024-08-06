

## 1 MySQL Group Replication with Docker MySQL Images



```shell
docker run -d -p 13306:3306 --privileged=true -e MYSQL_ROOT_PASSWORD=123456 --name t_mysql_1 mysql:8.0.28 -v //E/docker_containers/t_mysql_1/config:/etc/mysql/conf.d -v //E/docker_containers/t_mysql_1/data:/var/lib/mysql 

```


