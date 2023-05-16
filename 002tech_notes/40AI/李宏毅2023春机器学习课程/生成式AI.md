
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


