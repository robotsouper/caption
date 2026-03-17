---
name: translator
description: SRT subtitle translator for 深影字幕组, adds Chinese translations to English-only .srt files. Call this skill when the user asks to "translate", "翻译", "add Chinese translation", or translate an SRT file.
---

# SRT Translator

## Usage Scope
This skill applies ONLY to `.srt` files within the `caption` project folder.

## Steps

### Step 1: Read the Format Rules
Read the format checker skill at `.claude/skills/format_checker/SKILL.md` to load all 17 formatting rules. Every translation must comply with these rules.

### Step 2: Read the Target SRT File
Read the entire `.srt` file. Count the total number of subtitle blocks. Do not modify any of the original English lines, timestamps, punctuations... ONLY add chinese translation.

### Step 3: Translate and Write

Choose the approach based on file size:

#### Small files (≤100 blocks): Direct Write
- Translate all blocks inline
- Use the Write tool to write the translated SRT file directly (Chinese line above English line, 5-line structure per block)
- No Python script needed

#### Large files (>100 blocks): Parallel Chunks + Python Script
- Split the subtitle blocks into chunks of ~200 blocks each
- Use background Task agents to translate chunks in parallel. Each agent receives:
  - The chunk of English subtitle text (sequence numbers + English lines)
  - A summary of all relevant formatting rules (see below)
  - Instructions to return ONLY a Python dictionary literal mapping subtitle sequence number (int) to Chinese translation (str)
- Generate a temporary Python script (`_translate_tmp.py`) that:
  1. Defines all Chinese translations as a `dict[int, str]`
  2. Reads the original SRT file with `encoding="utf-8-sig"` (handles BOM)
  3. Parses each block and inserts the Chinese line above the English line
  4. Writes the output back to the same file with `encoding="utf-8"` and `newline="\n"`
  5. Run the script, then delete it after success

### Step 4: Verify
Read the beginning and end of the output file to confirm correct 5-line structure.

## Rules

### Chinese Punctuation (Rule 5)
- **NO full-width punctuation** except "" (quotation marks) and 《》 (book title marks)
- Replace all other Chinese punctuation (，。？！、：；) with **one space**
- Example: `你好 世界` not `你好，世界`

### Ellipsis (Rule 8)
- Use three half-width periods: `...`
- NOT `…` (full-width) or `..` (two periods)
- `--` or `-` at end of speech -> change to `...`

### Dialogue (Rule 10)
- 2+ speakers sharing one timecode: prefix each speaker with `- `
- First speaker starts at beginning: `- 台词一 - 台词二`
- Both Chinese AND English must have dashes

### Lyrics (Rule 11)
- Chinese lyrics: `♪text♪` (no spaces between ♪ and text)
- English lyrics: do NOT add ♪ (keep as-is)

### Long Sentences (Rule 12)
- Max ~22 Chinese characters per subtitle line
- Break by meaning groups (~10 chars per group) if needed

### Names (Rule 13)
- Regular English names (Linda, Charlie): keep in English, capitalize first letter
- Famous/established names: translate to Chinese (Marx->马克思, Obama->奥巴马)
- Known place names: translate (Los Angeles->洛杉矶)
- Unknown/ordinary place names: keep in English

### Numbers (Rule 14)
- Convert Arabic numbers to Chinese characters in Chinese subtitles (5->五, 2->两)
- Exception: keep Arabic for 3+ digit numbers or long sequences

### Annotations (Rule 15)
- For on-screen text or unfamiliar terms: use `{\an8}` at start
- Timecode matches the original subtitle being annotated

### Filler Words (Rule 16)
- "you know", "I mean", "ah", "oh" -> do not translate
- If filler word is the entire block -> leave Chinese line as minimal translation or empty
- Exception: plot-critical fillers should be translated

### Intensity Words (Rule 17)
- "bloody", "hell" used for emphasis -> translate contextually, not literally

### HTML Tags (Rule 9)
- Strip all `<i>`, `</i>`, `<b>`, `</b>`, font tags from English
- Strip narration/sound descriptions like `(sniffles)`, `ALICE:`
- If entire block is only a description -> mark for deletion

### Quotation Marks (Rule 7)
- For letters, prescriptions, news: quotes only at very beginning and very end
- Do NOT quote every line of a multi-line quote

## Sub-Agent Prompt Template
```
Translate the following English subtitles to Chinese. Return ONLY a Python dictionary literal (no markdown, no explanation) mapping subtitle sequence number (int) to Chinese translation (str).

Rules:
[Include the Translation Rules above]

Subtitles to translate:
[Paste subtitle blocks: sequence number + English text only]

Output format example:
{
    1: "不是吧",
    2: "一只超大的鳄鱼标本",
}
```

## Python Assembly Script Template
```python
import re

cn = {
    # ... all translations merged from sub-agents ...
}

filepath = r"<SRT_FILE_PATH>"

with open(filepath, "r", encoding="utf-8-sig") as f:
    content = f.read()

blocks = re.split(r'\n\n+', content.strip())
out_lines = []

for block in blocks:
    lines = block.strip().split('\n')
    if len(lines) < 2:
        continue
    seq = lines[0].strip()
    timecode = lines[1].strip()
    eng = '\n'.join(lines[2:])
    seq_num = int(seq)
    chinese = cn.get(seq_num, "")
    out_lines.append(f"{seq}\n{timecode}\n{chinese}\n{eng}\n")

with open(filepath, "w", encoding="utf-8", newline="\n") as f:
    f.write('\n'.join(out_lines) + '\n')

print(f"Written {len(out_lines)} translated blocks to SRT file.")
```

## Error Handling
- Always read the original file with `encoding="utf-8-sig"` to handle BOM
- Write output with `encoding="utf-8"` and `newline="\n"` for consistent line endings
- If a subtitle number has no translation in the dictionary, use an empty string (indicates a block that may need deletion per Rule 9/16)
- Delete the temporary Python script after successful execution
