# 2. Help Commands

[← Previous: Command Syntax](01-command-syntax.md) | [Home](README.md) | [Next: Executing Multiple Commands →](03-executing-multiple-commands.md)

---

Nobody memorises every command and every option. Linux ships with **built-in help** so you
can look things up instantly. There are **three common ways** to get help.

| Help command | What it does | Best for |
|--------------|--------------|----------|
| `whatis command` | One-line description of what a command is | A quick reminder |
| `command --help` | Short summary of usage and available options | Listing a command's options |
| `man command` | The full manual page (detailed reference) | Deep, complete documentation |

---

## 1) `whatis` — the one-liner

Gives a single-line summary so you instantly know what a command does.

```bash
$ whatis ls
ls (1) - list directory contents
```

Use it when you see an unfamiliar command and just want to know *what it is*.

---

## 2) `--help` — the quick option list

Most commands accept `--help` and print a quick usage guide with their main options.

```bash
$ ls --help
Usage: ls [OPTION]... [FILE]...
  -a, --all      do not ignore entries starting with .
  -l             use a long listing format
  -h, --human-readable   print sizes like 1K 234M 2G
  ...
```

Use it when you know the command but forgot the exact option you need.

---

## 3) `man` — the full manual

The `man` (manual) command opens the **complete reference manual** for a command. It
contains every option, examples, and detailed explanations.

```bash
$ man ls
```

Inside a `man` page:

| Key | Action |
|-----|--------|
| `Spacebar` | Scroll down one page |
| `↑` / `↓` | Scroll line by line |
| `/word` then `Enter` | Search for "word" |
| `n` | Jump to the next search match |
| `q` | Quit and return to the terminal |

> **Tip:** `man` pages are divided into sections. `whatis ls` showed `ls (1)` — the `(1)`
> means it lives in manual section 1 (user commands).

---

## Which one should I use?

```
Just need a reminder of what it does?  ->  whatis command
Forgot an option?                      ->  command --help
Want full details and examples?        ->  man command
```

---

## Try It Yourself

```bash
# One-line description
whatis pwd
whatis cp

# Quick option list
cp --help
mkdir --help

# Full manual (press q to quit)
man cp

# Bonus: search inside a man page
man ls
#   then type:  /sort   and press Enter to find sorting options
#   press n to see the next match, q to quit
```

**Challenge:** Use the help tools to answer these without searching the web:
1. What does the `wc` command do?
2. Which `cp` option copies folders and everything inside them?

<details>
<summary>Show answers</summary>

1. `whatis wc` → "print newline, word, and byte counts for each file."
2. `cp --help` → `-r` (or `-R`, `--recursive`) copies directories recursively.

</details>

---

## Key Takeaways

- `whatis` → quick one-line summary.
- `--help` → fast list of options.
- `man` → the full, detailed manual (quit with `q`).
- Reach for help *before* guessing — it is always available offline.

---

[← Previous: Command Syntax](01-command-syntax.md) | [Home](README.md) | [Next: Executing Multiple Commands →](03-executing-multiple-commands.md)
