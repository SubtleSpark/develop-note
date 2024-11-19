# maven使用

## 部分标签作用
### 标签大全
[[002tech_notes/11maven/maven 标签大全|标签大全]]

### packaging
[Maven小知识：1.packaging 2.properties 3.dependencyManagement标签作用](https://www.cnblogs.com/sensenh/p/15126715.html)

 >pom ---------> 父类型都为pom类型，没有java代码，也不执行任何代码，只是为了聚合工程或传递依赖用的
> jar ---------> 内部调用或者是作服务使用
> war ---------> 需要部署的项目

### properties
通常用来统一管理依赖版本

### dependencyManagement
通常用在父项目中，与dependencies不同。dependencyManagement中的依赖，**并不会真的被引入**，只是声明了依赖的默认版本号等信息。
子项目需要一个依赖时，仍然需要在dependencies中显示引入，但可以不写版本号。
子项目指定了版本号，就使用子项目的版本号。

## 多模块开发（父子工程）
[父子模块关系示例](https://blog.csdn.net/mazhongjia/article/details/106855388)

- 父模块
dependencies 中写所有模块公用的依赖
父模块model标签中引入子模块
版本号都由父项目管理，写在properties标签中

- 子模块
子模块的parent标签，设置为父模块坐标
dependencies 中写自己只有模块需要的依赖

- spring boot 入口文件
引入controller 层的依赖
设置包扫描路径 ComponentScan("xxxx")

## 依赖管理
### 依赖传递
依赖分为 直接、间接依赖。
通常只用声明直接依赖，间接依赖会自动引入。
**只有依赖范围为compile的依赖才会传递**。
依赖传递会导致依赖冲突。

### 依赖冲突处理
- 最短路径优先：层级浅的依赖优先使用
- 先声明优先：层级一样，先声明的依赖优先使用

## 其他
### idea创建Spring项目，自动生成的 mvnw 文件

mvnw是Maven Wrapper的缩写。因为我们安装Maven时，默认情况下，系统所有项目都会使用全局安装的这个Maven版本。但是，对于某些项目来说，它可能必须使用某个特定的Maven版本，这个时候，就可以使用Maven Wrapper，它可以负责给这个特定的项目安装指定版本的Maven，而其他项目不受影响。

**简单地说，Maven Wrapper就是给一个项目提供一个独立的，指定版本的Maven给它使用**

```shell
my-project
├── .mvn
│   └── wrapper
│       ├── MavenWrapperDownloader.java
│       ├── maven-wrapper.jar
│       └── maven-wrapper.properties
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
├── main
│   ├── java
│   └── resources
└── test
    ├── java
    └── resources
```