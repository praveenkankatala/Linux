# Part 4 — Network Services: DNS, NTP, Web & Proxy

*Topics 43–56.*

**In this part:** hostname management and name resolution with `dig`/`nslookup`/`getent`, how DNS actually resolves, the record types that matter, running your own BIND zone, time sync with `chrony` (why wrong clocks break TLS & AD), Nginx & Apache web servers, and Squid forward-proxy for controlled egress.

[⬅ Part 3](./03-package-and-patch-management.md) · [Index](./README.md)

---

## 43. Hostname / IP lookup

Two everyday jobs: managing **your own** host's name, and looking up **someone else's** name↔IP. The hostname is more than cosmetic — it appears in logs, prompts, monitoring, and TLS certificates, so set it properly with `hostnamectl` rather than editing files randomly.

### Your own hostname

```bash
hostname                       # current short hostname
hostname -f                       # fully-qualified (FQDN) if a domain is set
hostname -I                         # all IP addresses of this host
hostnamectl                           # full identity: hostname, OS, kernel, machine-id
sudo hostnamectl set-hostname web01.prod.lab    # set it PERMANENTLY
```

> [!NOTE]
> **Three kinds of hostname:** systemd tracks a **static** name (persistent, in `/etc/hostname`), a **transient** name (set by DHCP/mDNS at runtime), and a **pretty** name (free-form, e.g. "Prod Web 01"). `hostnamectl set-hostname` sets the static one, which is what you almost always want.

### Looking up other hosts

```bash
getent hosts google.com    # resolves via the FULL nsswitch chain (files+dns)
host google.com              # quick, human-friendly DNS lookup
nslookup google.com            # classic lookup (still everywhere)
dig google.com                   # the detailed, scriptable lookup tool
```

> [!TIP]
> **`getent` vs `dig`:** `dig`/`nslookup` query **DNS only**. `getent hosts` follows the same path the *system* does — so it also sees `/etc/hosts`. If an app resolves a name differently from `dig`, check `getent`: something in `/etc/hosts` or `nsswitch` is overriding DNS.

---

## 44. `nslookup` & `dig` — hands on

`dig` (Domain Information Groper) is the professional tool — precise, scriptable, shows the full answer. `nslookup` is simpler and still common. Learn to read a `dig` answer and DNS stops being mysterious.

```bash
dig google.com                # full answer with ANSWER SECTION
dig google.com +short           # just the IP(s) — perfect for scripts
dig google.com MX                 # mail servers for the domain
dig google.com NS                   # authoritative name servers
dig google.com TXT                    # SPF/DKIM/verification records
dig -x 8.8.8.8                          # REVERSE lookup: IP -> name (PTR)
dig @1.1.1.1 google.com                   # ask a SPECIFIC resolver (bypass local)
dig google.com +trace                       # follow resolution from the root down
```

### Reading a dig answer

- **ANSWER SECTION** — the actual records returned (A, AAAA, CNAME...). This is what you came for.
- **status: NOERROR** = success · **NXDOMAIN** = name doesn't exist · **SERVFAIL** = the resolver broke (often DNSSEC or an unreachable upstream).
- **TTL** (the number before the record type) — how many seconds the answer may be cached. High TTL = changes propagate slowly.
- **`;; Query time`** — how long the resolver took. Spikes here explain "the app is slow" when the network is fine.

> [!TIP]
> **`@resolver` isolates DNS problems:** `dig @8.8.8.8 name` vs `dig name` (your local resolver). If the public resolver answers but yours doesn't, the fault is your **local DNS config**, not the record itself. This one comparison splits "my DNS is broken" from "their record is broken".

---

## 45. DNS — Domain Name System

DNS is the internet's distributed phone book: it turns names humans remember into IPs machines route to. Understanding the **hierarchy** and the **record types** is what lets you diagnose resolution problems instead of just restarting things.

### How a lookup actually resolves

1. Your host asks its **resolver** (recursive server) for `www.example.com`.
2. The resolver asks a **root** server → "ask the `.com` servers".
3. It asks the **`.com` TLD** servers → "ask `example.com`'s name servers".
4. It asks `example.com`'s **authoritative** servers → gets the A record.
5. The resolver **caches** the answer (for its TTL) and returns it to you.

### Record types you must know

