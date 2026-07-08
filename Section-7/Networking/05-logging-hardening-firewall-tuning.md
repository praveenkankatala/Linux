# Part 5 — Logging, Hardening, Firewall & Tuning

*Topics 57–66.*

**In this part:** central logging with rsyslog (why, and a working collector + client setup), OS hardening — the high-impact checklist and how to apply it without locking yourself out, `traceroute` done right (`-T -p 443`), system tuning via `tuned`/`sysctl`/`ulimit`, and firewall management with `firewalld` — plus why the EC2 Security Group is usually the real gate.

[⬅ Part 4](./04-network-services-dns-ntp-web-proxy.md) · [Index](./README.md)

---

## 57. Central logger

On a single box, logs live locally in `/var/log` and via `journalctl`. Across a fleet that doesn't scale: if a host is compromised or crashes, its logs are lost or tampered with, and correlating an incident across ten servers by SSHing into each is hopeless. **Central logging** ships every host's logs to one collector, giving you a single searchable place, tamper-resistance, and cross-host correlation. `rsyslog` is the built-in mechanism; in bigger shops it feeds an ELK/Loki/CloudWatch pipeline.

### Why centralise

- **Survivability** — logs are safe on the collector even if the source host dies or is wiped.
- **Correlation** — trace a request across load balancer → app → database in one place, by timestamp (which is why NTP from Part 4 matters).
- **Security & compliance** — an attacker who clears local logs can't reach the ones already shipped off-box; auditors want immutable central retention.
- **Operational sanity** — one query instead of ten SSH sessions.

---

## 58. `rsyslog` — hands on

`rsyslog` is the default syslog daemon on RHEL. It reads messages by **facility** (`auth`, `cron`, `kern`, `mail`, `local0`–`7`...) and **severity** (`emerg`→`debug`) and routes them to files — or over the network to a central server.

```conf
# /etc/rsyslog.conf — 'facility.severity destination'
authpriv.*    /var/log/secure          # all auth logs -> secure
mail.*        -/var/log/maillog          # '-' = async write (faster)
*.emerg       :omusrmsg:*                 # emergencies -> all logged-in users
cron.*        /var/log/cron
```

### Set up a central collector + clients

```conf
# ON THE CENTRAL SERVER — /etc/rsyslog.d/server.conf — enable network reception
module(load="imtcp")
input(type="imtcp" port="514")

# Store each host's logs in its own directory:
template(name="PerHost" type="string"
    string="/var/log/central/%HOSTNAME%/%PROGRAMNAME%.log")
*.* ?PerHost
```

```bash
systemctl restart rsyslog
firewall-cmd --permanent --add-port=514/tcp && firewall-cmd --reload
```

```conf
# ON EACH CLIENT — /etc/rsyslog.d/forward.conf — forward everything to the collector
*.* @@10.0.1.99:514    # @@ = TCP (reliable); @ = UDP (lossy but lighter)
```

```bash
systemctl restart rsyslog
logger 'test message from client'    # inject a test log
# Then on the server: tail -f /var/log/central/<client>/root.log
```

