# 7. User & Group Management

[← Back to index](../README.md)

This topic has three parts: **auditing** who is on the system, **creating/modifying** accounts, and enforcing **password policy**.

---

## Part A — Auditing & discovery: who is on the system?

Before you change a running system, you need to know who's connected, what they're doing, and what privileges *your own* session has. Five commands cover this:

| Command | Function | Reads from |
|---------|----------|------------|
| `who` | Logged-in users, terminal, login time | `/var/run/utmp` |
| `w` | Who's logged in + uptime, load, live activity | `utmp` + `/proc` |
| `last` | Historical login/logout/reboot record | `/var/log/wtmp` |
| `id` | Your UID, primary group, and group memberships | `/etc/passwd`, `/etc/group` |
| `whoami` | The current username (one word) | process context |

### `who` — the active user roster

Answers one question: **who is logged in right now?** Fastest way to check if teammates are on a machine before you restart services.

**Under the hood:** Linux keeps a live binary record at `/var/run/utmp`. Every new session (SSH, console, terminal window) adds a record; logging out removes it. `who` just reads and formats that file.

```bash
$ who
ec2-user   pts/0   2026-06-18 10:15 (203.0.113.45)
adminuser  pts/1   2026-06-18 11:20 (198.51.100.12)
```

- **Column 1** — the account name
- **Column 2** — the terminal: `pts` = pseudo-terminal (SSH / terminal window); `tty` = a local physical console
- **Columns 3–4** — when the session started
- **Column 5** — the remote IP the user connected from

### `last` — the historical login audit trail

Where `who` shows *now*, `last` shows *history*: every successful login, logout, and reboot since the log was created or rotated. A cornerstone tool for security investigations.

**Under the hood:** login events are written to a **binary** store at `/var/log/wtmp` — harder to tamper with than a text file, and decoded by the `last` command.

```bash
$ last
ec2-user   pts/0        203.0.113.45    Thu Jun 18 10:15   still logged in
adminuser  pts/1        198.51.100.12   Thu Jun 18 08:30 - 09:15  (00:45)
reboot     system boot  6.1.0-21-amd64  Wed Jun 17 06:00   still running
```

- **still logged in** — session is active now
- **08:30 - 09:15 (00:45)** — login time, logout time, total duration (45 min)
- **reboot** entries — every system boot, useful for tracking uptime

### `w` — the real-time dashboard

`w` is like running `uptime`, `who`, and `ps` at once: it shows who's connected **and what they're running right now**.

**Under the hood:** reads `/var/run/utmp` for sessions, then queries `/proc` to map each terminal to its active command.

```bash
$ w
 11:32:04 up 1 day,  5:32,  2 users,  load average: 0.05, 0.02, 0.01
USER     TTY    FROM           LOGIN@  IDLE   JCPU   PCPU WHAT
ec2-user pts/0  203.0.113.45   10:15   1.00s  0.11s  0.02s w
admin    pts/1  198.51.100.12  11:20   8:12m  0.45s  0.45s nano index.html
```

- **Header** — uptime and load averages (same as `uptime`)
- **IDLE** — time since the user last typed (admin idle 8m 12s)
- **JCPU** — total CPU time of all processes on that terminal
- **PCPU** — CPU time of the current task
- **WHAT** — the command running right now (admin is editing a file in `nano`)

### `finger` — the legacy profile card

Prints a summary of a user: real name, home directory, shell, login status, mail.

```bash
$ finger ec2-user
Login: ec2-user           Name: Cloud Application User
Directory: /home/ec2-user Shell: /bin/bash
On since Thu Jun 18 10:15 (IST) on pts/0 from 203.0.113.45
No mail.
No Plan.
```

> ⚠️ **Security note:** `finger` is **deprecated and not installed by default** on modern systems. It leaks real names, valid usernames, and home-directory layouts that attackers can use to plan brute-force SSH attacks. Expect "command not found" unless someone installs it (`sudo dnf install finger`).

### `id` — identity & privilege check

Prints the security context your shell runs under. **Run this the moment you hit a "Permission denied" error** to confirm you're actually in the required group.

**Under the hood:** calls the kernel functions `getuid()`, `getgid()`, `getgroups()`, which look up `/etc/passwd` and `/etc/group`.

```bash
$ id
uid=1001(developer) gid=1001(developer) groups=1001(developer),10(wheel),990(docker)
```

- **uid=1001** — your user ID (the kernel tracks the number, not the name). **root is always uid=0.**
- **gid=1001** — your primary group; new files you create get this group by default
- **groups=…** — supplementary groups. Here `wheel` grants `sudo` rights, and `docker` lets you manage containers without root.

> 💡 Lost track of who you are after several `sudo`/`su` jumps? `whoami` prints just the active username.

### Quick summary

| Command | Best used for |
|---------|---------------|
| `who` | Checking who's on the server before a reboot |
| `last` | Investigating incidents / tracking reboot history |
| `w` | Watching resource use and what users are running |
| `finger` | Legacy full-profile lookup (rarely used today) |
| `id` | Troubleshooting permissions and group membership |

---

## Part B — Creating, modifying & deleting accounts

| Command | Effect |
|---------|--------|
| `sudo useradd -m <user>` | Create a user **and** a home directory under `/home` |
| `sudo passwd <user>` | Set/change that user's password |
| `sudo groupadd <group>` | Create a new group for shared permissions |
| `sudo usermod -aG wheel <user>` | **Append** a user to a supplementary group |
| `sudo userdel <user>` | Delete the account, **keep** the home directory |
| `sudo userdel -r <user>` | Delete the account **and** remove the home directory + mail |

> ⚠️ **Always use `-a` with `usermod -G`.** Running `usermod -G wheel user` *without* `-a` replaces the user's entire group list — removing them from every other group they belonged to. `-aG` = **append** to groups.

---

## Part C — Password aging (`chage`)

Password aging forces periodic password changes, limiting how long a stolen password stays useful. Managed with `chage` (**cha**nge a**ge**).

```bash
sudo chage [flag] [value] username
```

| Flag | Name | Meaning |
|------|------|---------|
| `-m 7` | Minimum age | User must keep a password **at least 7 days** (blocks instantly cycling back to the old one) |
| `-M 90` | Maximum age | Password valid for **90 days**, then a change is forced |
| `-W 6` | Warning period | Warn the user **6 days** before expiry |
| `-E 2026-12-31` | Account expiry | Hard-lock the account on a date (ideal for contractors) |
| `-l user` | List | Show current aging status |

> ⚠️ An expiry date **in the past** locks the account immediately. Setting `-E 2025-12-31` today would block login at once.

### Where the data lives

| File | Stores |
|------|--------|
| `/etc/passwd` | Username, UID, primary GID, home directory, login shell |
| `/etc/shadow` | Hashed password **and** all the aging values above |
| `/etc/group` | Group names and their members |
| `/etc/login.defs` | Default aging limits applied to **new** users |

## 🧪 Practice

1. Create a user and set a password:
   ```bash
   sudo useradd -m testuser
   sudo passwd testuser
   ```
2. View and then set the password policy:
   ```bash
   sudo chage -l testuser
   sudo chage -M 90 -m 7 -W 6 testuser
   ```
3. Grant admin rights and verify:
   ```bash
   sudo usermod -aG wheel testuser
   id testuser              # should list the wheel group
   ```
4. Clean up:
   ```bash
   sudo userdel -r testuser
   ```

**✅ Success check:** `chage -l testuser` shows max 90 / min 7 / warn 6, and `id testuser` lists `wheel`.

---
[← Previous: Package Management](06-package-management.md) | [Next: System Information →](08-system-information.md)
