# Java 中的自定义注解与 AOP
intercepter spring提供
![[attachments/Pasted image 20220517175632.png]]


切面表达式匹配效率？
![[attachments/Pasted image 20220517175702.png]]

## 1 场景题

![[attachments/Pasted image 20220517180906.png]]
![[attachments/Pasted image 20220517181043.png]]

### 1.1 通过
![[attachments/Pasted image 20220517181611.png]]

## 2 弊端
- 仅阅读Java主体代码，还不能完全描述软件行为，对维护者有难度
	- 文档支持
	- 遵循惯例
	- 克制，不要泛滥
- 损耗少量性能
	- 在B端领域可忽略不计
	- 追求极致性能时可不采用，天平的另一端就是全员严格要求的编码规范