# 2. Process Monitoring (`ps` & `top`)

[← Back to index](../README.md)

A **process** is any running program. Every process has a unique number called a **PID** (Process ID). Two tools let you see them: `ps` takes a still photo, `top` shows a live video.

## `ps` — a snapshot in time

`ps` shows the processes running at the exact moment you press Enter.

| Command | What it shows |
|---------|---------------|
| `ps` | Only processes in your current terminal |
| `ps aux` | **BSD style** — every process, with CPU/memory usage |
| `ps -ef` | **System V style** — every process, with parent PID (PPID) |
| `ps -ef \| grep sshd` | Filter the list for one program |

**Reading the columns:**

| Column | Meaning |
|--------|---------|
| USER | Who owns the process |
| PID | Unique process ID |
| PPID | Parent process ID (who started it) |
| %CPU / %MEM | Share of processor / RAM used |
| STAT | State: **R** running, **S** sleeping, **Z** zombie |
| COMMAND | The program being run |

```bash
$ ps -ef | grep sshd
root      1234     1  0 10:15 ?   00:00:00 /usr/sbin/sshd -D
```

> 💡 **Grep shows itself in the results.** To hide the search command, bracket the first letter:
> `ps -ef | grep [s]shd`

## `top` — a live dashboard

Run `top` for a self-refreshing view of CPU, memory, and the busiest processes. Press **`q`** to quit.

**Useful keys inside `top`:**

| Key | Action |
|-----|--------|
| `P` | Sort by CPU usage |
| `M` | Sort by memory usage |
| `k` | Kill a process (asks for the PID) |
| `u` | Show only one user's processes |
| `1` | Show each CPU core separately |
| `q` | Quit |

**Reading the header (top few lines):**

- **Load average** — workload over the last 1, 5, and 15 minutes. If it's consistently higher than your number of CPU cores, the system is overloaded.
- **%Cpu(s)** — watch `id` (idle). Near **0** means the CPU is maxed out. A high `wa` (I/O wait) means the **disk** is the bottleneck, not the CPU.
- **MiB Mem** — total, free, used, and cached memory.

> 💡 **`htop`** is a friendlier, colorful version of `top` with mouse support. Install it:
> `sudo dnf install htop` (RHEL/Amazon) or `sudo apt install htop` (Ubuntu).

## 🧪 Practice

1. Start a harmless background process:
   ```bash
   sleep 600 &
   ```
2. Find its PID:
   ```bash
   ps -ef | grep [s]leep
   ```
3. Open `top`, press `M` to sort by memory, then `q` to quit.

**✅ Success check:** You can locate the `sleep` process and read its PID. (You'll terminate it in the next topic.)

---
[← Previous: System Services](01-system-services.md) | [Next: Signals & Killing Processes →](03-signals-and-kill.md)
