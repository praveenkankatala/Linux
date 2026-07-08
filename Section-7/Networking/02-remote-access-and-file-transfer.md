# Part 2 — Remote Access & File Transfer

*Topics 17–28.*

**In this part:** SSH vs Telnet, key-based auth, hardening `sshd` without locking yourself out, `ss` for sockets, `ping` vs `curl` for the two layers of "is it up?", FTP (and why SFTP won), plus a locked-down `vsftpd` setup, `scp` for quick copies, and `rsync` for real backups.

[⬅ Part 1](./01-linux-networking-fundamentals.md) · [Index](./README.md)

---

## 17. SSH & Telnet

Both give you a remote shell, but only one is safe.

- **Telnet** sends everything — including your password — in **plain text**; anyone capturing packets reads it. It is obsolete for login and survives only as a quick **port-reachability test**.
- **SSH** (Secure Shell) encrypts the entire session and is the universal standard for remote administration, file copy, tunnelling and automation.

| | Telnet | SSH |
|---|---|---|
| Port | 23 | 22 |
| Encryption | None — plaintext | Strong (AES/ChaCha20) |
| Authentication | Password only | Password **or** key pair |
| Use it for | Testing if a TCP port is open | All real remote access |
| Verdict | **Never** for login | Default everywhere |

### Telnet's one legitimate use

```bash
telnet mail.example.com 25    # is the SMTP port open + responding?
telnet 10.0.1.50 3306          # can I reach MySQL from here?
# Connected => port open. 'Connection refused' => nothing listening.
# (nc -zv host port does the same thing more cleanly)
```

### SSH key-based authentication — how it works

You generate a **key pair**: a private key that never leaves your machine and a public key you place on the server in `~/.ssh/authorized_keys`. At login the server challenges you to prove you hold the private key. No password crosses the wire, and you can't be brute-forced.

```bash
ssh-keygen -t ed25519 -C 'devops@laptop'    # generate a modern key pair
ssh-copy-id user@10.0.1.50                   # install your public key on the server
ssh user@10.0.1.50                            # log in — no password prompt
ssh -i ~/.ssh/aws-key.pem ec2-user@<public-ip>  # EC2: point at the .pem
```

> [!NOTE]
> **ed25519 over RSA:** Prefer `ed25519` keys — they're shorter, faster and as secure as 4096-bit RSA. Use `rsa -b 4096` only when talking to an old system that doesn't support ed25519.

---

## 18. SSH configuration — hands on

The server daemon is `sshd`, configured in **`/etc/ssh/sshd_config`**. These are the hardening changes you make on every internet-facing box, in priority order.

```ini
# /etc/ssh/sshd_config — the settings that matter
PermitRootLogin no          # never allow direct root login
PasswordAuthentication no    # keys only — kills brute-force attacks
Port 2222                    # optional: move off 22 to cut log noise
AllowUsers devops deploy      # whitelist who may log in at all
MaxAuthTries 3                # limit password guesses per connection
ClientAliveInterval 300        # drop idle sessions after 5 min
```

```bash
# Validate, then reload — in that order
sudo sshd -t                     # ALWAYS validate config before reloading
sudo systemctl reload sshd        # apply without dropping existing sessions
```

> [!WARNING]
> **Keep one session open:** Before you set `PasswordAuthentication no` or change the port, **open a second SSH session and confirm you can still log in with your key**. If you lock yourself out on a cloud box with no console, you're rebuilding it. Never reload `sshd` on faith.

### The client-side config — `~/.ssh/config`

Stop typing long SSH commands. Define hosts once:

```ini
# ~/.ssh/config
Host prod-web
    HostName 10.0.1.50
    User ec2-user
    IdentityFile ~/.ssh/aws-prod.pem
    Port 22

# Now simply: ssh prod-web
```

> [!TIP]
> **ProxyJump for bastions:** To reach a private-subnet host through a bastion, add `ProxyJump bastion` under the target host. `ssh prod-db` then transparently hops through the jump box — no manual two-step SSH.

---

## 19. The `ss` command

`ss` (socket statistics) is the modern replacement for `netstat`. It reads kernel socket tables directly, so it's dramatically faster on busy servers. It answers two questions you ask constantly: **"what is my box listening on?"** and **"who is connected to me right now?"**

| Flag | Meaning |
|---|---|
| `-t` | TCP sockets |
| `-u` | UDP sockets |
| `-l` | Listening sockets only |
| `-n` | Numeric — don't resolve ports/hosts to names (faster) |
| `-p` | Show the process (PID/program) owning each socket |
| `-a` | All sockets (listening + established) |
| `-s` | Summary statistics by protocol/state |

> [!TIP]
> **The one command to memorise:** `ss -tulpn` = **t**cp + **u**dp + **l**istening + owning **p**id + **n**umeric. This single line tells you every service listening on the machine and which process owns it. It's the first thing to run when "the service won't start — port already in use".

---

