# 🐚 Shell Scripting: Zero to Advanced

> A hands-on, **practical** Bash course — from your very first script to enterprise-grade, robust automation. Learn by typing, running, breaking, and fixing real scripts.

![Bash](https://img.shields.io/badge/Bash-5.x-4EAA25?logo=gnubash&logoColor=white) ![Platform](https://img.shields.io/badge/Platform-Linux%20%7C%20macOS%20%7C%20WSL-blue) ![Level](https://img.shields.io/badge/Level-Beginner%20→%20Advanced-orange) ![Style](https://img.shields.io/badge/Approach-Learn%20by%20doing-success)

## 📖 About this course

This is a **coaching guide, not a reference manual.** Every concept comes with a script you should actually type and run. Reading about shell scripting teaches you almost nothing — the muscle memory of typing a command, hitting an error, and fixing it is where the learning happens.

The course is split into **5 phases** and **20 topics**, each in its own file so you can learn one focused idea at a time and track your progress.

## ✅ Prerequisites

- A terminal on **Linux, macOS, or Windows (WSL)**
- **bash 4.0+** (check with `bash --version`)
- Any text editor — `nano`, `vim`, or VS Code
- No prior scripting experience required

## 🗺️ Curriculum

### [Phase 1 — The Basics](phase-1-the-basics/README.md)  ·  _Learning by doing_

| # | Topic | Description |
| --- | --- | --- |
| 1 | [Environment, Your First Script, and chmod +x](phase-1-the-basics/01-environment-and-first-script.md) | Environment setup, your first script, and chmod +x |
| 2 | [Variables and the Golden Rule of Quoting](phase-1-the-basics/02-variables-and-quoting.md) | Variables and the golden rule of quoting |
| 3 | [User Input and Command Arguments](phase-1-the-basics/03-input-and-arguments.md) | User input (read) and command arguments ($1, $#) |
| 4 | [Arithmetic and Command Substitution](phase-1-the-basics/04-arithmetic-and-command-substitution.md) | Arithmetic and command substitution $(...) |

### [Phase 2 — Logic & Automation](phase-2-logic-and-automation/README.md)  ·  _Building utilities_

| # | Topic | Description |
| --- | --- | --- |
| 5 | [Exit Status and Conditionals](phase-2-logic-and-automation/05-exit-status-and-conditionals.md) | Exit status ($?) and if / elif / else |
| 6 | [String, Integer, and File Tests](phase-2-logic-and-automation/06-test-operators.md) | String vs integer vs file tests (-f, -d) |
| 7 | [for Loops](phase-2-logic-and-automation/07-for-loops.md) | for loops — iterating lists and files |
| 8 | [while Loops](phase-2-logic-and-automation/08-while-loops.md) | while loops — reading files line by line |

### [Phase 3 — Text Handling & I/O](phase-3-text-handling-and-io/README.md)  ·  _Data processing_

| # | Topic | Description |
| --- | --- | --- |
| 9 | [stdout, stderr, and Redirection](phase-3-text-handling-and-io/09-redirection.md) | stdout, stderr, and redirection (>, >>, 2>) |
| 10 | [Pipes and Core Filters](phase-3-text-handling-and-io/10-pipes-and-filters.md) | Pipes and core filters: grep, cut, sort, uniq |
| 11 | [Stream Editing with sed](phase-3-text-handling-and-io/11-sed.md) | Stream editing with sed (find and replace) |
| 12 | [Data Extraction with awk](phase-3-text-handling-and-io/12-awk.md) | Data extraction with awk (columns and printing) |

### [Phase 4 — Intermediate Efficiency](phase-4-intermediate-efficiency/README.md)  ·  _Writing clean code_

| # | Topic | Description |
| --- | --- | --- |
| 13 | [Functions](phase-4-intermediate-efficiency/13-functions.md) | Functions — reusability, arguments, local vs global |
| 14 | [Arrays and String Manipulation](phase-4-intermediate-efficiency/14-arrays-and-strings.md) | Arrays and string manipulation (slicing, length) |
| 15 | [Parsing Flags with getopts](phase-4-intermediate-efficiency/15-getopts.md) | Parsing flags with getopts (-f, -h) |

### [Phase 5 — Advanced & Robust Scripting](phase-5-advanced-robust-scripting/README.md)  ·  _Enterprise level_

| # | Topic | Description |
| --- | --- | --- |
| 16 | [Defensive Flags and Error Logging](phase-5-advanced-robust-scripting/16-strict-mode-and-logging.md) | Defensive flags (set -euo pipefail) + error logging |
| 17 | [Traps, Signals, and Cleanup](phase-5-advanced-robust-scripting/17-traps-and-signals.md) | Traps and signals for cleanup on exit |
| 18 | [Associative Arrays (Hash Maps)](phase-5-advanced-robust-scripting/18-associative-arrays.md) | Associative arrays (key-value / hash maps) |
| 19 | [Concurrency: Background Jobs, wait, Subshells](phase-5-advanced-robust-scripting/19-concurrency.md) | Concurrency: background jobs, wait, subshells |
| 20 | [Advanced awk and Process Substitution](phase-5-advanced-robust-scripting/20-advanced-awk-and-process-substitution.md) | Advanced awk and process substitution <(cmd) |

## 🧭 How to use this guide

This is a coaching guide, not a reference manual. Every concept comes with a script you should actually type and run. Reading shell scripting teaches you almost nothing; the muscle memory of typing `chmod +x`, hitting an error, and fixing it is where the learning happens.

- **Light gray boxes** are scripts or commands you type.
- **Dark boxes** are the terminal output you should expect to see.
- **Colored callouts** flag tips, gotchas, and hands-on exercises.
- Do the **Try It Yourself** exercises before moving on. Solutions follow each one.

> [!NOTE]
> **What is a shell?**
>
> A *shell* is the program that reads the commands you type and asks the operating system to run them. `bash` (Bourne-Again SHell) is the most common on Linux servers, so this course targets bash. Most examples also work in `sh`, `zsh`, and `ksh`, and we call out where they differ.

## 💡 The one habit that matters most

> [!IMPORTANT]
> **Always quote your variables:** write `"$var"`, `"$@"`, and `"${arr[@]}"`.
>
> Unquoted variables silently break the moment a value contains a space or is empty — the single most common source of shell bugs. This habit is introduced in Phase 1 and reinforced through every later phase.

## 🛠️ Recommended tooling

- **[ShellCheck](https://www.shellcheck.net/)** — a static analyzer that catches quoting and portability bugs automatically. Install it and run every script through it: `shellcheck myscript.sh`.
- **[Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html)** — a widely used convention for readable, maintainable scripts.
- **`set -euo pipefail`** — the defensive header covered in Phase 5 that turns silent failures into loud, catchable ones.

## 🚀 Getting started

Begin with **[Phase 1 → Topic 1](phase-1-the-basics/01-environment-and-first-script.md)** and work straight through. Do the **🎯 Try It Yourself** exercise in each topic before moving on — every exercise has a worked solution right below it.

---

_Happy scripting!_ ⭐ If this course helps you, consider starring the repo.
