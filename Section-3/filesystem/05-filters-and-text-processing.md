# 05 — Filters & Text Processing

A **filter** is a program that reads text from standard input, transforms it, and writes to standard output. Because filters share this contract, they chain effortlessly with pipes.

The mental model: **data flows through a pipeline of filters**, each doing one transformation.

The core toolbox: `cat`, `cut`, `sort`, `uniq`, `wc`, `tr`, `grep`, `awk`, `truncate`.
(`grep` and `awk` are big enough to get their own pages — see [06](06-grep.md) and [07](07-awk.md).)

- [cat as a filter](#cat-as-a-filter)
- [cut — extract columns](#cut--extract-columns)
- [sort](#sort)
- [uniq](#uniq)
- [wc — count lines, words, bytes](#wc--count-lines-words-bytes)
- [tr — translate or delete characters](#tr--translate-or-delete-characters)
- [truncate — set a file to an exact size](#truncate--set-a-file-to-an-exact-size)

---

## cat as a filter

Beyond dumping files, `cat` is the simplest filter — often the first stage of a pipeline — and has handy display flags.

```bash
$ cat -n access.log       # number every line
$ cat -b access.log       # number only non-blank lines
$ cat -A config.conf      # show hidden chars: tabs as ^I, line ends as $
$ cat -s file             # squeeze repeated blank lines into one
```

> **Note:** `cat -A` is a lifesaver for debugging config files where a stray tab or trailing space breaks parsing — you can finally *see* the invisible characters.

---

## cut — extract columns

`cut` pulls out fields or character ranges from each line.

```bash
# by DELIMITER (-d) and field (-f)
$ cut -d' ' -f1 access.log       # first space-separated field (the IP)
$ cut -d',' -f2,3 users.csv      # 2nd and 3rd comma-separated fields
$ cut -d':' -f1 /etc/passwd      # all usernames

# by CHARACTER position (-c)
$ cut -c1-8 file                 # first 8 characters of each line
```

| Flag | Meaning |
|------|---------|
| `-d` | Set the field **delimiter** |
| `-f` | Choose the **field(s)** — `-f1,3` or `-f2-4` |
| `-c` | Choose **character** positions instead of fields |

---

## sort

`sort` orders lines. By default it sorts **alphabetically** (as text), which is why `10` sorts before `9`. Use `-n` for real numeric order.

```bash
$ sort users.csv                 # alphabetical by whole line
$ sort -r users.csv              # reverse order
$ sort -n numbers.txt            # numeric sort (10 after 9)
$ sort -rn numbers.txt           # numeric, descending

# sort by a specific column with a delimiter
$ sort -t',' -k4 users.csv       # by field 4 (dept)
$ sort -t',' -k1 -n users.csv    # by field 1 (id), numerically

$ sort -u names.txt              # sort AND remove duplicates
$ du -h /var/log/* | sort -h     # human sizes (2K < 1M < 1G)
```

| Flag | Effect |
|------|--------|
| `-n` | Numeric sort |
| `-r` | Reverse |
| `-k N` | Sort by field number N |
| `-t C` | Field delimiter is `C` |
| `-u` | Output unique lines only |
| `-h` | Human-numeric sort (2K < 1M < 1G) |
| `-f` | Case-insensitive |

> **⚠️ Watch out:** `sort` **must** run before `uniq` — `uniq` only collapses *adjacent* duplicates.

---

## uniq

`uniq` filters **adjacent** identical lines. Its real power is `-c` (count), which builds frequency reports.

```bash
$ sort names.txt | uniq          # collapse adjacent dupes (sort first!)
$ sort names.txt | uniq -c       # prefix each line with its count
$ sort names.txt | uniq -d       # show ONLY lines that repeat
$ sort names.txt | uniq -u       # show ONLY lines that never repeat

# classic 'top N': count, then sort by count descending
$ cut -d' ' -f1 access.log | sort | uniq -c | sort -rn | head
      3 10.0.0.5
      2 10.0.0.7
      1 10.0.0.9
```

| Flag | Effect |
|------|--------|
| `-c` | Prefix each line with its count |
| `-d` | Only lines that are duplicated |
| `-u` | Only lines that never repeat |
| `-i` | Ignore case |

> **Tip:** `sort | uniq -c | sort -rn` is the single most useful log-analysis pipeline you'll ever learn: *count occurrences and rank them.* Memorize it.

---

## wc — count lines, words, bytes

```bash
$ wc access.log          # lines  words  bytes  filename
      6      24     198 access.log

$ wc -l access.log       # lines only -> 6
$ wc -w report.txt       # words only
$ wc -c file             # bytes  (-m for characters, multibyte-aware)

# count how many lines matched a filter
$ grep ERROR app.log | wc -l

# how many .conf files in a tree?
$ find /etc -name '*.conf' | wc -l
```

| Flag | Counts |
|------|--------|
| `-l` | Lines |
| `-w` | Words |
| `-c` | Bytes |
| `-m` | Characters (UTF-8 aware) |
| `-L` | Length of the longest line |

---

## tr — translate or delete characters

`tr` replaces or removes characters from a stream (it reads stdin only, so pipe into it).

```bash
$ echo "hello" | tr 'a-z' 'A-Z'        # HELLO  (uppercase)
$ echo "path1,path2" | tr ',' '\n'     # split on commas into lines
$ cat file | tr -d ' '                 # delete all spaces
$ cat file | tr -s ' '                 # squeeze repeated spaces into one
$ cat file.txt | tr -d '\r'            # strip Windows carriage returns
```

| Flag | Effect |
|------|--------|
| `-d` | **Delete** the listed characters |
| `-s` | **Squeeze** repeats of a character into one |
| `-c` | Use the **complement** (everything NOT listed) |

---

## truncate — set a file to an exact size

`truncate` shrinks or extends a file to a given size. Its most common real-world use is **emptying a log file to zero bytes without deleting it** — so the process writing to it keeps its file handle.

```bash
# empty a log in place (keeps ownership, inode, and open handles)
$ truncate -s 0 /var/log/myapp/app.log

# set a file to an exact size
$ truncate -s 100M placeholder.img

# grow or shrink RELATIVE to current size
$ truncate -s +10M file        # add 10 MB
$ truncate -s -5M file         # remove the last 5 MB

# same 'empty the file' effect using redirection (no command needed)
$ > /var/log/myapp/app.log
$ : > /var/log/myapp/app.log
```

> **⚠️ Watch out:** Deleting a log with `rm` while an app holds it open frees **no** disk until the process restarts (the inode stays alive). `truncate -s 0` reclaims the space immediately and is the safe way to clear an active log.

---

### ✅ Quick recap

- Extract columns → `cut -d',' -f2`.
- Order → `sort` (`-n` numeric, `-k` by column).
- Count & rank → `sort | uniq -c | sort -rn`.
- Count lines → `wc -l`.
- Transform characters → `tr`.
- Empty a live log → `truncate -s 0 file`.

---

⬅️ [Prev: Comparing Files](04-comparing-files.md) | 🏠 [Index](README.md) | ➡️ [Next: grep](06-grep.md)
