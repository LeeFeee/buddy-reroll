# Buddy Reroll

[English](README.md)

一个 Claude Code 技能（Skill），让你可以重新抽取陪伴宠物（Buddy），获得你想要的物种、稀有度和特征。

## 功能

- 18 种物种：duck、goose、blob、cat、dragon、octopus、owl、penguin、turtle、snail、ghost、axolotl、capybara、cactus、robot、rabbit、mushroom、chonk
- 5 种稀有度：common、uncommon、rare、epic、legendary
- 支持筛选：物种、稀有度、闪光、眼睛样式、帽子、属性
- 一键生效 — 自动修改 `~/.claude.json`，更换宠物立即生效

## 使用条件

- 通过 npm 安装的 Claude Code（非 native 安装）
- 非 OAuth 登录（如使用第三方 API 代理）
- Node.js（npm 安装的 Claude Code 自带）

> **重要提示**：本技能仅适用于**非** Claude OAuth 登录且**非**使用 Anthropic 官方 API 的用户。如果您使用了第三方 API 代理（自定义了 `ANTHROPIC_BASE_URL`），本技能可以正常使用。

## 安装

将 skill 目录克隆到 Claude Code 的 skills 文件夹：

```bash
git clone https://github.com/LeeFeee/buddy-reroll.git ~/.claude/skills/buddy-reroll
```

然后重启 Claude Code 或开启新对话即可使用。

## 使用方法

直接用自然语言告诉 Claude Code 你想要什么宠物：

```
"我想要一只传奇闪光龙"
"给我来只史诗闪光猫"
"换一只传说级企鹅"
"刷一只稀有度的 owl"
```

技能会自动执行以下流程：
1. 检测登录方式是否兼容
2. 搜索符合条件的宠物
3. 展示 5 个候选宠物及其完整属性
4. 询问你要让哪只生效
5. 修改配置文件，立即生效

## 手动运行脚本

你也可以直接运行脚本：

```bash
# 搜索特定宠物
node ~/.claude/skills/buddy-reroll/scripts/buddy-reroll.js --species dragon --rarity legendary --shiny

# 查看某个 userID 对应的宠物
node ~/.claude/skills/buddy-reroll/scripts/buddy-reroll.js --check <uid>

# 要求最低属性
node ~/.claude/skills/buddy-reroll/scripts/buddy-reroll.js --species cat --min-stats 80
```

### 脚本参数

| 参数 | 说明 |
|------|------|
| `--species <名称>` | 目标物种 |
| `--rarity <名称>` | 最低稀有度（common/uncommon/rare/epic/legendary） |
| `--shiny` | 要求闪光 |
| `--eye <字符>` | 眼睛样式（· ✦ × ◉ @ °） |
| `--hat <名称>` | 帽子样式（none/crown/tophat/propeller/halo/wizard/beanie/tinyduck） |
| `--min-stats [数值]` | 全属性最低值（默认 90） |
| `--count <数量>` | 搜索结果数量（默认 5） |
| `--max <数量>` | 最大迭代次数（默认 50000000） |
| `--check <uid>` | 查看某个 userID 对应的宠物 |

## 原理

脚本使用与 Claude Code 相同的 PRNG（Mulberry32）和哈希算法（FNV-1a）来生成宠物。它通过暴力搜索随机 userID，直到找到生成目标宠物的 ID，然后技能会更新 `~/.claude.json` 中的 `userID` 字段使其生效。

## 许可证

MIT