| Record | Maps | Example / use |
|---|---|---|
| **A** | Name → IPv4 | `web.example.com` → `10.0.1.5` |
| **AAAA** | Name → IPv6 | The IPv6 equivalent of A |
| **CNAME** | Name → another name (alias) | `www` → `example.com` |
| **MX** | Domain → mail servers | Email routing, with priorities |
| **NS** | Domain → authoritative servers | Delegation |
| **PTR** | IP → name (reverse) | Reverse DNS, used by mail anti-spam |
| **TXT** | Arbitrary text | SPF, DKIM, domain verification |
| **SOA** | Zone metadata | Serial, refresh, TTL defaults |

> [!NOTE]
> **TTL & caching = propagation delay:** When you change a DNS record, old answers linger in caches until their **TTL** expires. That's why "DNS changes take time to propagate". Before a planned cutover, **lower the TTL a day ahead** (e.g. to 300s) so the switch is fast, then raise it again afterwards.

---

## 46. Download, install & configure DNS (BIND)

**BIND** (`named`) is the reference DNS server. You'd run your own to serve internal names (`db.corp.local`) that public DNS can't know about, or to give a private network fast local resolution. Here's a minimal authoritative zone for a lab domain.

```bash
dnf install -y bind bind-utils
systemctl enable --now named
```

```conf
# /etc/named.conf — listen on the LAN and allow your subnet to query
options {
    listen-on port 53 { 127.0.0.1; 10.0.1.10; };
    allow-query { localhost; 10.0.1.0/24; };
    recursion yes;    # act as a resolver for the LAN
};

zone "lab.local" IN {
    type master;
    file "lab.local.zone";
};
```

```dns
; /var/named/lab.local.zone — the actual records
$TTL 3600
@   IN SOA  ns1.lab.local. admin.lab.local. (
        2024060101 ; serial (BUMP THIS on every edit)
        3600 1800 604800 86400 )
@   IN NS   ns1.lab.local.
ns1 IN A    10.0.1.10
web01 IN A  10.0.1.20
db01  IN A  10.0.1.30
app   IN CNAME web01.lab.local.
```

```bash
# Validate, restart, test
named-checkconf                                     # validate named.conf syntax
named-checkzone lab.local /var/named/lab.local.zone   # validate the zone
systemctl restart named
firewall-cmd --permanent --add-service=dns && firewall-cmd --reload
dig @10.0.1.10 web01.lab.local +short                 # test against YOUR server
```

> [!WARNING]
> **Bump the serial — every time:** The zone **serial** number is how secondaries and caches know the zone changed. Edit records but forget to increment the serial, and your changes are silently ignored. The convention is `YYYYMMDDNN`. `named-checkzone` catches syntax errors but not a forgotten serial bump.

---

## 47. NTP — time synchronisation

Every server's clock must agree, and NTP (Network Time Protocol) keeps them within milliseconds of a reference. This is not optional: **wrong time breaks TLS certificates, Kerberos/AD authentication, log correlation, database replication, and JWT tokens.** On modern systems the daemon is **`chrony`**, which handles laptops, VMs and jittery cloud clocks far better than the old `ntpd`.

### Why time drift is a silent killer

- **TLS** — certificates are valid only within a time window; a clock hours off = "certificate not yet valid" errors.
- **Kerberos / Active Directory** — rejects auth if the clock skews more than ~5 minutes. Directly relevant to AD-joined Linux.
- **Logs** — correlating events across hosts is impossible if their timestamps disagree.
- **Distributed systems** — databases, certificates and tokens all assume synchronised clocks.

---

## 48. `chronyd` — hands on

```bash
dnf install -y chrony
systemctl enable --now chronyd
```

```conf
# /etc/chrony.conf — point at time sources
pool 2.pool.ntp.org iburst        # public pool; iburst = sync fast at startup
# On EC2, prefer the link-local Amazon Time Sync Service:
# server 169.254.169.123 prefer iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3                      # allow big step-corrections during first 3 updates
```

```bash
# Verify synchronisation
chronyc sources -v               # which time servers, and their quality
chronyc sourcestats                # measured offset/jitter per source
chronyc tracking                     # AM I in sync? offset from true time
timedatectl                            # overall time status + 'System clock synchronized: yes'
timedatectl set-timezone Asia/Kolkata    # set the timezone
```

> [!TIP]
> **Reading `chronyc sources`:** The leading symbol matters: `*` = the source currently syncing to (good), `+` = an acceptable backup, `?` = unreachable, `x` = a "falseticker" (disagrees with the others). You want to see a `*`. On EC2 use `169.254.169.123` — it's free, on-network, and never rate-limited.

