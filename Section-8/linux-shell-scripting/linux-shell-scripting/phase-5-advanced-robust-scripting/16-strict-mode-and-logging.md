[🏠 Home](../README.md)  ·  [Phase 5 — Advanced & Robust Scripting](README.md)  ·  **Topic 16**

# Topic 16 — Defensive Flags and Error Logging

## 16.1  The unofficial standard header

Nearly every production bash script starts with this line. It converts silent, creeping failures into immediate, obvious ones.

*strict-header.sh*

```bash
#!/usr/bin/env bash
set -euo pipefail
IFS=$'\n\t'
```

| Flag | What it does | Why you want it |
| --- | --- | --- |
| `set -e` | Exit the moment any command fails | Stops the script barrelling on after a broken step |
| `set -u` | Error on use of an unset variable | Catches typos like `$USR` instead of `$USER` |
| `set -o pipefail` | A pipeline fails if ANY stage fails | Without it, only the last command's status counts |
| `IFS=$'\n\t'` | Split words only on newline/tab, not spaces | Safer word-splitting for filenames |

*pipefail-demo.sh*

```bash
# Why pipefail matters:
false | sort | cat
echo "without pipefail, status = $?"   # 0 — the failure was hidden!

set -o pipefail
false | sort | cat
echo "with pipefail, status = $?"      # 1 — the truth surfaces
```

```console
without pipefail, status = 0
with pipefail, status = 1
```

## 16.2  The gotchas of set -e

> [!WARNING]
> **set -e is not a magic safety net**
>
> `set -e` has surprising exceptions. A command in an `if` test, or one joined by `&&`/`||`, does NOT trigger an exit even if it fails. And `x=$(cmd_that_fails)` behaviour depends on context. `set -e` is valuable, but write your critical checks explicitly rather than assuming it catches everything.

*set-e-notes.sh*

```bash
set -e

# This does NOT exit — grep 'failing' in a condition is expected to fail:
if grep -q "needle" haystack.txt; then echo found; fi

# Handle a variable that might be unset, without tripping set -u:
name="${1:-anonymous}"    # default if $1 is missing
echo "Hello $name"
```

## 16.3  A reusable error-logging framework

Structured logging with severity levels and a `die` helper is the backbone of any serious script. Here is a compact, battle-tested pattern.

*logging-framework.sh*

```bash
#!/usr/bin/env bash
set -euo pipefail

readonly LOG_FILE="/tmp/$(basename "$0").log"

_log() {
  local level="$1"; shift
  local ts; ts="$(date '+%Y-%m-%d %H:%M:%S')"
  # Write to stderr AND append to the log file
  printf '%s [%-5s] %s\n' "$ts" "$level" "$*" | tee -a "$LOG_FILE" >&2
}

info() { _log INFO  "$@"; }
warn() { _log WARN  "$@"; }
error(){ _log ERROR "$@"; }
die()  { _log FATAL "$@"; exit 1; }

info "Job started"
warn "Disk at 85%"
command -v terraform >/dev/null || die "terraform not found on PATH"
info "All checks passed"
```

```console
2026-07-07 09:41:12 [INFO ] Job started
2026-07-07 09:41:12 [WARN ] Disk at 85%
2026-07-07 09:41:12 [FATAL] terraform not found on PATH
```

> [!TIP]
> **readonly for constants**
>
> Marking config values `readonly` (like `LOG_FILE` above) prevents a later line from accidentally reassigning them — a cheap guard that pairs well with `set -u`.

> [!IMPORTANT]
> **🎯 Try It Yourself — Exercise 16**
>
> Take any earlier script and add the strict header plus the `info`/`die` helpers. Make it `die` with a clear message if a required argument is missing, and `info` at the start and end. Confirm that removing the argument stops the script immediately.

---

⬅️ [Topic 15: Parsing Flags with getopts](../phase-4-intermediate-efficiency/15-getopts.md)  |  ⬆️ [Phase 5 index](README.md)  |  [Topic 17: Traps, Signals, and Cleanup](17-traps-and-signals.md) ➡️
