# lazyvim-agent

> [English](README.md)

一个 Claude Code sub-agent，专门用于诊断 neovim + LazyVim 的配置问题。

## 解决什么问题

修改 LazyVim 行为时，最大的困惑是：**这个行为到底是谁定义的？**

LazyVim 将配置分成多个层级叠加，任意一层都可以定义或覆盖按键、选项和行为：

```
┌─────────────────────────────────────────────────┐
│           用户配置 (~/.config/nvim/)              │  ← 唯一安全的修改位置
├─────────────────────────────────────────────────┤
│         LazyVim extras (lazyvim.json)            │  ← 可选功能包
├─────────────────────────────────────────────────┤
│      LazyVim 框架 (lazy/LazyVim/)                │  ← 框架默认值
├─────────────────────────────────────────────────┤
│         插件源码 (lazy/<plugin>/)                │  ← 上游行为
├─────────────────────────────────────────────────┤
│               neovim 内置默认值                   │  ← 基础层
└─────────────────────────────────────────────────┘
              优先级：上层覆盖下层
```

一个按键如 `gd` 可能在任意层被定义，也可能在中途被某层覆盖。不清楚行为的来源，随手写的修改可能与下层冲突，或被上层悄悄覆盖。

这个 agent 能追踪任意行为的来源，并给出只修改用户配置的精确方案。

## 安装

```bash
git clone https://github.com/shenyfg/lazyvim-agent ~/.local/share/lazyvim-agent
mkdir -p ~/.claude/agents
ln -sf ~/.local/share/lazyvim-agent/agents/lazyvim.md ~/.claude/agents/lazyvim.md
```

或者直接 symlink 本地路径：

```bash
mkdir -p ~/.claude/agents
ln -sf /path/to/lazyvim-agent/agents/lazyvim.md ~/.claude/agents/lazyvim.md
```

## 使用

在 Claude Code 中直接提问，agent 会被自动委派：

```
按下 gd 时发生了什么？
为什么 tab 宽度是 2？
我想禁用打开文件时自动跳到上次位置
<leader>cf 是哪个插件定义的？
```

## 工作原理

agent 按配置层级从高到低搜索：

```
用户配置 (~/.config/nvim/)
  └── LazyVim extras (lazyvim.json 中启用的)
        └── LazyVim 框架 (~/.local/share/nvim/lazy/LazyVim/)
              └── 插件源码 (~/.local/share/nvim/lazy/<plugin>/)
                    └── neovim 内置默认值
```

找到来源后，给出对应的修改代码，写到正确的用户配置文件中。

## 项目结构

```
lazyvim-agent/
├── README.md          # English
├── README.zh.md       # 中文
└── agents/
    └── lazyvim.md     # agent 定义（frontmatter + system prompt）
```
