`.bashrc` 是 Linux 中最常用的 Shell 配置文件之一，全称为 **Bash Run Commands**。

## 1. 名字的由来

这个名字可以拆解为两部分：

- **Bash**: 指的是 **Bourne Again Shell**，是 GNU 项目中对经典 Unix Shell (sh) 的增强版本，也是 Ubuntu 的默认 Shell 。[cnblogs](https://www.cnblogs.com/midworld/p/11006967.html)​
    
- **rc**: 源自 Unix 传统的命名习惯 **"Run Commands"**（运行命令）。
    
    - 这个术语最早追溯到 1965 年的 CTSS 系统，当时的脚本文件被称为 "runcom"。
        
    - 后来 Unix 继承了这一传统，许多配置文件都以 `rc` 结尾（如 `.vimrc`, `.zshrc`），意为“程序启动时需要自动运行的指令集合” 。csdn+1​
        

## 2. 这个文件是做什么的？

`.bashrc` 是一个**脚本文件**，当你每次打开一个新的终端窗口（交互式 Shell）时，系统会自动执行它里面的代码 。它的主要作用是**定制你的命令行环境**，让你使用起来更顺手。cloud.baidu+1​

常见的用途包括：

- **设置别名 (Alias)**：给长命令起个短名字。
    
    - 例如：`alias ll='ls -l'`，这样你以后输入 `ll` 就等于输入了 `ls -l` 。oschina+1​
        
- **配置环境变量**：告诉系统去哪里找软件。
    
    - 例如：`export PATH=$PATH:/my/custom/path`，把你安装软件的目录加到系统路径里 。[csdn](https://blog.csdn.net/Heyyellman/article/details/111565781)​
        
- **美化提示符**：修改命令行前面的那串字符（如颜色、显示当前目录等）。cnblogs+1​
    
- **加载自定义函数**：定义一些你常用的复杂操作函数 。[cnblogs](https://www.cnblogs.com/dclogs/p/18667213)​
    

## 3. 文件在哪里？

系统里通常有两类 `.bashrc`：

1. **用户级** (`~/.bashrc`)：位于你的主目录（`/home/用户名/.bashrc`），只影响你自己的账户。这是你最常修改的文件 。cnblogs+1​
    
2. **系统级** (`/etc/bash.bashrc`)：位于 `/etc` 目录，影响系统里的所有用户。通常由管理员维护，普通用户不应随意修改 。[cnblogs](https://www.cnblogs.com/dclogs/p/18667213)​
    

## 4. 如何让修改生效？

当你修改了 `.bashrc` 文件后，它不会立即生效，因为 Shell 只有在启动时才读取它。你需要运行以下命令来手动刷新：

bash

`source ~/.bashrc`

或者直接关闭当前终端，重新打开一个也可以 。csdn+1​

1. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/152616778/3d949d18-d551-4bfe-84b6-4a61b78092f0/image.jpg](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/152616778/3d949d18-d551-4bfe-84b6-4a61b78092f0/image.jpg)
2. [https://blog.csdn.net/Heyyellman/article/details/111565781](https://blog.csdn.net/Heyyellman/article/details/111565781)
3. [https://www.cnblogs.com/midworld/p/11006967.html](https://www.cnblogs.com/midworld/p/11006967.html)
4. [https://blog.csdn.net/juruiyuan111/article/details/122068561](https://blog.csdn.net/juruiyuan111/article/details/122068561)
5. [https://cloud.baidu.com/article/2825272](https://cloud.baidu.com/article/2825272)
6. [https://www.cnblogs.com/dclogs/p/18667213](https://www.cnblogs.com/dclogs/p/18667213)
7. [https://cloud.tencent.com/developer/article/2479952](https://cloud.tencent.com/developer/article/2479952)
8. [https://my.oschina.net/emacs_8747319/blog/17167604](https://my.oschina.net/emacs_8747319/blog/17167604)
9. [https://cloud.tencent.com/developer/article/1813083](https://cloud.tencent.com/developer/article/1813083)
10. [https://my.oschina.net/emacs_8774087/blog/17231645](https://my.oschina.net/emacs_8774087/blog/17231645)
11. [https://www.reddit.com/r/linux/comments/7oc5mt/what_are_some_useful_things_you_put_on_your/?tl=zh-hans](https://www.reddit.com/r/linux/comments/7oc5mt/what_are_some_useful_things_you_put_on_your/?tl=zh-hans)