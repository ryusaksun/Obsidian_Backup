`grep`（Global Regular Expression Print）是 Linux 系统中最强大的文本搜索工具。它的核心功能是：在文件中查找符合你要求的文本，并把这些行打印出来。

## 1. 基础语法

```bash
grep [选项] "关键词" 文件名
```

- 示例：`grep "error" server.log`
    - 在 `server.log` 文件中查找包含 "error" 的所有行。

## 2. 核心常用选项 (必记)

| 选项 | 作用 | 记忆场景 |
|---|---|---|
| -i | 忽略大小写 (Ignore case) | 找 "Error" 时也能搜到 "error" |
| -v | 反向查找 (Invert) | "除了这行我全都要"（剔除干扰行） |
| -n | 显示行号 (Number) | 告诉我代码在哪一行 |
| -r | 递归搜索 (Recursive) | 在整个项目文件夹里搜某个变量名 |
| -c | 统计行数 (Count) | 只需要知道出现了几次，不想看内容 |
| --color | 高亮显示 | 让搜到的关键词变红，一眼看到 |

## 3. 实战场景与组合拳

### 场景一：剔除干扰信息 (配合 `-v`)

这是最经典的用法，特别是在查看进程时。

```bash
ps -ef | grep python | grep -v grep
```

- 第一步：`ps -ef` 列出所有进程。
- 第二步：`grep python` 筛选出包含 python 的行（但这把 `grep python` 这个命令自己也算进去了）。
- 第三步：`grep -v grep` 把包含 "grep" 字样的行剔除掉，只剩下真正的 python 进程。

### 场景二：在整个目录里找代码 (配合 `-r`)

```bash
grep -r "db_password" /var/www/html/
```

- 作用：在 `/var/www/html/` 目录下及其所有子文件夹中，查找哪个文件包含了 "db_password"。
- 输出：它会显示 `文件名:匹配内容`，帮你快速定位配置文件。

### 场景三：查看日志的上下文 (配合 `-A` / `-B`)

报错信息通常不仅在报错的那一行，前后的日志也很重要。

- -A (After)：显示匹配行之后的 n 行。
- -B (Before)：显示匹配行之前的 n 行。
- -C (Context)：显示前后各 n 行。

```bash
grep -C 5 "Exception" app.log
```

- 作用：找到 "Exception" 发生的那一行，并把它的前后 5 行都打印出来，方便排查原因。

## 4. 正则表达式入门

`grep` 之所以强大，是因为支持正则表达式（配合 `-E` 选项更佳）：

- `^` (行首)：`grep "^root" /etc/passwd` （找以 root 开头的行）
- `$` (行尾)：`grep "bash$" /etc/passwd` （找以 bash 结尾的行）
- `.` (任意字符)：`grep "r..t"` （找 r 开头，t 结尾，中间任意两个字符的单词）

## 总结

- 简单搜：`grep "text" file`
- 搜目录：`grep -r "text" dir`
- 反向搜：`grep -v "text" file`
- 搜进程：`ps -ef | grep proc`
- 看上下文：`grep -C 5 "text" file`

---

**<font color="#2ecc71">✅ 已格式化</font>**
