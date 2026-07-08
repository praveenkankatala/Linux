[🏠 Home](../README.md)  ·  [Phase 3 — Text Handling & I/O](README.md)  ·  **Topic 9**

# Topic 9 — stdout, stderr, and Redirection

## 9.1  Redirecting stdout to a file

| Operator | Meaning |
| --- | --- |
| `> file` | Send stdout to file, **overwriting** it |
| `>> file` | Send stdout to file, **appending** |
| `< file` | Read stdin from file |
| `2> file` | Send stderr to file |
| `2>> file` | Append stderr to file |
| `&> file` | Send both stdout and stderr to file |
| `2>&1` | Send stderr to wherever stdout is currently going |

*redirect.sh*

```bash
echo "first line"  > out.txt   # creates / overwrites
echo "second line" >> out.txt  # appends
cat out.txt
```

```console
first line
second line
```

> [!WARNING]
> **> overwrites without warning**
>
> A single `>` silently truncates the target file to empty before writing. Run `command > important.log` by mistake and the old contents are gone. Use `>>` when you mean to add. You can enable `set -o noclobber` to make `>` refuse to overwrite an existing file.

## 9.2  Separating errors from output

This is why the streams exist. Save real results to one place, errors to another.

*split-streams.sh*

```bash
# find will hit some permission-denied dirs (stderr)
# and some real matches (stdout). Split them:
find /etc -name '*.conf' > results.txt 2> errors.txt

echo "Matches: $(wc -l < results.txt)"
echo "Errors:  $(wc -l < errors.txt)"
```

```console
Matches: 63
Errors:  4
```

## 9.3  /dev/null — the black hole

*devnull.sh*

```bash
command 2>/dev/null    # discard errors, keep output
command >/dev/null     # discard output, keep errors
command &>/dev/null    # discard everything (just check exit status)
```

## 9.4  Writing to stderr from your own script

Good scripts send error and diagnostic messages to stderr, not stdout, so they don't pollute the data a caller is trying to capture.

*log-helpers.sh*

```bash
#!/usr/bin/env bash
log()  { echo "[INFO]  $*"; }        # normal output -> stdout
err()  { echo "[ERROR] $*" >&2; }    # errors -> stderr

log "Starting job"
err "Could not reach database"
```

> [!TIP]
> **The 2>&1 ordering trap**
>
> `>file 2>&1` sends both streams to the file. But `2>&1 >file` does **not** — order matters: it points stderr at the *terminal* (where stdout currently is), then moves stdout to the file. Always put `2>&1` **last**.

> [!IMPORTANT]
> **🎯 Try It Yourself — Exercise 9**
>
> Write `safe-run.sh <command...>` that runs whatever command you pass it, appends its stdout to `run.log` and its stderr to `error.log` (both timestamped is a bonus), and prints `OK` or `FAILED` based on the exit status.

### Solution

*safe-run.sh*

```bash
#!/usr/bin/env bash
ts="$(date '+%F %T')"
echo "[$ts] $*" >> run.log

if "$@" >> run.log 2>> error.log; then
  echo "OK"
else
  echo "FAILED (exit $?)" >&2
fi
```

---

⬅️ [Topic 8: while Loops](../phase-2-logic-and-automation/08-while-loops.md)  |  ⬆️ [Phase 3 index](README.md)  |  [Topic 10: Pipes and Core Filters](10-pipes-and-filters.md) ➡️
