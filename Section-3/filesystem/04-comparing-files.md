# 04 — Comparing Files: `diff` & `cmp`

Both compare files, but they answer different questions:

- **`diff`** shows *how text differs, line by line* — ideal for configs and code.
- **`cmp`** shows *whether two files differ at all*, byte by byte — ideal for binaries and quick equality checks.

Set up two versions to compare:

```bash
$ printf 'alpha\nbeta\ngamma\n'          > v1.txt
$ printf 'alpha\nBETA\ngamma\ndelta\n'   > v2.txt
```

---

## diff — line-by-line differences

```bash
$ diff v1.txt v2.txt
2c2
< beta
---
> BETA
3a4
> delta
```

Reading the default output:

- `2c2` → line 2 **changed**.
- `<` is the **left** file (v1), `>` is the **right** file (v2).
- `3a4` → after line 3, v2 **added** line 4 (`delta`).
- You'll also see `d` for **deleted** lines.

### The formats you'll actually use

```bash
# unified format (-u): the format of patches and code review
$ diff -u v1.txt v2.txt
--- v1.txt
+++ v2.txt
@@ -1,3 +1,4 @@
 alpha
-beta
+BETA
 gamma
+delta

# side-by-side (-y): easiest to eyeball
$ diff -y v1.txt v2.txt
alpha             alpha
beta            | BETA
gamma             gamma
                > delta

# ignore case / whitespace
$ diff -i v1.txt v2.txt      # ignore case
$ diff -w v1.txt v2.txt      # ignore all whitespace

# compare whole directories recursively
$ diff -r /etc/old /etc/new
```

> **Tip:** `diff -u a b > change.patch` captures a change; `patch < change.patch` applies it. This is the backbone of code review and config-drift detection.

---

## cmp — do these files differ, and where?

```bash
$ cmp v1.txt v2.txt
v1.txt v2.txt differ: byte 7, line 2

# silent mode for scripts: exit code 0 = identical, 1 = differ
$ cmp -s fileA fileB && echo "same" || echo "different"

# great for binaries where diff is meaningless
$ cmp app-v1.bin app-v2.bin
```

---

## Which one should I use?

| | `diff` | `cmp` |
|--|--------|-------|
| **Best for** | Text files, configs, code | Binaries, quick equality check |
| **Output** | Which lines changed, and how | First differing byte/line |
| **Exit code** | 0 same, 1 differ, 2 error | 0 same, 1 differ, 2 error |

---

### ✅ Quick recap

- Human-readable text differences → `diff -u a b`.
- "Are these two files identical?" in a script → `cmp -s a b`.

---

⬅️ [Prev: File Display](03-file-display.md) | 🏠 [Index](README.md) | ➡️ [Next: Filters & Text Processing](05-filters-and-text-processing.md)
