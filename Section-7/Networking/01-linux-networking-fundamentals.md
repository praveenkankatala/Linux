# Part 1 — Linux Networking Fundamentals

*Interfaces, files, commands & tools. Topics 1–16.*

**In this part:** the client-server model, enabling connectivity (link → address → route → DNS), the network files that still run everything, the `iproute2` commands that replaced `ifconfig`, `curl` vs `wget`, NIC naming & bonding, and a modern-utility cheat sheet.

[⬅ Back to index](./README.md)

---

## 1. How to read this guide

This part covers the foundations you need before touching any Linux network service: how a Linux box talks to the network, which files and kernel objects actually control connectivity, and the everyday commands you use to inspect and fix it.

**The mental model to hold throughout:** a network request leaves your process, gets a **name resolved to an IP** (DNS), the kernel decides **which interface and gateway** to send it out of (routing table), wraps it in frames tied to a **MAC address** (ARP), and a **firewall** decides whether it may pass. Almost every "the network is broken" ticket is a failure in one of those layers, and every command below inspects exactly one of them.

> [!TIP]
> **How to practise:** Spin up two small VMs or EC2 instances in the same subnet. One acts as **server**, one as **client**. Nearly every hands-on in this series works with just those two hosts. Run every command twice — once on a healthy system to learn the *normal* output, once after deliberately breaking something (`ip link set eth0 down`) so you recognise the *failure* signature.

---

## 2. The client–server relationship

Almost all networking is one program asking another for something.

- The **server** is the long-running process that *listens* on a well-known port and waits.
- The **client** is the process that *initiates* a connection to that port, sends a request, and reads a response.

The same physical machine is routinely both — your web server (server on `:443`) is also an SSH client when it pulls code from GitHub.

### The three things that define a listening service

| Element | Meaning |
|---|---|
| **IP address** | Which network interface it accepts connections on (`0.0.0.0` = all, `127.0.0.1` = localhost only) |
| **Port** | A 16-bit number (0–65535). Ports below 1024 are "privileged" and need root to bind (22 SSH, 80 HTTP, 443 HTTPS, 53 DNS, 3306 MySQL) |
| **Protocol** | Almost always **TCP** (reliable, connection-oriented: SSH, HTTP, DB) or **UDP** (fire-and-forget: DNS, NTP, syslog) |

A connection is uniquely identified by the **4-tuple**: source IP, source port, destination IP, destination port. That's why one server on port 443 can hold thousands of simultaneous connections — each client uses a different high-numbered source port.

```bash
# Prove the model on one host: start a listener, connect to it

# Terminal 1 — the 'server'
nc -l 9000                    # netcat listens on TCP 9000

# Terminal 2 — the 'client'
nc 127.0.0.1 9000              # connect; type text, it appears on the server

# See the established 4-tuple while both are running
ss -tnp | grep 9000
```

> [!NOTE]
> **TCP handshake:** TCP opens with a **3-way handshake**: client SYN → server SYN-ACK → client ACK. If you see connections stuck in `SYN-SENT` (client) or `SYN-RECV` (server), the packets are leaving but the reply is being dropped — almost always a firewall or security group, not the app.

---

## 3. Enabling internet on Linux

"Getting on the internet" is four independent things that must all succeed. Debug them **in this order** — do not touch DNS until you can ping an IP, and do not touch routing until the interface is up.

| # | Layer | What must be true | How to check |
|---|---|---|---|
| 1 | Link | The interface (NIC) is UP with carrier | `ip link show` |
| 2 | Address | The interface has an IP + subnet mask | `ip addr show` |
| 3 | Route | A default gateway exists to reach non-local IPs | `ip route show` |
| 4 | DNS | Names resolve to IPs | `cat /etc/resolv.conf` + `dig` |

```bash
# Run these five lines and the failing layer reveals itself
ip link show eth0        # state UP? Look for 'state UP' and 'LOWER_UP'
ip addr show eth0        # is there an 'inet 10.x.x.x/24' line?
ip route show            # is there a 'default via <gateway>' line?
ping -c3 8.8.8.8          # can I reach the internet BY IP? (tests 1-3)
ping -c3 google.com       # can I reach it BY NAME? (tests DNS = step 4)
```

