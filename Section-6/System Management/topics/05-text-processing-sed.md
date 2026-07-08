# 5. Text Processing with `sed`

[← Back to index](../README.md)

`sed` (**s**tream **ed**itor) edits text automatically — perfect for find-and-replace or deleting lines across a file without opening an editor.

> **How sed thinks:** it reads your file **one line at a time** into a temporary buffer (the *pattern space*), checks the line against your instruction, then prints or discards it — repeating for every line.

## Substitution (find & replace)

```bash
sed 's/search/replacement/flags' filename
```

- **`s`** — the substitute command
- **`g`** flag — **global**: replace *every* match on a line (not just the first)
- **`i`** flag — **ignore case** when matching
- **`-i`** option — edit the file **in place** (save changes). Without it, `sed` only prints to the screen and the file is unchanged.

```bash
sed 's/sed/AWK/' file        # first match on each line, printed to screen
sed 's/sed/AWK/g' file       # every match on each line
sed 's/sed/AWK/gi' file      # every match, ignoring case
sed -i 's/sed/AWK/g' file    # actually save the changes to the file
```

> ⚠️ **Preview before you save.** Run without `-i` first to see the result. Add `-i` only once you're happy — it overwrites the file permanently.

## Deleting lines

The `d` command discards matching lines instead of printing them.

| Command | Effect |
|---------|--------|
| `sed '2d' file` | Delete line 2 |
| `sed '2,3d' file` | Delete lines 2 through 3 |
| `sed '$d' file` | Delete the **last** line (`$` = end of file) |
| `sed '/Amazon/d' file` | Delete every line containing "Amazon" |
| `sed '2,3!d' file` | Keep **only** lines 2–3 (`!` inverts the match) |

## `sed` vs `awk` — which one?

| Tool | Best for |
|------|----------|
| `sed` | Fast find-and-replace and deleting lines |
| `awk` | Column-based data, arithmetic, and complex logic |

## 🧪 Practice

1. Create a test file:
   ```bash
   cat > sample.txt <<EOF
   Hello Amazon Linux
   sed is very powerful
   We are learning sed command
   sed helps in text processing
   EOF
   ```
2. Preview a global replace (nothing is saved yet):
   ```bash
   sed 's/sed/AWK/g' sample.txt
   ```
3. Delete lines 2–3 (preview):
   ```bash
   sed '2,3d' sample.txt
   ```
4. Now save a change permanently:
   ```bash
   sed -i 's/Amazon/AWS/g' sample.txt
   cat sample.txt
   ```

**✅ Success check:** After step 4, the first line reads **"Hello AWS Linux"**.

---
[← Previous: Task Scheduling](04-task-scheduling.md) | [Next: Package Management →](06-package-management.md)
