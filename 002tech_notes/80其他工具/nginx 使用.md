Nginx 是一个高性能的开源 Web 服务器和反向代理服务器。以下是 Nginx 的基本使用方法：

## 1 安装、启动
### 1.1 安装
首先需要安装 Nginx。在 Linux 中，可以使用包管理器进行安装。例如，在 Ubuntu 中，可以使用以下命令安装 Nginx：

```sh
sudo apt-get update sudo apt-get install nginx
```

### 1.2 启动 Nginx
安装完成后，可以使用以下命令启动 Nginx：
```sh
sudo systemctl start nginx
```

如果要在系统启动时自动启动 Nginx，请使用以下命令：
```sh
sudo systemctl enable nginx
```

### 1.3 停止 Nginx
```sh
sudo systemctl stop nginx
```

## 2 配置文件
Nginx 配置文件位于 `/etc/nginx/nginx.conf`。 在更改 Nginx 配置文件后，可以使用以下命令测试配置是否正确，如果配置正确，则会显示一条成功的消息。
```sh
sudo nginx -t
```

在更改 Nginx 配置文件后，可以使用以下命令重新加载配置文件，这会重新加载配置文件，而无需停止正在运行的 Nginx 服务。
```sh
sudo systemctl reload nginx
```


## 3 错误记录
### 3.1 nginx: [emerg] bind() to 0.0.0.0:18008 failed (13: Permission denied)
启动后无法绑定端口，分为两种情况
1. 端口号小于 1024，需要使用 root 权限启动 nginx
```shell
sudo systemctl status nginx
```
2. 如果端口号大于1024，是因为端口没有打开http权限
```shell
# 查看有http权限的端口
semanage port -l | grep http_port_t

# 添加http权限端口
semanage port -a -t http_port_t  -p tcp 18008
```

