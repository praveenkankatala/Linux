# 🐧 Linux Command-Line Notes

> Practical, beginner-friendly notes on core Linux commands — shell productivity, compression, file viewing, comparison, text processing, and editing with `vi`/`vim`.
> Every topic includes **plain-English explanations** and **hands-on examples you can copy and run**.

Targeted at **Amazon Linux / RHEL / Ubuntu** using the **Bash** shell, but 95% applies to any Linux or macOS terminal.

---

## 📚 Topics

| # | Topic | What you'll learn |
|---|-------|-------------------|
| 01 | [Shell Productivity](01-shell-productivity.md) | Tab completion, command history, pipes & redirection |
| 02 | [Compression & Archiving](02-compression-and-archiving.md) | `gzip`, `gunzip`, `tar`, reading `.gz` in place |
| 03 | [File Display Commands](03-file-display.md) | `cat`, `less`, `head`, `tail`, `more`, `nl` |
| 04 | [Comparing Files](04-comparing-files.md) | `diff` and `cmp` |
| 05 | [Filters & Text Processing](05-filters-and-text-processing.md) | `cut`, `sort`, `uniq`, `wc`, `tr`, `truncate` |
| 06 | [grep — Searching Text](06-grep.md) | Pattern search, regex, context lines |
| 07 | [awk — Column Processing](07-awk.md) | Fields, filters, calculations, group-by |
| 08 | [Editing Files](08-file-editing.md) | Redirection, here-docs, `tee`, `sed`, `nano` |
| 09 | [vi / vim Editor](09-vi-vim-editor.md) | The editor on every server, end to end |
| ⭐ | [Cheat Sheet](CHEATSHEET.md) | Everything on one page |

---

## 🚀 How to use these notes

1. Open a terminal on any Linux machine.
2. Read a topic top to bottom.
3. **Type every example yourself** — the shaded `bash` blocks are meant to be run, not just read. Muscle memory is the goal.

### Sample files used throughout

Many examples reference these two files. Create them once and keep them in your working directory so every example is reproducible.

```bash
# a small web server access log
cat > access.log <<'EOF'
10.0.0.5 GET /index.html 200
10.0.0.7 GET /login 200
10.0.0.5 POST /api/order 500
10.0.0.9 GET /index.html 200
10.0.0.7 GET /assets/app.js 404
10.0.0.5 GET /index.html 200
EOF

# a small CSV of users
cat > users.csv <<'EOF'
id,name,role,dept
101,ravi,engineer,platform
102,anita,manager,platform
103,sam,engineer,data
104,leah,analyst,data
EOF
```

### Conventions

| Symbol | Meaning |
|--------|---------|
| `$` | The shell prompt — you don't type it |
| `#` | A comment in a command block |
| `<name>` | A placeholder you replace with a real value |
| `Ctrl+X` | Hold **Ctrl** and press **X** |

---

## 🗺️ The one pipeline to remember

If you learn only one thing from these notes, learn this. It answers *"count how often each thing appears, ranked from most to least."*

```bash
sort file | uniq -c | sort -rn | head
```

You'll use it for top IPs, top URLs, top error messages — endlessly.

---

## 📄 License

Free to use, share, and adapt. Attribution appreciated but not required.