> [!TIP]
> **The classic split-result:** `ping 8.8.8.8` works but `ping google.com` fails ⇒ networking is **fine**, DNS is broken. Fix `/etc/resolv.conf`. This single distinction resolves a huge share of connectivity tickets.

---

## 4. Network configuration & connectivity

On modern RHEL-family systems, **NetworkManager** owns the configuration. You rarely edit files by hand any more — you use `nmcli`. On Amazon Linux and older CentOS you may still meet legacy `ifcfg-*` scripts; both are covered below.

### The modern way — `nmcli` (NetworkManager CLI)

```bash
nmcli device status              # list NICs and their connection state
nmcli connection show            # list saved connection profiles

# Set a STATIC IP permanently
nmcli con mod eth0 ipv4.addresses 192.168.10.20/24
nmcli con mod eth0 ipv4.gateway 192.168.10.1
nmcli con mod eth0 ipv4.dns '8.8.8.8 1.1.1.1'
nmcli con mod eth0 ipv4.method manual   # manual = static; 'auto' = DHCP
nmcli con up eth0                       # apply
```

### The legacy way — `ifcfg` scripts (older CentOS / Amazon Linux 2)

```ini
# /etc/sysconfig/network-scripts/ifcfg-eth0
TYPE=Ethernet
BOOTPROTO=none        # 'none'/'static' = manual, 'dhcp' = automatic
NAME=eth0
DEVICE=eth0
ONBOOT=yes             # bring up at boot — the field people forget
IPADDR=192.168.10.20
PREFIX=24
GATEWAY=192.168.10.1
DNS1=8.8.8.8
```

Apply with: `nmcli con reload && nmcli con up eth0`

> [!WARNING]
> **Cloud gotcha:** On **EC2**, never hard-code a static IP the way you would on-prem. The instance gets its private IP from the VPC via DHCP; overriding it can black-hole the instance so you lose SSH. Change addressing through the **VPC/ENI**, not inside the guest.

---

## 5. Network components — the vocabulary

These are the objects every later command manipulates. Learn them once and the rest of the module is naming, not new concepts.

| Component | What it is | Analogy |
|---|---|---|
| **NIC / Interface** | The network card — physical (`eth0`, `ens5`) or virtual (`lo`, `bond0`, `docker0`) | The door your traffic leaves through |
| **IP address** | Logical address of an interface on a network (`10.0.1.5`) | Your street address |
| **Subnet mask / CIDR** | Which part of the IP is "network" vs "host" (`/24` = 256 addresses) | Which neighbourhood you're in |
| **MAC address** | 48-bit hardware address burned into the NIC | The device's fingerprint |
| **Gateway / Router** | The IP traffic is sent to when the destination is off-subnet | The exit onto the highway |
| **DNS resolver** | Server that turns names into IPs | The phone book |
| **Port** | Numbered endpoint identifying a service on a host | The apartment number |
| **ARP table** | Local map of IP → MAC for the current subnet | Post-it note of who lives where nearby |

> [!NOTE]
> **CIDR quick math:** `/24` = 256 addresses · `/16` = 65,536 · `/25` = 128 · `/30` = 4 (2 usable — classic point-to-point). In every subnet the **first** address is the network ID and the **last** is broadcast, so usable hosts = total − 2.

---

## 6. Understanding the network on Linux — hands on

A guided tour of a live host. Run each block and read the annotation before moving on. This is the single most useful thing to internalise in the whole module.

```bash
# Step 1 — interfaces at a glance
ip -brief address              # one line per NIC: name, state, IPs (best overview)
# eth0  UP    10.0.1.42/24 fe80::8c1:...   <- healthy: UP + has an IPv4
# lo    UNKNOWN 127.0.0.1/8                <- loopback, always present

# Step 2 — routing decides the exit
ip route
# default via 10.0.1.1 dev eth0   <- the gateway for the internet
# 10.0.1.0/24 dev eth0 proto kernel ... <- the local subnet, no gateway needed

# Step 3 — who am I talking to on my subnet? (ARP / neighbours)
ip neigh                        # IP <-> MAC map, REACHABLE/STALE per entry

# Step 4 — what am I listening on / connected to?
ss -tulpn                       # t=tcp u=udp l=listening p=pid n=numeric
```

> [!TIP]
> **Build the reflex:** `ip a` → `ip r` → `ip n` → `ss -tulpn` in that order tells you the complete network state of a box in under ten seconds. Make it muscle memory.

