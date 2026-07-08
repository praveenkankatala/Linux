# 1. System Services (`systemctl` / `service`)

[← Back to index](../README.md)

## What is a service?

A **service** (also called a **daemon**) is a program that runs quietly in the background and waits to do work — a web server waiting for visitors, a database waiting for queries, or the SSH server waiting for you to log in. Most services start automatically when the machine boots.

Modern Linux uses a system manager called **systemd** to start, stop, and supervise these services. The tool you use to control it is **`systemctl`**.

> **Analogy:** Think of `systemctl` as the control panel for every background program on your machine — an on/off switch, a status light, and a "start automatically" toggle for each one.

## Core commands

| Action | Modern (`systemctl`) | Legacy (`service` / `chkconfig`) |
|--------|----------------------|----------------------------------|
| Start a service | `systemctl start <svc>` | `service <svc> start` |
| Stop a service | `systemctl stop <svc>` | `service <svc> stop` |
| Restart a service | `systemctl restart <svc>` | `service <svc> restart` |
| Check status | `systemctl status <svc>` | `service <svc> status` |
| Enable at boot | `systemctl enable <svc>` | `chkconfig <svc> on` |
| Disable at boot | `systemctl disable <svc>` | `chkconfig <svc> off` |

Replace `<svc>` with the service name, e.g. `nginx`, `sshd`, `crond`.

## Example

```bash
$ sudo systemctl start nginx        # start the web server
$ systemctl status nginx            # check it
● nginx.service - The nginx HTTP server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled)
     Active: active (running) since Thu 2026-06-18 10:15:02 UTC
$ sudo systemctl enable nginx       # make it start on every boot
```

## Key ideas

- **`enable` vs `start` are different things.** `start` runs it *now*. `enable` makes it start *on boot*. You usually want both.
- **`reload` vs `restart`.** After editing a config file (like `nginx.conf`), prefer `systemctl reload nginx` — it applies changes **without downtime**. `restart` fully stops and starts the service (brief outage).
- **Quick check:** `systemctl is-active nginx` returns just `active` or `inactive` — handy in scripts.
- **Find what broke:** `systemctl --failed` lists every service that failed to start.

> 💡 On modern systems the old `service nginx start` still works — it's quietly translated to `systemctl start nginx` behind the scenes.

## 🧪 Practice

1. Check whether the cron service is running:
   ```bash
   systemctl status crond
   ```
2. Stop it, confirm it stopped, then start it again:
   ```bash
   sudo systemctl stop crond
   systemctl is-active crond      # should say: inactive
   sudo systemctl start crond
   systemctl is-active crond      # should say: active
   ```
3. Make sure it starts on boot:
   ```bash
   sudo systemctl enable crond
   systemctl is-enabled crond     # should say: enabled
   ```

**✅ Success check:** `is-active` returns `active` and `is-enabled` returns `enabled`.

---
[← Back to index](../README.md) | [Next: Process Monitoring →](02-process-monitoring.md)
