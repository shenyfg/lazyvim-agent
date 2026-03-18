# lazyvim-agent

> [中文版](README.zh.md)

A Claude Code sub-agent for diagnosing neovim + LazyVim configuration issues.

## Problem

When tweaking LazyVim behavior, the hardest question is: **where does this behavior come from?**

LazyVim stacks multiple layers of configuration, and any layer can define or override a keymap, option, or behavior:

```
┌─────────────────────────────────────────────────┐
│            Your config (~/.config/nvim/)         │  ← only safe place to edit
├─────────────────────────────────────────────────┤
│         LazyVim extras (lazyvim.json)            │  ← optional feature packs
├─────────────────────────────────────────────────┤
│      LazyVim framework (lazy/LazyVim/)           │  ← opinionated defaults
├─────────────────────────────────────────────────┤
│         Plugin source (lazy/<plugin>/)           │  ← upstream behavior
├─────────────────────────────────────────────────┤
│            neovim built-in defaults              │  ← baseline
└─────────────────────────────────────────────────┘
         priority: higher layer wins
```

A single keypress like `gd` could be defined at any of these layers — or overridden halfway up the stack. Without knowing exactly which layer owns a behavior, any fix you write may silently conflict with a lower layer or get shadowed by a higher one.

This agent traces the source of any behavior and gives you a precise fix that only touches your user config.

## Installation

```bash
git clone https://github.com/shenyfg/lazyvim-agent ~/.local/share/lazyvim-agent
mkdir -p ~/.claude/agents
ln -sf ~/.local/share/lazyvim-agent/agents/lazyvim.md ~/.claude/agents/lazyvim.md
```

Or symlink a local path directly:

```bash
mkdir -p ~/.claude/agents
ln -sf /path/to/lazyvim-agent/agents/lazyvim.md ~/.claude/agents/lazyvim.md
```

## Usage

Ask questions directly in Claude Code — the agent is dispatched automatically:

```
What happens when I press gd?
Why is tab width 2?
I want to disable restoring cursor position on file open
Which plugin defines <leader>cf?
```

## How It Works

The agent searches the config stack from highest to lowest priority:

```
User config (~/.config/nvim/)
  └── LazyVim extras (enabled in lazyvim.json)
        └── LazyVim framework (~/.local/share/nvim/lazy/LazyVim/)
              └── Plugin source (~/.local/share/nvim/lazy/<plugin>/)
                    └── neovim built-in defaults
```

Once the source is found, it generates the correct override code and tells you exactly which file to put it in.

## Project Structure

```
lazyvim-agent/
├── README.md          # English
├── README.zh.md       # 中文
└── agents/
    └── lazyvim.md     # agent definition (frontmatter + system prompt)
```
