# SRT Format Checker — Claude Code Skill

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) custom skill that checks SRT subtitle files against the formatting standards of **深影字幕组** (Shenying Fansub Team).

## What It Does

Point it at any `.srt` file and it will audit every subtitle block against 17 formatting rules, then output a structured report listing every violation with subtitle number, line number, current content, the rule violated, and a suggested fix.

The skill is **read-only** — it never modifies your SRT files.

## Rules Checked

| # | Rule | Description |
|---|------|-------------|
| 1 | Five-Line Structure | Each block must follow: number → timecode → Chinese → English → blank line |
| 2 | Chinese Above English | Chinese on line 3, English on line 4, never reversed |
| 3 | Blank Lines | Exactly one blank line between blocks, no extras |
| 4 | Single Line Display | No manual line breaks within a subtitle line |
| 5 | Chinese Punctuation | No full-width punctuation except `""` and `《》`; replace others with a space |
| 6 | English Punctuation | Must be half-width with a space after each mark |
| 7 | Quotation Marks | Quotes only at the very start and end of a multi-line quote |
| 8 | Ellipsis | Must be three half-width periods `...`, not `…` or `..`; convert trailing `--`/`-` |
| 9 | HTML Tag Cleanup | Remove `<i>`, `<b>`, font tags, and narration/sound descriptions |
| 10 | Dialogue Format | Multi-speaker lines use `- ` dashes in both Chinese and English |
| 11 | Lyrics Format | Chinese lyrics wrapped in `♪...♪` (no spaces); English lyrics have no `♪` |
| 12 | Long Sentences | Max ~22 Chinese chars per line; English lines are not split |
| 13 | Names & Proper Nouns | Capitalize English names; translate well-known names/places to Chinese |
| 14 | Number Localization | Arabic → Chinese characters in Chinese lines (except 3+ digit numbers) |
| 15 | Annotation Subtitles | `{\an8}` tag for on-screen text annotations |
| 16 | Filler Words | "you know", "I mean", etc. should not be translated; delete if standalone |
| 17 | Intensity Words | Translate emphasis words contextually, not literally |

## Installation

1. Copy the `format_checker` folder into your project's `.claude/skills/` directory:

```
your-project/
└── .claude/
    └── skills/
        └── format_checker/
            └── SKILL.md
```

2. (Optional) Add a trigger to your project's `CLAUDE.md` so the skill is invoked automatically:

```markdown
- When the user asks to "check format" or "检查格式", use format_checker in .claude/skills
```

## Usage

In Claude Code, simply ask:

```
检查格式 Merteuil.S01.E06.srt
```

or

```
check the format of Merteuil.S01.E06.srt
```

Claude will read the file, run all 17 checks, and output a report like:

```
=== SRT 格式检查报告 ===
文件：Merteuil.S01.E06.srt

--- 发现的问题 ---

【问题 1】违反规则 5：中文标点
  轴号：12 | 行号：45
  当前内容：你好，世界
  问题说明：中文逗号应替换为空格
  修改建议：你好 世界

--- 合规项目 ---
✓ 规则 1：五行结构 — 全部通过
✓ 规则 2：中上英下 — 全部通过
...

--- 统计 ---
共检查 290 个字幕块，发现 3 处问题
```

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI or VS Code extension
- SRT files in the same project directory

## License

MIT
