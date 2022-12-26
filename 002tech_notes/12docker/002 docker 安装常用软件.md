# 1 常用软件安装
## 1 tomcat
```shell
docker run -d -p 8080:8080 --name mytc8 tomcat
```

### 1.1 踩坑
-   现象：
最新的tomcat将原本的资源文件换了位置。启动后访问tomcat主页会报404。

-   解决方法
启动后将webapps.dist/ 下的文件，移动到webapps/。localhost:8080就能看到tomcat主页 

## 2 mysql
### 2.1 方式1
```shell
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 --name mysql mysql:5.7
```
- 缺陷：
容器删除后，数据库数据也被删除

### 2.2 方式2 使用数据卷安装
```shell
docker run -d -p 13306:3306 --privileged=true \
-v /Users/jun/container/mysql/log:/var/log/mysql \
-v /Users/jun/container/mysql/data:/var/lib/mysql \
-v /Users/jun/container/mysql/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=123456 \
--name mysql arm64v8/mysql:8.0.28-oracle \
--binlog-format=ROW
```
- 好处：
数据卷将数据保存在了宿主机上，容器删除后，数据得以保存


### 2.3 踩坑
-   现象
mysql无法插入中文

-   解决方法
配置问题，mysql5默认字符集不支持中文，修改字符集即可。mysql8默认utf8支持中文

## 3 redis
```shell
docker run -d -p 16379:6379 --privileged=true
-v /Users/jun/container/redis/date:/data
-v /Users/jun/container/redis/redis.conf:/etc/redis/redis.conf
--name myRedis0 redis redis-server /etc/redis/redis.conf
```

- privileged 表明root用户是否有真正的root权限
  ==redis.conf 中要设置 daemonize no==。不要以后台方式运行。会导致容器运行后，因为没有前台进程而停止。[[002tech_notes/12docker/001 docker 基本命令#启动守护式容器|启动守护式容器]]


## 4 zookeeper
```shell
docker run -d -e TZ="Asia/Shanghai" -p 12181:2181 -v /Users/jun/container/zookeeper/data:/data --name zookeeper --restart always zookeeper
```