---

## 7. Network files & components

Before NetworkManager, everything was files, and those files still exist and still work. Knowing them means you can fix a box even when the fancy tools are broken.

| File | Purpose | Notes |
|---|---|---|
| `/etc/hostname` | The system's hostname | One line. `hostnamectl set-hostname` is the safe way to change it |
| `/etc/hosts` | Static name→IP overrides | Checked **before** DNS. Great for pinning a name in a lab |
| `/etc/resolv.conf` | Which DNS servers to use | `nameserver`, `search`, `domain` lines. Often auto-generated — don't hand-edit if NM/systemd-resolved manages it |
| `/etc/nsswitch.conf` | Resolution *order* (files vs DNS) | The `hosts:` line decides whether `/etc/hosts` or DNS wins |
| `/etc/sysconfig/network-scripts/ifcfg-*` | Per-interface config (legacy RHEL) | `IPADDR`, `GATEWAY`, `ONBOOT`... |
| `/etc/services` | Name↔port reference table | Read-only map: `http 80/tcp`. Informational, not enforced |
| `/proc/net/*` | Live kernel network state | `/proc/net/tcp`, `/proc/net/dev` — what `ss`/`ip` read under the hood |

> [!NOTE]
> **nsswitch is the referee:** If `hosts:` in `/etc/nsswitch.conf` reads `files dns`, a name in `/etc/hosts` **beats** DNS. Reverse them and DNS wins. This is why a stale `/etc/hosts` entry can silently override a correct DNS record.

---

## 8. Network files — hands on

```bash
# Pin a hostname to an IP locally (no DNS needed) — perfect for labs
echo '10.0.1.50 app-server app-server.lab' | sudo tee -a /etc/hosts
ping -c2 app-server              # resolves from /etc/hosts instantly

# Change DNS resolvers by hand
sudo tee /etc/resolv.conf <<'EOF'
search lab.local
nameserver 8.8.8.8
nameserver 1.1.1.1
EOF

# Confirm resolution order
grep '^hosts:' /etc/nsswitch.conf   # expect: hosts: files dns
getent hosts app-server              # shows which source answered
```

> [!WARNING]
> **`resolv.conf` gets overwritten:** If NetworkManager or `systemd-resolved` manages the box, your hand-edited `/etc/resolv.conf` is regenerated on the next network event. Set DNS via `nmcli con mod ... ipv4.dns` instead, or the change won't survive.

---

## 9. Common network commands — old vs new

The modern **`iproute2`** suite (`ip`, `ss`) replaced the deprecated **`net-tools`** (`ifconfig`, `netstat`, `route`, `arp`). Learn the new ones; recognise the old ones because they still appear in scripts and old answers online.

| Task | Modern (use this) | Legacy (deprecated) |
|---|---|---|
| Show interfaces & IPs | `ip addr` / `ip a` | `ifconfig` |
| Bring interface up/down | `ip link set eth0 up` | `ifconfig eth0 up` |
| Show routing table | `ip route` | `route -n` |
| Add a route | `ip route add ...` | `route add ...` |
| Show sockets/ports | `ss -tulpn` | `netstat -tulpn` |
| Show ARP/neighbours | `ip neigh` | `arp -n` |
| Test reachability | `ping` | `ping` (unchanged) |
| Trace path | `tracepath` / `traceroute` | `traceroute` |
| DNS lookup | `dig` / `host` | `nslookup` |

---

## 10. Common network commands — hands on

```bash
ip a                            # all interfaces + addresses
ip -s link show eth0             # per-interface RX/TX stats & errors
ip route get 8.8.8.8              # WHICH interface/gateway a packet would use
ss -tulpn                        # every listening TCP/UDP port + owning PID
ss -tnp state established         # current live connections
ping -c4 -i0.2 10.0.1.1            # 4 pings, 0.2s apart
ip neigh show                     # ARP cache
mtr google.com                    # ping + traceroute combined, live
```

> [!TIP]
> **`ip route get` is underused:** `ip route get <ip>` answers "which NIC and gateway will this packet actually use?" without sending anything. On multi-homed hosts (two NICs) it instantly tells you if traffic is leaving the wrong interface.

---

## 11. `curl` vs `wget`

Both fetch data over HTTP(S)/FTP, but they have different jobs.

