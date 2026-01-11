`usermod` 是 **User Modify**（用户修改）的缩写，是 Linux 中用于修改现有用户账户属性的核心命令 。当创建用户后，你需要调整其权限、主目录、Shell 类型或过期时间等属性时，就需要用到它。csdn+1​

该命令的基本语法为：  
`usermod [选项] 用户名`[zzxworld](https://www.zzxworld.com/posts/linux-usermod-command-guide)​

## 1. 核心应用场景与示例

|场景|命令示例|详细说明|
|---|---|---|
|**赋予管理员权限**|`sudo usermod -aG sudo ryuichi`|将用户加入到 `sudo` 组中，这是**最常用的用法**。  <br>**重要：** 必须使用 `-aG`（append + groups）而不是 `-G`，否则用户会被**踢出**之前所在的所有其他组 zzxworld+1​。|
|**锁定/解锁账户**|`sudo usermod -L ryuichi`  <br>`sudo usermod -U ryuichi`|`-L` (Lock) 会在密码前加感叹号，禁止用户密码登录（常用于员工离职或安全封锁）；`-U` (Unlock) 则是解锁 csdn+1​。|
|**修改默认 Shell**|`sudo usermod -s /bin/zsh ryuichi`|将用户的默认 Shell 改为 Zsh 或 Bash。如果你安装了 Oh-My-Zsh，就需要这一步 [zzxworld](https://www.zzxworld.com/posts/linux-usermod-command-guide)​。|
|**修改主目录**|`sudo usermod -d /new/home -m ryuichi`|`-d` 指定新路径，`-m` (move) 告诉系统把旧主目录里的文件自动搬过去。如果不加 `-m`，新目录会是空的 csdn+1​。|
|**修改用户名**|`sudo usermod -l new_name old_name`|`-l` (login) 用于重命名账户。注意：修改用户名前，该用户必须已完全退出登录（不能有任何运行中的进程）juejin+1​。|

## 2. 关键参数速查表

- `-a` (Append): **追加**模式。仅与 `-G` 配合使用，表示“把用户加到这个新组，但保留旧组”。如果不加 `-a`，用户就会只属于新组，旧组全没了！csdn+1​
    
- `-G` (Groups): 指定**附加组**（Secondary Groups）。
    
- `-g` (Group): 修改**主组**（Primary Group）。通常不建议修改主组，除非你非常清楚文件归属权的影响 。juejin+1​
    
- `-u` (UID): 修改用户的数字 ID。
    
- `-e` (Expire): 设置账户的过期日期（格式 `YYYY-MM-DD`），常用于临时账户 。wangchujiang+1​
    

## 3. 注意事项

- **权限要求**：必须以 `root` 身份或使用 `sudo` 才能执行 。[csdn](https://blog.csdn.net/lisanmengmeng/article/details/146144022)​
    
- **登录状态**：在修改用户的 UID、用户名或主目录时，该用户**必须未登录**，且没有运行任何进程。如果强行修改可能会报错 `user is currently used by process 1234` 。cloud.tencent+1​
    

1. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/152616778/3d949d18-d551-4bfe-84b6-4a61b78092f0/image.jpg](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/152616778/3d949d18-d551-4bfe-84b6-4a61b78092f0/image.jpg)
2. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/152616778/eee834de-f5bc-425c-b002-f21d3ffd6d11/image.jpg](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/152616778/eee834de-f5bc-425c-b002-f21d3ffd6d11/image.jpg)
3. [http://www.runoob.com/linux/linux-comm-usermod.html](http://www.runoob.com/linux/linux-comm-usermod.html)
4. [https://blog.csdn.net/lisanmengmeng/article/details/146144022](https://blog.csdn.net/lisanmengmeng/article/details/146144022)
5. [https://juejin.cn/post/7333278321688920116](https://juejin.cn/post/7333278321688920116)
6. [https://cloud.tencent.com/developer/article/2393287](https://cloud.tencent.com/developer/article/2393287)
7. [https://www.zzxworld.com/posts/linux-usermod-command-guide](https://www.zzxworld.com/posts/linux-usermod-command-guide)
8. [https://blog.csdn.net/longshenlmj/article/details/44080893](https://blog.csdn.net/longshenlmj/article/details/44080893)
9. [https://wangchujiang.com/linux-command/c/usermod.html](https://wangchujiang.com/linux-command/c/usermod.html)
10. [https://blog.csdn.net/qq_34431979/article/details/130903867](https://blog.csdn.net/qq_34431979/article/details/130903867)
11. [https://www.aliyun.com/sswb/257113.html](https://www.aliyun.com/sswb/257113.html)