# 03 — File Display Commands

You'll read files constantly — configs, logs, CSVs. Pick the right viewer for the size and purpose of the file.

| Command | Use it when you want to... |
|---------|----------------------------|
| `cat` | Dump a whole (small) file to the screen |
| `tac` | Dump a file **reversed**, last line first |
| `less` | Scroll a large file interactively (**best default**) |
| `more` | Simple forward-only pager (older) |
| `head` | See the **first** N lines |
| `tail` | See the **last** N lines (and follow live) |
| `nl` | Print with line numbers |
| `od` / `xxd` | Inspect the bytes of a binary file |

---

## cat & tac

```bash
$ cat users.csv            # print the whole file
$ cat -n access.log        # with line numbers
$ cat file1 file2 > both   # concatenate two files into one
$ tac access.log           # reverse line order (read newest log lines first)
```

---

## less — the pager you should default to

`less` loads lazily, so it opens huge files instantly. Keys **inside** `less`:

| Key | Action |
|-----|--------|
| `Space` / `b` | Page down / up |
| `/text` | Search **forward** for `text` (`n` = next, `N` = previous) |
| `?text` | Search **backward** |
| `G` / `g` | Jump to **end** / **start** of file |
| `F` | Follow mode — like `tail -f`; press `Ctrl+C` to stop |
| `q` | Quit |

```bash
$ less /var/log/messages
$ journalctl -u sshd | less     # page piped output
```

---

## head & tail

```bash
$ head access.log           # first 10 lines (default)
$ head -n 3 access.log      # first 3 lines
$ tail access.log           # last 10 lines
$ tail -n 20 access.log     # last 20 lines

# skip a header: print from line 2 to the end
$ tail -n +2 users.csv
```

### The DevOps favorite: watch a log grow in real time

```bash
$ tail -f /var/log/nginx/access.log
$ tail -f app.log | grep --line-buffered ERROR   # follow AND filter
```

> **Tip:** `tail -F` (capital **F**) keeps following even if the file is rotated and recreated — safer than `-f` for logs managed by `logrotate`.

---

## Numbering lines with nl

```bash
$ nl access.log            # number every non-blank line
$ nl -ba access.log        # number ALL lines including blanks
```

---

## Peeking at binary files

```bash
$ file /bin/ls            # what kind of file is it?
$ xxd /bin/ls | head      # hex + ASCII dump
$ od -c data.bin | head   # octal / character dump
```

---

### ✅ Quick recap

- Small file → `cat`. Big file → `less`.
- First/last lines → `head -n N` / `tail -n N`.
- Live log → `tail -F file`.

---

⬅️ [Prev: Compression](02-compression-and-archiving.md) | 🏠 [Index](README.md) | ➡️ [Next: Comparing Files](04-comparing-files.md)
