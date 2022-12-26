# docker file

## 1 基本介绍
### 1.1 docker File 作用
docker file是一个文本文件，用于生成新的镜像。是由一条条构建镜像所需要的指令和参数构成的脚本。

### 1.2 dockerFile 基础知识
1. docker 保留字指令必须大写
2. 按照从上到下的顺序执行
3. `#` 表示备注
4. 每条指令都会创建心的镜像层，并提交。

### 1.3 docker 执行 DockerFile 大致流程
1. 从基础镜像运行一个容器
2. 执行一条命令，修改容器
3. 执行类似 docker commit 提交一个新的镜像层
4. 基于刚刚提交的镜像，运行一个新容器
5. 执行下一条指令，直到完成全部指令

![[attachments/Pasted image 20220605220343.png]]


### 1.4 mysql 镜像的 dockerFile 示例
https://github.com/mysql/mysql-docker/blob/mysql-server/8.0/Dockerfile