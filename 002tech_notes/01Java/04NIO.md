# NIO

![[attachments/网络模型.svg]]


## 1 常用命令手册

### 1.1 netstat
#### 1.1.1 mac下常用参数
| 参数 | 说明                                                                                                           |
| ---- | -------------------------------------------------------------------------------------------------------------- |
| -a   | 显示所有连接信息，包括常用于服务器的一些端口监听连接不加-a的话，默认不显示LISTEN连接信息的                     |
| -n   | 不显示别名信息，用数字代替，可以加快命令的执行速度比如：mysql的端口用3306直接显示，而不是用mysql这样的名称显示 |
| -p   | 显示指定网络协议的连接，全部协议在/etc/protocols中比如：-ptcp 或者-p udp                                       |
| -v   | 显示更多的信息，可以显示对应连接的进程ID(PID)等当需要查看网络连接是属于哪个进程时可以使用                      |



### 1.2 其他命令
```shell
# 追踪命令产生的系统调用
strace -ff -o out java testMain -Xmx128m

# 查看当前 shell 的所有文件描述符。0、1、2 分别代表标准输入，标准输出，错误输出
cd /proc/$$/fd

# 查看文件描述符细节
lsof -op $$ 

# 查看page cache
pcstat 文件名

# 查看系统的配置项，对应配置文件 /etc/sysctl.conf
sysctl -a | grep dirty
vim /etc/sysctl.conf

```

## 2 磁盘IO
### 2.1 java Buffer IO 为什么比 Direct IO快？
Buffer IO在jvm 中有一块缓冲区，写满后调用一次 system call 写入内存。
而原始 IO 每次写入都会调用一次 system call，减少了用户态和内核态的切换

- Direct  IO
![[attachments/Pasted image 20220722002108.png  | 500]]
- buffer  IO
![[attachments/Pasted image 20220722002011.png | 900]]

### 2.2 磁盘IO模型
![[attachments/磁盘IO模型.svg]]

通过 `Channel` 获取的 `MappedByteBuffer`，获取了一个 `mmap`（一个文件 - 堆外内存 `pagecache` 的映射）。调用`map.put("abc".getBytes())` 的时候，不用通过系统调用就可以将数据写入 `pagecache`。

直接IO，会跳过操作系统管理的 page cache，更灵活。MySql会使用直接IO。



