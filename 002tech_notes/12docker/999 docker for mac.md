# docker for mac 踩坑

## 1 mac 进入 docker 虚拟层

MAC OS中，docker有两层虚拟机，一层是docker本身的虚拟层（linux），然后是docker里容器的虚拟层，所以网上文章提到的/var/lib/docker 文件夹在MAC系统中是找不到的。
不同版本的 docker 有不同进入 docker 虚拟层的方法。下面是部分能查到的办法：

- docker for mac 版本
![[attachments/Pasted image 20220603131524.png]]

- docker 版本
![[attachments/Pasted image 20220603131615.png]]

### 1.1 使用 docker 的 ttl 进入虚拟层
能查到最多的解答，但是我的版本找不到这个 tty 文件
```shell
screen ~/Library/Containers/com.docker.docker/Data/vms/0/tty
```

### 1.2 将docker虚拟层挂载在容器
```shell
docker run -it --name docker_vm --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
```
生成了一个容器，将docker虚拟层的相关文件挂载到了这个容器中。



## 2 修改已创建的 docker 容器端口映射
**docker run 可以指定端口映射，但是容器一旦生成，就没有一个命令可以直接修改**。通常间接的办法是，保存镜像，再创建一个新的容器，在创建时指定新的端口映射。有没有办法不保存镜像而直接修改已有的这个容器呢？有。在stackoverflow上面找到答案了，原帖如下
[stackoverflow 修改容器配置](https://stackoverflow.com/questions/19335444/how-do-i-assign-a-port-mapping-to-an-existing-docker-container)

操作步骤：

1. 停止容器
2. 停止docker服务(systemctl stop docker)
3. 修改这个容器的hostconfig.json文件中的端口（原帖有人提到，如果config.v2.json里面也记录了端口，也要修改）。如果无法找到这个文件夹。则需要[[002tech_notes/12docker/999 docker for mac#1 mac 进入 docker 虚拟层|进入docker虚拟层]]
```shell
cd /var/lib/docker/3b6ef264a040* #这里是CONTAINER ID
vi hostconfig.json


# 如果之前没有端口映射, 应该有这样的一段:
"PortBindings":{}

# 增加一个映射, 这样写:
# 前一个数字是容器端口, 后一个是宿主机端口。而修改现有端口映射更简单, 把端口号改掉就行.(HostPort 不用加协议)
"PortBindings":{"3306/tcp":[{"HostIp":"","HostPort":"3307"}]}

```
4. 启动docker服务（systemctl start docker）
5. 启动容器


## 3 网络
