# spring源码大局观

## 1 构建源码学习环境
- IDEA 构建 spring 5.2.9.RELEASE 版本源码
https://blog.csdn.net/YogaK/article/details/119822158


- 课程中使用的源码版本是当时的5.3.0-snapshot
Upgrade JDK 15 CI image to JDK 15 build 26 Sam Brannen 2020/6/12, 15:56 ==30385dae==

- 我使用的版本是 5.3.0 release版


## 2 为什么要Spring这个框架
**解决类与类之间强耦合的问题**
怎么解决的？通过IoC容器


### 2.1 控制反转 Inversion of Control
![[attachments/IoC示例.svg]]
使用控制反转前，UserService 和 LoginService 如果想使用 UserMapper ，必须在xxService中自己写取得 UserMapper 对象的逻辑。
使用控制反转后，不再由要使用该依赖的对象手动获取，而是由 IOC 容器控制依赖的注入。
**所有的类都会在spring容器中登记，告诉spring你是个什么东西，你需要什么东西，然后spring会在系统运行到适当的时候，把你要的东西主动给你，同时也把你交给其他需要你的东西。所有的类的创建、销毁都由 spring来控制，也就是说控制对象生存周期的不再是引用它的对象，而是spring。对于某个具体的对象而言，以前是它控制其他对象，现在是所有对象都被spring控制，所以这叫控制反转。**


### 2.2 依赖注入 Dependency Inject
IoC的一个重点是在系统运行中，动态的向某个对象提供它所需要的其他对象。这一点是通过DI（Dependency Injection，依赖注入）来实现的。
DI是由Martin Fowler 在2004年初的一篇论文中首次提出的。他总结：**控制的什么被反转了？就是：获得依赖对象的方式反转了。**


## 3 spring IOC 容器架构图
<iframe 
		width=100%
		height=800
		src = https://www.processon.com/view/link/5ee18fa4e401fd1fd287bee3
</iframe>


### 3.1 BeanDefinition
![[attachments/Pasted image 20220515194328.png|700]]
| 属性名            | 作用                                                                 |
| ----------------- | -------------------------------------------------------------------- |
| scope             | 单例模式：每次getBean从缓存池获取；原型模式：每次 getBean 创建新实例 |
| dependsOn         | 若果 ClassA 配置了 DependsOn ClassB 。那么在 B 处理完成后才会处理 A  |
| autowireCandidate | 这个类是否可以Autowire到别的类中                                     |
| initMethodName    |                                                                      |
| destroyMethodName |                                                                      |

### 3.2 BeanFactoryPostProcessor


### 3.3 BeanPostProcessor
![[attachments/Pasted image 20220516235710.png|600]]
两个方法分别在实例化之前和之后调用

![[002tech_notes/04spring/spring 源码解析/Untitled Diagram.svg]]

## 4 快速过一遍源码

`org.springframework.context.support.AbstractApplicationContext` 的`refresh()` 这个方法执行完后，容器就启动起来了

## 5 三级缓存与循环依赖
### 5.1 三级缓存
DefaultSingletonBeanRegistry 中存放了三级缓存

``` java
/** 一级缓存 Cache of singleton objects: bean name to bean instance. */  
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);  
  
/** 三级缓存 Cache of singleton factories: bean name to ObjectFactory. */  
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);  
  
/** 二级缓存 Cache of early singleton objects: bean name to bean instance. */  
private final Map<String, Object> earlySingletonObjects = new ConcurrentHashMap<>(16);
```

### 5.2 循环依赖
#### 5.2.1 三种循环依赖
1. 原型模式产生循环依赖（无法解决）
2. 单例循环依赖 - 构造参数产生依赖（无法解决）
3. 单例循环依赖 - setter产生依赖（可以解决, 靠三级缓存解决）

