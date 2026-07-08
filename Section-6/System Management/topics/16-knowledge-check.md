# 16. Knowledge Check

[← Back to index](../README.md)

Answer from memory first, then verify on a test box. Answers are at the bottom — no peeking!

## Questions

1. Which command makes a service start automatically at boot?
2. What's the difference between `kill -15` and `kill -9`?
3. Write a cron line that runs `/opt/backup.sh` every day at 2 AM.
4. How do you replace every occurrence of "foo" with "bar" in `file.txt` **and save it**?
5. On Amazon Linux, which command installs the `tree` package?
6. Why must you use `-a` with `usermod -G`?
7. Which file stores password hashes and aging data?
8. Which modern command replaces `netstat -tulpn`?
9. What does `journalctl -u sshd --since "1 hour ago"` show?
10. Inside `screen`, how do you leave a job running in the background?

---

## Answers

<details>
<summary>Click to reveal answers</summary>

1. `systemctl enable <service>`
2. `-15` (SIGTERM) asks the process to exit **cleanly**; `-9` (SIGKILL) **forces** an immediate kill with no chance to save or clean up.
3. `0 2 * * * /opt/backup.sh`
4. `sed -i 's/foo/bar/g' file.txt`
5. `sudo dnf install tree` (Amazon Linux is RPM/Red Hat based — `apt` won't work).
6. Without `-a`, `-G` **replaces** the user's entire group list, removing them from all other groups. `-aG` appends instead.
7. `/etc/shadow`
8. `ss -tulpn`
9. SSH server (sshd) log entries from the last 60 minutes.
10. Press **Ctrl+A** then **D** to detach (leaves it running). Typing `exit` would kill it.

</details>

---
[← Previous: Hands-On Labs](15-hands-on-labs.md) | [Back to index →](../README.md)
