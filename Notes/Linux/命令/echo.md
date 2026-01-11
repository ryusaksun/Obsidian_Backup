`echo` 命令就像是 Linux 世界里的"复读机"或"扬声器"。它的最基本功能是在终端屏幕上打印一段文字，但配合其他符号使用时，它能发挥出巨大的威力。

## 1. 基础用法 (Hello World)

最简单的用法，就是把后面的内容打印出来：

```bash
echo "Hello World"
# 输出: Hello World
```

## 2. 核心功能：查看变量

这是运维和开发中最常用的功能，用来查看系统环境变量的值。

```bash
echo $PATH
# 输出系统的 PATH 路径

echo $USER
# 输出当前用户名
```

注意：在 Linux 中，引用变量必须在前面加 `$` 符号。

## 3. 进阶选项 (必会)

### `-n`：不换行

默认情况下，`echo` 输出完会自动换行。如果不想换行（比如做进度条时）：

```bash
echo -n "Loading..."
echo "Done"
# 输出: Loading...Done (在同一行)
```

### `-e`：启用转义字符

让 `echo` 能看懂 `\n` (换行)、`\t` (Tab 缩进)、`\033` (颜色) 等特殊符号。

```bash
# \n 表示换行
echo -e "第一行\n第二行"

# 打印带颜色的文字 (比如红色的 Error)
echo -e "\033[31mError: File not found\033[0m"
```

## 4. 最强组合：配合重定向 (`>` / `>>`)

`echo` 很少单独使用，它通常用来生成文件或修改配置文件。

- 新建文件并写入内容：

```bash
echo "nameserver 8.8.8.8" > /etc/resolv.conf
```

- 解释：把 "nameserver 8.8.8.8" 这句话写入文件，覆盖原有内容。

- 追加内容到文件末尾：

```bash
echo "export JAVA_HOME=/usr/local/java" >> ~/.bashrc
```

- 解释：在配置文件的最后加一行，不破坏已有内容。

## 总结

- 看变量：`echo $VAR`
- 写文件：`echo "text" > file`
- 改配置：`echo "config" >> file`
- 不换行：`echo -n`

---

**<font color="#2ecc71">✅ 已格式化</font>**
