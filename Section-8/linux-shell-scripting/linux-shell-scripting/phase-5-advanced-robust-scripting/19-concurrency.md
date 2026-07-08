[🏠 Home](../README.md)  ·  [Phase 5 — Advanced & Robust Scripting](README.md)  ·  **Topic 19**

# Topic 19 — Concurrency: Background Jobs, wait, Subshells

When tasks are independent — pinging 50 hosts, processing 100 files — running them one at a time wastes time. Backgrounding with `&` and synchronising with `wait` lets you do them in parallel.

## 19.1  Background a job with & and reap it with wait

*parallel-basic.sh*

```bash
#!/usr/bin/env bash
echo "Starting three slow tasks in parallel..."

sleep 3 & echo "  task A pid $!"
sleep 2 & echo "  task B pid $!"
sleep 1 & echo "  task C pid $!"

wait                       # block until ALL background jobs finish
echo "All tasks done (took ~3s, not 6s)"
```

```console
Starting three slow tasks in parallel...
  task A pid 20481
  task B pid 20482
  task C pid 20483
All tasks done (took ~3s, not 6s)
```

| Token | Meaning |
| --- | --- |
| `cmd &` | Run cmd in the background, keep going |
| `$!` | PID of the most recently backgrounded job |
| `wait` | Wait for all background jobs |
| `wait $pid` | Wait for one specific job and get ITS exit status |
| `jobs` | List active background jobs in this shell |

## 19.2  Collecting exit statuses

A bare `wait` loses individual results. To know which job failed, keep the PIDs and `wait` on each.

*parallel-ping.sh*

```bash
#!/usr/bin/env bash
hosts=("8.8.8.8" "1.1.1.1" "192.0.2.1")   # last one won't respond
declare -A pid_of

for h in "${hosts[@]}"; do
  ping -c1 -W1 "$h" &>/dev/null &
  pid_of[$!]="$h"
done

for pid in "${!pid_of[@]}"; do
  if wait "$pid"; then echo "UP   ${pid_of[$pid]}"
  else echo "DOWN ${pid_of[$pid]}"; fi
done
```

```console
UP   8.8.8.8
UP   1.1.1.1
DOWN 192.0.2.1
```

## 19.3  Throttling with xargs -P

Launching thousands of jobs at once will overwhelm a machine. `xargs -P N` runs at most N in parallel — the simplest robust throttle in shell.

*throttle.sh*

```bash
# Convert every .png to .jpg, 4 at a time
ls *.png | xargs -P4 -I{} sh -c 'convert "{}" "${1%.png}.jpg"' _ {}

# Or with GNU parallel if available:
# parallel -j4 convert {} {.}.jpg ::: *.png
```

## 19.4  Subshells: ( ) vs { }

Parentheses run commands in a **subshell** — a child process with its own copy of variables and working directory. Changes inside don't affect the parent. Braces group commands in the *current* shell.

*subshell.sh*

```bash
dir="/home"

( cd /tmp; echo "inside subshell: $PWD" )   # child — cd is local
echo "back in parent:  $PWD"                # unchanged!

x=1
( x=99; echo "subshell x=$x" )              # child copy
echo "parent x=$x"                          # still 1
```

```console
inside subshell: /tmp
back in parent:  /home/devops
subshell x=99
parent x=1
```

> [!WARNING]
> **The subshell variable trap (revisited)**
>
> This is why `cmd | while read ...` can't update outer variables — the pipe puts the `while` in a subshell whose changes vanish. Now you can see the mechanism. The fix, again, is `while read ...; done < <(cmd)` from Topic 20.

> [!IMPORTANT]
> **🎯 Try It Yourself — Exercise 19**
>
> You have a list of URLs in `urls.txt`. Write a script that checks each one's HTTP status with `curl -s -o /dev/null -w '%{http_code}'` in parallel (background each, keep the PID→URL map), then reports which returned a non-200. Bonus: throttle to 5 concurrent with `xargs -P5`.

---

⬅️ [Topic 18: Associative Arrays (Hash Maps)](18-associative-arrays.md)  |  ⬆️ [Phase 5 index](README.md)  |  [Topic 20: Advanced awk and Process Substitution](20-advanced-awk-and-process-substitution.md) ➡️
