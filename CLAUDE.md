# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个 Obsidian 学术笔记知识库，所有内容均为 Markdown 格式。主要包含 LLM/扩散模型、Docker、Linux、应用数学等领域的学习笔记。

## 交流规则

- 使用中文回复
- 简洁明了，直接给出答案
- 代码、文件路径、命令保留英文

## 目录结构

```
MyVault/
├── Notes/                    # 学术笔记主目录
│   ├── LLM/                  # 大语言模型相关（扩散模型、Transformer等）
│   ├── Docker/               # Docker 容器技术
│   ├── Linux/                # Linux 命令和系统知识
│   ├── 任性科技/             # 小波变换、VAE 研究
│   ├── 知识/                 # 通用技术知识
│   ├── 健康/                 # 健康相关笔记
│   └── 存档/                 # 研一期末复习资料（应用数学、图像处理等）
├── Excalidraw/               # 绘图文件（跳过格式化）
└── .obsidian/                # Obsidian 配置
```

## Markdown 格式化规则

### 核心原则

格式化只改格式，不改内容。除非特别要求，保持原始语义完整。

### 标题规范

- 禁止 H1 标题（`#`）：Obsidian 显示文件名为标题，正文使用 `##` 起始
- 如发现 H1，降级为 H2

### 内容规范

- 删除加粗标记 `**`（除非明确要求保留）
- 删除文件末尾的 URL 参考链接列表
- 清理无关数字乱码（如 11111、+4、77777）
- 不需要 YAML Front Matter 和 TOC

### 格式化标记

完成格式化后，在文件末尾添加：
```markdown
---

**<font color="#2ecc71">✅ 已格式化</font>**
```

已有此标记的文件无需再次格式化。

### Obsidian 特殊语法

- 保留 `[[WikiLink]]` 内部链接
- 保留图片嵌入 `![[图片名.png]]`（不要删除 `!`）

### LaTeX 公式与颜色

**正确做法**：LaTeX 内部使用 `\color{}`
```markdown
<font color="#00b0f0">文字</font> $\color{#00b0f0}{\epsilon}$ <font color="#00b0f0">更多文字</font>
```

**禁止**：font 标签包裹 LaTeX 公式（会导致渲染失败）
```markdown
<!-- 错误 -->
<font color="#00b0f0">公式 $\epsilon$ 无法渲染</font>
```

### 表格中的 LaTeX

表格单元格内的 LaTeX 绝对值符号 `|` 会被误认为列分隔符，需改为 `\mid`：
```markdown
<!-- 错误 -->
| $1 - |p-q|$ |

<!-- 正确 -->
| $1 - \mid p-q \mid$ |
```

## 格式化工作流程

1. 检查文件末尾是否已有格式化标记，有则跳过
2. 检查并修复 font 标签包裹 LaTeX 的问题
3. H1 降级为 H2
4. 删除加粗、参考链接、乱码
5. 添加格式化标记

## Git 操作

格式化前先备份：
```bash
git add Notes/ Excalidraw/
git commit -m "Backup: $(date '+%Y-%m-%d %H:%M:%S') 新增笔记提交"
git push
```

格式化完成后提交：
```bash
git add Notes/
git commit -m "Format: 格式化 XXX.md"
```

## 跳过的文件

- `*.excalidraw.md` - Excalidraw 绘图文件
- `*.bak` - 备份文件
- `.obsidian/` - Obsidian 配置目录
