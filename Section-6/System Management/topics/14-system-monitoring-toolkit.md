# 14. System Monitoring Toolkit

[← Back to index](../README.md)

A round-up of commands for diagnosing a server that's slow, out of space, or having network trouble — grouped into compute/storage and networking.

## Compute, storage & I/O

- **`top`** / **`free -h`** — live CPU and memory (see topics 2 and 8)
- **`df -h`** — disk space per filesystem
- **`iostat`** — CPU and per-device disk I/O stats; catches a disk bottleneck
- **`dmesg`** — kernel messages; the first place to look after plugging in new hardware

```bash
$ iostat
Device   tps   kB_read/s   kB_wrtn/s   kB_read   kB_wrtn
xvda     2.15      45.20       12.80    128400     36320
```

> 💡 `iostat` comes from the `sysstat` package. If it's missing:
> `sudo dnf install sysstat` (RHEL/Amazon) or `sudo apt install sysstat` (Ubuntu).

## Networking — old vs new

Older tutorials still show deprecated tools. Learn the **modern** `ip`/`ss` versions:

| Legacy (deprecated) | Modern (standard) | Purpose |
|---------------------|-------------------|---------|
| `ifconfig` | `ip addr` | Interface configuration and IP addresses |
| `route -n` | `ip route` | Routing table |
| `netstat -tulpn` | `ss -tulpn` | Listening ports and open sockets |

```bash
$ ip addr                    # show interfaces and IPs
$ ss -tulpn                  # show listening ports (who's accepting connections)
State   Recv-Q  Send-Q  Local Address:Port
LISTEN  0       128           0.0.0.0:22          # sshd
LISTEN  0       128           0.0.0.0:80          # web server
```

- **`ip addr`** — which network interfaces exist and their IP addresses
- **`ip route`** — how traffic leaves the machine (the routing table)
- **`ss -tulpn`** — every listening port and the program behind it (faster than the old `netstat`)

## 🧪 Practice

1. Find your machine's IP address and default route:
   ```bash
   ip addr
   ip route
   ```
2. List everything listening for connections:
   ```bash
   ss -tulpn
   ```
3. Check disk I/O (install `sysstat` first if needed):
   ```bash
   iostat
   ```

**✅ Success check:** You can identify your IP address, your open listening ports (e.g. 22 for SSH), and current disk activity.

---
[← Previous: System Maintenance & Recovery](13-system-maintenance-recovery.md) | [Next: Hands-On Labs →](15-hands-on-labs.md)
