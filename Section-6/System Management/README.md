# 🐧 Linux System Administration — Learning & Practice Guide

A beginner-friendly, hands-on guide to managing a real Linux system. Every topic is explained in plain language, with the **why**, the **commands**, real **terminal output**, and a **practice section** you can run yourself.

> Tested on **Amazon Linux / RHEL** family (systemd). Where **Debian / Ubuntu** differs, it's called out. Remember: Linux is **case-sensitive** — every command is lowercase.

---

## 📚 Topics

| # | Topic | What you'll learn |
|---|-------|-------------------|
| 1 | [System Services](topics/01-system-services.md) | Start, stop, enable, and troubleshoot services with `systemctl` |
| 2 | [Process Monitoring](topics/02-process-monitoring.md) | See what's running with `ps` and `top` |
| 3 | [Signals & Killing Processes](topics/03-signals-and-kill.md) | Stop frozen processes safely with `kill` |
| 4 | [Task Scheduling](topics/04-task-scheduling.md) | Automate jobs with `cron` and `at` |
| 5 | [Text Processing with sed](topics/05-text-processing-sed.md) | Find, replace, and delete text in files |
| 6 | [Package Management](topics/06-package-management.md) | Install software with `apt` and `dnf`/`yum` |
| 7 | [User & Group Management](topics/07-user-group-management.md) | Create users, set password policy, audit logins |
| 8 | [System Information](topics/08-system-information.md) | Inspect CPU, RAM, disk, and hardware |
| 9 | [System Utilities](topics/09-system-utilities.md) | Everyday helpers: date, cal, uptime, bc, and more |
| 10 | [Environment Variables](topics/10-environment-variables.md) | Understand `$PATH`, `$HOME`, and `export` |
| 11 | [Terminal & Session Management](topics/11-terminal-session-management.md) | Persistent sessions with `screen` and `script` |
| 12 | [Logging & Diagnostics](topics/12-logging-and-diagnostics.md) | Read logs with `journalctl` and generate an SOS report |
| 13 | [System Maintenance & Recovery](topics/13-system-maintenance-recovery.md) | Reboot, runlevels, and resetting a lost root password |
| 14 | [System Monitoring Toolkit](topics/14-system-monitoring-toolkit.md) | Modern `ip`/`ss` networking and I/O monitoring |

## 🧪 Practice & Review

- [Hands-On Labs](topics/15-hands-on-labs.md) — 6 guided labs with verification steps
- [Knowledge Check](topics/16-knowledge-check.md) — quiz yourself (answers included)
- [Cheat Sheet](CHEATSHEET.md) — one-line lookups for every command

---

## 🚀 How to use this guide

1. **Read a topic** top to bottom — each builds from concept → command → example.
2. **Spin up a throwaway VM** (an AWS EC2 free-tier instance, a local VM, or a container) so you can break things safely.
3. **Run the practice section** at the end of each topic.
4. **Tip:** start a session recording first so you can review what you did:
   ```bash
   script my_session.log   # everything you type/see is saved
   # ... practice ...
   exit                    # stops recording
   ```

## 🗺️ Prerequisites

You should be comfortable with basic navigation (`cd`, `ls`, `pwd`), viewing files (`cat`, `less`), and connecting to a server over SSH. Everything else is explained here.

---

*Built for learners. Contributions and corrections welcome — open an issue or PR.*