> [!TIP]
> **`@@` vs `@` and logrotate:** `@@host` = **TCP** (won't lose messages), `@host` = **UDP** (lighter, can drop under load). Use TCP for anything you care about. And remember `logrotate` (`/etc/logrotate.d/`) — without rotation your central `/var/log` fills the disk and logging stops silently.

---

## 59. OS hardening

Hardening is reducing a system's **attack surface** — turning off what you don't need and locking down what you do. The principle is **least privilege / least exposure**: every open port, running service, and permissive setting is something an attacker can use. Frameworks like **CIS Benchmarks** and **DISA STIGs** codify this into checklists; below are the highest-value controls.

### The high-impact hardening checklist

| Area | Control | Why |
|---|---|---|
| **SSH** | Key-only auth, no root login, limit users | Kills the #1 remote attack vector |
| **Services** | Disable/remove anything unused | Fewer services = smaller attack surface |
| **Firewall** | Default-deny inbound, allow only needed ports | Nothing reachable that shouldn't be |
| **Users** | No shared accounts; sudo not root; strong password policy | Accountability + limit blast radius |
| **Updates** | Automated security patching | Close known CVEs fast (Part 3) |
| **SELinux** | Keep it enforcing | Contains a compromised service |
| **Auditing** | `auditd` + central logs | Detect and investigate |
| **Filesystem** | `noexec`/`nosuid` on `/tmp`, correct perms | Block a common exploitation path |

---

## 60. OS hardening — hands on

```bash
# Step 1 — inventory the attack surface: audit what's exposed and running
ss -tulpn                                       # every listening port — close what you don't need
systemctl list-unit-files --state=enabled          # services starting at boot
systemctl disable --now cups avahi-daemon            # example: kill printing/mDNS if unused
```

```conf
# Step 2 — enforce a password policy — /etc/security/pwquality.conf
minlen = 14
minclass = 3        # need chars from >=3 classes (upper/lower/digit/special)
maxrepeat = 3
```

```bash
# Step 3 — lock accounts after failed logins (pam_faillock)
faillock --user someuser    # view lockouts; configure via /etc/security/faillock.conf

# Step 4 — keep SELinux enforcing and confirm
getenforce                    # must say 'Enforcing'
setenforce 1                    # if it says Permissive

# Step 5 — turn on auditd for a tamper log of security events
systemctl enable --now auditd
ausearch -m avc -ts today       # what SELinux denied today

# Step 6 — restrict /tmp (common exploit staging ground)
# mount /tmp with nosuid,noexec,nodev in /etc/fstab
```

> [!WARNING]
> **Change one thing at a time — with a way back:** Hardening locks things down; a mistake locks *you* out. Always keep a **second session open** and, on cloud, know how to reach the **serial/console** before tightening SSH or the firewall. Apply changes incrementally and test after each — never paste a whole STIG in one go on a live box.

> [!TIP]
> **Automate it, don't click it:** For a fleet, encode hardening as **Ansible roles** (e.g. the CIS/ansible-lockdown roles) or bake it into a **golden AMI**. That gives you repeatability, drift detection and an audit trail — far better than hand-hardening each host. This is exactly where infra-as-code pays off.

---

## 61. Traceroute — the concept

`ping` tells you *whether* a host is reachable; **`traceroute`** tells you *the path* packets take to get there and *where* they stop. It works by sending packets with increasing **TTL** values — TTL 1 expires at the first router (which reports back), TTL 2 at the second, and so on — mapping each hop. When connectivity fails "somewhere in the middle", `traceroute` finds the somewhere.

### When you reach for it

- A remote host is unreachable and you need to know **which hop** drops the traffic (your gateway? the ISP? the destination's edge?).
- Latency is high and you want to see **where** it spikes along the path.
- Asymmetric routing or a misconfigured router is suspected.

---

## 62. `traceroute` — hands on

```bash
traceroute google.com                 # default UDP probes, hop by hop
traceroute -n google.com                # numeric — skip reverse DNS, much faster
traceroute -T -p 443 host                 # TCP to port 443 — gets through firewalls
                                            #   that block UDP/ICMP (the realistic test)
tracepath google.com                          # no-root alternative, also discovers MTU
mtr -rw google.com                              # traceroute + ping combined, best of both
```

### Reading the output

- Each numbered line is a **hop** (router). Three time values = three probes to that hop.
- **`* * *`** at a hop — that router didn't reply. Often harmless (many routers deprioritise/block probe replies); only a problem if every hop from there on is also `*`.
- **Latency that jumps and stays high** from a certain hop onward — congestion or a long link (e.g. an intercontinental leg) begins there.
- **Stops before the destination** with all `*` after a point — that's likely where traffic is actually being dropped (a down link or a firewall).

> [!NOTE]
> **Use `-T -p 443`, not plain `traceroute`:** Default `traceroute` uses UDP/ICMP, which many firewalls block — giving misleading `* * *` even when the real service works fine. `traceroute -T -p 443` traces using **TCP to the actual service port**, so you test the path your application traffic really takes. This distinction separates a real network fault from a cosmetic one.

---

## 63. Tuning the system

Tuning is adjusting kernel and resource parameters so a server performs well under its specific load. Defaults are conservative, general-purpose compromises; a busy web server, database or NAT gateway each want different settings. The main levers are **`sysctl`** (kernel parameters), **`ulimit`** (per-process resource limits), and **`tuned`** (RHEL's profile-based auto-tuner).

### The three tuning levers

| Lever | Controls | Set via |
|---|---|---|
| **`sysctl`** | Kernel params: network buffers, connection limits, VM behaviour | `/etc/sysctl.d/*.conf` |
| **`ulimit`** | Per-process limits: open files, processes, memory | `/etc/security/limits.d/*.conf` |
| **`tuned`** | Whole pre-built profiles for a workload type | `tuned-adm profile <name>` |

---

## 64. Tuning system — hands on

```bash
# tuned — the easy, supported starting point on RHEL
tuned-adm list                                  # available profiles
tuned-adm active                                  # current profile
tuned-adm profile throughput-performance             # for busy servers/DBs
tuned-adm profile virtual-guest                        # sensible default for VMs/EC2
```

```conf
# sysctl — targeted kernel tuning for a network-heavy host
# /etc/sysctl.d/99-net-tuning.conf
net.core.somaxconn = 4096                    # bigger listen backlog (high-conn servers)
net.ipv4.tcp_max_syn_backlog = 8192            # survive SYN bursts
net.ipv4.ip_local_port_range = 1024 65000        # more ephemeral ports for a proxy/LB
net.ipv4.tcp_fin_timeout = 15                      # reclaim closed connections faster
vm.swappiness = 10                                   # prefer RAM over swap (DB hosts)
net.ipv4.ip_forward = 1                                # enable routing (only on gateways/NAT)
```

```bash
sudo sysctl --system              # load all *.conf and apply
sysctl net.core.somaxconn           # verify a value took effect
```

```conf
# ulimit — raise open-file limits for a high-connection service
# /etc/security/limits.d/nginx.conf
nginx soft nofile 65535
nginx hard nofile 65535
```

```bash
ulimit -n    # current shell's open-file limit
```

> [!WARNING]
> **Measure, change one thing, re-measure:** Tuning without a baseline is guessing, and copy-pasted "magic" sysctl values can make things worse. Benchmark first, change **one** parameter, benchmark again. And always persist settings in `/etc/sysctl.d/` — a live `sysctl -w` value is lost on reboot.

> [!TIP]
> **The "too many open files" classic:** A high-traffic Nginx/Java/DB service dies with `Too many open files` because every connection is a file descriptor and the default limit (often 1024) is too low. The fix is **both** the ulimit (`nofile`) and, for systemd services, `LimitNOFILE=` in the unit — the shell limit alone doesn't cover a daemon started by systemd.

---

## 65. Firewall management

The Linux firewall is the kernel's **netfilter** framework. You don't touch it directly — you use a front-end. On RHEL that's **`firewalld`** (zone-based, dynamic, the default); underneath sits **`nftables`** (modern) which replaced **`iptables`** (legacy but still everywhere in scripts and Docker). On Ubuntu the friendly front-end is **`ufw`**. The golden rule for any firewall: **default-deny inbound, explicitly allow only what you need.**

### The firewall landscape

| Tool | Layer | Where you meet it |
|---|---|---|
| **netfilter** | Kernel framework (the engine) | Never directly |
| **nftables** | Modern rule syntax | RHEL 8+/Ubuntu under the hood |
| **iptables** | Legacy rule syntax | Old scripts, Docker, K8s, tons of docs |
| **firewalld** | Zone-based front-end (default RHEL) | What you actually run on RHEL |
| **ufw** | Simple front-end (default Ubuntu) | What you run on Ubuntu |
| **Security Groups** | AWS instance-level firewall | EC2 — often the *real* gate |

> [!NOTE]
> **firewalld zones:** `firewalld` groups rules into **zones** (`public`, `internal`, `trusted`, `dmz`...), each a trust level with its own allowed services. An interface is assigned to a zone. For a simple server you work almost entirely in the **public** zone — but the model scales to multi-NIC hosts with different trust per interface.

---

## 66. Firewall management — hands on (`firewalld`)

```bash
# Inspect current state first
systemctl enable --now firewalld
firewall-cmd --state                    # running?
firewall-cmd --get-active-zones           # which zone each NIC is in
firewall-cmd --list-all                     # everything allowed in the default zone
```

```bash
# The permanent + reload pattern — --permanent survives reboot, then --reload applies
firewall-cmd --permanent --add-service=https      # by named service
firewall-cmd --permanent --add-port=8080/tcp         # by explicit port
firewall-cmd --reload                                  # apply permanent rules

# Remove access
firewall-cmd --permanent --remove-service=cockpit
firewall-cmd --reload

# Restrict a port to ONE source subnet (rich rule)
firewall-cmd --permanent --add-rich-rule=\
  'rule family=ipv4 source address=10.0.1.0/24 port port=3306 protocol=tcp accept'
firewall-cmd --reload
```

> [!WARNING]
> **`--permanent` or it's gone at reboot:** A `firewall-cmd` rule **without** `--permanent` is runtime-only and vanishes on reload/reboot. A rule **with** `--permanent` only takes effect after `--reload`. The safe habit: add with `--permanent`, then `--reload`. To test something risky live, add it runtime-only first — if you lock yourself out, a reboot restores access.

> [!TIP]
> **On EC2, check the Security Group first:** Two firewalls stack on EC2: the **VPC Security Group** (outside the instance) and the **guest firewall** (`firewalld`/`ufw` inside). A blocked port is far more often the Security Group. When a port "won't open", verify the SG allows it **before** debugging `firewalld` — it saves hours. Both must permit the traffic for it to flow.

### `iptables` — recognise it (legacy but everywhere)

```bash
iptables -L -n -v                                       # list rules with packet counts
iptables -A INPUT -p tcp --dport 22 -j ACCEPT              # allow SSH
iptables -A INPUT -j DROP                                     # default-deny the rest (order matters!)

# Note: Docker and Kubernetes write iptables/nftables rules directly —
# which is why "my firewalld rule is ignored" often means Docker overrode it.
```

---

## Bonus: `ufw` — Ubuntu's firewall front-end — hands on

`ufw` (Uncomplicated Firewall) is Ubuntu/Debian's default front-end over the same `iptables`/`nftables` engine that `firewalld` manages on RHEL. It trades zones for a simpler allow/deny list — good for a single-purpose server, less suited to multi-NIC, multi-trust-level boxes.

```bash
# Enable it and check status — do this over console/local access first,
# NOT over the SSH connection you're about to firewall (see warning below)
sudo apt-get install -y ufw
sudo ufw status verbose             # inspect before enabling anything
```

```bash
# Always allow SSH BEFORE enabling — this is the #1 way people lock themselves out
sudo ufw allow OpenSSH               # or: sudo ufw allow 22/tcp
sudo ufw enable                        # turn the firewall on (default-deny inbound)
```

```bash
# Everyday rules
sudo ufw allow 80/tcp                        # allow HTTP by port
sudo ufw allow https                           # allow by service name (from /etc/services)
sudo ufw allow from 10.0.1.0/24 to any port 3306  # restrict a port to one subnet
sudo ufw deny 23                                    # explicitly block a port (Telnet)
sudo ufw delete allow 80/tcp                          # remove a rule
sudo ufw status numbered                                # list rules with index numbers (for delete by number)
sudo ufw reload                                           # apply changes without disabling
```

| Job | `firewalld` (RHEL) | `ufw` (Ubuntu) |
|---|---|---|
| Check status | `firewall-cmd --state` | `ufw status verbose` |
| Allow a port | `firewall-cmd --permanent --add-port=80/tcp` | `ufw allow 80/tcp` |
| Allow a service | `firewall-cmd --permanent --add-service=https` | `ufw allow https` |
| Remove a rule | `firewall-cmd --permanent --remove-service=X` | `ufw delete allow X` |
| Apply changes | `firewall-cmd --reload` | `ufw reload` (rules apply immediately, no separate permanent/runtime split) |
| Restrict by source | rich rule (see topic 66) | `ufw allow from <subnet> to any port <port>` |

> [!WARNING]
> **Allow SSH before you enable `ufw`.** Unlike `firewalld`'s public zone (which usually ships with SSH pre-allowed), a fresh `ufw enable` defaults to deny-all-inbound. If you enable it before adding `ufw allow OpenSSH`, your current SSH session may keep working until it drops — but the *next* connection attempt will be refused and you're locked out with no rule to undo remotely. Confirm the SSH rule is listed in `ufw status` before running `ufw enable`, and keep console/serial access available as a fallback, same as any firewall change.

> [!NOTE]
> **`ufw` is still `iptables`/`nftables` underneath.** Rules you add with `ufw` show up in `iptables -L` (or `nft list ruleset`) just like `firewalld` rules do. Docker's habit of writing its own `iptables` rules directly can override or bypass `ufw` the same way it does `firewalld` — if a container's port is reachable when `ufw` says it shouldn't be, suspect Docker.

---

## Where this leaves you

You now have the full arc: from how a packet leaves the box (Part 1), through remote access and file transfer (Part 2), package and patch management (Part 3), the core network services (Part 4), to the operational disciplines that keep a fleet observable, secure, fast and defended (Part 5).

The throughline for a DevOps engineer is that **every one of these is codifiable** — ship logs with a config-managed `rsyslog`, harden with Ansible roles, tune via `/etc/sysctl.d` in your image build, and manage firewalls as declarative rules. Learn them by hand here, then automate them everywhere.

> [!TIP]
> **The end-to-end troubleshooting flow:** Reachable? → `ping`/`mtr`. Right path? → `traceroute -T -p <port>`. Port open? → `ss -tulpn` (local), `nc -zv` (remote), then the firewall + Security Group. Name resolving? → `dig` vs `getent`. Service answering? → `curl -fsS`. Clock sane? → `chronyc tracking`. What broke & when? → `journalctl -u <svc> -e` + central logs. Walk these in order and almost nothing stays mysterious.

---

[⬅ Part 4](./04-network-services-dns-ntp-web-proxy.md) · [Index](./README.md)
