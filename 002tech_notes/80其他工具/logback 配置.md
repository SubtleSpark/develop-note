# logback 配置
[logback官网](https://logback.qos.ch/manual/configuration.html)
## 1 logback的配置介绍
![[attachments/Pasted image 20220630165615.png]]
### 1.1 Configuring loggers, or the `<logger>` element
#### 1.1.1 level inheritance（日志等级继承）

At this point you should have at least some understanding of [level inheritance](https://logback.qos.ch/manual/architecture.html#effectiveLevel) and the [basic selection rule](https://logback.qos.ch/manual/architecture.html#basic_selection). Otherwise, and unless you are an Egyptologist, logback configuration will be no more meaningful to you than are hieroglyphics.

A logger is configured using the `<logger>` element. A `<logger>` element takes exactly one mandatory name attribute, an optional level attribute, and an optional additivity attribute, admitting the values _true_ or _false_. The value of the level attribute admitting one of the case-insensitive string values TRACE, DEBUG, INFO, WARN, ERROR, ALL or OFF. The special case-insensitive value _INHERITED_, or its synonym _NULL_, will force the level of the logger to be inherited from higher up in the hierarchy. This comes in handy if you set the level of a logger and later decide that it should inherit its level.


### 1.2 Configuring the root logger, or the `<root>` element

The `<root>` element configures the root logger. It supports a single attribute, namely the level attribute. It does not allow any other attributes because the additivity flag does not apply to the root logger. Moreover, since the root logger is already named as "ROOT", it does not allow a name attribute either. The value of the level attribute can be one of the case-insensitive strings TRACE, DEBUG, INFO, WARN, ERROR, ALL or OFF. Note that the level of the root logger cannot be set to INHERITED or NULL.

1. root 标签配置了 root logger。它只支持一个 lever 属性。
2. additivity 在 root logger上没有作用。
3. name 属性已经被赋值为了ROOT，所以 name 属性也没有用。

## 2 logback的默认配置

如果配置文件 logback-test.xml 和 logback.xml 都不存在，那么 logback 默认地会调用BasicConfigurator ，创建一个最小化配置。最小化配置由一个关联到根 logger 的ConsoleAppender 组成。输出用模式为%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n 的 PatternLayoutEncoder 进行格式化。root logger 默认级别是 DEBUG。

-   Logback的配置文件
　　Logback 配置文件的语法非常灵活。正因为灵活，所以无法用 DTD 或 XML schema 进行定义。尽管如此，可以这样描述配置文件的基本结构：以`<configuration>`开头，后面有零个或多个`<appender>`元素，有零个或多个`<logger>`元素，有最多一个`<root>`元素。

-   Logback默认配置的步骤
1.  尝试在 classpath下查找文件logback-test.xml；
2.  如果文件不存在，则查找文件logback.xml；
3.  如果两个文件都不存在，logback用BasicConfigurator自动对自己进行配置，这会导致记录输出到控制台。



## 3 总结

### 3.1 输出源选择
logback的配置，需要配置输出源appender，打日志的logger（子节点）和root（根节点），实际上，它输出日志是从子节点开始，子节点如果有输出源直接输入，如果无，判断配置的addtivity，是否向上级传递，即是否向root传递，传递则采用root的输出源，否则不输出日志。

### 3.2 日志级别Level
日志记录器（Logger）的行为是分等级的： 分为OFF、FATAL、ERROR、WARN、INFO、DEBUG、ALL或者您定义的级别。  
Log4j建议只使用四个级别，**优先级从高到低分别是 ERROR、WARN、INFO、DEBUG，优先级高的将被打印出来。（logback通用）**  
通过定义级别，可以作为应用程序中相应级别的日志信息的开关。  
**比如在这里定义了INFO级别，则应用程序中所有DEBUG级别的日志信息将不被打印出来。（设置INFO级别，即：>=INFO 生效）**  
**项目上生产环境的时候一定得把debug的日志级别重新调为warn或者更高，避免产生大量日志。**