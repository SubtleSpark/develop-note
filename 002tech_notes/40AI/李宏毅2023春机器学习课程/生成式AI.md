
## 1 ChatGPT 原理剖析
### 1.1 ChatGPT 真正做的事
[课堂PPT](https://drive.google.com/file/d/1MPoOeCuO2sX1NVRdJYC79XXdK8un6h1H/view)
从GPT1到GPT3，再到ChatGPT，模型真正做的事其实就是**文字接龙**。如下面这个图片：
1. 型首先接受输入`什么是机器学习？`
2. 模型输出了下一个字概率分布
3. 根据概率选择下一个字
4. 加上生成的文字，将新的字符串重新输入到模型中。
5. 重复上面的过程，知道输出的制定的符号表示结束
![[attachments/Pasted image 20230516215335.png | 550]]

## 2 ChatGPT是怎样炼成的
ChatGPT 没有直接的论文参考，但根据[官方文档](https://openai.com/blog/chatgpt/)摘要中可以知道: *ChatGPT is a sibling model to [InstructGPT](https://openai.com/blog/instruction-following/)*。它们论文中的图片
### 2.1 训练的几个阶段
**1. 学习文字接龙（预训练，自监督学习）**
通过网上大量的文字资料，可以训练出一个会文字接龙的模型。这一步中的数据集不是人工标注的，可以通过程序自动生成，所以这一步叫做**自监督学习**。预训练后得到的模型是之后很多模型的基础，于是这个预训练的模型被叫做 **基石模型(Foundation Model)**。

学习了文字接龙后，GPT 可以完成补充句子的任务，但对它提问可能不会产生让人期待的结果。比如，在同样的输入下，GPT可能输出下面三个结果。在文字接龙的任务下，这几种输出都是 makes sense 的。于是，接下来的任务就变成了引导 GPT 输出我们期望的输出。

> 输出1：台湾最高的山是哪座山  -> *玉山*
> 输出2:  台湾最高的山是哪座山  ->  *，谁来告诉我？*
> 输出3:  台湾最高的山是哪座山  ->  *A 玉山，B 雪山*


**2. 人类老师引导 GPT 接龙的方向 （监督学习）**
提供一些人工标注好的问题和答案，针对这些数据再进行训练。这些数据不需要很多就可以达到较好的效果，因为这些答案本来就是 GPT 有能力产生的，新数据只是让 GPT 更偏向产生这种答案。InstructGPT 在这一步使用了几万条标注的数据。

**3. 训练 teacher model 模仿人类老师**
人类问 GPT 一个问题，让 GPT 产生很多的答案，再由人类老师标注这些答案中哪些是好的哪些是坏的。不同于上一步，这一步中人类不再需要提供完整的答案，只用评价输出的好坏。
收集【问题 答案 评分】这样的数据之后，就可以训练一个 teacher model 让它来模仿人类的评判规则和喜好。

**4. 用 Reinforcement learning 让 GPT 向 teacher model 学习**

**总结**
在第一阶段时，GPT 是想说啥就说啥，后面的几个步骤是引导 GPT 说人类想要它说的。

### 2.2 预训练的帮助



### 2.3 参考
[PPT - ChatGPT 原理剖析 (2/3) — 預訓練 (Pre-train)](https://drive.google.com/file/d/1Ue4J-Kk4ScFTq-UIs8hW9E2h5BAbu-oo/view)
[video - ChatGPT (可能)是怎麼煉成的 - GPT 社會化的過程](https://www.youtube.com/watch?v=e0aKI2GGZNg)
[PPT - ChatGPT (可能)是怎麼煉成的 - GPT 社會化的過程](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbDJHYmFYX3JFUmx2aFl6THNKSUhaMjBZYzVEd3xBQ3Jtc0tsRHhfU0RhNnlNZGtGNk5BYko2SHBZOHRMa1Y3bTdRa3ZhTVVpNVZEVVgtSW0tTEJ0M3VMelhiVVZkTVV2azJUM1k2cXA4NTNnc2xJYm5rVmpTTFNEVWd6WERfOEJqb1lXNDBPTjNvSHNIMDNSZHlFVQ&q=https%3A%2F%2Fdocs.google.com%2Fpresentation%2Fd%2F1vDT11ec_nY6P0o--NHq9col5XEE4tHBw%2Fedit%3Fusp%3Dsharing%26ouid%3D115046073158939078465%26rtpof%3Dtrue%26sd%3Dtrue&v=e0aKI2GGZNg)


## 3 生成式学习策略

### 3.1 各个击破



### 3.2 一步到位



