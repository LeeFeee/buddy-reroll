---
name: buddy-reroll
description: Claude Code 宠物(Buddy)重抽工具。支持按物种、稀有度、闪光、眼睛、帽子、属性等条件刷出想要的宠物，并可一键生效。支持并行多任务刷宠，高效完成苛刻条件筛选。当用户提到"换宠物"、"刷宠物"、"reroll buddy"、"buddy"、"宠物"时触发此技能。
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

### 第 1 步：解析用户需求 → 填充变量

从用户描述中提取筛选条件，填充以下变量：

#### 变量定义表

| 变量名 | 说明 | 可选值 | 来源示例 |
|--------|------|--------|----------|
| `${SPECIES}` | 物种 | duck, goose, blob, cat, dragon, octopus, owl, penguin, turtle, snail, ghost, axolotl, capybara, cactus, robot, rabbit, mushroom, chonk | "龙" → dragon |
| `${RARITY}` | 最低稀有度 | common, uncommon, rare, epic, legendary | "传奇" → legendary |
| `${SHINY}` | 是否闪光 | `--shiny` 或 空 | "闪光" → `--shiny` |
| `${EYE}` | 眼睛样式 | · ✦ × ◉ @ ° 或 空 | "眼睛@样子" → @ |
| `${HAT}` | 帽子 | none, crown, tophat, propeller, halo, wizard, beanie, tinyduck 或 空 | "戴皇冠" → crown |
| `${MIN_STATS}` | 全属性最低值 | 数字 或 空 | "全属性80+" → 80 |
| `${MAX_ITERATIONS}` | 循环次数 | 数字，默认500，极端条件建议2000+ | 根据条件苛刻程度判断 |

#### 属性条件变量

当用户对单个属性有特定要求时，填充以下变量：

| 变量名 | 说明 | 格式 |
|--------|------|------|
| `${STAT_CONDITION}` | 属性判断条件 | bash 条件表达式，多个用 `&&` 连接 |

**属性名称对照：**
- `debugging` - DEBUGGING 属性
- `patience` - PATIENCE 属性  
- `chaos` - CHAOS 属性
- `wisdom` - WISDOM 属性
- `snark` - SNARK 属性

**常用条件模板：**

| 用户需求 | ${STAT_CONDITION} 值 |
|----------|---------------------|
| 全属性 >= N | `[ "$debugging" -ge N ] && [ "$patience" -ge N ] && [ "$chaos" -ge N ] && [ "$wisdom" -ge N ] && [ "$snark" -ge N ]` |
| 全属性 = N | `[ "$debugging" = "N" ] && [ "$patience" = "N" ] && [ "$chaos" = "N" ] && [ "$wisdom" = "N" ] && [ "$snark" = "N" ]` |
| 某属性 >= N | `[ "$<属性名>" -ge N ]` |
| 某属性 = N | `[ "$<属性名>" = "N" ]` |
| CHAOS=100 且其他 >= N | `[ "$chaos" = "100" ] && [ "$debugging" -ge N ] && [ "$patience" -ge N ] && [ "$wisdom" -ge N ] && [ "$snark" -ge N ]` |

**重要：涉及全属性判断时，必须包含全部5个属性，不要遗漏 DEBUGGING！**

### 第 2 步：选择刷宠模式

**判断逻辑：**
```
IF 用户需求可以用脚本内置参数完全满足（无特殊属性条件）
   → 使用简单模式
ELSE IF 用户需求包含自定义属性条件
   → 使用并行模式
```

#### 简单模式

脚本内置参数已能满足需求，直接调用：

```bash
node ~/.claude/skills/buddy-reroll/scripts/buddy-reroll.js \
  --species ${SPECIES} \
  --rarity ${RARITY} \
  ${SHINY} \
  ${MIN_STATS:+--min-stats $MIN_STATS}
```

**适用场景：** 无特殊属性条件，只需物种/稀有度/闪光/全属性最低值。

#### 并行模式

用户有自定义属性条件，需手动判断。启动 4 个并行后台任务。

**完整模板：**

