
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
ChatGPT 没有直接的论文参考，但根据[官方文档](https://openai.com/blog/chatgpt/)摘要中可以知道: ChatGPT is a sibling model to [InstructGPT](https://openai.com/blog/instruction-following/)。

### 2.1 参考

https://www.youtube.com/watch?v=e0aKI2GGZNg
[ChatGPT (可能)是怎麼煉成的 - GPT 社會化的過程PPT](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbDJHYmFYX3JFUmx2aFl6THNKSUhaMjBZYzVEd3xBQ3Jtc0tsRHhfU0RhNnlNZGtGNk5BYko2SHBZOHRMa1Y3bTdRa3ZhTVVpNVZEVVgtSW0tTEJ0M3VMelhiVVZkTVV2azJUM1k2cXA4NTNnc2xJYm5rVmpTTFNEVWd6WERfOEJqb1lXNDBPTjNvSHNIMDNSZHlFVQ&q=https%3A%2F%2Fdocs.google.com%2Fpresentation%2Fd%2F1vDT11ec_nY6P0o--NHq9col5XEE4tHBw%2Fedit%3Fusp%3Dsharing%26ouid%3D115046073158939078465%26rtpof%3Dtrue%26sd%3Dtrue&v=e0aKI2GGZNg)
