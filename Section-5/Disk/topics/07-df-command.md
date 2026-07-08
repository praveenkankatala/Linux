# 07 · The `df` Command — Checking Disk Usage

[⬅ Previous: Disk Partitioning](06-disk-partitioning.md) · [Back to index](../README.md) · [Next: LVM ➡](08-lvm.md)

---

## 🎯 What is `df`?

`df` = **disk free**. It answers one urgent question: **"Am I about to run out of disk space?"** It reports how full each *mounted filesystem* is.

> ⛽ **Analogy:** `df` is the fuel gauge for each of your filesystems. One glance tells you which tank is nearly empty.

---

## 🧪 Hands-on

### The everyday command

```bash
df -h            # -h = human-readable (G/M/K instead of raw blocks)
```
```text
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda2       20G   6.4G  14G   32% /
/dev/xvdf1       10G   33M   10G    1% /data
```

### Useful variations

```bash
df -hT           # also show the filesystem TYPE (xfs/ext4/tmpfs)
df -i            # show INODE usage instead of bytes
df -h /data      # just one filesystem
```

> [!IMPORTANT]
> **You can be "full" with free space left — via inodes!**
> Every file uses one **inode** (a metadata slot). A filesystem with millions of tiny files can run out of *inodes* while showing free *bytes*. If writes fail with "No space left on device" but `df -h` looks fine, check `df -i`.

---

## 🆚 `df` vs `du` — don't confuse them

| | `df` | `du` |
|---|------|------|
| Measures | Free space at the **filesystem** level | Space used by **files/directories** |
| Speed | Instant (reads metadata) | Slower (walks the tree) |
| Question it answers | "How full is this disk?" | "What is eating my space?" |

```bash
# du companion — find the biggest directories under /var
sudo du -sh /var/* | sort -rh | head -10
```

> [!TIP]
> **The classic gotcha:** `df` says the disk is 100% full, but `du` of the files adds up to much less. The cause is almost always a **deleted-but-still-open file** — a process is still writing to a file you "removed," so the space isn't freed until the process closes it. Find it with:
> ```bash
> sudo lsof | grep deleted
> ```
> Then restart the offending process (often a log-writing service).

---

## ✅ Key takeaways

- `df -h` = quick, human-readable disk usage per filesystem.
- `df -hT` adds the filesystem type; `df -i` shows **inode** usage.
- `df` = filesystem-level free space; `du` = space used by files.
- "Full but `df` looks fine" → check inodes (`df -i`) or deleted-open files (`lsof | grep deleted`).

## 💬 Interview questions

1. *Disk shows full but you can't find big files — why?* → out of inodes, or a deleted-but-open file held by a process.
2. *Difference between `df` and `du`?* → df = filesystem free space; du = per-file/dir usage.

---

[⬅ Previous: Disk Partitioning](06-disk-partitioning.md) · [Back to index](../README.md) · [Next: LVM ➡](08-lvm.md)