```bash
for i in {1..${MAX_ITERATIONS}}; do
  # 1. 执行脚本获取结果
  result=$(node ~/.claude/skills/buddy-reroll/scripts/buddy-reroll.js \
    --species ${SPECIES} \
    --rarity ${RARITY} \
    ${SHINY} 2>/dev/null)

  # 2. 提取属性值
  stats=$(echo "$result" | grep "stats:" | head -1)
  debugging=$(echo "$stats" | grep -oP 'DEBUGGING:\K[0-9]+')
  patience=$(echo "$stats" | grep -oP 'PATIENCE:\K[0-9]+')
  chaos=$(echo "$stats" | grep -oP 'CHAOS:\K[0-9]+')
  wisdom=$(echo "$stats" | grep -oP 'WISDOM:\K[0-9]+')
  snark=$(echo "$stats" | grep -oP 'SNARK:\K[0-9]+')

  # 3. 判断是否满足条件（使用 ${STAT_CONDITION}）
  if ${STAT_CONDITION}; then
    echo "✅ 找到符合条件的宠物！"
    echo "$result"
    exit 0
  fi
done
echo "未找到符合条件的宠物"
```

**启动方式：** 使用 4 个并行的 Bash 工具调用，设置 `run_in_background: true`，`timeout: 600000`。

**检查结果：**
```bash
sleep 60 && for f in /tmp/claude-*/tasks/*.output; do
  [ -s "$f" ] && cat "$f"
done
```

**停止任务：** 找到结果后用 `TaskStop` 停止所有后台任务。

### 第 3 步：展示结果

整理脚本输出，展示：物种、稀有度、眼睛、帽子、闪光状态、属性、uid。

### 第 4 步：询问用户

用 AskUserQuestion 工具让用户选择要生效的宠物。

### 第 5 步：应用修改

用 Read/Edit 工具修改 `~/.claude.json` 中的 `userID` 字段。

## 查看当前宠物

```bash
node ~/.claude/skills/buddy-reroll/scripts/buddy-reroll.js --check <userID>
```

## 变量填充示例

### 示例：用户说 "刷全属性80以上的传奇闪光龙"

**解析过程：**
1. "龙" → `${SPECIES}` = `dragon`
2. "传奇" → `${RARITY}` = `legendary`
3. "闪光" → `${SHINY}` = `--shiny`
4. "全属性80以上" → `${STAT_CONDITION}` = `[ "$debugging" -ge 80 ] && [ "$patience" -ge 80 ] && [ "$chaos" -ge 80 ] && [ "$wisdom" -ge 80 ] && [ "$snark" -ge 80 ]`
5. 条件苛刻 → `${MAX_ITERATIONS}` = `2000`

**选择模式：** 有自定义属性条件 → 并行模式

**生成代码：**
```bash
for i in {1..2000}; do
  result=$(node ~/.claude/skills/buddy-reroll/scripts/buddy-reroll.js --species dragon --rarity legendary --shiny 2>/dev/null)
  stats=$(echo "$result" | grep "stats:" | head -1)
  debugging=$(echo "$stats" | grep -oP 'DEBUGGING:\K[0-9]+')
  patience=$(echo "$stats" | grep -oP 'PATIENCE:\K[0-9]+')
  chaos=$(echo "$stats" | grep -oP 'CHAOS:\K[0-9]+')
  wisdom=$(echo "$stats" | grep -oP 'WISDOM:\K[0-9]+')
  snark=$(echo "$stats" | grep -oP 'SNARK:\K[0-9]+')

  if [ "$debugging" -ge 80 ] && [ "$patience" -ge 80 ] && [ "$chaos" -ge 80 ] && [ "$wisdom" -ge 80 ] && [ "$snark" -ge 80 ]; then
    echo "✅ 找到符合条件的宠物！"
    echo "$result"
    exit 0
  fi
done
```

### 示例：用户说 "刷一个普通的闪光鸭子"

**解析过程：**
1. "鸭子" → `${SPECIES}` = `duck`
2. "普通" → `${RARITY}` = `common`
3. "闪光" → `${SHINY}` = `--shiny`
4. 无属性条件 → `${STAT_CONDITION}` = 空

**选择模式：** 无特殊属性条件 → 简单模式

**生成代码：**
```bash
node ~/.claude/skills/buddy-reroll/scripts/buddy-reroll.js --species duck --rarity common --shiny
```