---

## 49. Nginx

Nginx is a high-performance web server and **reverse proxy**. Its event-driven architecture handles tens of thousands of concurrent connections on modest hardware, which is why it fronts most of the web. You'll use it three ways: serving static files, **reverse-proxying** to app servers, and **load-balancing** across several backends.

### The three roles

| Role | What it does | Typical use |
|---|---|---|
| **Web server** | Serves static files (HTML/CSS/JS/images) | Static sites, SPA front-ends |
| **Reverse proxy** | Forwards requests to a backend app, returns the response | In front of Node/Java/Python apps; TLS termination |
| **Load balancer** | Spreads traffic across multiple backends | Scale-out, high availability |

---

## 50. Nginx — hands on

```bash
# Install and confirm it serves
dnf install -y nginx
systemctl enable --now nginx
firewall-cmd --permanent --add-service=http --add-service=https
firewall-cmd --reload
curl -I http://localhost    # expect: HTTP/1.1 200 OK
```

```nginx
# /etc/nginx/conf.d/app.conf — reverse proxy + load balancer in one file
upstream backend {                 # define a pool (add servers to load-balance)
    server 127.0.0.1:8080;
    server 127.0.0.1:8081;          # add a second = round-robin LB
}

server {
    listen 80;
    server_name app.lab.local;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;                     # pass client IP through
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

```bash
# Validate, then reload
nginx -t                      # ALWAYS test config before reloading
systemctl reload nginx          # apply with zero downtime
```

> [!WARNING]
> **`nginx -t` before every reload:** `systemctl reload nginx` on a broken config can drop the running server. `nginx -t` validates syntax first and points at the exact line. Make `nginx -t && systemctl reload nginx` a single habit — never reload blind.

> [!TIP]
> **`reload` vs `restart`:** **`reload`** re-reads config while keeping existing connections alive (zero downtime). **`restart`** kills and re-spawns the process (brief outage). Use `reload` for config changes; reserve `restart` for binary upgrades or when the process is wedged.

---

## 51. New system utility commands

A grab-bag of modern utilities that make operating a Linux box faster. These aren't networking-specific but you reach for them during every incident.

| Command | Replaces / adds | Use |
|---|---|---|
| `systemctl` | `service` + `chkconfig` | Start/stop/enable services, check status |
| `journalctl` | Reading `/var/log` by hand | Query systemd logs with filters |
| `ss` | `netstat` | Sockets & ports (Part 2) |
| `ip` | `ifconfig`/`route`/`arp` | All interface/route/neighbour ops (Part 1) |
| `nmcli` | Editing `ifcfg` files | Network config via NetworkManager |
| `timedatectl` | `date`/`hwclock` juggling | Time, timezone, sync status |
| `hostnamectl` | Editing `/etc/hostname` | Hostname & machine identity |
| `lsof` | — | Which process holds a file/port/socket |
| `watch` | — | Re-run a command every N seconds, live |

---

## 52. New system utilities — hands on

```bash
# systemctl — the service control centre
systemctl status nginx                                # is it running? recent log lines inline
systemctl --failed                                       # everything that failed to start
systemctl list-units --type=service --state=running

# journalctl — logs with a query language
journalctl -u nginx -f                                  # follow one service's logs live
journalctl -u nginx --since '1 hour ago'
journalctl -p err -b                                       # only errors, this boot
journalctl -k                                                 # kernel messages (dmesg equivalent)
```

```bash
# lsof — who is using this port/file?
lsof -i :8080                       # which process owns port 8080
lsof -i -P -n | grep LISTEN           # all listeners (numeric)
lsof /var/log/app.log                   # who has this file open

