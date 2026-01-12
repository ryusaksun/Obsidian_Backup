`usermod` 是 User Modify（用户修改）的缩写，是 Linux 中用于修改现有用户账户属性的核心命令。当创建用户后，你需要调整其权限、主目录、Shell 类型或过期时间等属性时，就需要用到它。

该命令的基本语法为：
`usermod [选项] 用户名`

## 核心应用场景与示例

| 场景 | 命令示例 | 详细说明 |
| --- | --- | --- |
| 赋予管理员权限 | `sudo usermod -aG sudo ryuichi` | 将用户加入到 `sudo` 组中，这是最常用的用法。 重要：必须使用 `-aG`（append + groups）而不是 `-G`，否则用户会被踢出之前所在的所有其他组。 |
| 锁定/解锁账户 | `sudo usermod -L ryuichi` <br>`sudo usermod -U ryuichi` | `-L` (Lock) 会在密码前加感叹号，禁止用户密码登录（常用于员工离职或安全封锁）；`-U` (Unlock) 则是解锁。 |
| 修改默认 Shell | `sudo usermod -s /bin/zsh ryuichi` | 将用户的默认 Shell 改为 Zsh 或 Bash。如果你安装了 Oh-My-Zsh，就需要这一步。 |
| 修改主目录 | `sudo usermod -d /new/home -m ryuichi` | `-d` 指定新路径，`-m` (move) 告诉系统把旧主目录里的文件自动搬过去。如果不加 `-m`，新目录会是空的。 |
| 修改用户名 | `sudo usermod -l new_name old_name` | `-l` (login) 用于重命名账户。注意：修改用户名前，该用户必须已完全退出登录（不能有任何运行中的进程）。 |

## 关键参数速查表

- `-a` (Append): 追加模式。仅与 `-G` 配合使用，表示"把用户加到这个新组，但保留旧组"。如果不加 `-a`，用户就会只属于新组，旧组全没了！

- `-G` (Groups): 指定附加组（Secondary Groups）。

- `-g` (Group): 修改主组（Primary Group）。通常不建议修改主组，除非你非常清楚文件归属权的影响。

- `-u` (UID): 修改用户的数字 ID。

- `-e` (Expire): 设置账户的过期日期（格式 `YYYY-MM-DD`），常用于临时账户。

## 注意事项

- 权限要求：必须以 `root` 身份或使用 `sudo` 才能执行。

- 登录状态：在修改用户的 UID、用户名或主目录时，该用户必须未登录，且没有运行任何进程。如果强行修改可能会报错 `user is currently used by process 1234`。

---

**<font color="#2ecc71">✅ 已格式化</font>**
