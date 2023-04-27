## 1 langchain是什么
langchain是一个能将各种 AI 模型集成在一起形成自定义功能的 AI 框架。
- git地址：https://github.com/hwchase17/langchain
- 官方文档：https://python.langchain.com/en/latest/
- 视频教程 ：https://www.youtube.com/watch?v=kYRB-vJFy38&list=PLqZXAkvF1bPNQER9mLmDbntNfSpzdDIU5&index=2

注意官方文档的右下角，有一个搜索框，不同于普通文档基于关键字匹配的检索方法，这个搜索功能是 Mendable 提供的。Mendable 本身是一个基于 langchain 框架实现的文档检索工具。

## 2 langchain 组件

### 2.1 Prompts
#### 2.1.1 Template
在预先定义好的一个 template 中预先插入占位符，然后使用输入将占位符填充。这样生成固定格式的 Prompt。

#### 2.1.2 Examples
FewShotPrompt：一种 Prompt 的技巧，在这个 prompt 中会有一些例子，告诉 LLM 如何规范地输出。
这里的 Example 就是用于填充这个方法中例子的部分。

#### 2.1.3 Selector
在 Example 中可能会定义大量的例子，如果全部都填充入 Prompt 会造成大量的 token 消耗。Selector 的目的就是从这些 Examples 中选出符合要求的几个，



#### 2.1.4 Output Parsers

