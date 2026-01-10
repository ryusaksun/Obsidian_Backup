`more` 是一个古老但经典的 **分页显示工具**，用于逐页阅读长文件内容，避免屏幕瞬间被大量文本刷屏。虽然现在功能更强大的 `less` 命令更受欢迎（有一句名言：“less is more”），但 `more` 依然是许多精简系统（如 Docker 容器）中的默认工具。

## 1. 核心作用

当你直接使用 `cat` 查看一个几千行的文件时，内容会瞬间滚到最后。而 `more` 会让内容**停在第一页**，等待你的指令再往下翻。

## 2. 常用操作按键

进入 `more` 界面后，你需要用键盘控制翻页：

|按键|功能|记忆口诀|
|---|---|---|
|**空格键 (Space)**|向下翻**一整页**|"大步往下走"|
|**回车键 (Enter)**|向下滚**一行**|"小心慢走"|
|**b**|往回翻**一页** (Back)|"Back 回去"|
|**q**|**退出**查看 (Quit)|"Quit 走人"|
|**=**|显示当前行号|"等于第几行"|
|**h**|显示帮助|"Help"|

_(注：部分极简版 `more` 可能不支持 `b` 回翻，这是它不如 `less` 的地方)_runoob+1​

## 3. 常用命令格式

## 基础用法

bash

`more 文件名`

_示例_：`more /var/log/syslog`

## 配合管道符（最常用）

当你执行命令（如 `ls` 或 `cat`）输出太长时，用 `more` 接住它。

bash

`ls -l /etc | more`

_解释_：列出 /etc 下所有文件，用 more 分页显示。

## 进阶技巧

- **从第 n 行开始看**：
    
    bash
    
    `more +100 file.txt`
    
    _作用_：跳过前 99 行，直接从第 100 行开始显示 。wangchujiang+1​
    
- **搜索关键词**：
    
    bash
    
    `more +/Error log.txt`
    
    _作用_：打开文件后，直接跳转到第一个包含 "Error" 的地方 。comate.baidu+1​
    
- **压缩空行**：
    
    bash
    
    `more -s file.txt`
    
    _作用_：如果文件里有连续很多空行，显示时把它们合并成一个空行，看着更紧凑 。[wangchujiang](https://wangchujiang.com/linux-command/c/more.html)​
    

## 总结

`more` 是查看长文本的基础工具。虽然 `less` 功能更全（支持随意上下滚动、高亮搜索），但在资源受限或未安装 `less` 的环境里，`more` 依然是你的好帮手。

1. [http://www.runoob.com/linux/linux-comm-more.html](http://www.runoob.com/linux/linux-comm-more.html)
2. [https://blog.csdn.net/AJLLOVE/article/details/143302575](https://blog.csdn.net/AJLLOVE/article/details/143302575)
3. [https://wangchujiang.com/linux-command/c/more.html](https://wangchujiang.com/linux-command/c/more.html)
4. [https://www.cnblogs.com/hcgk/p/18937269](https://www.cnblogs.com/hcgk/p/18937269)
5. [https://blog.csdn.net/weixin_56303229/article/details/143795001](https://blog.csdn.net/weixin_56303229/article/details/143795001)
6. [https://lanlan2017.github.io/blog/65cb3430/](https://lanlan2017.github.io/blog/65cb3430/)
7. [https://initroot.com/tutorial/linux/file/linuxfileviewmoreless.html](https://initroot.com/tutorial/linux/file/linuxfileviewmoreless.html)
8. [https://comate.baidu.com/zh/page/stttjivjdua](https://comate.baidu.com/zh/page/stttjivjdua)
9. [https://c.biancheng.net/linux/jlw6qph.html](https://c.biancheng.net/linux/jlw6qph.html)
10. [https://developer.aliyun.com/article/612848](https://developer.aliyun.com/article/612848)