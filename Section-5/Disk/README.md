# 🐧 Linux System Administration — From Zero to Production

> A hands-on, beginner-friendly guide to the Linux concepts every DevOps / SRE / SysAdmin actually uses.
> Every topic has **plain-English explanations**, **diagrams**, and **copy-paste practicals**.

<p align="center">
  <img alt="Linux" src="https://img.shields.io/badge/Linux-RHEL%20%7C%20Amazon%20Linux-EE0000?logo=linux&logoColor=white">
  <img alt="Level" src="https://img.shields.io/badge/Level-Beginner%20→%20Advanced-2E74B5">
  <img alt="Format" src="https://img.shields.io/badge/Style-Hands--on%20Labs-3C8C5C">
  <img alt="License" src="https://img.shields.io/badge/License-MIT-lightgrey">
</p>

---

## 📖 What is this?

This repository teaches Linux the way it's used on real servers and in the cloud — the **boot process**, **storage & LVM**, **RAID**, **filesystems**, **NFS**, **databases**, and the **LAMP stack**.

It's written so that **anyone can follow along**, even without a Linux background. Each lesson answers three questions:

1. **What is it?** — in plain language, with an analogy.
2. **Why does it matter?** — where you'll actually use it.
3. **How do I do it?** — real commands you can run, with expected output.

> [!NOTE]
> All examples target the **RHEL family** (RHEL, Rocky, AlmaLinux, CentOS Stream, and **Amazon Linux 2 / 2023**) which use `systemd`, `dnf`/`yum`, and `XFS`. Where a command differs on **Debian/Ubuntu**, it's clearly marked.

---

## 🗺️ Learning Roadmap

Follow the topics in order for a smooth ramp, or jump straight to what you need.

### Part 1 — Boot & Startup
| # | Topic | What you'll learn |
|---|-------|-------------------|
| 01 | [Run Levels & systemd Targets](topics/01-run-levels.md) | Machine states 0–6 and their modern replacements |
| 02 | [The Boot Process](topics/02-boot-process.md) | Every stage from power-on to login prompt |
| 03 | [systemd-analyze](topics/03-systemd-analyze.md) | Measure and speed up boot time |
| 04 | [MOTD & Login Banners](topics/04-motd.md) | Customise what users see at login |

### Part 2 — Storage, LVM, RAID & Filesystems
| # | Topic | What you'll learn |
|---|-------|-------------------|
| 05 | [Linux Storage Basics](topics/05-linux-storage.md) | How disks are named and inspected |
| 06 | [Disk Partitioning](topics/06-disk-partitioning.md) | `fdisk`, `parted`, MBR vs GPT |
| 07 | [The `df` Command](topics/07-df-command.md) | Check disk usage the right way |
| 08 | [LVM — Logical Volume Manager](topics/08-lvm.md) | Resize storage with **zero downtime** |
| 09 | [Swap Space](topics/09-swap-space.md) | Add memory overflow safely |
| 10 | [XFS & `xfs_info`](topics/10-xfs-filesystem.md) | Grow and repair the default RHEL filesystem |
| 11 | [RAID](topics/11-raid.md) | Combine disks for speed and redundancy |
| 12 | [Filesystem Check (`fsck`)](topics/12-filesystem-check-fsck.md) | Detect and repair corruption |
| 13 | [Backups, `dd` & `lsblk`](topics/13-backup-dd-lsblk.md) | Clone disks and image drives |

### Part 3 — Network Storage, Databases & Distros
| # | Topic | What you'll learn |
|---|-------|-------------------|
| 14 | [NFS — Network File System](topics/14-nfs.md) | Share a folder across the network |
| 15 | [SATA & SAS](topics/15-sata-sas.md) | The disk interfaces underneath it all |
| 16 | [MySQL / MariaDB](topics/16-mysql-mariadb.md) | Install and run a database server |
| 17 | [LAMP Stack](topics/17-lamp-stack.md) | Build a full web application stack |
| 18 | [Linux Distributions](topics/18-linux-distributions.md) | Choose the right distro and translate commands |

---

## 🚀 How to use this repo

- **Reading only?** Click any topic above — it renders beautifully on GitHub.
- **Practising?** Spin up a throwaway VM or EC2 instance (Amazon Linux 2023 is free-tier friendly) and run the commands as you read.
- **Interviewing?** Each file ends with **key takeaways** and common **interview questions**.

> [!TIP]
> Practise on a **disposable machine**, not your daily driver. Storage and boot commands can wipe data if mistyped. A `t3.micro` EC2 instance or a local VM (VirtualBox / Multipass) is perfect.

---

## 🧭 Legend used throughout

| Symbol | Meaning |
|--------|---------|
| `command` | Something you type in the terminal |
| 💡 **TIP** | A shortcut or best practice |
| ⚠️ **WARNING** | Something that can break things — read carefully |
| 🧪 **Hands-on** | A step-by-step lab to try yourself |

---

## 📂 Repository structure

```
linux-mastery/
├── README.md              ← you are here
└── topics/
    ├── 01-run-levels.md
    ├── 02-boot-process.md
    ├── 03-systemd-analyze.md
    ├── 04-motd.md
    ├── 05-linux-storage.md
    ├── 06-disk-partitioning.md
    ├── 07-df-command.md
    ├── 08-lvm.md
    ├── 09-swap-space.md
    ├── 10-xfs-filesystem.md
    ├── 11-raid.md
    ├── 12-filesystem-check-fsck.md
    ├── 13-backup-dd-lsblk.md
    ├── 14-nfs.md
    ├── 15-sata-sas.md
    ├── 16-mysql-mariadb.md
    ├── 17-lamp-stack.md
    └── 18-linux-distributions.md
```

---

## 🤝 Contributing & License

Found a typo or want to add an example? PRs welcome.
Released under the **MIT License** — free to use, share, and learn from.

---

<p align="center"><i>⭐ If this helped you, consider starring the repo so others can find it too.</i></p>
