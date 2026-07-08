[🏠 Home](../README.md)

# Phase 1 — The Basics
> _Learning by doing_

## 📚 Topics in this phase

| # | Topic | What you'll learn |
| --- | --- | --- |
| 1 | [Environment, Your First Script, and chmod +x](01-environment-and-first-script.md) | Environment setup, your first script, and chmod +x |
| 2 | [Variables and the Golden Rule of Quoting](02-variables-and-quoting.md) | Variables and the golden rule of quoting |
| 3 | [User Input and Command Arguments](03-input-and-arguments.md) | User input (read) and command arguments ($1, $#) |
| 4 | [Arithmetic and Command Substitution](04-arithmetic-and-command-substitution.md) | Arithmetic and command substitution $(...) |

## ✅ Phase 1 Checklist

Before moving to Phase 2, make sure you can do all of these from memory:

- Write a shebang, save a script, `chmod +x` it, and run it three different ways.
- Assign a variable with no spaces around `=` and read it with `"$var"`.
- Explain why `"$file"` is safer than `$file`.
- Read interactive input with `read -rp`, and read arguments with `$1`, `$#`, `"$@"`.
- Do integer math with `$(( ))` and capture command output with `$(...)`.

> [!NOTE]
> **Next up — Phase 2**
>
> You can now write linear scripts. Phase 2 adds *decisions and repetition*: exit status, `if/elif/else`, string vs integer vs file tests, and `for`/`while` loops. That's where scripts start becoming real tools.

---
⬅️ _First phase_  |  🏠 [Home](../README.md)  |  [Phase 2 — Logic & Automation](../phase-2-logic-and-automation/README.md) ➡️
