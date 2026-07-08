[🏠 Home](../README.md)

# Phase 5 — Advanced & Robust Scripting
> _Enterprise level_

A script that works on your laptop and a script you'd trust to run unattended on a fleet of servers are different things. This phase covers the difference: failing fast and loudly, cleaning up after yourself no matter how you exit, modelling structured data, doing work in parallel, and wielding awk and process substitution for problems that would otherwise need a 'real' programming language.

> [!NOTE]
> **The reliability mindset**
>
> Assume everything can fail: commands, network, disk, the user pressing Ctrl-C mid-run. Robust scripts detect failure early, leave no mess behind, and report clearly. That's what turns a script into a tool your team relies on.

## 📚 Topics in this phase

| # | Topic | What you'll learn |
| --- | --- | --- |
| 16 | [Defensive Flags and Error Logging](16-strict-mode-and-logging.md) | Defensive flags (set -euo pipefail) + error logging |
| 17 | [Traps, Signals, and Cleanup](17-traps-and-signals.md) | Traps and signals for cleanup on exit |
| 18 | [Associative Arrays (Hash Maps)](18-associative-arrays.md) | Associative arrays (key-value / hash maps) |
| 19 | [Concurrency: Background Jobs, wait, Subshells](19-concurrency.md) | Concurrency: background jobs, wait, subshells |
| 20 | [Advanced awk and Process Substitution](20-advanced-awk-and-process-substitution.md) | Advanced awk and process substitution <(cmd) |

## ✅ Phase 5 Checklist

- Open scripts with `set -euo pipefail` and understand where `set -e` won't save you.
- Build a logging framework (`info`/`warn`/`error`/`die`) that writes to stderr and a file.
- Use `trap cleanup EXIT` to remove temp files and locks no matter how the script ends.
- Model key-value data with `declare -A` associative arrays and use them for counting.
- Run independent work in parallel with `&` / `wait`, throttle with `xargs -P`, and reason about subshells.
- Group and aggregate with advanced `awk`, and use `<(cmd)` / `>(cmd)` to avoid temp files and subshell traps.

## Where to go next

You now have the full toolkit. To keep sharpening it: run every script through **ShellCheck** (`shellcheck script.sh`) — it catches quoting and portability bugs automatically and will teach you as it flags things. Read the scripts already on your systems in `/etc/init.d` and `/usr/bin`. And adopt a style guide (Google's Shell Style Guide is excellent) so your scripts stay consistent as they grow.

> [!NOTE]
> **You've completed the course**
>
> From `chmod +x` to parallel awk aggregation across five phases. The only thing left is reps: automate one annoying manual task this week with a script that uses the strict header, a function or two, and a trap. That's how this becomes second nature.

---
⬅️ [Phase 4 — Intermediate Efficiency](../phase-4-intermediate-efficiency/README.md)  |  🏠 [Home](../README.md)  |  _Final phase_ 🎉
