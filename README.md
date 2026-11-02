# mxcat-changelog

公开的 GitHub 技能源：提供 `mxcat-changelog`，按统一规范更新项目中的 CHANGELOG，可通过 `npx skills` 安装到 Cursor、Claude Code 等 Agent。

源仓库：https://github.com/maoxiancat/mxcat-changelog

## 安装

项目：

```bash
npx -y skills add maoxiancat/mxcat-changelog --skill mxcat-changelog
```

全局：

```bash
npx -y skills add maoxiancat/mxcat-changelog --skill mxcat-changelog -g
```

仅安装到 Cursor 和 Claude Code：

```bash
npx -y skills add maoxiancat/mxcat-changelog --skill mxcat-changelog -a cursor -a claude-code
```

## 验证

列出源里的技能（应出现 `mxcat-changelog`，且不要安装）：

```bash
npx -y skills add maoxiancat/mxcat-changelog --list
```

本仓库作为本地路径时也可以列出：

```bash
npx -y skills add /path/to/mxcat-changelog --list
```
