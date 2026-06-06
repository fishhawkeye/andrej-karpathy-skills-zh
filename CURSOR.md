# 在 Cursor 中使用本仓库

本项目包含一个 **Cursor 项目规则**，这样在你这里工作时，受 Karpathy 启发的行为准则会自动生效。

## 在本仓库中使用

1. 在 Cursor 中打开此文件夹。
2. 规则文件 [`.cursor/rules/karpathy-guidelines.mdc`](.cursor/rules/karpathy-guidelines.mdc) 已提交并设置了 `alwaysApply: true`，因此不需要额外的安装步骤。
3. 在 Cursor 中，你可以在 **Settings → Rules**（或项目规则界面）中确认，`karpathy-guidelines` 应该会出现在那里。

## 在其他项目中使用相同的准则

**Cursor（推荐）：** 将 `.cursor/rules/karpathy-guidelines.mdc` 复制到目标项目的 `.cursor/rules/` 目录中（如需则创建文件夹）。可以根据需要调整或与现有规则合并。

**其他工具：** 如果某个工具只支持根目录的指令文件，则将 [`CLAUDE.md`](CLAUDE.md) 复制到该项目中（或将其内容合并到你现有的指令文件中）。

## 可选：个人 Agent 技能

如果你希望将相同内容作为 `~/.cursor/skills` 下的可复用技能使用，可以使用 [`skills/karpathy-guidelines/SKILL.md`](skills/karpathy-guidelines/SKILL.md)。你可以将其复制或符号链接到你的个人技能目录中，使用你习惯的目录结构。

## Claude Code 与 Cursor 的区别

- **Claude Code：** 通过插件市场安装，按照 [`README.md`](README.md) 中的说明操作；插件会暴露本仓库中的技能。按项目使用也可以依赖 `CLAUDE.md`。
- **Cursor：** 使用已提交的 `.cursor/rules/` 文件，如上所述。Cursor 默认不会读取 `.claude-plugin/` 或 `CLAUDE.md`。

## 给贡献者的说明

当你修改四条原则时，请保持 **[`CLAUDE.md`](CLAUDE.md)** 和 **[`.cursor/rules/karpathy-guidelines.mdc`](.cursor/rules/karpathy-guidelines.mdc)** 同步。如果发布的技能/插件文本也需要匹配，请同时更新 **[`skills/karpathy-guidelines/SKILL.md`](skills/karpathy-guidelines/SKILL.md)**。
