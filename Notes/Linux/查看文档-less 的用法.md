`less` 是一款比 `more` 更强大的**文件阅读工具**。它的名字来源于“Less is more”（少即是多），但功能上它其实是 `more` 的“超级增强版”。

它最大的特点是**支持随意上下翻页**，而且在打开大文件时不会一次性加载整个文件（速度极快），非常适合查看几百兆甚至几个 G 的日志文件 。csdn+1​

## 1. 核心优势

- **随意回滚**：不仅能往下翻，还能往回翻（这是 `more` 的痛点）。
    
- **秒开大文件**：它只加载你屏幕上看到的这部分内容，不像编辑器那样把整个文件读入内存。
    
- **强大的搜索**：支持类似 `vi` 编辑器的搜索体验（高亮显示）。
    

## 2. 常用操作按键（类似 Vim）

|功能|按键|记忆提示|
|---|---|---|
|**退出**|`q`|Quit|
|**向下翻一页**|`空格` 或 `f`|Forward|
|**向上翻一页**|`b`|Back|
|**向下滚一行**|`j` 或 `回车`|Vim 风格|
|**向上滚一行**|`k`|Vim 风格|
|**跳到文件头**|`g`|Go top|
|**跳到文件尾**|`G`|Go bottom|
|**搜索内容**|`/关键词`|比如 `/error`，按 `n` 找下一个，`N` 找上一个|
|**显示行号**|`-N` (启动时)|`less -N`|

## 3. 实用技巧

## 实时监控日志（类似 tail -f）

在 `less` 运行中，按下 **`F`** (Shift+f) 键，它就会进入“实时滚动模式”，一旦文件有新内容写入，屏幕会自动滚动更新。

- **退出监控**：按 `Ctrl + C` 回到普通查看模式。
    
- **场景**：正在排查问题，既想看历史日志，又想盯着最新日志。
    

## 配合管道符

这是最常见的用法，用于查看长命令的输出。

bash

`ps -ef | less # 或者 dpkg -l | less`

## 在查看时才想起来要行号？

如果你已经打开了文件，但忘了加 `-N` 参数：  
直接在 `less` 界面里输入 `-N` 然后回车，行号就会显示出来（再次输入可关闭）。

## 总结

在 Linux 系统管理中，**`less` 是查看文件的首选工具**。除非环境非常简陋只有 `more`，否则建议统统习惯使用 `less`。

1. [https://envader.plus/article/292](https://envader.plus/article/292)
2. [https://atmarkit.itmedia.co.jp/ait/articles/1702/09/news031.html](https://atmarkit.itmedia.co.jp/ait/articles/1702/09/news031.html)
3. [https://popinsight.jp/blog/?p=12235](https://popinsight.jp/blog/?p=12235)
4. [https://blog.csdn.net/weixin_43840640/article/details/98937639](https://blog.csdn.net/weixin_43840640/article/details/98937639)
5. [https://blog.csdn.net/lanlangaogao/article/details/125539792](https://blog.csdn.net/lanlangaogao/article/details/125539792)
6. [https://raspi.taneyats.com/entry/command-less](https://raspi.taneyats.com/entry/command-less)
7. [https://qiita.com/Tri_Engi/items/fc3e725c5280eb0d1d9d](https://qiita.com/Tri_Engi/items/fc3e725c5280eb0d1d9d)
8. [https://tech-lab.sios.jp/archives/32185](https://tech-lab.sios.jp/archives/32185)
9. [https://www.runoob.com/linux/linux-comm-less.html](https://www.runoob.com/linux/linux-comm-less.html)
10. [https://atmarkit.itmedia.co.jp/ait/articles/1702/10/news022.html](https://atmarkit.itmedia.co.jp/ait/articles/1702/10/news022.html)