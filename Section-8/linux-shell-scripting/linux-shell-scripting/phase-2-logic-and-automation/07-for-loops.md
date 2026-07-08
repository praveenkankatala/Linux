[🏠 Home](../README.md)  ·  [Phase 2 — Logic & Automation](README.md)  ·  **Topic 7**

# Topic 7 — for Loops

## 7.1  Looping over a list of words

*for-list.sh*

```bash
#!/usr/bin/env bash
for env in dev staging prod; do
  echo "Deploying to $env..."
done
```

```console
Deploying to dev...
Deploying to staging...
Deploying to prod...
```

## 7.2  Looping over files (globbing)

This is the everyday workhorse: apply an action to every file matching a pattern. Let the shell expand the glob — never loop over `$(ls)`.

*for-files.sh*

```bash
#!/usr/bin/env bash
# Compress every .log file in the current directory
for logfile in ./*.log; do
  [[ -e "$logfile" ]] || continue   # skip if no matches
  echo "Compressing $logfile"
  gzip "$logfile"
done
```

> [!WARNING]
> **Never loop over $(ls)**
>
> `for f in $(ls)` breaks the moment a filename contains a space or a wildcard character — it splits `My File.txt` into two iterations. Loop over a glob (`for f in ./*.log`) instead: the shell hands you each real filename intact. The `|| continue` guard handles the case where nothing matches.

## 7.3  C-style and numeric-range loops

*for-range.sh*

```bash
# C-style: full control over the counter
for (( i = 1; i <= 5; i++ )); do
  echo "Attempt $i"
done

# Brace expansion for a fixed range
for n in {1..3}; do echo "item $n"; done

# Step by 2 using {start..end..step}
for even in {0..10..2}; do echo -n "$even "; done; echo
```

```console
Attempt 1
Attempt 2
Attempt 3
Attempt 4
Attempt 5
item 1
item 2
item 3
0 2 4 6 8 10
```

## 7.4  break and continue

*break-continue.sh*

```bash
for i in {1..10}; do
  (( i % 2 == 0 )) && continue   # skip even numbers
  (( i > 7 )) && break           # stop once past 7
  echo "odd: $i"
done
```

```console
odd: 1
odd: 3
odd: 5
odd: 7
```

> [!IMPORTANT]
> **🎯 Try It Yourself — Exercise 7**
>
> Write `bulk-rename.sh` that loops over every `*.txt` file in the current directory and prints the command that would rename it to `.bak` (e.g. `mv notes.txt notes.bak`). Once the output looks right, make it actually run the `mv`. Guard against the no-match case.

### Solution

*bulk-rename.sh*

```bash
#!/usr/bin/env bash
for f in ./*.txt; do
  [[ -e "$f" ]] || { echo "No .txt files here."; break; }
  new="${f%.txt}.bak"     # strip .txt, add .bak
  echo "mv '$f' '$new'"
  mv "$f" "$new"
done
```

> [!NOTE]
> `${f%.txt}` removes the shortest matching `.txt` suffix — a taste of the string manipulation covered in Phase 4, Topic 14.

---

⬅️ [Topic 6: String, Integer, and File Tests](06-test-operators.md)  |  ⬆️ [Phase 2 index](README.md)  |  [Topic 8: while Loops](08-while-loops.md) ➡️