## 20. `ss` command — hands on

```bash
ss -tulpn                        # everything listening + owning PID
ss -tnp state established         # current live TCP connections
ss -tan state syn-sent             # stuck outbound handshakes = firewall/SG
ss -tn dst 10.0.1.50                # connections to a specific host
ss -tn sport = :443                 # who is connected to my HTTPS port
ss -s                                # totals: how many sockets in each state

# 'Address already in use' when starting a service — find the culprit
ss -tulpn | grep :8080
```

> [!NOTE]
> **Reading the output:** Columns are `State`, `Recv-Q`, `Send-Q`, `Local Address:Port`, `Peer Address:Port`, `Process`. A large **Recv-Q** on a listening socket means the app isn't accepting connections fast enough (backlog filling). A large **Send-Q** on an established socket means the peer isn't reading — often a network stall.

---

## 21. `curl` & `ping` — two different layers

Two different layers of "is it working". **`ping`** tests raw reachability at the network layer (ICMP) — can packets get to the host and back. **`curl`** tests the application layer — is the *service* actually answering HTTP. A host can ping fine while its web server is down, so you need both.

| | `ping` | `curl` |
|---|---|---|
| Layer | Network (ICMP) | Application (HTTP/S, FTP...) |
| Answers | Is the host alive & reachable? | Is the service responding correctly? |
| Typical failure | Host down / firewall / no route | App crashed / wrong status code / TLS error |
| In a health check | Coarse liveness | Real end-to-end validation |

---

## 22. `curl` — hands on

```bash
# Measure exactly where time goes in a request
curl -w 'DNS:%{time_namelookup} Connect:%{time_connect} ' \
     -w 'TTFB:%{time_starttransfer} Total:%{time_total}\n' \
     -o /dev/null -s https://example.com

# Just the HTTP status code (perfect for monitoring scripts)
curl -s -o /dev/null -w '%{http_code}\n' https://example.com

# Test against a specific vhost by name
curl -H 'Host: app.example.com' http://10.0.1.50/

# Ignore a self-signed cert in a lab (never in prod)
curl -k https://10.0.1.50/
```

> [!TIP]
> **The `-w` timing trick:** When "the site is slow", `curl -w` splits the total into DNS + TCP connect + TLS + time-to-first-byte. It tells you instantly whether the delay is DNS, the network, or the application — before you open a single dashboard.

---

## 23. `ping` — hands on

```bash
ping -c4 8.8.8.8                 # 4 packets then stop (Linux pings forever otherwise)
ping -c4 -i0.2 10.0.1.1            # 0.2s between packets — faster feedback
ping -c1 -W1 10.0.1.50              # 1 packet, 1s timeout — scriptable up/down probe
ping -s1472 -M do 8.8.8.8            # MTU probe: 1472+28=1500; 'do'=don't fragment
```

### What the output tells you

- **`time=1.2 ms`** — round-trip latency. Consistent low values = healthy. Rising/jumpy = congestion or an overloaded host.
- **Packet loss %** at the end — anything above 0% on a LAN is a red flag (bad cable, saturated link, failing NIC).
- **`Destination Host Unreachable`** — no route or the gateway can't reach it (routing problem).
- **`Request timeout` / 100% loss** — packets leave but nothing returns: host down, or a firewall/security group is dropping ICMP.

> [!WARNING]
> **No reply ≠ host down:** Many firewalls and **all default EC2 security groups block ICMP**. A server can be perfectly healthy and serving traffic on 443 while silently ignoring every ping. Confirm liveness with `curl`/`nc` on the real service port before declaring a host dead.

---

## 24. FTP — File Transfer Protocol

FTP is the legacy protocol for transferring files. Like Telnet, plain FTP is **insecure** — credentials and data travel unencrypted. It's still met on old appliances and vendor drop-boxes, so you should understand it, but for anything you control use **SFTP** (file transfer over SSH) or **SCP** instead.

| Protocol | Port(s) | Encrypted? | When you'd use it |
|---|---|---|---|
| FTP | 21 (control) + data | No | Legacy systems only |
| FTPS | 21 + TLS | Yes | FTP that had TLS bolted on |
| **SFTP** | 22 (over SSH) | Yes | Default choice — already have SSH |
| **SCP** | 22 (over SSH) | Yes | Quick one-off copies |

> [!NOTE]
> **Active vs passive FTP:** FTP's biggest operational pain is its **second data connection**. In **active** mode the server connects back to the client (breaks through NAT/firewalls); in **passive** mode the client opens both connections (firewall-friendly). Behind NAT you almost always want **passive** mode. This dual-connection design is precisely why SFTP/SCP — single port 22 — won.

---

## 25. FTP server — hands on (`vsftpd`)

`vsftpd` (Very Secure FTP Daemon) is the standard FTP server on RHEL. Setup, then lock it down with TLS.

```bash
sudo dnf install -y vsftpd
sudo systemctl enable --now vsftpd
```

