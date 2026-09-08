---
name: JAIIB-Note-Maker
description: >-
  Master skill for making JAIIB exam notes from YouTube video links.
  Works for all three subjects: AFM, IE&IFS, PPB.
  Extracts the actual video transcript using yt-dlp, reads the video's
  flow, detects the nature of content (numerical/theoretical/MCQ-heavy/
  practical), and produces concise, MCQ-focused notes saved in the correct
  subject module folder. Activate whenever the user provides a YouTube link
  for JAIIB preparation.
---

# JAIIB Note Maker — Master Skill

## Purpose
Convert any JAIIB YouTube lecture into concise, MCQ-focused notes.
- Works for **AFM**, **IE&IFS**, and **PPB**
- Notes follow the **video's own teaching flow** — not a rigid template
- Style adapts to the **nature of the video**
- Notes are saved directly into the correct **subject → module** folder

---

## Step 1 — Extract Transcript

Run this command every time. The `android` player_client bypasses YouTube's 429 rate-limit:

```powershell
python -m yt_dlp --skip-download --write-auto-sub --sub-lang en --sub-format srv1 `
  --extractor-args "youtube:player_client=android" `
  --output "transcript" "<YouTube_URL>"
```

- Output file: `transcript.en.srv1` in the current working directory
- Strip XML tags to get plain text for reading

If transcript download fails → tell the user, do not invent notes.

---

## Step 2 — Read and Detect Video Nature

Read the full transcript. Before writing a single note, determine:

| Signal in transcript | Video Nature | Note Style |
|----------------------|-------------|------------|
| Formulas, calculations, step-by-step numbers | **Numerical** | Lead with formula → worked example → traps |
| Definitions, concepts, comparisons | **Theoretical** | Concept flow → differences table → MCQ traps |
| Instructor solves MCQs explicitly | **MCQ-heavy** | Reproduce those MCQs + explain wrong options |
| Journal entries, T-accounts, ledgers | **Accounting/Practical** | Show entries clearly → common errors |
| Mix of above | **Mixed** | Follow the video's own sequence |

---

## Step 3 — Identify Subject and Module

### Workspace folder map

```
D:\JAIIB-2026\
├── 01_IE_IFS\
│   ├── 01_Module_A_Indian_Economic_Architecture\
│   ├── 02_Module_B_Economic_Concepts_Related_to_Banking\
│   ├── 03_Module_C_Indian_Financial_Architecture\
│   └── 04_Module_D_Financial_Products_and_Services\
├── 02_PPB\
│   ├── 01_Module_A_General_Banking_Operations\
│   ├── 02_Module_B_Functions_of_Banks\
│   ├── 03_Module_C_Banking_Technology\
│   └── 04_Module_D_Ethics_in_Banks_and_Financial_Institutions\
└── 03_AFM\
    ├── 01_Module_A_Accounting_Principles_and_Processes\
    ├── 02_Module_B_Financial_Statements_and_Core_Banking_Systems\
    ├── 03_Module_C_Financial_Management\
    └── 04_Module_D_Taxation_and_Fundamentals_of_Costing\
```

Determine from transcript content which subject and module the video belongs to.
If ambiguous, save in the closest match and note it at the top of the file.

### Filename convention
`<SUBJECT>_<ModuleCode>_<TopicName>.md`

Examples:
- `AFM_A02_Nature_and_Purpose_of_Accounting.md`
- `IEIFS_B04_RBI_Functions.md`
- `PPB_A03_KYC_and_AML.md`

Do NOT overwrite an existing file — append `_v2` if file already exists.

---

## Step 4 — Write the Notes

### The golden rule
> Follow the video's flow. Write what the instructor actually taught, in the order they taught it. Do not reorganise into a textbook structure.

### What to always include (adapt the format to the video's nature)

**For every video:**
- Topic name, Subject, Module, YouTube URL at the top (3 lines, no decoration)
- Core content following the video's sequence
- Any MCQs the instructor solved — reproduce them with answer + explanation of why wrong options are wrong
- MCQ traps — points the instructor flagged as exam-tricky
- A tight quick-revision list at the end (bullet points, no prose)

**For numerical videos (add these):**
- Formula box before the example
- Step-by-step calculation — show each step on its own line
- Common calculation mistakes flagged

**For theoretical videos (add these):**
- Differences/comparison table if the instructor compared two concepts
- Key definitions in bold, as the instructor stated them

**For accounting/practical videos (add these):**
- Journal entries in standard two-column format
- Note which side (Dr/Cr) and why

### What to NEVER do
- Do not write a transcript
- Do not add textbook content not covered in the video
- Do not mix content from different subjects
- Do not use a rigid 9-section template
- Do not pad notes with generic exam tips
- Do not guess — if something in the video is unclear, mark it **[VERIFY]**

---

## Step 5 — Save the File

```powershell
# Use Python to write (avoids PowerShell here-string issues with special chars)
python -c "
content = '''<notes content here>'''
with open(r'D:\\JAIIB-2026\\<subject_folder>\\<module_folder>\\<filename>.md', 'w', encoding='utf-8') as f:
    f.write(content)
print('Saved.')
"
```

Confirm save with file size. Clean up `transcript.en.srv1` after saving.

---

## Token Efficiency Rules

To keep token usage low on every video:
1. Run yt-dlp → read transcript in one shot
2. Detect nature → decide format before writing anything
3. Write notes once — no drafts, no back-and-forth
4. Do not re-read the transcript after notes are written
5. Do not create implementation plans for note-making — just do it
