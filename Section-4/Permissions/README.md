# Linux Essentials — Commands, Help & File Permissions

A beginner-friendly, hands-on guide to core Linux concepts. Every topic has a clear
explanation, real command examples, diagrams, and a **Try It Yourself** practical
section so you can learn by doing.

> No prior Linux experience needed. If you can open a terminal, you can follow along.

---

## Contents

| # | Topic | What you'll learn |
|---|-------|-------------------|
| 1 | [Command Syntax](01-command-syntax.md) | How every Linux command is structured |
| 2 | [Help Commands](02-help-commands.md) | Look up any command with `whatis`, `--help`, `man` |
| 3 | [Executing Multiple Commands](03-executing-multiple-commands.md) | Run and chain commands with `;`, `&&`, `\|\|`, `\|` |
| 4 | [Users & Permissions](04-users-and-permissions.md) | `whoami`, the `rwx` model, and `u/g/o` levels |
| 5 | [Viewing Permissions (`ls -l`)](05-viewing-permissions-ls-l.md) | Read the 10-character permission string |
| 6 | [Changing Permissions — Symbolic Mode](06-chmod-symbolic-mode.md) | `chmod u/g/o +/-/= rwx` |
| 7 | [Changing Permissions — Numeric Mode](07-chmod-numeric-mode.md) | The octal `chmod 755` system |
| 8 | [Access Control Lists (ACL)](08-access-control-lists-acl.md) | Fine-grained per-user permissions |
| 9 | [Quick Reference Cheat Sheet](09-quick-reference.md) | Every command on one page |

---

## How to use this guide

1. Read the topics in order — each one builds on the last.
2. Keep a terminal open and run the examples yourself.
3. Complete the **Try It Yourself** section at the end of each file.
4. Bookmark the [Cheat Sheet](09-quick-reference.md) for daily use.

## Who this is for

- Absolute beginners starting with Linux
- Students preparing for Linux / cloud / DevOps interviews
- Anyone who wants a clean, practical permissions reference

---

### Recommended practice environment

You don't need to install anything risky. Any of these works:

- A Linux machine or VM (Ubuntu, RHEL, Amazon Linux, etc.)
- **WSL** (Windows Subsystem for Linux) on Windows
- The built-in **Terminal** on macOS (most commands are identical)
- A free online sandbox such as a cloud shell

---

> **Tip:** Throughout this guide, a `$` at the start of a line means "type this in your
> terminal." You don't type the `$` itself — it just represents the command prompt.

**Start here → [1. Command Syntax](01-command-syntax.md)**
