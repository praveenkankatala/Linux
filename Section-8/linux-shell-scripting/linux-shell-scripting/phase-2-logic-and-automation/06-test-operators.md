[🏠 Home](../README.md)  ·  [Phase 2 — Logic & Automation](README.md)  ·  **Topic 6**

# Topic 6 — String, Integer, and File Tests

The `[[ ... ]]` construct compares things. Which operator you use depends on **what kind** of data you're comparing. Mixing them up is a classic source of bugs — string operators on numbers, or vice versa.

## 6.1  String comparisons

| Test | True when |
| --- | --- |
| `[[ "$a" == "$b" ]]` | Strings are equal |
| `[[ "$a" != "$b" ]]` | Strings differ |
| `[[ -z "$a" ]]` | String is empty (zero length) |
| `[[ -n "$a" ]]` | String is non-empty |
| `[[ "$a" == pre* ]]` | String matches a glob pattern |
| `[[ "$a" < "$b" ]]` | Sorts before (lexicographic) |

*string-test.sh*

```bash
env="prod"

if [[ "$env" == "prod" ]]; then
  echo "Production — extra confirmations required"
fi

if [[ "$env" == pr* ]]; then
  echo "Environment name starts with 'pr'"
fi
```

## 6.2  Integer comparisons

For numbers, use the word operators inside `[[ ]]`, or the symbol operators inside `(( ))`. Do not use `==`/`<` for numbers — they compare as text, so `"10" < "9"` is *true*.

| [[ ]] form | (( )) form | Meaning |
| --- | --- | --- |
| `-eq` | `==` | equal |
| `-ne` | `!=` | not equal |
| `-gt` | `>` | greater than |
| `-ge` | `>=` | greater or equal |
| `-lt` | `<` | less than |
| `-le` | `<=` | less or equal |

*int-test.sh*

```bash
count=42

# Both of these are equivalent and correct for numbers:
if [[ "$count" -gt 40 ]]; then echo "over 40 (word form)"; fi
if (( count > 40 ));      then echo "over 40 (symbol form)"; fi
```

> [!WARNING]
> **The bug that hides until 10**
>
> `[[ "$a" > "$b" ]]` on numbers compares them as *strings*: `[[ "9" > "10" ]]` is true because '9' sorts after '1'. Use `-gt` inside `[[ ]]`, or switch to `(( a > b ))`. This bug passes every single-digit test and then fails in production.

## 6.3  File tests

These operators ask the filesystem about a path. They are the backbone of any script that touches files.

| Test | True when |
| --- | --- |
| `-e path` | Path exists (any type) |
| `-f path` | Exists and is a regular file |
| `-d path` | Exists and is a directory |
| `-r / -w / -x path` | You can read / write / execute it |
| `-s path` | Exists and is not empty |
| `-L path` | Is a symbolic link |
| `file1 -nt file2` | file1 is newer than file2 |

*file-test.sh*

```bash
#!/usr/bin/env bash
target="/etc/nginx/nginx.conf"

if [[ ! -e "$target" ]]; then
  echo "$target does not exist" >&2
  exit 1
elif [[ -f "$target" && -r "$target" ]]; then
  echo "$target is a readable file — proceeding"
else
  echo "$target exists but is not a readable file" >&2
  exit 1
fi
```

> [!NOTE]
> **Combine tests with && and ||**
>
> Inside `[[ ]]` you can join conditions: `[[ -f "$f" && -s "$f" ]]` means 'a regular file that is not empty'. `!` negates: `[[ ! -d "$d" ]]` means 'not a directory'.

> [!IMPORTANT]
> **🎯 Try It Yourself — Exercise 6**
>
> Write `logrotate-lite.sh <logfile>`. If the file exists and is larger than 0 bytes, rename it to `<logfile>.old` (use `mv`) and create a fresh empty one with `touch`. If it doesn't exist, print a message and exit 1. Test the number comparison and the file tests together.

### Solution

*logrotate-lite.sh*

```bash
#!/usr/bin/env bash
log="$1"

if [[ -z "$log" ]]; then
  echo "Usage: $0 <logfile>" >&2; exit 1
fi

if [[ -f "$log" && -s "$log" ]]; then
  mv "$log" "${log}.old"
  touch "$log"
  echo "Rotated $log -> ${log}.old"
else
  echo "$log missing or empty — nothing to rotate" >&2
  exit 1
fi
```

---

⬅️ [Topic 5: Exit Status and Conditionals](05-exit-status-and-conditionals.md)  |  ⬆️ [Phase 2 index](README.md)  |  [Topic 7: for Loops](07-for-loops.md) ➡️
