# 06 — `grep`: Searching Text with Patterns

`grep` prints lines that match a pattern. It's probably the command you'll type most often. The name means **g**lobal **r**egular **e**xpression **p**rint.

- [Everyday usage](#everyday-usage)
- [Context lines](#context-lines-essential-for-logs)
- [Regular expressions](#regular-expressions)
- [Flag reference](#flag-reference)

---

## Everyday usage

```bash
$ grep ERROR app.log             # lines containing ERROR
$ grep -i error app.log          # case-insensitive
$ grep -v DEBUG app.log          # INVERT: lines NOT containing DEBUG
$ grep -n ERROR app.log          # show line numbers
$ grep -c ERROR app.log          # count matching lines
$ grep -w cat file               # match the whole WORD 'cat', not 'category'
$ grep -r "TODO" ./src           # recursive search through a directory
$ grep -l "api_key" ./config/*   # list only the FILE NAMES that match
```

---

## Context lines (essential for logs)

When you find an error, you usually want the lines around it too.

```bash
$ grep -A 3 ERROR app.log        # match + 3 lines AFTER
$ grep -B 3 ERROR app.log        # match + 3 lines BEFORE
$ grep -C 3 ERROR app.log        # match + 3 lines on both sides (Context)
```

---

## Regular expressions

With `-E` (extended regex) you get the full pattern language:

| Symbol | Means |
|--------|-------|
| `^` | Start of line |
| `$` | End of line |
| `.` | Any single character |
| `*` | Zero or more of the previous |
| `+` | One or more of the previous |
| `[...]` | Any character in the set |
| `\|` | OR (alternation) |
| `{n}` | Exactly n repetitions |

```bash
$ grep -E '^10\.0\.0\.' access.log      # lines starting with that subnet
$ grep -E ' 500$' access.log            # lines ending in status 500
$ grep -E '(4|5)[0-9]{2}$' access.log   # any 4xx or 5xx status at line end

# -o prints ONLY the matched text (great for extraction)
$ grep -Eo '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' access.log   # pull out all IPs
```

---

## Flag reference

| Flag | Effect |
|------|--------|
| `-i` | Case-insensitive |
| `-v` | Invert — show non-matching lines |
| `-n` | Prefix line numbers |
| `-c` | Count matching lines |
| `-r` / `-R` | Recurse into directories |
| `-w` | Match whole words only |
| `-o` | Print only the matched text, not the whole line |
| `-l` | Print only names of files that match |
| `-E` | Extended regex (enables `+ ? \| ( )` without backslashes) |
| `-A N` / `-B N` / `-C N` | Show N lines After / Before / around each match |

> **Tip:** `grep -rn --include='*.py' 'def main' .` searches only Python files recursively and shows line numbers — a fast way to navigate an unfamiliar codebase.

---

### ✅ Quick recap

- Find text → `grep -i pattern file`.
- Search a whole project → `grep -rn pattern .`.
- Count matches → `grep -c pattern file`.
- Extract just the match → `grep -Eo 'regex' file`.

---

⬅️ [Prev: Filters](05-filters-and-text-processing.md) | 🏠 [Index](README.md) | ➡️ [Next: awk](07-awk.md)
