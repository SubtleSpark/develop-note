# lottery 项目开发笔记
## 1 环境、配置规范
> [环境、配置、规范](https://gitcode.net/KnowledgePlanet/Lottery/-/wikis/%E7%AC%AC-2-%E9%83%A8%E5%88%86-%E9%A2%86%E5%9F%9F%E5%BC%80%E5%8F%91//%E7%AC%AC01%E8%8A%82%EF%BC%9A%E7%8E%AF%E5%A2%83%E3%80%81%E9%85%8D%E7%BD%AE%E3%80%81%E8%A7%84%E8%8C%83)

### 1.1 开发环境
-   JDK 1.8
-   SpringBoot 2.6.0
-   Dubbo 2.7.10
-   DB-ROUTER `自研分库分表路由组件，带着你一起写个SpringBoot Starter`
-   vue `开发H5大转盘抽奖`
-   微信公众号 `对接提供API，回复抽奖`
-   Docker `本地和云服务都可以`

### 1.2 开发规范
**分支命名**：日期_姓名首字母缩写_功能单词，如：`210804_xfg_buildFramework`
**提交规范**：`作者，type: desc` 如：`小傅哥，fix：修复查询用户信息逻辑问题` _参考Commit message 规范_
```
# 主要type
feat:     增加新功能
fix:      修复bug

# 特殊type
docs:     只改动了文档相关的内容
style:    不影响代码含义的改动，例如去掉空格、改变缩进、增删分号
build:    构造工具的或者外部依赖的改动，例如webpack，npm
refactor: 代码重构时使用
revert:   执行git revert 打印的 message

# 暂不使用type
test:     添加测试或者修改现有测试
perf:     提高性能的改动
ci:       与CI（持续集成服务）有关的改动
chore:    不修改src或者test的其余修改，例如构建过程或辅助工具的变动
```

## 2 搭建(DDD + RPC)架构
[搭建(DDD + RPC)架构](https://gitcode.net/KnowledgePlanet/Lottery/-/wikis/%E7%AC%AC-2-%E9%83%A8%E5%88%86-%E9%A2%86%E5%9F%9F%E5%BC%80%E5%8F%91//%E7%AC%AC02%E8%8A%82%EF%BC%9A%E6%90%AD%E5%BB%BADDD%E5%9B%9B%E5%B1%82%E6%9E%B6%E6%9E%84)
我们创建的项目结构并不是一个简单的 MVC 结构，而是需要基于 DDD 四层架构进行模块化拆分，并把分布式组件 RPC 结合进行，所以这里我们需要进行框架搭建。

### 2.1 DDD 分层架构介绍
> DDD（Domain-Driven Design 领域驱动设计）是由Eric Evans最先提出，目的是对软件所涉及到的领域进行建模，以应对系统规模过大时引起的软件复杂性的问题。整个过程大概是这样的，开发团队和领域专家一起通过通用语言(Ubiquitous Language)去理解和消化领域知识，从领域知识中提取和划分为一个一个的子领域（核心子域，通用子域，支撑子域），并在子领域上建立模型，再重复以上步骤，这样周而复始，构建出一套符合当前领域的模型。

依靠领域驱动设计的设计思想，通过**事件风暴** #unknown 建立领域模型，合理划分领域逻辑和物理边界，建立领域对象及服务矩阵和服务架构图，定义符合DDD分层架构思想的代码结构模型，保证业务模型与代码模型的一致性。通过上述设计思想、方法和过程，指导团队按照DDD设计思想完成微服务设计和开发。

**服务架构调用关系**
![[attachments/Pasted image 20220522012054.png]]

- **接口层{interfaces}**
	- 接口服务位于用户接口层，用于处理用户发送的Restful请求和解析用户输入的配置文件等，并将信息传递给应用层。
- **应用层{application}**
	- 应用服务位于应用层。用来表述应用和用户行为，负责服务的组合、编排和转发，负责处理业务用例的执行顺序以及结果的拼装。
	- 应用层的服务包括应用服务和领域事件相关服务。
	- 应用服务可对微服务内的领域服务以及微服务外的应用服务进行组合和编排，或者对基础层如文件、缓存等数据直接操作形成应用服务，对外提供粗粒度的服务。
	- 领域事件服务包括两类：领域事件的发布和订阅。通过事件总线和消息队列实现异步数据传输，实现微服务之间的解耦。
- **领域层{domain}**
	- 领域服务位于领域层，为完成领域中跨实体或值对象的操作转换而封装的服务，领域服务以与实体和值对象相同的方式参与实施过程。
   - 领域服务对同一个实体的一个或多个方法进行组合和封装，或对多个不同实体的操作进行组合或编排，对外暴露成领域服务。领域服务封装了核心的业务逻辑。实体自身的行为在实体类内部实现，向上封装成领域服务暴露。
   - 为隐藏领域层的业务逻辑实现，所有领域方法和服务等均须通过领域服务对外暴露。
   - 为实现微服务内聚合之间的解耦，原则上禁止跨聚合的领域服务调用和跨聚合的数据相互关联。
- **基础层{infrastructure}**
   - 基础服务位于基础层。为各层提供资源服务（如==数据库、缓存==等），实现各层的解耦，降低外部资源变化对业务逻辑的影响。
   - **基础服务主要为仓储服务，通过依赖反转的方式为各层提供基础资源服务**，领域服务和应用服务调用仓储服务接口，利用仓储实现持久化数据对象或直接访问基础资源。

## 3 跑通广播模式RPC过程调用
> [跑通广播模式RPC过程调用](https://gitcode.net/KnowledgePlanet/Lottery/-/wikis/%E7%AC%AC-2-%E9%83%A8%E5%88%86-%E9%A2%86%E5%9F%9F%E5%BC%80%E5%8F%91/%E7%AC%AC03%E8%8A%82%EF%BC%9A%E8%B7%91%E9%80%9A%E5%B9%BF%E6%92%AD%E6%A8%A1%E5%BC%8FRPC%E8%BF%87%E7%A8%8B%E8%B0%83%E7%94%A8)

### 3.1 pom 文件配置
按照现有工程的结构模块分层，包括：
-   lottery-application，应用层，引用：`domain`
-   lottery-common，通用包，引用：`无`
-   lottery-domain，领域层，引用：`infrastructure`
-   lottery-infrastructure，基础层，引用：`无`
-   lottery-interfaces，接口层，引用：`application`、`rpc`
-   lottery-rpc，RPC接口定义层，引用：`common`

![[002tech_notes/lottery 抽奖系统学习/lottery模块依赖关系.svg]]
### 3.2 对 Dubbo 的初步理解
1. 服务的提供者和调用者引入同一个接口，想当于定义了一个接口的方法名、参数、返回值。
2. 提供者实现该接口，并在实现类中添加注解`org.apache.dubbo.config.annotation.Service`。
3. 调用者通过`org.apache.dubbo.config.annotation.Reference`注解，注入该接口的实现，就可以像调用本地方法一样完成远程调用


```java
// 定义一个接口
public interface IActivityBooth {  
    ActivityRes queryActivityById(ActivityReq req);    
}

// 提供者实现该接口
@Service  
public class ActivityBooth implements IActivityBooth {  
    @Resource  
    private IActivityDao activityDao;  
  
    @Override  
    public ActivityRes queryActivityById(ActivityReq req) {  
        ActivityRes = /*查询出的结果*/；
        return ActivityRes;  
    }  
  
}

// 消费者注入接口
public class ApiTest {  
    @Reference(interfaceClass = IActivityBooth.class, url = "dubbo://127.0.0.1:20880")  
    private IActivityBooth activityBooth;  
  
    @Test  
    public void test_rpc() {  
        ActivityReq req = new ActivityReq();  
        req.setActivityId(100001L);  
        ActivityRes result = activityBooth.queryActivityById(req);  
        logger.info("测试结果：{}", JSON.toJSONString(result));  
    }  
}
```


## 4 库表设计
[抽奖活动策略库表设计](https://gitcode.net/KnowledgePlanet/Lottery/-/wikis/%E7%AC%AC-2-%E9%83%A8%E5%88%86-%E9%A2%86%E5%9F%9F%E5%BC%80%E5%8F%91//%E7%AC%AC04%E8%8A%82%EF%BC%9A%E6%8A%BD%E5%A5%96%E6%B4%BB%E5%8A%A8%E7%AD%96%E7%95%A5%E5%BA%93%E8%A1%A8%E8%AE%BE%E8%AE%A1)
基本诉求：活动配置、奖品概率配置、奖品梳理配置、记录用户的抽奖数据
![[attachments/Pasted image 20220813144436.png]]

## 5 抽奖策略领域模块开发
[第05节：抽奖策略领域模块开发](https://gitcode.net/KnowledgePlanet/Lottery/-/wikis/%E7%AC%AC-2-%E9%83%A8%E5%88%86-%E9%A2%86%E5%9F%9F%E5%BC%80%E5%8F%91//%E7%AC%AC05%E8%8A%82%EF%BC%9A%E6%8A%BD%E5%A5%96%E7%AD%96%E7%95%A5%E9%A2%86%E5%9F%9F%E6%A8%A1%E5%9D%97%E5%BC%80%E5%8F%91)
### 5.1 领域功能结构
![[attachments/Pasted image 20220827152632.png | 500]]
-   model，用于提供vo、req、res 和 aggregates 聚合对象。
-   repository，提供仓储服务，其实也就是对Mysql、Redis等数据的统一包装。
-   service，是具体的业务领域逻辑实现层，在这个包下定义了algorithm抽奖算法实现和具体的抽奖策略包装 draw 层，对外提供抽奖接口 IDrawExec#doDrawExec

### 5.2 DrawExecImpl 继承结构
![[attachments/Pasted image 20220827113417.png | 700]]



## 6 模板模式处理抽奖流程
[第06节：模板模式处理抽奖流程](https://gitcode.net/KnowledgePlanet/Lottery/-/wikis/%E7%AC%AC-2-%E9%83%A8%E5%88%86-%E9%A2%86%E5%9F%9F%E5%BC%80%E5%8F%91/%E7%AC%AC06%E8%8A%82%EF%BC%9A%E6%A8%A1%E6%9D%BF%E6%A8%A1%E5%BC%8F%E5%A4%84%E7%90%86%E6%8A%BD%E5%A5%96%E6%B5%81%E7%A8%8B)

### 6.1 抽奖策略继承结构
![[attachments/Pasted image 20220827163144.png | 700]]
### 6.2 DrawExecImpl 继承结构
![[attachments/Pasted image 20220827154040.png | 600]]
相较于上一节的继承体系，变化主要在于添加了`AbstractDrawBase`，提供了 `doDrawExec` 的模板方法。
-   `DrawConfig`：配置抽奖策略，SingleRateRandomDrawAlgorithm、EntiretyRateRandomDrawAlgorithm
-   `DrawStrategySupport`：提供抽奖策略数据支持，便于查询策略配置、奖品信息。通过这样的方式隔离职责。
-   `AbstractDrawBase`：**抽象类定义模板方法流程**，在抽象类的 `doDrawExec` 方法中，处理整个抽奖流程，并提供在流程中需要使用到的抽象方法，由 `DrawExecImpl` 服务逻辑中做具体实现。


## 7 简单工厂模式搭建发奖领域（工厂模式）
[第07节：简单工厂搭建发奖领域](https://gitcode.net/KnowledgePlanet/Lottery/-/wikis/%E7%AC%AC-2-%E9%83%A8%E5%88%86-%E9%A2%86%E5%9F%9F%E5%BC%80%E5%8F%91//%E7%AC%AC07%E8%8A%82%EF%BC%9A%E7%AE%80%E5%8D%95%E5%B7%A5%E5%8E%82%E6%90%AD%E5%BB%BA%E5%8F%91%E5%A5%96%E9%A2%86%E5%9F%9F)




## 8 活动领域的配置与状态（状态模式）
[活动领域的配置与状态](https://gitcode.net/KnowledgePlanet/Lottery/-/wikis/%E7%AC%AC-2-%E9%83%A8%E5%88%86-%E9%A2%86%E5%9F%9F%E5%BC%80%E5%8F%91//%E7%AC%AC08%E8%8A%82%EF%BC%9A%E6%B4%BB%E5%8A%A8%E9%A2%86%E5%9F%9F%E7%9A%84%E9%85%8D%E7%BD%AE%E4%B8%8E%E7%8A%B6%E6%80%81)
### 8.1 调整模块依赖关系
主要调整了domain 和 infrastructure 的依赖关系。domain中定义Repository 的接口方法。infrastructure 中负责实现该方法。
![[002tech_notes/lottery 抽奖系统学习/调整后的依赖关系.svg]]



