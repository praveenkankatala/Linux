# 12. Logging & Diagnostics

[← Back to index](../README.md)

> *"If it isn't in the logs, it didn't happen."*

When a service fails or a login is refused, the logs are the first place to look. Linux has two logging systems: traditional text files in `/var/log`, and the modern `systemd` journal read with `journalctl`.

## Traditional log files (`/var/log`)

| File | Tracks |
|------|--------|
| `/var/log/messages` | General system activity (the catch-all) |
| `/var/log/secure` | Authentication: logins and `sudo` attempts |
| `/var/log/cron` | Scheduled-task execution |
| `/var/log/maillog` | Mail server activity |
| `/var/log/boot.log` | Startup events |
| `/var/log/httpd/` | Apache access and error logs |

> 💡 **Ubuntu names differ:** `/var/log/messages` → `/var/log/syslog`, and `/var/log/secure` → `/var/log/auth.log`.

### Reading logs

```bash
sudo tail -f /var/log/secure                 # follow new lines live (Ctrl+C to stop)
sudo grep "Failed password" /var/log/secure  # search for failed logins
```

> 💡 **Log rotation:** files like `messages.1` or `messages.gz` are *old* logs kept by the `logrotate` utility. It compresses and eventually deletes old data so logs don't fill the disk — which is why the newest events are always in the plain, un-numbered file (e.g. `messages`).

## `journalctl` — the systemd journal

Modern systems collect logs from the kernel and all services into one indexed binary store, queried with `journalctl`. Because it's indexed, searches by service or time are near-instant.

| Command | Shows |
|---------|-------|
| `sudo journalctl -u sshd` | Logs for one service (unit) |
| `journalctl --since "1 hour ago"` | Logs from a relative time window |
| `journalctl --since "today"` | Logs since midnight |
| `sudo journalctl -b` | Logs from the current boot only |
| `sudo journalctl -f` | Follow logs live (like `tail -f`) |
| `sudo journalctl -p err` | Only error-level and worse |
| `sudo journalctl --disk-usage` | How much space the journal uses |

Combine filters for precision:

```bash
sudo journalctl -u sshd --since "2026-06-09 00:00:00" --until "2026-06-10 12:00:00"
sudo journalctl -u sshd -f
```

> ⚠️ **Permissions matter.** Without `sudo`, a normal user sees only their own logs — so a service query may return `-- No entries --` simply for lack of permission. (It can also just mean the service was quiet in that window.)

## The SOS report — the system's "black box"

When a failure is too complex for single commands, RHEL and Amazon Linux generate an **SOS report**: one archive bundling configuration (`/etc`), runtime status, kernel parameters, and logs.

```bash
sudo dnf install sos     # install if missing
sudo sosreport           # generate the report
# output: /var/tmp/sosreport-*.tar.xz
```

> 💡 **Enterprise reality:** open a support ticket with AWS, Red Hat, or Oracle about a crashing server and the first reply is almost always: *"Please generate and upload an sosreport."*

## 🧪 Practice

1. Watch authentication logs live, then trigger an entry:
   ```bash
   sudo tail -f /var/log/secure
   # in a second terminal, run  sudo whoami  — watch it appear, then Ctrl+C
   ```
2. Query SSH logs for the last hour and errors since today:
   ```bash
   sudo journalctl -u sshd --since "1 hour ago"
   sudo journalctl --since today -p err
   ```

**✅ Success check:** You can read authentication events and filter journal logs by service and time.

---
[← Previous: Terminal & Session Management](11-terminal-session-management.md) | [Next: System Maintenance & Recovery →](13-system-maintenance-recovery.md)