- **`curl`** is a Swiss-army client for *talking to* APIs and services — it prints to stdout, speaks every method, and is the tool you script.
- **`wget`** is a *downloader* — it saves files, follows links, and can mirror whole sites and resume broken downloads.

| Need | Reach for | Why |
|---|---|---|
| Call a REST API, send JSON, set headers | **curl** | Full control of method, headers, body; prints response |
| Download a file to disk | **wget** (or `curl -O`) | `wget` saves by default; `curl` needs `-O` |
| Resume a huge interrupted download | **`wget -c`** | Built-in resume |
| Mirror/recurse a website | **`wget -r`** | curl can't recurse |
| Health-check / scripting in pipelines | **`curl -fsS`** | Clean exit codes, silent, fails on HTTP errors |
| Follow redirects | `curl -L` / `wget` (auto) | curl needs `-L` explicitly |

---

## 12. `curl` & `wget` — hands on

```bash
# --- curl for APIs ---
curl -s https://api.github.com/users/torvalds   # GET, print JSON
curl -I https://example.com                      # headers only (HEAD)
curl -L http://example.com                        # follow redirects

# POST JSON to an API
curl -X POST https://httpbin.org/post \
  -H 'Content-Type: application/json' \
  -d '{"name":"devops","role":"engineer"}'

# The scripting pattern — fail loudly, stay quiet on success
curl -fsS https://example.com/health || echo 'health check FAILED'
```

```bash
# --- wget for downloads ---
wget https://example.com/big.iso                          # save to ./big.iso
wget -c https://example.com/big.iso                        # resume a partial file
wget -O app.tar.gz https://example.com/download             # save under a chosen name
wget -q --show-progress https://example.com/f.zip            # quiet + progress bar
wget --mirror -p --convert-links https://site.tld/            # mirror a whole site
```

> [!NOTE]
> **curl flag decoder:** `-f` fail on HTTP ≥400 (vital in scripts) · `-s` silent · `-S` show errors even when silent · `-L` follow redirects · `-O` save with remote name · `-o file` save with chosen name · `-I` headers only · `-H` add header · `-d` send body · `-X` set method.

---

## 13. NIC — Network Interface Cards

A NIC is the endpoint the kernel attaches an IP to. Naming has evolved: old kernels used `eth0`/`eth1` (assigned in boot order, so unpredictable across reboots), modern **Predictable Network Interface Names** encode the hardware slot — `ens5`, `enp0s3`, `eno1` — so a card always keeps the same name. On EC2 you'll see `ens5`/`eth0`; the loopback `lo` is always present.

### Reading interface names

- `en` = Ethernet · `wl` = wireless LAN · `ww` = WWAN (cellular)
- `o<index>` = onboard (`eno1`) · `s<slot>` = hotplug/PCI slot (`ens5`) · `p<bus>s<slot>` = PCI bus+slot (`enp0s3`)
- `lo` = loopback · `bond0` = bonded interface · `br0` = bridge · `vlan10`/`eth0.10` = VLAN sub-interface

```bash
ip link show                        # all links + MAC + MTU + state
ethtool eth0                        # speed, duplex, link detected (physical NICs)
ethtool -i eth0                     # driver + firmware behind the NIC
cat /sys/class/net/eth0/mtu          # current MTU (default 1500)
ip link set eth0 mtu 9000            # jumbo frames (needs end-to-end support)
```

> [!WARNING]
> **MTU matters in the cloud:** EC2 supports **jumbo frames (MTU 9001)** inside a VPC but the path to the internet is capped at 1500. A mismatched MTU causes large packets to silently vanish — connections hang after the handshake. If SSH connects but `scp` of a big file stalls, suspect MTU.

---

## 14. NIC bonding — hands on

**Bonding** (a.k.a. teaming/link aggregation) combines multiple physical NICs into one logical interface for **redundancy** (survive a cable/switch failure) or **throughput** (aggregate bandwidth). The behaviour is set by the **bonding mode**.

| Mode | Name | Gives you | Needs switch config? |
|---|---|---|---|
| 0 | `balance-rr` | Round-robin — throughput + redundancy | Yes |
| 1 | `active-backup` | One active NIC, others standby — pure redundancy | No (most common) |
| 4 | `802.3ad` (LACP) | Standard link aggregation — throughput + redundancy | Yes (LACP on switch) |
| 6 | `balance-alb` | Adaptive load balancing, no switch help needed | No |

