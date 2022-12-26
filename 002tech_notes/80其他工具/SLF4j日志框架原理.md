# SLF4j日志框架原理
## 1 Java中的日志框架
| 名称     | Jar包                                 | 描述                                                 |
|:-------- | ------------------------------------- | ---------------------------------------------------- |
| slf4j    | Slf4j-api.jar                         | 门面日志框架                                         |
| jcl      | commons-logging.jar                   | 门面日志框架                                         |
| log4j    | log4j.jar                             | 底层日志框架，Log4j 1.x                              |
| reload4j | reload4j.jar                          | 底层日志框架，log4j 1.x 的分支版本，修复了安全漏洞。 |
| log4j2   | log4j-api.jar，log4j-core.jar         | 底层日志框架，Log4j 2.x                              |
| jul      | JDK                                   | 底层日志框架                                         |
| logback  | logback-classic.jar, logback-core.jar | 底层日志框架                                         |

除了jcl、jul 剩余的日志框架的作者是同一个人。


## 2 什么是门面日志框架


## 3 为什么用门面日志框架
![[attachments/Pasted image 20220504021640.png]]
没有门面日志框架，在代码层面就要针对不同的框架写不同的接入代码


## 4 相关资料
[10分钟让你彻底明白Java SPI，附实例代码演示](https://www.bilibili.com/video/BV1RY4y1v7mN/?spm_id_from=333.788.recommend_more_video.0)
#todo 