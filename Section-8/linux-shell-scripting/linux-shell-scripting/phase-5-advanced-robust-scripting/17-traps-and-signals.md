[🏠 Home](../README.md)  ·  [Phase 5 — Advanced & Robust Scripting](README.md)  ·  **Topic 17**

# Topic 17 — Traps, Signals, and Cleanup

A `trap` runs a command when the script receives a signal or is about to exit. Its number-one use is **cleanup**: removing temp files, releasing locks, and restoring state — reliably, even if the script crashes or the user hits Ctrl-C.

## 17.1  Common signals

| Signal | Triggered by | Note |
| --- | --- | --- |
| `EXIT` | The script ending for any reason | The workhorse for cleanup |
| `INT` (2) | Ctrl-C | User interrupt |
| `TERM` (15) | `kill <pid>` | Polite termination request |
| `HUP` (1) | Terminal closed | Often used to trigger config reload |
| `ERR` | Any command failing (bash) | Great for a debug/error hook |
| `KILL` (9) | `kill -9` | Cannot be trapped — never runs cleanup |

## 17.2  The trap ... EXIT cleanup pattern

Register cleanup once, at the top. Because it's on `EXIT`, it runs whether the script finishes normally, errors out, or is interrupted — you never leak the temp file.

*trap-cleanup.sh*

```bash
#!/usr/bin/env bash
set -euo pipefail

workdir="$(mktemp -d)"      # create a private temp directory

cleanup() {
  local rc=$?
  rm -rf "$workdir"
  echo "Cleaned up $workdir (exit code $rc)"
}
trap cleanup EXIT           # run cleanup no matter how we leave

echo "Working in $workdir"
echo "data" > "$workdir/file.txt"
# ... do real work; even if this crashes, cleanup still runs ...
```

```console
Working in /tmp/tmp.a1B2c3
Cleaned up /tmp/tmp.a1B2c3 (exit code 0)
```

> [!TIP]
> **Capture $? first inside the handler**
>
> `local rc=$?` must be the very first line of the handler — it grabs the exit status that triggered the trap before any command inside the handler overwrites it. Then you can log or branch on the real result.

## 17.3  Handling Ctrl-C gracefully

*trap-int.sh*

```bash
#!/usr/bin/env bash
interrupted=0
on_int() { interrupted=1; echo; echo "Interrupt received — finishing current step..."; }
trap on_int INT

for i in {1..5}; do
  (( interrupted )) && { echo "Stopping early after step $((i-1))"; break; }
  echo "Step $i"; sleep 1
done
echo "Done."
```

> [!NOTE]
> Instead of dying abruptly on Ctrl-C, this sets a flag and lets the loop reach a safe stopping point — the pattern you want for scripts that must not be interrupted mid-operation (e.g. mid-write).

## 17.4  A production-shaped example: a lock file

*lockfile.sh*

```bash
#!/usr/bin/env bash
set -euo pipefail
LOCK="/tmp/myjob.lock"

if ! ( set -o noclobber; : > "$LOCK" ) 2>/dev/null; then
  echo "Another instance is already running." >&2; exit 1
fi
trap 'rm -f "$LOCK"' EXIT   # release the lock on exit

echo "Lock acquired — doing work"
sleep 2
echo "Work complete"
```

> [!WARNING]
> **kill -9 skips your trap**
>
> `SIGKILL` (9) and `SIGSTOP` cannot be trapped — the kernel removes the process immediately, so cleanup never runs and stale lock files can linger. Design for it: check for a stale lock on startup, or use `flock` which the kernel releases automatically when the process dies.

> [!IMPORTANT]
> **🎯 Try It Yourself — Exercise 17**
>
> Write a script that downloads (simulate with `mktemp` + `echo`) to a temp file, processes it, and moves the result into place. Use `trap ... EXIT` so that if processing fails partway, the temp file is always removed and the destination is never left half-written.

---

⬅️ [Topic 16: Defensive Flags and Error Logging](16-strict-mode-and-logging.md)  |  ⬆️ [Phase 5 index](README.md)  |  [Topic 18: Associative Arrays (Hash Maps)](18-associative-arrays.md) ➡️
