官方示例
<iframe
		width=100%
		height=700
		src="https://mybatis.org/mybatis-3/zh/configuration.html#映射器（mappers）">
</iframe>
最近使用mybatis的时候一直很疑惑 为什么接口名必须与Mybatis的映射文件名一定要一模一样，如果不一样就会报如下错
```java
org.apache.ibatis.binding.BindingException: Invalid bound statement (not found): org.xx.demo.mapper.xx.xx
```
**究其原因是mybatis-config.xml的配置文件的原因**

1. 在注册映射文件时使用`<package name="包名">`标签时，需要映射文件名和接口名一样，不然会报错。
2. 在注册映射文件时使用`<mapper class="">`mapper标签的class属性时，需要映射文件名和接口名一样，不然会报错。
3. 在注册映射文件时使用`<mapper resource="org/xx/demo/mapper/xx.xml"/>`，不需要映射文件名和接口名一样

