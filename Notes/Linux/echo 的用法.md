`echo` 命令就像是 Linux 世界里的“复读机”或“扬声器”。它的最基本功能是**在终端屏幕上打印一段文字**，但配合其他符号使用时，它能发挥出巨大的威力。

## 1. 基础用法 (Hello World)

最简单的用法，就是把后面的内容打印出来：

bash

`echo "Hello World" # 输出: Hello World`

## 2. 核心功能：查看变量

这是运维和开发中最常用的功能，用来查看系统环境变量的值。

bash

`echo $PATH # 输出系统的 PATH 路径 echo $USER # 输出当前用户名`

_注意_：在 Linux 中，引用变量必须在前面加 `$` 符号 。eng-entrance+1​

## 3. 进阶选项 (必会)

## `-n`：不换行

默认情况下，`echo` 输出完会自动换行。如果不想换行（比如做进度条时）：

bash

`echo -n "Loading..." echo "Done" # 输出: Loading...Done (在同一行)`

## `-e`：启用转义字符

让 `echo` 能看懂 `\n` (换行)、`\t` (Tab 缩进)、`\033` (颜色) 等特殊符号 。runoob+1​

bash

`# \n 表示换行 echo -e "第一行\n第二行" # 打印带颜色的文字 (比如红色的 Error) echo -e "\033[31mError: File not found\033[0m"`

## 4. 最强组合：配合重定向 (`>` / `>>`)

`echo` 很少单独使用，它通常用来**生成文件**或**修改配置文件**。

- **新建文件并写入内容**：
    
    bash
    
    `echo "nameserver 8.8.8.8" > /etc/resolv.conf`
    
    _解释_：把 "nameserver 8.8.8.8" 这句话写入文件，**覆盖**原有内容。
    
- **追加内容到文件末尾**：
    
    bash
    
    `echo "export JAVA_HOME=/usr/local/java" >> ~/.bashrc`
    
    _解释_：在配置文件的最后加一行，**不破坏**已有内容 。qiita+1​
    

## 总结

- **看变量**：`echo $VAR`
    
- **写文件**：`echo "text" > file`
    
- **改配置**：`echo "config" >> file`
    
- **不换行**：`echo -n`
    

1. [https://qiita.com/empty948/items/6031785e23d8fdbb1a5a](https://qiita.com/empty948/items/6031785e23d8fdbb1a5a)
2. [https://eng-entrance.com/linux-command-echo](https://eng-entrance.com/linux-command-echo)
3. [https://www.sejuku.net/blog/50830](https://www.sejuku.net/blog/50830)
4. [http://www.runoob.com/linux/linux-shell-echo.html](http://www.runoob.com/linux/linux-shell-echo.html)
5. [https://unicornee.ai/articles/echo-command-complete-guide/](https://unicornee.ai/articles/echo-command-complete-guide/)
6. [https://envader.plus/course/1/scenario/1006](https://envader.plus/course/1/scenario/1006)
7. [https://atmarkit.itmedia.co.jp/ait/articles/1705/26/news013.html](https://atmarkit.itmedia.co.jp/ait/articles/1705/26/news013.html)
8. [https://bashdo.com/command/echo/%E5%88%9D%E5%BF%83%E8%80%85%E5%90%91%E3%81%91bash-shell-echo%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89%E3%82%92%E4%BD%BF%E3%81%84%E3%81%93%E3%81%AA%E3%81%99%E6%96%B9%E6%B3%95%E3%81%A8%E6%B4%BB%E7%94%A8/](https://bashdo.com/command/echo/%E5%88%9D%E5%BF%83%E8%80%85%E5%90%91%E3%81%91bash-shell-echo%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89%E3%82%92%E4%BD%BF%E3%81%84%E3%81%93%E3%81%AA%E3%81%99%E6%96%B9%E6%B3%95%E3%81%A8%E6%B4%BB%E7%94%A8/)
9. [https://www.miraclejob.com/recommend/detail?cd=4022](https://www.miraclejob.com/recommend/detail?cd=4022)