# 4. Task Scheduling (`crontab` & `at`)

[← Back to index](../README.md)

Linux can run jobs for you automatically. Use **`cron`** for jobs that repeat, and **`at`** for a job that runs once.

## `crontab` — recurring jobs

Great for "every night at 2 AM, back up the database."

| Command | Purpose |
|---------|---------|
| `crontab -e` | Edit your scheduled jobs (opens an editor) |
| `crontab -l` | List your jobs |
| `crontab -r` | Remove **all** your jobs (careful!) |

### The five time fields

Every cron line starts with five fields, then the command:

```
┌── minute (0–59)
│ ┌── hour (0–23)
│ │ ┌── day of month (1–31)
│ │ │ ┌── month (1–12)
│ │ │ │ ┌── day of week (0–6, Sun–Sat)
│ │ │ │ │
* * * * *  command-to-run
```

### Examples

```bash
0 2 * * *    /path/to/backup.sh                 # every day at 02:00
*/15 * * * * /usr/bin/python3 /opt/job.py        # every 15 minutes
0 8 * * 1    /home/user/weekly_report.sh         # every Monday at 08:00
```

### Cron pro-tips

- **Always use full paths.** Cron runs with a bare environment. Write `/usr/bin/python3 /home/user/script.py`, not `python script.py`.
- **Capture output.** Cron is silent — send output to a log so you can debug:
  ```bash
  0 2 * * * /path/backup.sh >> /var/log/backup.log 2>&1
  ```
- **No seconds field.** The smallest interval cron supports is **one minute**.

## `at` — one-time jobs

Runs a command **once** at a future time, then forgets it.

```bash
at 14:00               # then type your command(s), press Ctrl+D to save
at now + 30 minutes
at 09:00 AM July 4
```

**Managing pending jobs:**

| Command | Purpose |
|---------|---------|
| `atq` | List pending jobs (shows a job ID) |
| `atrm <id>` | Remove a pending job |
| `at -c <id>` | Show what a pending job will run |

> ⚠️ The `atd` background service must be running for `at` to work. Check it:
> `systemctl status atd`

## 🧪 Practice

1. Add a cron job that logs the date every minute:
   ```bash
   crontab -e
   # add this line, save, and exit:
   * * * * * date >> ~/cron_test.log
   ```
2. Wait two minutes, then check:
   ```bash
   cat ~/cron_test.log
   ```
3. Schedule a one-off job:
   ```bash
   echo 'touch ~/at_done' | at now + 1 minute
   atq                      # see it queued
   ```

**✅ Success check:** `cron_test.log` gains a new timestamped line each minute, and `~/at_done` appears after a minute. Clean up with `crontab -e` (remove the line) when done.

---
[← Previous: Signals & Killing Processes](03-signals-and-kill.md) | [Next: Text Processing with sed →](05-text-processing-sed.md)
