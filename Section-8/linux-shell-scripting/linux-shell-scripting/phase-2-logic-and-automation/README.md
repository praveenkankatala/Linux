[🏠 Home](../README.md)

# Phase 2 — Logic & Automation
> _Building utilities_

In Phase 1 your scripts ran top to bottom, doing the same thing every time. Real utilities make **decisions** and **repeat work**. This phase gives you both. By the end you can write scripts that check conditions, react to failures, and loop over hundreds of files.

> [!NOTE]
> **The mental model**
>
> Every command returns a hidden number when it finishes: its *exit status*. `0` means success; anything else means failure. Conditionals and loops are built entirely on top of this one idea, so we start there.

## 📚 Topics in this phase

| # | Topic | What you'll learn |
| --- | --- | --- |
| 5 | [Exit Status and Conditionals](05-exit-status-and-conditionals.md) | Exit status ($?) and if / elif / else |
| 6 | [String, Integer, and File Tests](06-test-operators.md) | String vs integer vs file tests (-f, -d) |
| 7 | [for Loops](07-for-loops.md) | for loops — iterating lists and files |
| 8 | [while Loops](08-while-loops.md) | while loops — reading files line by line |

## ✅ Phase 2 Checklist

- Read `$?` after a command and set your own exit code with `exit N`.
- Write `if / elif / else / fi` and branch on both tests and command success.
- Pick the right comparison: `==`/`-z` for strings, `-eq`/`-gt` (or `(( ))`) for integers, `-f`/`-d` for files.
- Loop over a glob of files safely (never over `$(ls)`), and use `break`/`continue`.
- Read a file line by line with `while IFS= read -r line; do ... done < file`.

> [!NOTE]
> **Next up — Phase 3**
>
> You can now make decisions and repeat work. Phase 3 is about *data*: stdout vs stderr, redirection, pipes, and the classic filters `grep`, `cut`, `sort`, `uniq`, `sed`, and `awk`. This is where shell truly shines.

---
⬅️ [Phase 1 — The Basics](../phase-1-the-basics/README.md)  |  🏠 [Home](../README.md)  |  [Phase 3 — Text Handling & I/O](../phase-3-text-handling-and-io/README.md) ➡️
