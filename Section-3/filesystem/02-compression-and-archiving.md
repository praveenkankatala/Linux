# 02 — Compression & Archiving

Two separate ideas often get merged:

- **Archiving** = bundling many files into one (`tar`)
- **Compression** = shrinking bytes (`gzip`, `bzip2`, `xz`)

`tar` can call a compressor for you, which is why you see `.tar.gz` everywhere.

- [The tools at a glance](#the-tools-at-a-glance)
- [gzip & gunzip](#gzip--gunzip)
- [Reading compressed files without unzipping](#reading-compressed-files-without-unzipping)
- [tar — bundling directories](#tar--bundling-directories)

---

## The tools at a glance

| Tool | Job | Extension |
|------|-----|-----------|
| `tar` | Bundle files/dirs into one archive | `.tar` |
| `gzip` / `gunzip` | Fast compression, good ratio | `.gz` |
| `bzip2` | Slower, smaller than gzip | `.bz2` |
| `xz` | Slowest, smallest (best ratio) | `.xz` |
| `zip` / `unzip` | Archive + compress (cross-platform) | `.zip` |

---

## gzip & gunzip

`gzip` compresses a **single file in place**: it replaces `report.txt` with `report.txt.gz` and removes the original. `gunzip` (or `gzip -d`) reverses it.

```bash
# compress (original is replaced by report.txt.gz)
$ gzip report.txt
$ ls
report.txt.gz

# decompress (the .gz is replaced by report.txt)
$ gunzip report.txt.gz
$ gzip -d report.txt.gz          # identical to gunzip

# keep the original AND make a .gz  (-k = keep)
$ gzip -k access.log
$ ls
access.log  access.log.gz

# control the ratio: -1 = fastest ... -9 = smallest
$ gzip -9 bigfile.log
```

---

## Reading compressed files without unzipping

The `z`-family tools read gzipped content directly. This is priceless for rotated logs like `messages-20250601.gz` that you must not permanently unzip.

```bash
$ zcat access.log.gz             # cat a .gz to the screen
$ zless /var/log/messages-*.gz   # page through a rotated log
$ zgrep "ERROR" app.log.gz       # grep INSIDE a .gz
$ zdiff old.log.gz new.log.gz    # diff two .gz files
```

> **Tip:** In real operations you rarely decompress rotated logs. Reach for `zgrep`/`zcat` to search them in place — it saves disk and leaves the archive untouched.

---

## tar — bundling directories

`tar` is the workhorse for packaging directories. Memorize the flags as words:

| Flag | Meaning (mnemonic) |
|------|--------------------|
| `c` | **c**reate an archive |
| `x` | e**x**tract an archive |
| `t` | lis**t** contents (table) |
| `v` | **v**erbose — show file names |
| `f` | **f**ile — the archive name follows (always needed) |
| `z` | filter through g**z**ip (`.tar.gz`) |
| `j` | filter through bzip2 (`.tar.bz2`) |
| `J` | filter through xz (`.tar.xz`) |
| `C` | **C**hange to a directory before acting |

### Create an archive

```bash
# bundle a directory into a plain .tar
$ tar -cvf logs.tar /var/log/myapp/

# bundle AND gzip-compress in one step (most common)
$ tar -czvf logs.tar.gz /var/log/myapp/

# bzip2 (smaller) or xz (smallest)
$ tar -cjvf logs.tar.bz2 /var/log/myapp/
$ tar -cJvf logs.tar.xz  /var/log/myapp/
```

### List contents (without extracting)

```bash
$ tar -tvf logs.tar.gz          # always inspect BEFORE you extract
-rw-r--r-- root/root 1240 2025-06-01 09:10 var/log/myapp/app.log
-rw-r--r-- root/root  512 2025-06-01 09:10 var/log/myapp/error.log
```

### Extract an archive

```bash
# extract into the current directory
$ tar -xzvf logs.tar.gz

# extract into a specific directory  (-C)
$ mkdir /tmp/restore
$ tar -xzvf logs.tar.gz -C /tmp/restore

# extract just ONE file from the archive
$ tar -xzvf logs.tar.gz var/log/myapp/error.log
```

> **Note:** Modern GNU `tar` auto-detects compression on **extract**, so `tar -xvf logs.tar.gz` works even without `z`. Keep `z`/`j`/`J` when **creating**, where it is required.

> **⚠️ Watch out:** Never extract an untrusted archive blindly. Run `tar -tvf` first and confirm paths are **relative** (`var/log/...`), not **absolute** (`/etc/...`) — a malicious archive can overwrite system files.

### Practical one-liners

```bash
# back up /etc with a date-stamped name
$ tar -czvf etc-backup-$(date +%F).tar.gz /etc

# copy a directory tree to a remote host over ssh (no temp file)
$ tar -czf - ./app | ssh user@host "tar -xzf - -C /opt"

# exclude noise while archiving
$ tar -czvf src.tar.gz ./project --exclude='*.log' --exclude='node_modules'
```

---

### ✅ Quick recap

- Compress one file: `gzip -k file` → `gunzip file.gz`.
- Search a `.gz` without unzipping: `zgrep`, `zcat`, `zless`.
- Create: `tar -czvf out.tar.gz dir/` — List: `tar -tvf` — Extract: `tar -xzvf out.tar.gz -C /path`.

---

⬅️ [Prev: Shell Productivity](01-shell-productivity.md) | 🏠 [Index](README.md) | ➡️ [Next: File Display](03-file-display.md)