```bash
# Active-backup bond of eth1 + eth2 using nmcli (mode 1)
nmcli con add type bond con-name bond0 ifname bond0 \
  bond.options 'mode=active-backup,miimon=100'
nmcli con add type ethernet con-name bond0-p1 ifname eth1 master bond0
nmcli con add type ethernet con-name bond0-p2 ifname eth2 master bond0
nmcli con mod bond0 ipv4.addresses 192.168.10.30/24 ipv4.method manual
nmcli con up bond0

# Verify — shows active slave and per-NIC link status
cat /proc/net/bonding/bond0
```

> [!TIP]
> **`miimon` is the heartbeat:** `miimon=100` tells the kernel to check each NIC's link every 100 ms. Without it the bond won't notice a dead cable and won't fail over. Always set it.

> [!WARNING]
> **Not for standard EC2:** Classic kernel bonding generally does **not** apply to normal EC2 instances — AWS gives redundancy at the ENI/AZ level. Bonding is an on-prem / bare-metal (including EC2 bare-metal) technique. Know it for interviews and data-centre work.

---

## 15. New network utilities

Beyond the classics, a handful of modern tools make diagnosis far faster. These are the ones worth installing on any box you operate.

**`mtr` — traceroute + ping, live.** Shows every hop **and** the loss/latency at each hop, updating continuously. The fastest way to prove *where* on the path packets are being lost.

```bash
mtr google.com
mtr -rw -c100 8.8.8.8       # report mode: 100 packets, then a clean table
```

**`nmap` — port & host discovery.** Scans which ports are open on a host and what's listening. Essential for verifying a firewall change actually opened (or closed) a port.

```bash
nmap -sT -p22,80,443 10.0.1.50    # is the server actually listening?
nmap -sn 10.0.1.0/24               # which hosts are alive on the subnet
```

**`tcpdump` — packet capture.** Captures raw packets so you can see exactly what's on the wire. When "the app says connection refused" you use `tcpdump` to prove whether packets even arrive.

```bash
sudo tcpdump -i eth0 port 443 -nn
sudo tcpdump -i any host 10.0.1.50 -w capture.pcap   # save for Wireshark
```

**`nc` (netcat) — the network multitool**

```bash
nc -zv 10.0.1.50 22    # is port 22 reachable? (z=scan, v=verbose)
nc -l 9000              # open an ad-hoc listener for testing
```

---

## 16. Tool cheat sheet — print this

The "which tool do I reach for" summary for the whole module.

| Tool | One-line job | Signature command | Install (RHEL) |
|---|---|---|---|
| `ip` | Interfaces, addresses, routes, neighbours | `ip a` / `ip r` | `iproute` (default) |
| `ss` | Listening ports & live connections | `ss -tulpn` | `iproute` (default) |
| `ping` | Is the host reachable at all? | `ping -c4 host` | `iputils` (default) |
| `mtr` | Where on the path is loss happening? | `mtr -rw host` | `dnf install mtr` |
| `traceroute` | The hop-by-hop path | `traceroute host` | `dnf install traceroute` |
| `dig` | DNS resolution detail | `dig host` | `dnf install bind-utils` |
| `curl` | Talk to HTTP APIs / health checks | `curl -fsS url` | `curl` (default) |
| `wget` | Download / mirror files | `wget -c url` | `dnf install wget` |
| `nmap` | Which ports/hosts are open | `nmap -p22 host` | `dnf install nmap` |
| `tcpdump` | See the actual packets | `tcpdump -i eth0 -nn` | `dnf install tcpdump` |
| `nc` | Port test / ad-hoc listener | `nc -zv host 22` | `dnf install nmap-ncat` |
| `ethtool` | Physical NIC speed/driver/link | `ethtool eth0` | `dnf install ethtool` |

> [!TIP]
> **Decision shortcut:** **Reachable?** → `ping`. **Which port open?** → `nmap`/`nc`. **Where's the loss?** → `mtr`. **Name resolving?** → `dig`. **What's actually on the wire?** → `tcpdump`. **What's my box listening on?** → `ss -tulpn`.

---

[⬅ Back to index](./README.md) · [Next: Part 2 — Remote Access & File Transfer ➡](./02-remote-access-and-file-transfer.md)
