`apt update` 和 `apt upgrade` 是两个经常一起使用但功能截然不同的命令。简单来说：**`update` 是为了“获取信息”，`upgrade` 是为了“执行行动”。**

## 1. `sudo apt update`（更新列表）

- **作用**：它**不安装**任何软件，也**不升级**任何现有软件。
    
- **机制**：它会访问 `/etc/apt/sources.list` 里配置的软件源服务器，下载最新的**“软件包清单”**到你的本地电脑 。csdn+1​
    
- **目的**：让你的系统知道“现在世界上有哪些软件是新的，版本号是多少，在哪里下载”。如果不运行这一步，你的电脑就以为老的软件还是最新的 。[51cto](https://www.51cto.com/article/717893.html)​
    

## 2. `sudo apt upgrade`（升级软件）

- **作用**：它会**真正地下载并安装**新版本的软件。
    
- **机制**：它会对比你本地已安装的软件版本和刚才通过 `update` 获取到的最新清单。如果发现某个软件在清单里有更新的版本，它就会把那个新版本下载下来并安装覆盖旧版本 。pingcode+1​
    
- **前提**：必须先运行 `update`，否则 `upgrade` 根本不知道有哪些软件需要更新 。[reddit](https://www.reddit.com/r/linux4noobs/comments/zftf5d/sudo_apt_update_sudo_apt_grade_explanation_im/)​
    

## 3. 一个生活的比喻

想象你去逛超市（你的 Linux 系统）：

- **`apt update`**：就像是你**拿起当天的特价传单看了一遍**。你现在知道了今天的猪肉降价了，苹果出新品种了。但此时你的购物车还是空的，你什么都没买。
    
- **`apt upgrade`**：就像是你**根据传单去货架上拿东西**。因为你看过传单（运行过 update），所以你径直走向了打折区，把旧的商品换成了新的。
    

## 总结

|命令|行为|作用|结果|
|---|---|---|---|
|`apt update`|下载文件清单|检查更新|知道有哪些新东西，但系统软件没变|
|`apt upgrade`|下载并安装软件|执行更新|系统里的软件变成了新版本|

**最佳实践**：  
永远组合使用它们，先检查后升级：

bash

`sudo apt update && sudo apt upgrade`

（`&&` 表示前半句成功了才执行后半句）

1. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/152616778/3d949d18-d551-4bfe-84b6-4a61b78092f0/image.jpg](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/152616778/3d949d18-d551-4bfe-84b6-4a61b78092f0/image.jpg)
2. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/152616778/eee834de-f5bc-425c-b002-f21d3ffd6d11/image.jpg](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/152616778/eee834de-f5bc-425c-b002-f21d3ffd6d11/image.jpg)
3. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/152616778/b50fb15f-bc94-4369-b397-61da335e9f45/image.jpg](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/152616778/b50fb15f-bc94-4369-b397-61da335e9f45/image.jpg)
4. [https://blog.csdn.net/MonsterUFO/article/details/132999560](https://blog.csdn.net/MonsterUFO/article/details/132999560)
5. [https://www.51cto.com/article/717893.html](https://www.51cto.com/article/717893.html)
6. [https://blog.csdn.net/taoxicun/article/details/122902625](https://blog.csdn.net/taoxicun/article/details/122902625)
7. [https://docs.pingcode.com/ask/46357.html](https://docs.pingcode.com/ask/46357.html)
8. [https://juejin.cn/post/7140168136166211620](https://juejin.cn/post/7140168136166211620)
9. [https://www.reddit.com/r/linux4noobs/comments/zftf5d/sudo_apt_update_sudo_apt_grade_explanation_im/](https://www.reddit.com/r/linux4noobs/comments/zftf5d/sudo_apt_update_sudo_apt_grade_explanation_im/)
10. [https://www.cnblogs.com/zxdplay/p/16780243.html](https://www.cnblogs.com/zxdplay/p/16780243.html)
11. [https://qiita.com/Toda-KAN/items/8e0d4630145590c9bd95](https://qiita.com/Toda-KAN/items/8e0d4630145590c9bd95)
12. [https://nakaterux.hatenablog.com/entry/2024/09/17/092358](https://nakaterux.hatenablog.com/entry/2024/09/17/092358)
13. [https://www.freecodecamp.org/japanese/news/sudo-apt-get-update-vs-upgrade-what-is-the-difference/](https://www.freecodecamp.org/japanese/news/sudo-apt-get-update-vs-upgrade-what-is-the-difference/)