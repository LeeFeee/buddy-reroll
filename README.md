# Buddy Reroll

[中文说明](README_CN.md)

A Claude Code skill that lets you reroll your companion pet (buddy) to get the exact species, rarity, and traits you want.

## Features

- 18 species: duck, goose, blob, cat, dragon, octopus, owl, penguin, turtle, snail, ghost, axolotl, capybara, cactus, robot, rabbit, mushroom, chonk
- 5 rarity tiers: common, uncommon, rare, epic, legendary
- Filter by: species, rarity, shiny, eye style, hat, stats
- One-click apply — automatically updates your `~/.claude.json` to make the pet take effect instantly

## Requirements

- Claude Code installed via npm (not native installer)
- Non-OAuth login (e.g., third-party API proxy)
- Node.js (comes with npm-installed Claude Code)

> **Important**: This skill only works for users who are NOT logged in via Claude OAuth or using the official Anthropic API. If you're using a third-party API proxy (custom `ANTHROPIC_BASE_URL`), this skill will work for you.

## Install

Copy the skill directory to your Claude Code skills folder:

```bash
git clone https://github.com/LeeFeee/buddy-reroll.git ~/.claude/skills/buddy-reroll
```

Then restart Claude Code or start a new conversation.

## Usage

Simply tell Claude Code what pet you want in natural language:

```
"I want a legendary shiny dragon"
"给我来只史诗闪光猫"
"Reroll my buddy to an epic penguin"
"换一只传说级龙"
```

The skill will:
1. Check if your login method is compatible
2. Search for matching pets
3. Show you 5 options with full stats
4. Ask which one you'd like to apply
5. Update your config to make it take effect instantly

## Manual Script Usage

You can also run the script directly:

```bash
# Search for a specific pet
node ~/.claude/skills/buddy-reroll/scripts/buddy-reroll.js --species dragon --rarity legendary --shiny

# Check what a specific userID produces
node ~/.claude/skills/buddy-reroll/scripts/buddy-reroll.js --check <uid>

# With minimum stats requirement
node ~/.claude/skills/buddy-reroll/scripts/buddy-reroll.js --species cat --min-stats 80
```

### Script Options

| Option | Description |
|--------|-------------|
| `--species <name>` | Target species |
| `--rarity <name>` | Minimum rarity (common/uncommon/rare/epic/legendary) |
| `--shiny` | Require shiny variant |
| `--eye <char>` | Eye style (· ✦ × ◉ @ °) |
| `--hat <name>` | Hat style (none/crown/tophat/propeller/halo/wizard/beanie/tinyduck) |
| `--min-stats [val]` | All stats >= value (default: 90) |
| `--count <n>` | Number of results (default: 5) |
| `--max <n>` | Max iterations (default: 50000000) |
| `--check <uid>` | Check what a userID produces |

## How It Works

The script uses the same PRNG (Mulberry32) and hash algorithm (FNV-1a) as Claude Code to generate buddy pets. It brute-forces random userIDs until it finds ones that produce your desired pet, then the skill updates the `userID` field in `~/.claude.json`.

## License

MIT
