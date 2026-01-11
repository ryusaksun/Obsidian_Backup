`sudo` 是 Linux 中最重要的命令之一，它赋予了普通用户管理系统的能力。

## 1. 名字的含义与来源

- **全称**：**S**uper**U**ser **DO**（超级用户去做）。runoob+1​
    
- **含义**：
    
    - `SuperUser`：指系统中的最高权限者（通常是 root）。
        
    - `DO`：执行动作。
        
    - 合起来就是：“**以超级用户的身份去执行后面的命令**”。
        
- **读音**：通常读作 `/ˈsuːduː/`（"苏-杜"）或者 `/ˈsuːdoʊ/`（"苏-兜"）。
    

## 2. 它解决了什么问题？

在 `sudo` 出现之前，如果你想安装软件或修改系统配置，你必须用 `su` 切换到 root 账户。这样做有两个巨大的风险：

1. **权限过大**：一旦切换成 root，你做的**任何**操作（包括手滑误删文件）都是不可挽回的 。[linux.digibeatrix](https://www.linux.digibeatrix.com/zh/security-and-user-management-zh/sudo-command-complete-guide/)​
    
2. **密码共享**：所有管理员都必须知道 root 的密码。如果一个管理员离职了，就得修改 root 密码并通知所有人，非常麻烦 。csdn+1​
    

`sudo` 完美解决了这两个问题：

- **临时提权**：它只让你的**这一条命令**拥有管理员权限，命令执行完立刻变回普通百姓，大大降低了误操作风险 。[linux.digibeatrix](https://www.linux.digibeatrix.com/zh/security-and-user-management-zh/sudo-command-complete-guide/)​
    
- **审计追踪**：它会记录谁在什么时间执行了什么命令（记在 `/var/log/auth.log` 里），方便事后追责 。[csdn](https://blog.csdn.net/tian830937/article/details/134650610)​
    
- **只需自己的密码**：你只需要输入**你自己的密码**来验证身份，而不需要知道 root 的密码 。[linux.digibeatrix](https://www.linux.digibeatrix.com/zh/security-and-user-management-zh/sudo-command-complete-guide/)​
    

## 3. 一个生动的例子

想象你去银行办理业务：

- **su (切换 root)**：就像你直接**抢了行长的工牌和钥匙**，你可以打开金库、修改任何账户，没人能拦你，但你也可能不小心炸了银行。
    
- **sudo (临时提权)**：就像你**填了一张申请单**，柜员验证了你的身份证（输入你自己的密码），然后帮你按下了那个“转账”按钮。只有这一次操作是特权的，做完你就还是普通客户。
    

1. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/152616778/3d949d18-d551-4bfe-84b6-4a61b78092f0/image.jpg](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/152616778/3d949d18-d551-4bfe-84b6-4a61b78092f0/image.jpg)
2. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/152616778/eee834de-f5bc-425c-b002-f21d3ffd6d11/image.jpg](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/152616778/eee834de-f5bc-425c-b002-f21d3ffd6d11/image.jpg)
3. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/152616778/b50fb15f-bc94-4369-b397-61da335e9f45/image.jpg](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/152616778/b50fb15f-bc94-4369-b397-61da335e9f45/image.jpg)
4. [https://blog.csdn.net/tian830937/article/details/134650610](https://blog.csdn.net/tian830937/article/details/134650610)
5. [http://www.runoob.com/linux/linux-comm-sudo.html](http://www.runoob.com/linux/linux-comm-sudo.html)
6. [https://documentation.suse.com/zh-cn/sle-micro/6.0/html/Micro-sudo-run-commands-as-superuser/index.html](https://documentation.suse.com/zh-cn/sle-micro/6.0/html/Micro-sudo-run-commands-as-superuser/index.html)
7. [https://wangchujiang.com/linux-command/c/sudo.html](https://wangchujiang.com/linux-command/c/sudo.html)
8. [https://www.linux.digibeatrix.com/zh/security-and-user-management-zh/sudo-command-complete-guide/](https://www.linux.digibeatrix.com/zh/security-and-user-management-zh/sudo-command-complete-guide/)
9. [https://my.oschina.net/emacs_8799256/blog/17290970](https://my.oschina.net/emacs_8799256/blog/17290970)
10. [https://cloud.tencent.com/developer/article/2393293](https://cloud.tencent.com/developer/article/2393293)
11. [https://docs.redhat.com/zh-cn/documentation/red_hat_enterprise_linux/9/html/configuring_basic_system_settings/managing-sudo-access_configuring-basic-system-settings](https://docs.redhat.com/zh-cn/documentation/red_hat_enterprise_linux/9/html/configuring_basic_system_settings/managing-sudo-access_configuring-basic-system-settings)
12. [https://blog.csdn.net/Amentos/article/details/129291596](https://blog.csdn.net/Amentos/article/details/129291596)