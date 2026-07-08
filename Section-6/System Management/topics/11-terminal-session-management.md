# 11. Terminal & Session Management

[← Back to index](../README.md)

Tools for managing your terminal itself — clearing the screen, recording what you do, and keeping work alive when your connection drops.

## Everyday basics

- **`clear`** — wipes the screen. Faster: press **Ctrl + L**.
- **`exit`** — closes the shell or session. **Ctrl + D** does the same (sends "end of file"). After `sudo su -`, `exit` drops you back to your normal user.

## `script` — record a terminal session

`script` saves **everything** shown in your terminal (what you type and what comes back) to a file — perfect for lab logs, documentation, and proving what you did.

```bash
script lab_session.log   # start recording (default file: "typescript")
# ... run your commands ...
exit                     # stop recording
cat lab_session.log      # review the whole session
```

## `screen` — keep sessions alive

On a remote server, if your SSH connection drops mid-task, the task dies with it. **`screen`** creates a session that keeps running even after you disconnect — essential for long jobs.

| Command / key | Action |
|---------------|--------|
| `screen -S name` | Start a **named** session |
| `screen -ls` | List running sessions |
| `screen -r name` | Reattach to a session |
| **Ctrl+A** then **D** | **Detach** (leave it running in the background) |
| **Ctrl+A** then **K** | Kill the current session |

**Real-world example:** downloading a 50 GB file over SSH:

```bash
screen -S bigdownload    # start a named session
# start the download...
# press Ctrl+A then D to detach, then log out and go to sleep
# next morning:
screen -r bigdownload    # reattach — download finished while you were away
```

> ⚠️ **Detach, don't exit.** Typing `exit` inside `screen` **kills** the session. To leave a job running, detach with **Ctrl+A then D**.

> 💡 **`tmux`** is the modern alternative — same idea, plus split panes and windows.

## 🧪 Practice

1. Record a short session:
   ```bash
   script demo.log
   uptime
   df -h
   exit
   cat demo.log         # your commands and their output are saved
   ```
2. Try `screen`:
   ```bash
   screen -S test
   # inside: run  top
   # press Ctrl+A then D to detach
   screen -ls           # see your detached session
   screen -r test       # reattach; press q to quit top; type exit to close
   ```

**✅ Success check:** `demo.log` contains your commands, and you can detach and reattach a `screen` session.

---
[← Previous: Environment Variables](10-environment-variables.md) | [Next: Logging & Diagnostics →](12-logging-and-diagnostics.md)
