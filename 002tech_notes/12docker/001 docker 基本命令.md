# docker 基本命令
## 1 查看容器状态 ps
| 参数 | 作用 | 备注 |
| :--: | -- | -- |
| -a | 列出所有容器 | Run container in background and print container ID|
| -l | 显示最近创建的容器| |
| -n | 显示最近创建的n个容器||
| -q | 静默模式，只显示容器编号 | |


```shell
# 显示正在运行的5个容器
docker ps -n 5
```


## 2 查看容器日志 logs
```shell
docker logs ID
```


## 3 容器的运行
### 3.1 启动容器 run
#### 3.1.1 启动交互式容器
```shell
docker run -it
```
| 参数 | 作用 | 备注 |
| :--: | -- | -- |
| -d | 后台运行（守护式容器） | Run container in background and print container ID|
| -i | 交互模式| |
| -t | 分配一个伪终端|--tty|
| -P | 随机端口映射 | |
| -p | 指定端口映射 | |

#### 3.1.2 启动守护式容器
```shell
jun@jundeMBP ~ % docker run -d ubuntu c0e4ccb1c47580adf39a626f408db27723576a754f491943c2395e114f790fb9
jun@jundeMBP ~ % docker ps CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
```

现象：容器创建、启动后马上退出了 原因：Docker容器后台运行必须要有前台进程。如果容器运行的命令不是一直挂起的命令，就会自动退出。 上面ubuntu容器创建后，内没有前台的进程，就自动退出了。但如果交互模式运行后，用ctrl+p+q退出，容器内还有一个前台运行的命令行，就不会自动退出。

```shell
docker run -d amd64/redis
```

容器内的redis默认配置不是以守护进程启动的，docker认为有前台进程，启动后不会自动退出。

### 3.2 绑定容器数据卷

![](https://cdn.nlark.com/yuque/0/2022/png/1309324/1646926225912-693e3e25-467c-41df-bf71-cb76cff731a1.png)
```shell
docker run --privileged=true -v /宿主机路径:/容器内路径:ro -w /foo -it ubuntu bash
```

-   --privileged Give extended privileges to this container
-   没有目录会自动创建
-   可以在后面加上ro、rw控制只读、读写（默认读写）
[数据卷不存在的情况](https://blog.csdn.net/qq_34556414/article/details/108589588)

### 3.3 退出机制
1.  exit。 退出后容器停止
2.  ctrl +p+q。退出后容器继续运行

### 3.4 启动、停止、强制停止容器
```shell
docker start ID/容器名
docker stop ...
docker kill ...
```

### 3.5 重新进入容器 exec attach
```shell
docker exec -it ID
docker attach -it ID
```
- 两个命令有区别，推荐 exec
#unknown  有什么区别？

## 4 容器镜像操作
### 4.1 导入、导出容器 import、export

**导出一个容器的内容，留作一个tar归档文件。相当于把整个容器的文件系统备份了**
``` shell
docker export 容器ID > 文件名.tar
```

**导入拷贝的文件系统形成新的镜像**
``` shell
cat 文件名.tar｜ docker import -镜像用户/镜像名:版本号
```

![](https://cdn.nlark.com/yuque/0/2022/png/1309324/1646409033050-3bdf2135-36e6-4e74-8807-9458921b2ff3.png)

### 4.2 提交容器副本，成为一个新镜像 commit
```shell
docker commit -m="描述信息" -a="作者" 容器ID 创建镜像名:[标签名]
```


### 4.3 推送镜像到远程 push

![](https://cdn.nlark.com/yuque/0/2022/png/1309324/1646724561544-0e9e859c-8ce8-4d5a-9c8b-755ac5cdca33.png)

```shell
docker tag rhel-httpd registry-host:5000/myadmin/rhel-httpd

docker push registry-host:5000/myadmin/rhel-httpd
```

  

## 5 容器、主机文件相互拷贝 cp
```shell
# 从容器复制文件到主机
docker cp 容器ID:容器内路径 目的主机路径

# 反过来就可以将文件从主机复制到容器
```

## 6 查看容器信息 docker inspect
