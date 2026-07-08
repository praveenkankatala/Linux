[🏠 Home](../README.md)  ·  [Phase 3 — Text Handling & I/O](README.md)  ·  **Topic 10**

# Topic 10 — Pipes and Core Filters

A pipe `|` connects the stdout of one command to the stdin of the next. Chaining small filters is the heart of shell data processing.

## 10.1  grep — find matching lines

| Option | Effect |
| --- | --- |
| `-i` | Case-insensitive |
| `-v` | Invert — show lines that do NOT match |
| `-c` | Count matching lines |
| `-n` | Show line numbers |
| `-r` | Recurse into directories |
| `-E` | Extended regex (enables +, ?, \|, ()) |
| `-o` | Print only the matched part |

*grep-demo.sh*

```bash
# Find failed SSH logins, count them, show the offending IPs
grep -i "failed password" /var/log/auth.log | wc -l
grep -oE "([0-9]{1,3}\.){3}[0-9]{1,3}" /var/log/auth.log | head
```

## 10.2  cut — slice out columns

*cut-demo.sh*

```bash
# /etc/passwd is colon-delimited: user:x:uid:gid:info:home:shell
cut -d: -f1     /etc/passwd | head -3   # just usernames
cut -d: -f1,7   /etc/passwd | head -3   # username + shell
```

```console
root
daemon
bin
root:/bin/bash
daemon:/usr/sbin/nologin
bin:/usr/sbin/nologin
```

> [!NOTE]
> `-d` sets the delimiter, `-f` selects fields. `cut` is fast but only handles a *single-character* delimiter and can't cope with variable whitespace — that's `awk`'s job (Topic 12).

## 10.3  sort and uniq — the counting combo

`uniq` only collapses *adjacent* duplicates, so you almost always `sort` first. The `sort | uniq -c | sort -rn` chain is one of the most useful one-liners in all of shell.

*top-ips.sh*

```bash
# Top 5 IPs by number of failed logins
grep "Failed password" /var/log/auth.log \
  | grep -oE "([0-9]{1,3}\.){3}[0-9]{1,3}" \
  | sort \
  | uniq -c \
  | sort -rn \
  | head -5
```

```console
     47 203.0.113.12
     31 198.51.100.7
     22 192.0.2.44
      9 203.0.113.99
      5 198.51.100.2
```

| Command | Key options |
| --- | --- |
| `sort` | `-n` numeric, `-r` reverse, `-k2` by column 2, `-t:` set delimiter, `-u` unique |
| `uniq` | `-c` prefix counts, `-d` only duplicates, `-u` only uniques, `-i` ignore case |

## 10.4  Other filters you'll reach for

| Filter | Does |
| --- | --- |
| `wc -l / -w / -c` | Count lines / words / bytes |
| `head -n N / tail -n N` | First / last N lines |
| `tail -f` | Follow a file as it grows (live logs) |
| `tr` | Translate or delete characters |
| `tee file` | Write to a file AND pass through the pipe |
| `xargs` | Turn stdin into arguments for another command |

> [!IMPORTANT]
> **🎯 Try It Yourself — Exercise 10**
>
> From `/etc/passwd`, produce a count of how many accounts use each login shell, sorted from most common to least. (Field 7 is the shell, delimiter `:`.) Expected shape: a count next to each shell path.

### Solution

*shell-census.sh*

```bash
cut -d: -f7 /etc/passwd | sort | uniq -c | sort -rn
```

```console
     28 /usr/sbin/nologin
      3 /bin/bash
      1 /bin/sync
      1 /usr/bin/zsh
```

---

⬅️ [Topic 9: stdout, stderr, and Redirection](09-redirection.md)  |  ⬆️ [Phase 3 index](README.md)  |  [Topic 11: Stream Editing with sed](11-sed.md) ➡️
