# Skill: format_checker

## Description
SRT subtitle format checker for 深影字幕组. Checks specified `.srt` files in this project against the team's translation and formatting standards.

## Scope
This skill applies ONLY to files within the `caption` project folder.

## When to Use
When the user asks to "check format", "检查格式", "check the format of", or review an SRT file for compliance.

## How to Use
1. Read the target `.srt` file, DO NOT modify anything in the .srt file
2. Check every subtitle block against ALL rules below
3. Output a report listing every violation with: subtitle number, line number, current content, which rule is violated, and suggested fix
4. At the end, list which rules passed with no issues

## Format Rules

### Rule 1: SRT Five-Line Structure
Each subtitle block must follow this exact structure:
```
序号 (number only)
HH:MM:SS,mmm --> HH:MM:SS,mmm
中文字幕
英文字幕
(blank line)
```
- Timecodes must NOT be modified
- Sequence numbers do NOT need to be renumbered after deletions (gaps like 1,2,5,6 are OK)
- There must be NO blank line between the timecode and the subtitle text

### Rule 2: Chinese Above, English Below
- Line 3 of each block = Chinese subtitle
- Line 4 of each block = English subtitle
- Never reversed

### Rule 3: Blank Line Between Blocks
- Exactly ONE blank line between subtitle blocks
- No extra blank lines (delete if more than one)
- No blank lines between timecode and subtitle text
- File should not end with multiple blank lines

### Rule 4: Single Line Display
- Both Chinese and English subtitles must be on a single line each
- No manual line breaks within a subtitle line

### Rule 5: Punctuation in Chinese Subtitles
- **NO full-width symbols allowed** in Chinese subtitles, EXCEPT:
  - Quotation marks "" (allowed)
  - Book title marks 《》 (allowed)
- All other Chinese punctuation (，。？！、：；) must be replaced with **one space**
- That means: no commas, periods, question marks, exclamation marks, colons, semicolons in Chinese lines

### Rule 6: English Subtitle Punctuation
- All English punctuation must be **half-width** (not full-width)
- If full-width punctuation appears in English, convert to half-width
- There must be **a space after** each punctuation mark (e.g., "Hello, world" not "Hello,world")

### Rule 7: Quotation Marks ""
- For letters, prescriptions, news, etc.: add quotes only at the very beginning and very end
- Do NOT add quotes to every line of a multi-line quote

### Rule 8: Ellipsis ...
- Ellipsis = three half-width periods: `...`
- NOT `…` (shift+6 full-width ellipsis)
- NOT `..` (only two periods)
- Unfinished speech shown as `--` or `-` at end of line → change to `...` (both Chinese AND English)
- Do NOT use find-replace for `--` because timecodes contain `-->`
- Chinese ellipsis: keep if appropriate; English ellipsis: always keep

### Rule 9: HTML Tag Cleanup
- Delete all `<i>`, `</i>`, `<b>`, `</b>` and similar HTML tags from English subtitles
- Delete font color/size codes (e.g., `<font color=...>`)
- Delete narration/sound descriptions (e.g., `(sniffles)`, `(door slams)`, `(Laugh)`, `ALICE:`)
- If an entire subtitle block contains only such descriptions, delete the whole block

### Rule 10: Dialogue Format
- When 2+ speakers share one timecode, add a hyphen-dash before each speaker's line:
  - First dash: starts at beginning, followed by a space: `- 台词`
  - Subsequent dashes: space before AND after: ` - 台词`
- BOTH Chinese and English lines must have dashes
- Example CN: `- 当老大不容易 - 是啊 - 你记住这点`
- Example EN: `- It ain't easy being king. - Yeah. - You remember that.`
- If English is missing dashes, add them based on video content
- Multi-line dialogue must be merged into one line first

### Rule 11: Lyrics Format
- Chinese lyrics: add ♪ at BOTH beginning and end, **no space** between ♪ and text
  - Correct: `♪烟尘弥漫丘陵♪`
  - Wrong: `♪ 烟尘弥漫丘陵 ♪` (spaces)
  - Wrong: `♪烟尘弥漫丘陵♪♪` (double ♪)
  - Wrong: `♪烟尘弥漫丘陵` (missing closing ♪)
- English lyrics: do NOT add ♪

### Rule 12: Long Sentence Handling
- English subtitles: do NOT break lines (line splitting is done by team lead)
- Chinese subtitles: break by meaning groups (~10 characters per group)
- Each English line must have a corresponding Chinese line above it
- Max ~22 Chinese characters per subtitle line (audience needs to read in 2-4 seconds)

### Rule 13: Names and Proper Nouns
- Regular English names (Linda, Charlie): keep in English, **capitalize first letter**
- Famous/established names: translate to Chinese (Marx→马克思, Obama→奥巴马)
- Known place names: translate to Chinese (Los Angeles→洛杉矶)
- Unknown/ordinary place names: keep in English

### Rule 14: Number Localization
- In Chinese subtitles, convert Arabic numbers to Chinese characters (5→五, 2→两)
- Exceptions (keep Arabic):
  - Numbers with more than 2 digits (e.g., 233, 4567)
  - Long sequences of numbers (e.g., 11,12,13...)

### Rule 15: Annotation Subtitles (硬字幕)
- Used for: explaining on-screen text OR unfamiliar terms in dialogue
- Must include `{\an8}` at the start of the text (positions subtitle at top of screen)
- Timecode must match the original subtitle being annotated
- Sequence number can be duplicated (software auto-corrects)
- Content should be concise and clear
- Common knowledge (DNA, CEO, 911) does NOT need annotation

### Rule 16: Filler Words
- "you know", "I mean", "ah", "oh" etc. → do not translate
- If a filler word occupies an entire subtitle block by itself → delete the whole block
- Exception: if the filler word is plot-critical, translate it

### Rule 17: Intensity Words
- Words like "bloody", "hell" used for emphasis → translate contextually
- Do NOT translate literally (e.g., don't translate "hell" as 地狱 when it's just emphasis)

## Report Format
Output the report in this structure:

```
=== SRT 格式检查报告 ===
文件：[filename]

--- 发现的问题 ---

【问题 N】违反规则 X：[规则名称]
  轴号：XX | 行号：XX
  当前内容：...
  问题说明：...
  修改建议：...

(repeat for each issue)

--- 合规项目 ---
✓ 规则 X：[规则名称] — 全部通过
(list all rules that passed)

--- 统计 ---
共检查 XX 个字幕块，发现 XX 处问题
```
