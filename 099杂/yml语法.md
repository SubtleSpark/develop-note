# yml 语法

## 1 基本语法

### 1.1 特性
-   大小写敏感
-   使用缩进表示层级关系
-   **缩进不允许使用tab，只允许空格**
-   缩进的空格数不重要，只要相同层级的元素左对齐即可
-   '#'表示注释

### 1.2 yaml 数组
以 `-` 开头的行表示构成一个数组
yaml 支持多维数组

```yaml
# 数组
key: 
    - value1
    - value2

# 行内表示法
key: [value1, value2]


# 二维数组
arrs:  
-  
  - k: a1k  
	v: a1v  
  - k: a2k  
	v: a2v  
-  
  - k: b1k  
	v: b1v  
  - k: b2k  
	v: b2v

```

### 1.3 yaml 对象
对象键值对使用冒号结构表示 `key: value`，冒号后面要加一个空格。
也可以使用 `key:{key1: value1, key2: value2, ...}`（行内表示）。

```yaml
# 对象列表
companies:
    -
        id: 1
        name: company1
        price: 200W
    -
        id: 2
        name: company2
        price: 500W


# 行内表示
companies: [{id: 1,name: company1,price: 200W},{id: 2,name: company2,price: 500W}]
```


### 1.4 yaml 纯量
纯量是最基本的，不可再分的值，包括：
-   字符串
-   布尔值
-   整数
-   浮点数
-   Null
-   时间
-   日期

```yaml
boolean: 
    - TRUE  #true,True都可以
    - FALSE  #false，False都可以

float:
    - 3.14
    - 6.8523015e+5  #可以使用科学计数法

int:
    - 123
    - 0b1010_0111_0100_1010_1110    #二进制表示

null:
    nodeName: 'node'
    parent: ~  #使用~表示null
string:
    - 哈哈
    - 'Hello world'  #可以使用双引号或者单引号包裹特殊字符
    - newline
      newline2    #字符串可以拆成多行，每一行会被转化成一个空格
date:
    - 2018-02-17    #日期必须使用ISO 8601格式，即yyyy-MM-dd
datetime: 
    -  2018-02-17T15:02:31+08:00    #时间使用ISO 8601格式，时间和日期之间使用T连接，最后使用+代表时区
```

### 1.5 锚点、引用
`&`  锚点和  `*`  别名，可以用来引用。参考以下。
[YAML 入门教程 | 菜鸟教程 (runoob.com)](https://www.runoob.com/w3cnote/yaml-intro.html)






