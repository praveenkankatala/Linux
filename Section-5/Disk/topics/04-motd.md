# 04 · MOTD — Message of the Day & Login Banners

[⬅ Previous: systemd-analyze](03-systemd-analyze.md) · [Back to index](../README.md) · [Next: Linux Storage ➡](05-linux-storage.md)

---

## 🎯 What is the MOTD?

The **MOTD (Message of the Day)** is the **banner shown after a user logs in**.

> 📋 **Analogy:** It's the notice board on the office door. As soon as someone walks in (logs in), they see: "This is PRODUCTION — don't touch without a change ticket," plus useful facts like uptime and disk usage.

**Why admins use it:**
- **Legal / security warnings** ("Authorized use only, activity is logged").
- **Environment labels** ("⚠️ PRODUCTION" vs "🧪 DEV") so nobody runs the wrong command on the wrong box.
- **Quick facts** — hostname, uptime, disk usage at a glance.

---

## 📁 The files involved

| File | Shown | When |
|------|-------|------|
| `/etc/motd` | After login | Every interactive login |
| `/etc/motd.d/*` | After login (merged) | RHEL 8+ / newer — drop-in fragments |
| `/etc/issue` | **Before** login | At the local console prompt |
| `/etc/issue.net` | **Before** login | At the remote (SSH) prompt |

> [!NOTE]
> **`issue` vs `motd` — a key distinction:**
> `/etc/issue` and `/etc/issue.net` appear **before** you type your password (at the prompt). `/etc/motd` appears **after** you've logged in. Put legal "unauthorized access prohibited" warnings in `issue.net` so they're seen *before* login.

---

## 🧪 Hands-on

### 1. Static MOTD (the simplest)

```bash
sudo tee /etc/motd >/dev/null <<'EOF'
***********************************************************
  PRODUCTION SERVER — authorized use only.
  All activity is logged and monitored.
  Raise a change ticket before modifying anything.
***********************************************************
EOF
```
Log out and back in — the banner appears after authentication.

### 2. Drop-in fragments (RHEL 8+ / Amazon Linux 2023)

Instead of one big file, add modular pieces in `/etc/motd.d/`:

```bash
sudo mkdir -p /etc/motd.d
echo 'Env: PROD | Owner: platform-team | Region: ap-south-1' \
  | sudo tee /etc/motd.d/10-banner
# All files in /etc/motd.d are concatenated at login.
```

### 3. Dynamic MOTD — show live host info

**Debian / Ubuntu** runs executable scripts in `/etc/update-motd.d/` at login:

```bash
sudo tee /etc/update-motd.d/99-hostinfo >/dev/null <<'EOF'
#!/bin/bash
echo "Host  : $(hostname -f)"
echo "Uptime: $(uptime -p)"
echo "Load  : $(cut -d' ' -f1-3 /proc/loadavg)"
echo "Disk  : $(df -h / | awk 'NR==2{print $5" used of "$2}')"
EOF
sudo chmod +x /etc/update-motd.d/99-hostinfo
```

**RHEL family** doesn't run update-motd by default. A common pattern is a small script that a **cron job or systemd timer** runs to regenerate a file under `/etc/motd.d/`:

```bash
cat >/usr/local/bin/gen-motd.sh <<'EOF'
#!/bin/bash
{
  echo "===== $(hostname -s) ====="
  echo "Kernel: $(uname -r)"
  echo "Uptime: $(uptime -p)"
} > /etc/motd.d/20-dynamic
EOF
chmod +x /usr/local/bin/gen-motd.sh
# then schedule it (cron or a systemd .timer)
```

### 4. Pre-login banner (console + SSH)

```bash
# The text shown BEFORE login
echo 'Authorized access only. Disconnect NOW if not authorized.' \
  | sudo tee /etc/issue.net

# Make SSH actually display it:
sudo sed -i 's|^#\?Banner.*|Banner /etc/issue.net|' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

---

## 💡 Good to know: how the MOTD is printed

The MOTD is displayed by a **PAM module** called `pam_motd`, configured in `/etc/pam.d/login` and `/etc/pam.d/sshd`. If your MOTD ever "doesn't show up," check that `pam_motd` is present and enabled in those PAM files.

---

## ✅ Key takeaways

- `/etc/motd` = banner **after** login; `/etc/issue`/`issue.net` = **before** login.
- RHEL 8+ uses drop-in fragments in `/etc/motd.d/`.
- Dynamic MOTD: scripts in `/etc/update-motd.d/` (Debian) or a timer/cron writing to `/etc/motd.d/` (RHEL).
- SSH banners are enabled via the `Banner` directive in `sshd_config`.

## 💬 Interview questions

1. *Difference between `/etc/motd` and `/etc/issue`?* → motd is post-login; issue is pre-login.
2. *How do you show a legal warning before SSH login?* → set `Banner /etc/issue.net` in `sshd_config`.

---

[⬅ Previous: systemd-analyze](03-systemd-analyze.md) · [Back to index](../README.md) · [Next: Linux Storage ➡](05-linux-storage.md)