```ini
# /etc/vsftpd/vsftpd.conf — key directives
anonymous_enable=NO       # no anonymous access
local_enable=YES           # allow local system users
write_enable=YES            # allow uploads
chroot_local_user=YES        # jail users to their home dir (important)
pasv_min_port=40000            # fix the passive port range...
pasv_max_port=40100             # ...so you can open it in the firewall
```

```bash
# Firewall + test
sudo firewall-cmd --permanent --add-service=ftp
sudo firewall-cmd --permanent --add-port=40000-40100/tcp
sudo firewall-cmd --reload
sudo systemctl restart vsftpd

# Test from the client
ftp 10.0.1.50
```

> [!WARNING]
> **The passive-port trap:** The #1 `vsftpd` support ticket: login works, directory listing hangs forever. Cause: the passive data port range isn't open in the firewall / security group. **Always** pin `pasv_min`/`max_port` and open that exact range.

---

## 26. The `scp` command

`scp` (secure copy) copies files between hosts over SSH. Same auth as SSH — reuse your keys — encrypted end to end. Perfect for quick, one-off transfers where you don't need the smartness of `rsync`.

```bash
scp file.txt user@10.0.1.50:/tmp/                              # local -> remote
scp user@10.0.1.50:/var/log/app.log ./                          # remote -> local
scp -r ./mydir user@10.0.1.50:/opt/                               # -r for whole directories
scp -i ~/.ssh/key.pem app.jar ec2-user@<ip>:/opt/app/               # with a specific key
scp -P 2222 file.txt user@host:/tmp/                                 # note: CAPITAL -P for port
```

> [!NOTE]
> **`-p` vs `-P`:** In `scp` the **capital `-P`** sets the port (lowercase `-p` preserves timestamps) — the reverse of `ssh`, where port is lowercase `-p`. This inconsistency has cost every admin at least one confused minute.

---

## 27. `scp` command — hands on

```bash
# Copy between two REMOTE hosts (from your laptop, via -3)
scp -3 user@host-a:/data/f.gz user@host-b:/data/

# Bandwidth-limit a large copy so you don't saturate the link (kbit/s)
scp -l 8192 big.iso user@10.0.1.50:/tmp/

# Preserve mode/timestamps and show progress-ish verbosity
scp -pv config.tar.gz user@10.0.1.50:/etc/backup/
```

> [!TIP]
> **When to stop using `scp`:** `scp` re-copies the **entire** file every time and has no resume. For anything repeated, large, or interruptible, switch to `rsync` (next topic) — it only sends the differences.

---

## 28. The `rsync` command

`rsync` is the professional's file-sync tool. Its superpower is the **delta-transfer algorithm**: it compares source and destination and sends **only the changed blocks**. A nightly 50 GB backup where 200 MB changed transfers ~200 MB, not 50 GB. It also resumes, preserves permissions, and can mirror exactly. This is what real backup and deployment jobs use.

### The flags you'll actually use

| Flag | Effect | Why it matters |
|---|---|---|
| `-a` | Archive: recursive + preserve perms, times, symlinks, owner | The default you almost always want |
| `-v` | Verbose — list what's transferred | See what changed |
| `-z` | Compress in transit | Faster over slow/WAN links |
| `-P` | `--partial --progress` — resume + progress bar | Survives interruptions |
| `--delete` | Remove files on dest not in source | TRUE mirror — dangerous, see warning |
| `-n` | `--dry-run` — show what *would* happen | Always run this first |
| `-e ssh` | Transport over SSH | Encrypted; add `-i key.pem` |

```bash
# The everyday backup command
rsync -avzP /var/www/ user@10.0.1.50:/backup/www/

# Exact mirror (dest becomes identical to source) — test with -n FIRST
rsync -avn --delete /src/ /dest/     # dry-run: review the deletions
rsync -av --delete /src/ /dest/       # then run for real

# Over SSH with a key, excluding noise
rsync -avzP -e 'ssh -i ~/.ssh/key.pem' \
  --exclude '*.tmp' --exclude '.git' \
  ./app/ ec2-user@<ip>:/opt/app/
```

> [!WARNING]
> **The trailing-slash rule — memorise it:** `rsync -a /src/ /dest/` copies the **contents** of `src` into `dest`. `rsync -a /src /dest/` copies the **folder itself**, creating `/dest/src/`. One slash changes everything. Combined with `--delete`, the wrong slash can wipe the wrong directory — which is exactly why you always `-n` first.

> [!TIP]
> **Why rsync beats scp for automation:** Idempotent (re-running only syncs new changes), resumable (`-P`), bandwidth-friendly (`-z`, `--bwlimit`), and mirror-capable (`--delete`). Every serious backup/deploy script uses `rsync`, not `scp`.

---

[⬅ Part 1](./01-linux-networking-fundamentals.md) · [Index](./README.md) · [Next: Part 3 — Package & Patch Management ➡](./03-package-and-patch-management.md)
