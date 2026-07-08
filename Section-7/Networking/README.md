# 🐧 Linux Networking — A Practical, Hands-On Guide

A complete, no-fluff walkthrough of Linux networking for sysadmins, DevOps engineers, and SREs — from "why is the network broken" fundamentals all the way to firewall hardening and kernel tuning.

Every topic follows the same pattern: **what it is → why it matters → the exact commands → the mistake everyone makes**. Nothing here is theoretical filler; every command has been run and every warning comes from a real failure mode.

> **Target platforms:** RHEL / CentOS / Rocky Linux / AlmaLinux / Amazon Linux 2 & 2023, with Ubuntu/Debian differences called out wherever they matter.

---

## 📚 How this guide is organized

The content is split into 5 parts, 66 topics total. Read them in order the first time — each part builds on the one before it.

| Part | File | Covers | Topics |
|---|---|---|---|
| **1** | [`01-linux-networking-fundamentals.md`](./01-linux-networking-fundamentals.md) | Client-server model, interfaces, IP/routing/DNS, network files, `iproute2`, `curl`/`wget`, NIC bonding | 1–16 |
| **2** | [`02-remote-access-and-file-transfer.md`](./02-remote-access-and-file-transfer.md) | SSH & hardening, `ss`, `ping` vs `curl`, FTP/SFTP, `scp`, `rsync` | 17–28 |
| **3** | [`03-package-and-patch-management.md`](./03-package-and-patch-management.md) | RPM/YUM/DNF, patch lifecycle, automated patching script, local repos, `apt`/`dpkg` | 29–42 |
| **4** | [`04-network-services-dns-ntp-web-proxy.md`](./04-network-services-dns-ntp-web-proxy.md) | Hostnames, DNS & BIND, `chrony`/NTP, Nginx, Apache, Squid proxy | 43–56 |
| **5** | [`05-logging-hardening-firewall-tuning.md`](./05-logging-hardening-firewall-tuning.md) | Central logging (rsyslog), OS hardening, `traceroute`, kernel tuning, `firewalld`/`iptables`/`ufw` | 57–66 + bonus |

---

## 🧭 The mental model (hold onto this throughout)

A network request leaves your process, gets a **name resolved to an IP** (DNS), the kernel decides **which interface and gateway** to send it out of (routing table), wraps it in frames tied to a **MAC address** (ARP), and a **firewall** decides whether it may pass.

Almost every *"the network is broken"* ticket is a failure in exactly one of those layers:

```
Link  →  Address  →  Route  →  DNS  →  Firewall
(NIC up?)  (IP set?)  (gateway?)  (resolves?)  (allowed?)
```

Debug top to bottom, in order. Don't touch DNS until you can ping an IP. Don't touch routing until the interface is up.

---

## 🛠️ How to practise this yourself

1. Spin up **two small VMs** (local VirtualBox/VMware, or two EC2 instances in the same subnet). One is the **server**, one is the **client**. Almost every hands-on exercise in this guide only needs those two hosts.
2. Run every command **twice**: once on a healthy system to learn the *normal* output, once after deliberately breaking something (`ip link set eth0 down`, stop a service, block a port) so you recognise the *failure* signature.
3. Keep a **second terminal session open** before touching SSH config or the firewall — if you lock yourself out, you want a way back in.

---

## 🗺️ End-to-end troubleshooting flow

The single most useful cheat sheet in this whole guide — walk these in order and almost nothing stays mysterious:

| Question | Tool |
|---|---|
| Reachable at all? | `ping` / `mtr` |
| Right path to get there? | `traceroute -T -p <port>` |
| Port open? | `ss -tulpn` (local), `nc -zv` (remote), then firewall / Security Group |
| Name resolving? | `dig` vs `getent hosts` |
| Service actually answering? | `curl -fsS` |
| Clock sane? | `chronyc tracking` |
| What broke, and when? | `journalctl -u <service> -e` + central logs |

---

## ✅ Prerequisites

- Comfortable with a Linux shell (`bash`)
- `sudo`/root access on a test VM or cloud instance
- No prior networking certification needed — this guide teaches the concepts as it goes

## 📌 License / Use

Personal study notes, free to fork, star, and adapt for your own learning.
