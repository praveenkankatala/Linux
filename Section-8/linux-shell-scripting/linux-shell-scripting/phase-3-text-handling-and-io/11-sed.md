[🏠 Home](../README.md)  ·  [Phase 3 — Text Handling & I/O](README.md)  ·  **Topic 11**

# Topic 11 — Stream Editing with sed

`sed` (stream editor) transforms text as it flows past. Its most common job by far is find-and-replace across a file or a stream.

## 11.1  Substitution: s/find/replace/

*sed-sub.sh*

```bash
echo "hello world" | sed 's/world/shell/'

# Replace ALL occurrences on each line with the g flag
echo "a-a-a" | sed 's/a/X/g'

# Case-insensitive with the I flag
echo "Error ERROR error" | sed 's/error/OK/gI'
```

```console
hello shell
X-X-X
OK OK OK
```

## 11.2  Editing files: -i and the backup habit

*sed-inplace.sh*

```bash
# Preview first (safe) — prints to screen, file untouched
sed 's/8080/9090/g' server.conf

# Edit in place, keeping a .bak backup (recommended)
sed -i.bak 's/8080/9090/g' server.conf
```

> [!WARNING]
> **-i is destructive — always back up**
>
> `sed -i` rewrites the file with no undo. Get in the habit of `sed -i.bak` which writes `server.conf.bak` alongside the change. On macOS the syntax differs (`sed -i '' ...`), a common cross-platform trap.

## 11.3  Choosing a different delimiter

When your text contains slashes (like file paths), pick another delimiter so you don't have to escape every `/`.

*sed-delim.sh*

```bash
# Ugly — every slash escaped:
echo "/opt/app" | sed 's/\/opt\/app/\/srv\/app/'

# Clean — use | as the delimiter instead:
echo "/opt/app" | sed 's|/opt/app|/srv/app|'
```

```console
/srv/app
/srv/app
```

## 11.4  Deleting and printing specific lines

*sed-lines.sh*

```bash
sed '/^#/d'      config.ini   # delete comment lines
sed '/^$/d'      config.ini   # delete blank lines
sed -n '10,20p'  bigfile.txt  # print only lines 10-20
sed '2d'         data.csv     # delete line 2 (e.g. a header)
```

> [!IMPORTANT]
> **🎯 Try It Yourself — Exercise 11**
>
> You have a config file `app.conf` containing `DEBUG=true` and `PORT=3000`. Write a script that flips `DEBUG` to `false` and changes the port to `8080`, editing the file in place but keeping a backup. Verify by printing the two changed lines afterward with grep.

### Solution

*reconfigure.sh*

```bash
#!/usr/bin/env bash
conf="app.conf"
sed -i.bak \
  -e 's/^DEBUG=.*/DEBUG=false/' \
  -e 's/^PORT=.*/PORT=8080/' \
  "$conf"

grep -E '^(DEBUG|PORT)=' "$conf"
```

```console
DEBUG=false
PORT=8080
```

> [!NOTE]
> `-e` lets you chain multiple edits in one pass. `^DEBUG=.*` anchors to the line start and matches the whole value so the replacement is total.

---

⬅️ [Topic 10: Pipes and Core Filters](10-pipes-and-filters.md)  |  ⬆️ [Phase 3 index](README.md)  |  [Topic 12: Data Extraction with awk](12-awk.md) ➡️
