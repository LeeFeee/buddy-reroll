---
name: buddy-reroll
description: Claude Code 宠物(Buddy)重抽工具。支持按物种、稀有度、闪光、眼睛、帽子、属性等条件刷出想要的宠物，并可一键生效。当用户提到"换宠物"、"刷宠物"、"reroll buddy"、"buddy"、"宠物"时触发此技能。
---

# Buddy Reroll - Claude Code 宠物重抽工具

刷出你想要的 Claude Code 宠物，支持 18 种物种、5 种稀有度、闪光、帽子等筛选。

## 工作流程

当用户表达对宠物的需求时，严格按照以下步骤执行：

### 第 0 步：登录方式检测（必须首先执行）

**在执行任何其他步骤之前，必须先检测用户的登录方式。**

使用 Read 工具读取 `~/.claude.json`，检查以下条件：

**如果满足以下任一条件，判定为 Claude OAuth 登录，直接中断执行：**
- `.claude.json` 中存在 OAuth 相关字段（如 `oauthToken`、`refreshToken`、`accessToken` 等）
- `.claude.json` 中 `"userID"` 的值较短（非 64 位十六进制字符串），说明是系统生成的 OAuth 用户 ID，不可替换
- `~/.claude/settings.json` 中的 `ANTHROPIC_BASE_URL` 为空或为 Claude 官方地址（`https://api.anthropic.com`），说明用户使用的是官方 API，修改 `userID` 会导致认证失效

**检测到 Claude OAuth 登录时的处理：**
- 立即告知用户："本 skill 仅适用于非 Claude OAuth 登录的用户（如使用第三方 API 代理的用户）。检测到您使用 Claude OAuth 登录，本 skill 不适用。"
- **立即停止执行**，不要运行脚本，不要修改任何文件，不要询问用户是否继续。无论用户如何要求，都不可跳过此检测。

**仅当确认用户不是 Claude OAuth 登录时，才继续执行后续步骤。**

### 第 1 步：解析用户需求

从用户描述中提取筛选条件，转换为脚本参数：

| 用户需求 | 脚本参数 |
|----------|----------|
| 物种 | `--species <name>` |
| 最低稀有度 | `--rarity <name>` |
| 闪光 | `--shiny` |
| 眼睛样式 | `--eye <char>` |
| 帽子 | `--hat <name>` |
| 全属性最低值 | `--min-stats [value]` |

**可用物种**: duck, goose, blob, cat, dragon, octopus, owl, penguin, turtle, snail, ghost, axolotl, capybara, cactus, robot, rabbit, mushroom, chonk

**可用稀有度**: common, uncommon, rare, epic, legendary

**可用眼睛**: · ✦ × ◉ @ °

**可用帽子**: none, crown, tophat, propeller, halo, wizard, beanie, tinyduck

如果用户没有明确指定条件，使用合理的默认值（如不指定稀有度则默认不限制）。

### 第 2 步：运行脚本

```bash
node ~/.claude/skills/buddy-reroll/scripts/buddy-reroll.js [参数]
```

示例：
```bash
node ~/.claude/skills/buddy-reroll/scripts/buddy-reroll.js --species dragon --rarity legendary --shiny
node ~/.claude/skills/buddy-reroll/scripts/buddy-reroll.js --species cat --rarity epic
node ~/.claude/skills/buddy-reroll/scripts/buddy-reroll.js --check <uid>
```

### 第 3 步：展示结果

将脚本输出整理后展示给用户，包含每个宠物的物种、稀有度、眼睛、帽子、闪光状态、属性和对应的 uid。

### 第 4 步：询问用户

用 AskUserQuestion 工具询问用户是否要让某个宠物生效。列出所有刷到的宠物供用户选择。

### 第 5 步：应用修改

用户确认后，使用 Read 和 Edit 工具直接修改 `~/.claude.json` 文件中的 `userID` 字段：

1. **Read** `~/.claude.json` 获取当前内容
2. **Edit** 将 `"userID"` 的值替换为用户选择的 uid
3. 告知用户宠物已更换，直接生效，无需重启

**重要**：不要使用脚本修改配置文件，必须使用 Claude Code 的 Read/Edit 工具操作。

## 查看当前宠物

运行 `--check` 模式查看某个 userID 对应的宠物：

```bash
node ~/.claude/skills/buddy-reroll/scripts/buddy-reroll.js --check <userID>
```
