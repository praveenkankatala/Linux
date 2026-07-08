[🏠 Home](../README.md)

# Phase 3 — Text Handling & I/O
> _Data processing_

The Unix philosophy is: small tools that each do one thing, connected by streams of text. This phase teaches you to route those streams (redirection), chain the tools (pipes), and reach for the right filter for the job. Master this and you can process log files, CSVs, and command output with one-liners that would take pages of code elsewhere.

> [!NOTE]
> **The three standard streams**
>
> Every program is born with three channels: **stdin** (0, its input), **stdout** (1, normal output), and **stderr** (2, error messages). Keeping stdout and stderr separate is what lets you save results while still seeing errors.

## 📚 Topics in this phase

| # | Topic | What you'll learn |
| --- | --- | --- |
| 9 | [stdout, stderr, and Redirection](09-redirection.md) | stdout, stderr, and redirection (>, >>, 2>) |
| 10 | [Pipes and Core Filters](10-pipes-and-filters.md) | Pipes and core filters: grep, cut, sort, uniq |
| 11 | [Stream Editing with sed](11-sed.md) | Stream editing with sed (find and replace) |
| 12 | [Data Extraction with awk](12-awk.md) | Data extraction with awk (columns and printing) |

## ✅ Phase 3 Checklist

- Redirect stdout and stderr independently (`>`, `>>`, `2>`, `&>`, `2>&1` last).
- Send your own errors to stderr with `>&2`, and discard noise with `/dev/null`.
- Build pipelines with `grep`, `cut`, `sort`, `uniq -c`, `head`, `wc`.
- Do find-and-replace with `sed 's/.../.../g'` and edit files safely with `sed -i.bak`.
- Extract and summarise columns with `awk` fields, patterns, `printf`, and `END`.

> [!NOTE]
> **Next up — Phase 4**
>
> You can now transform data fluently. Phase 4 makes your *code* clean and reusable: functions, arrays and string manipulation, and professional flag parsing with `getopts`.

---
⬅️ [Phase 2 — Logic & Automation](../phase-2-logic-and-automation/README.md)  |  🏠 [Home](../README.md)  |  [Phase 4 — Intermediate Efficiency](../phase-4-intermediate-efficiency/README.md) ➡️
