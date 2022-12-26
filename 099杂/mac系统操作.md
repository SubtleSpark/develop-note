# mac 系统操作
## 1 给用户添加组
1. 先进入root用户，执行 `dscl`。不使用root用户会导致添加组的时候没有权限。
2. 进入 /Local/Default/Groups 文件夹下。里面是所有的用户组
3. 执行 `-append _mysql GroupMembership jun` 。给 jun 用户添加了 `_mysql` 组权限
```shell
jundeMacBook-Pro:~ root# dscl
Entering interactive mode... (type "help" for commands)
> cd /Local/Default/Groups/
> /Local/Default/Groups > -append _mysql GroupMembership jun
```


## 2 netstat
### 2.1 mac下常用参数
| 参数 | 说明                                                                                                           |
| ---- | -------------------------------------------------------------------------------------------------------------- |
| -a   | 显示所有连接信息，包括常用于服务器的一些端口监听连接不加-a的话，默认不显示LISTEN连接信息的                     |
| -n   | 不显示别名信息，用数字代替，可以加快命令的执行速度比如：mysql的端口用3306直接显示，而不是用mysql这样的名称显示 |
| -p   | 显示指定网络协议的连接，全部协议在/etc/protocols中比如：-ptcp 或者-p udp                                       |
| -v   | 显示更多的信息，可以显示对应连接的进程ID(PID)等当需要查看网络连接是属于哪个进程时可以使用                      |

## 3 lsof


## 4 ps
| 参数      | 说明                                             | 举例               |
| --------- | ------------------------------------------------ | ------------------ |
| -ax -e -A | 都是输出全部的进程，默认只输出当前用户开启的进程 |                    |
| -f        | 显示更多信息                                     |                    |
| -p        | 显示指定pid的信息                                |                    |
| -u  -U    | 显示指定uid 或者用户名的信息                     | ps -u jun; ps -u 0 |
| -o        | 制定输出的列，可以使用ps -L 查看所有的关键字     | ps -o user -o pid  |
| -O        | 和 -o类似，但是是追加显示列                      |                    |


## 5 man 2 查看系统调用的文档
![[attachments/Pasted image 20220717225600.png | 500]]
> man 2 fork


## 6 strace 追踪一个命令产生的系统调用
```shell
strace -ff -o log java SocketBIO
```