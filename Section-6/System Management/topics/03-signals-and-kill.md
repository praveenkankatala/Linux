# 3. Signals & Killing Processes (`kill`)

[← Back to index](../README.md)

Despite the scary name, `kill` doesn't only "destroy" — it **sends a signal** to a process by its PID. Signals can ask a process to stop, force it to stop, or tell it to reload.

## The three signals you'll actually use

| Signal | Name | What it does |
|--------|------|--------------|
| `-15` | SIGTERM | **Polite request** (default). Lets the process save its work, close files, and exit cleanly. |
| `-9` | SIGKILL | **Hard stop.** The kernel kills it instantly — no chance to save. Last resort only. |
| `-1` | SIGHUP | **Reload.** Tells a service (nginx, Apache) to re-read its config without a full restart. |

## Usage

```bash
kill 1234          # send SIGTERM to PID 1234 — try this first
kill -9 1234       # force kill a frozen process
pkill nginx        # kill by name instead of PID
```

## Rules to remember

- **Try gentle first.** Run `kill -15` (or plain `kill`), wait a few seconds, and only escalate to `kill -9` if it's still stuck. `-9` can leave temp files and corrupt data because the program never gets to clean up.
- **Permissions matter.** You can only kill **your own** processes. Use `sudo` to kill processes owned by root or another user.
- **Find the PID first** using `ps` or `top` (see the previous topic).

## 🧪 Practice

Continuing from the previous topic's `sleep` process:

1. Find the PID:
   ```bash
   ps -ef | grep [s]leep
   ```
2. Terminate it gracefully:
   ```bash
   kill -15 <PID>
   ```
3. Confirm it's gone:
   ```bash
   ps -ef | grep [s]leep      # no result = success
   ```

**✅ Success check:** The `sleep` process no longer appears in the list.

---
[← Previous: Process Monitoring](02-process-monitoring.md) | [Next: Task Scheduling →](04-task-scheduling.md)
