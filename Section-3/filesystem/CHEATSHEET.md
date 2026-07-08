# ⭐ Linux Cheat Sheet

Everything from these notes on one page. [Back to Index](README.md).

---

## Shell productivity

| Task | Command |
|------|---------|
| Complete a name/path | `Tab` (twice to list options) |
| Recall previous command | `Up arrow` or `!!` |
| Search history | `Ctrl+R` then type |
| Reuse last argument | `!$` |
| Chain commands | `cmd1 \| cmd2 \| cmd3` |
| Save output to file | `cmd > file` (append: `>>`) |
| Redirect errors | `cmd 2> err.log` |

## Compression & archiving

| Task | Command |
|------|---------|
| Gzip a file (keep original) | `gzip -k file` |
| Ungzip | `gunzip file.gz` |
| Search inside a `.gz` | `zgrep pattern file.gz` |
| Create `tar.gz` | `tar -czvf out.tar.gz dir/` |
| List a `tar.gz` | `tar -tvf out.tar.gz` |
| Extract to a dir | `tar -xzvf out.tar.gz -C /path` |

## Viewing files

| Task | Command |
|------|---------|
| First / last N lines | `head -n N` / `tail -n N` |
| Follow a live log | `tail -F file` |
| Page a big file | `less file` |
| Whole small file | `cat file` |

## Comparing files

| Task | Command |
|------|---------|
| Line-by-line diff | `diff -u a b` |
| Side-by-side diff | `diff -y a b` |
| Are two files identical? | `cmp -s a b` |

## Filters & text processing

| Task | Command |
|------|---------|
| Extract a column | `cut -d',' -f2 file` |
| Sort numerically desc | `sort -rn file` |
| Sort by CSV column 4 | `sort -t',' -k4 file` |
| Count + rank occurrences | `sort \| uniq -c \| sort -rn` |
| Count lines / words | `wc -l` / `wc -w` |
| Uppercase | `tr 'a-z' 'A-Z'` |
| Strip carriage returns | `tr -d '\r'` |
| Empty a log in place | `truncate -s 0 file` |

## grep

| Task | Command |
|------|---------|
| Search (case-insensitive) | `grep -i pattern file` |
| Search recursively w/ line #s | `grep -rn pattern .` |
| Invert (lines NOT matching) | `grep -v pattern file` |
| Count matching lines | `grep -c pattern file` |
| Context around match | `grep -C 3 pattern file` |
| Extract just the match | `grep -Eo 'regex' file` |

## awk

| Task | Command |
|------|---------|
| Print a column | `awk '{print $3}' file` |
| CSV column | `awk -F',' '{print $2}' file` |
| Filter by a field | `awk '$4 == 500' file` |
| Group-by count | `awk '{c[$1]++} END{for(k in c) print c[k],k}'` |
| Sum a column | `awk '{s+=$5} END{print s}' file` |

## Editing files (no interactive editor)

| Task | Command |
|------|---------|
| Quick write / append | `echo "x" > file` / `>>` |
| Multi-line block | `cat > f <<'EOF' ... EOF` |
| Write a root-owned file | `echo x \| sudo tee /etc/file` |
| Find & replace in place | `sed -i.bak 's/old/new/g' file` |
| Delete matching lines | `sed -i '/pat/d' file` |

## vi / vim

| Task | Keys |
|------|------|
| Insert / append | `i` / `a` |
| Open line below / above | `o` / `O` |
| Escape to Normal mode | `Esc` |
| Move (l/d/u/r) | `h` `j` `k` `l` |
| Word / line-end / line-start | `w` / `$` / `0` |
| Top / bottom of file | `gg` / `G` |
| Go to line N | `NG` or `:N` |
| Delete char / line | `x` / `dd` |
| Change word / line | `cw` / `cc` |
| Yank / paste | `yy` / `p` |
| Undo / redo / repeat | `u` / `Ctrl-r` / `.` |
| Search | `/text` then `n` / `N` |
| Replace all in file | `:%s/old/new/g` (add `c` to confirm) |
| Delete matching lines | `:g/pat/d` |
| Save / save-quit | `:w` / `:wq` |
| Quit / quit-discard | `:q` / `:q!` |

---

## 🗺️ The pipeline to remember above all others

```bash
# "count how often each thing appears, ranked most to least"
sort file | uniq -c | sort -rn | head
```

---

🏠 [Back to Index](README.md)