# watch — a live dashboard from any command
watch -n1 'ss -s'                          # socket counts, refreshed every second
watch -n2 'systemctl --failed'               # keep an eye on failing units
```

> [!TIP]
> **The incident-response combo:** "Port already in use" → `lsof -i :PORT`. "Service won't start" → `systemctl status X` then `journalctl -u X -e`. "Is it recovering?" → `watch`. These three cover most of what you do in the first five minutes of an incident.

---

## 53. Squid server — forward proxy

Squid is a **forward proxy** and caching server. Where Nginx (reverse proxy) sits in front of *your servers* to protect them, Squid sits in front of *your clients* to control and cache their outbound traffic. In enterprises it's how you give internal servers controlled internet access, enforce allow-lists, and cache downloads — very relevant to locked-down/regulated networks.

### Forward vs reverse proxy — don't confuse them

| | Forward proxy (Squid) | Reverse proxy (Nginx) |
|---|---|---|
| Sits in front of | Clients (outbound) | Servers (inbound) |
| Hides | The clients from the internet | The servers from clients |
| Controls | What internal hosts may reach outside | How outside traffic reaches your apps |
| Classic use | Corporate egress filtering + caching | TLS termination + load balancing |

---

## 54. Squid server — hands on

```bash
dnf install -y squid
systemctl enable --now squid    # listens on 3128 by default
```

```conf
# /etc/squid/squid.conf — allow your LAN, define what it may reach
acl localnet src 10.0.1.0/24
acl allowed_sites dstdomain .redhat.com .amazonaws.com
http_access allow localnet allowed_sites
http_access deny all              # default-deny everything else
http_port 3128
```

```bash
# Point clients at it and test the policy
systemctl restart squid
firewall-cmd --permanent --add-port=3128/tcp && firewall-cmd --reload

# On a CLIENT, route traffic through the proxy
export http_proxy=http://10.0.1.15:3128
export https_proxy=http://10.0.1.15:3128

curl -I https://www.redhat.com    # allowed -> 200
curl -I https://example.org         # blocked -> 403 by Squid
tail -f /var/log/squid/access.log     # watch what clients request
```

> [!NOTE]
> **Where proxies fit in DevOps:** Air-gapped or PCI/pharma-regulated servers often have **no direct internet** — all egress goes through a Squid proxy with an allow-list (e.g. only AWS and OS-repo domains). Your `dnf`, `curl` and package tools then honour `http_proxy`/`https_proxy`. Knowing Squid explains why "the server has no internet but can still `dnf update`".

---

## 55. Web servers — Apache vs Nginx

The two dominant web servers on Linux are **Apache HTTPD** and **Nginx**. Both serve HTTP; they differ in architecture. Apache is process/thread-per-connection and endlessly modular; Nginx is event-driven and excels at high concurrency and proxying. Many stacks run both: Nginx out front for TLS and static files, Apache behind it for application logic.

| | Apache (`httpd`) | Nginx |
|---|---|---|
| Model | Process/thread per connection | Event-driven, async |
| Best at | Dynamic content, `.htaccess`, modules | Static files, reverse proxy, high concurrency |
| Config | `/etc/httpd/`, per-dir `.htaccess` | `/etc/nginx/`, central only |
| Concurrency | Good | Excellent (C10k+) |
| Typical role | App server behind a proxy | Front-end proxy / static / LB |

---

## 56. Web servers — hands on (Apache)

```bash
# Stand up Apache in five lines
dnf install -y httpd
systemctl enable --now httpd
firewall-cmd --permanent --add-service=http && firewall-cmd --reload
echo '<h1>Hello from Apache</h1>' > /var/www/html/index.html
curl http://localhost    # see your page
```

```apache
# A name-based virtual host: /etc/httpd/conf.d/site1.conf
<VirtualHost *:80>
    ServerName site1.lab.local
    DocumentRoot /var/www/site1
    ErrorLog /var/log/httpd/site1-error.log
    CustomLog /var/log/httpd/site1-access.log combined
</VirtualHost>
```

```bash
# Validate + the SELinux step people miss
apachectl configtest    # == 'Syntax OK' before you reload
systemctl reload httpd

# SELinux gotcha on RHEL — non-default docroots/ports need a label
sudo semanage fcontext -a -t httpd_sys_content_t '/var/www/site1(/.*)?'
sudo restorecon -Rv /var/www/site1
```

> [!WARNING]
> **SELinux blocks "correct" configs:** On RHEL with SELinux **enforcing**, Apache is denied access to files outside the default docroot, custom ports, and outbound proxy connections — even when permissions look right. Symptom: 403 or "permission denied" despite `chmod 644`. Fix with `semanage`/`restorecon`, or check `sudo ausearch -m avc -ts recent` to see what SELinux blocked. Don't just disable SELinux — label correctly.

---

[⬅ Part 3](./03-package-and-patch-management.md) · [Index](./README.md) · [Next: Part 5 — Logging, Hardening, Firewall & Tuning ➡](./05-logging-hardening-firewall-tuning.md)
