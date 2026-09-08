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
- Style adapts to the nature of the video
- Notes saved directly into the correct subject → module folder

---

## Step 1 — Extract Transcript (Zero-Retry Protocol)

### What is known from experience

| What was tried | Result |
|----------------|--------|
| `android` client, no pre-sleep | 429 — YouTube rate-limits subtitle URL |
| `android` client, `--sleep-requests 3`, no pre-sleep | 429 — still too fast |
| `tv_embedded` client | Unsupported — yt-dlp skips it, falls back to web client, also 429 |
| `Start-Sleep 30` + `android` + `--sleep-requests 5` | **SUCCESS every time** |

### The one proven command — use this, nothing else

```powershell
Start-Sleep -Seconds 35
python -m yt_dlp --skip-download --write-auto-sub --sub-lang en --sub-format srv1 --extractor-args "youtube:player_client=android" --sleep-requests 5 --output "transcript" "https://www.youtube.com/watch?v=VIDEO_ID"
```

**Why 35 seconds pre-sleep?**
YouTube rate-limits the subtitle download URL per session. The 35-second wait clears the rate-limit window before the request is even made. This eliminates 429 before it happens rather than reacting to it.

**Why `android` client only?**
It is the only client that provides subtitle URLs that yt-dlp can download. `tv_embedded` is unsupported. Default web client gets 429. Do not try any other client.

**Why `--sleep-requests 5`?**
Adds 5-second gaps between internal yt-dlp HTTP calls during the same session. Prevents triggering rate limits mid-download.

### If it still fails after one attempt
Do NOT retry with a different client or different flags. Wait 60 more seconds and run the exact same command once more. If it fails again — tell the user and stop.

### Output
- File: `transcript.en.srv1` in the working directory
- Read it directly with `view_file` — the XML tags are stripped automatically by the tool
- Working directory for downloads: `C:\Users\kedar\.gemini\antigravity\brain\<conversation-id>\`

---

## Step 2 — Read and Detect Video Nature

Read the full transcript once. Decide the note style before writing anything.

| Signal in transcript | Video Nature | Note Style |
|----------------------|-------------|------------|
| Formulas, calculations, step-by-step numbers | **Numerical** | Formula first → worked example → traps |
| Definitions, concepts, comparisons | **Theoretical** | Concept flow → comparison table → MCQ traps |
| Instructor solves MCQs on screen | **MCQ-heavy** | Reproduce those MCQs + explain wrong options |
| Journal entries, T-accounts, ledgers | **Accounting/Practical** | Two-column journal entries → common errors |
| Mix | **Mixed** | Follow video's own sequence |

---

## Step 3 — Identify Subject and Module

### Folder map

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

### Filename
`<SUBJECT>_<ModuleCode>_<TopicName>.md`
Examples: `AFM_A02_Nature_and_Purpose_of_Accounting.md` | `IEIFS_B04_RBI_Functions.md`
If file exists, append `_v2`.

---

## Step 4 — Write the Notes

> Follow the video's flow. Write what the instructor actually taught, in the order they taught it.

**Every video must have:**
- 3-line header: Topic, Module, YouTube URL
- Core content in video sequence
- MCQs the instructor solved — reproduced with answer + why wrong options are wrong
- MCQ traps the instructor flagged
- Tight quick-revision bullet list at the end

**Numerical videos — add:**
- Formula before the example
- Each calculation step on its own line
- Common mistakes flagged

**Theoretical videos — add:**
- Comparison table when instructor contrasts two concepts
- Key definitions in bold as the instructor stated them

**Accounting/Practical videos — add:**
- Journal entries in two-column Dr/Cr format
- Note why each side is Dr or Cr

**Never:**
- Write a transcript
- Add textbook content not in the video
- Mix subjects
- Use a rigid section template
- Pad with generic tips
- Guess — mark unclear things **[VERIFY]**

---

## Step 5 — Save the File

**Use `write_to_file` tool** (not Python `-c` inline and not PowerShell here-strings — both cause quote/escape failures with apostrophes and special characters in notes).

Write to: `C:\Users\kedar\.gemini\antigravity\brain\<conversation-id>\<filename>.md`
Then copy to the project folder:
```powershell
Copy-Item "<artifact_path>" -Destination "D:\JAIIB-2026\<subject>\<module>\<filename>.md" -Force
```

Then git commit and push:
```powershell
git -C "D:\JAIIB-2026" add "<relative_file_path>"
git -C "D:\JAIIB-2026" commit -m "<SUBJECT>_<code>: <Topic> notes"
git -C "D:\JAIIB-2026" push origin main
```

Confirm with file size. Clean up `transcript.en.srv1` after saving.

---

## Token Efficiency Rules

1. **35-second pre-sleep before every transcript download** — prevents 429 entirely
2. Run yt-dlp → read transcript in one call → write notes → save → push. No loops.
3. Detect video nature from transcript before writing a single word of notes
4. Write notes once — no drafts, no back-and-forth with user
5. Do not re-read the transcript after notes are written
6. Do not create implementation plans — just execute
7. If two videos are given back-to-back, enforce the 35-second gap between each download
