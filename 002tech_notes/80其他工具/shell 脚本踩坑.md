# shell 脚本踩坑

## 1 脚本中 pwd
### 1.1 实验1
实验1：在`~/dev/script/test.sh` 下有如下脚本。分别在两个文件工作目录下调用这个脚本，查看结果。
![[attachments/Pasted image 20220605193938.png]]
结论：shell 脚本中 `pwd` 的结果，取决于执行这个脚本时的路径。而不是脚本身的路径。

### 1.2 实验2
实验2：在`~/dev/test2.sh` 下有如下脚本。脚本调用了`~/dev/script/test.sh`。
![[attachments/Pasted image 20220605194713.png]]
结论：嵌套调用脚本`pwd`结果，取决于执行这个脚本时的路径。而不是脚本身的路径。
