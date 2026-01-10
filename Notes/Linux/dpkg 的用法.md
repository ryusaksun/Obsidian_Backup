`dpkg` 是 **Debian Package** 的缩写，它是 Ubuntu 和 Debian 系统中最底层的包管理工具。你可以把它理解为这些 Linux 系统的“安装包管理器”核心 。e-words+1​

简单来说，它的地位类似于：

- **在 Windows 中**：双击 `.exe` 或 `.msi` 安装包时，负责把文件解压并复制到系统文件夹的那个底层程序。
    
- **在 RedHat/CentOS 中**：`rpm` 命令。
    

## dpkg 和 apt 的区别

新手常搞混这两个命令。

- **`dpkg` (底层工具)**：
    
    - **能做**：安装本地的 `.deb` 文件、查看已安装软件列表、删除软件。
        
    - **不能做**：**自动下载软件**、**自动解决依赖关系**。如果你装 A 软件需要 B 库，用 dpkg 装 A 会直接报错告诉你缺 B，它不会帮你去下载 B 。marusuke-blog+1​
        
- **`apt` (高层工具)**：
    
    - 它是基于 `dpkg` 构建的智能管家。当你运行 `apt install A`，它会自动从网上下载 A 和 B，然后调用 `dpkg` 把它们按顺序装好 。[marusuke-blog](https://marusuke-blog.com/linux-command-dpkg/)​
        

## 什么时候必须用 dpkg？

虽然平时 99% 的情况都用 `apt`，但在以下场景你必须用 `dpkg`：

1. **安装下载好的 `.deb` 文件**：如果你从官网（比如 Google Chrome、VS Code）下载了一个安装包文件（不是通过商店），需要用 `dpkg -i` 来安装。
    
2. **查询本地已安装软件的详细列表**：`apt` 的列表显示有时不够直观，`dpkg -l` 更常被用来做精准排查。
    

## 常用命令速查

- **列出已安装软件**：`dpkg -l`[linux.joho](https://linux.joho.info/command/dpkg/)​
    
- **安装本地安装包**：`sudo dpkg -i 软件包名.deb`[marusuke-blog](https://marusuke-blog.com/linux-command-dpkg/)​
    
- **查看某软件装在哪里**：`dpkg -L 软件名`[linux.joho](https://linux.joho.info/command/dpkg/)​
    
- **查看某个文件属于哪个软件**：`dpkg -S /path/to/file`[linux.joho](https://linux.joho.info/command/dpkg/)​
    

1. [https://e-words.jp/w/dpkg.html](https://e-words.jp/w/dpkg.html)
2. [https://eng-entrance.com/linux-package-dpkg](https://eng-entrance.com/linux-package-dpkg)
3. [https://qiita.com/miyuki_samitani/items/a49f955ada17b4c75c1d](https://qiita.com/miyuki_samitani/items/a49f955ada17b4c75c1d)
4. [https://manpages.ubuntu.com/manpages/trusty/ja/man1/dpkg.1.html](https://manpages.ubuntu.com/manpages/trusty/ja/man1/dpkg.1.html)
5. [https://linux.joho.info/command/dpkg/](https://linux.joho.info/command/dpkg/)
6. [https://marusuke-blog.com/linux-command-dpkg/](https://marusuke-blog.com/linux-command-dpkg/)
7. [https://chaba-lab.com/command-dpkg-20250127/](https://chaba-lab.com/command-dpkg-20250127/)
8. [https://ava.hosting/ja/faq/a-guide-to-using-the-dpkg-command-on-linux/](https://ava.hosting/ja/faq/a-guide-to-using-the-dpkg-command-on-linux/)
9. [https://www.infraexpert.com/infra/linux08.html](https://www.infraexpert.com/infra/linux08.html)
10. [https://kusanagi.dht-jpn.co.jp/2020/02/linux%E3%81%AE%E3%83%91%E3%83%83%E3%82%B1%E3%83%BC%E3%82%B8%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0%E3%82%92%E7%9F%A5%E3%82%8D%E3%81%86%E7%AC%AC2%E5%9B%9E%E5%AE%AE%E5%B4%8E%E6%82%9F%E6%B0%8F/](https://kusanagi.dht-jpn.co.jp/2020/02/linux%E3%81%AE%E3%83%91%E3%83%83%E3%82%B1%E3%83%BC%E3%82%B8%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0%E3%82%92%E7%9F%A5%E3%82%8D%E3%81%86%E7%AC%AC2%E5%9B%9E%E5%AE%AE%E5%B4%8E%E6%82%9F%E6%B0%8F/)