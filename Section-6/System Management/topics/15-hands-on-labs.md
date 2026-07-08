# 15. Hands-On Labs

[← Back to index](../README.md)

Six guided labs that tie the topics together. Run them on a **throwaway VM or EC2 instance** so you can break things safely.

> 💡 Start by recording your session: `script section5_labs.log` — then `exit` when done to save the log.

---

## Lab 1 — Service control

1. Check whether cron is running: `systemctl status crond`
2. Stop it, confirm, then start it again and confirm.
3. Make sure it starts on boot: `sudo systemctl enable crond`

**✅ Check:** `systemctl is-active crond` → `active`, and `systemctl is-enabled crond` → `enabled`.

---

## Lab 2 — Processes & signals

1. Start a dummy background process: `sleep 600 &`
2. Find its PID: `ps -ef | grep [s]leep`
3. Watch it in `top`, then quit with `q`.
4. Terminate it gracefully: `kill -15 <PID>`

**✅ Check:** `ps -ef | grep [s]leep` no longer lists the process.

---

## Lab 3 — Scheduling

1. Add a cron job that appends the date every minute:
   ```
   * * * * * date >> ~/cron_test.log
   ```
2. List it with `crontab -l`, wait two minutes, then `cat ~/cron_test.log`.
3. Schedule a one-off job:
   ```bash
   echo 'touch ~/at_done' | at now + 1 minute
   atq
   ```

**✅ Check:** `cron_test.log` gains timestamped lines and `~/at_done` appears after a minute. Remove the cron line with `crontab -e` when done.

---

## Lab 4 — sed text processing

1. Create a test file:
   ```bash
   cat > sample.txt <<EOF
   Hello Amazon Linux
   sed is very powerful
   We are learning sed command
   sed helps in text processing
   EOF
   ```
2. Preview a replacement: `sed 's/sed/AWK/g' sample.txt`
3. Delete lines 2–3: `sed '2,3d' sample.txt`
4. Save a change in place: `sed -i 's/Amazon/AWS/g' sample.txt`

**✅ Check:** `cat sample.txt` shows "Hello AWS Linux" on the first line.

---

## Lab 5 — User & password policy

1. Create a user and set a password:
   ```bash
   sudo useradd -m testuser
   sudo passwd testuser
   ```
2. View aging: `sudo chage -l testuser`
3. Apply a policy: `sudo chage -M 90 -m 7 -W 6 testuser`
4. Add to the wheel group and verify: `sudo usermod -aG wheel testuser` then `id testuser`

**✅ Check:** `chage -l` shows max 90 / min 7 / warn 6, and `id testuser` lists `wheel`. Clean up: `sudo userdel -r testuser`.

---

## Lab 6 — Logs & diagnostics

1. Tail the secure log live: `sudo tail -f /var/log/secure` (open a second SSH session to generate entries, then Ctrl+C).
2. Query SSH logs for the last hour: `sudo journalctl -u sshd --since "1 hour ago"`
3. Show only errors since today: `sudo journalctl --since today -p err`

**✅ Check:** You can read authentication events and filter them by service and time.

---
[← Previous: System Monitoring Toolkit](14-system-monitoring-toolkit.md) | [Next: Knowledge Check →](16-knowledge-check.md)
