# 8. System Information & Hardware

[← Back to index](../README.md)

When you log into a new or misbehaving server, these read-only commands tell you exactly what hardware and OS you're working with. They're safe to run anywhere.

## CPU & kernel

- **`lscpu`** — clean summary of CPU architecture, cores, and threads
- **`cat /proc/cpuinfo`** — raw, per-core detail
- **`uname -a`** — kernel version, hostname, and architecture (`x86_64` / `aarch64`)

## Memory (RAM)

- **`free -h`** — RAM and swap in human-readable units (the `-h` is essential — turns bytes into MB/GB)
- **`cat /proc/meminfo`** — deep breakdown of memory, swap, and buffers

## Storage

- **`df -h`** — free space on every mounted filesystem
- **`lsblk`** — disks and partitions shown as a tree

## Hardware inventory

- **`sudo lshw`** — full hardware summary (motherboard, RAM slots, CPU)
- **`sudo dmidecode -t system`** — serial number, manufacturer, BIOS version

## Fast troubleshooting reference

| To find out… | Run |
|--------------|-----|
| How much RAM is left? | `free -h` |
| Is the disk full? | `df -h` |
| Which OS / kernel? | `uname -a` or `cat /etc/os-release` |
| How many CPU cores? | `lscpu` |

```bash
$ free -h
               total        used        free      shared  buff/cache   available
Mem:           7.6Gi       1.2Gi       4.8Gi        12Mi       1.6Gi       6.1Gi
Swap:             0B          0B          0B

$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1       20G  6.2G   14G  32% /
```

> 💡 **Virtual machines show less.** On EC2, Docker, and other virtual environments, `dmidecode` and `lshw` may show limited data because the "hardware" is virtual.

## 🧪 Practice

Answer these about your own machine using one command each:

1. How many CPU cores does it have? → `lscpu`
2. How much free RAM? → `free -h`
3. How full is the root disk? → `df -h /`
4. Which kernel version? → `uname -r`

**✅ Success check:** You can state your machine's cores, free RAM, disk usage %, and kernel version.

---
[← Previous: User & Group Management](07-user-group-management.md) | [Next: System Utilities →](09-system-utilities.md)
