# 15 · SATA & SAS — Disk Interfaces

[⬅ Previous: NFS](14-nfs.md) · [Back to index](../README.md) · [Next: MySQL / MariaDB ➡](16-mysql-mariadb.md)

---

## 🎯 What are SATA and SAS?

They're the **interfaces (the "plug and protocol") that connect a drive to the computer** — the physical layer underneath everything in the storage topics. When you spec a server or read a hardware inventory, these terms come up.

> 🛣️ **Analogy:** If data is traffic and the disk is a city, then SATA and SAS are the **roads** connecting them. SATA is a normal two-lane road (fine for most cars). SAS is a wider highway with better engineering (for heavy, constant enterprise traffic). NVMe is a dedicated expressway straight into the CPU.

---

## 📊 Comparison

| | **SATA** | **SAS** | **NVMe** |
|---|---|---|---|
| Full name | Serial ATA | Serial Attached SCSI | Non-Volatile Memory Express |
| Target market | Consumer / bulk storage | Enterprise / servers | High-performance SSD |
| Typical speed | 6 Gbps | 12–24 Gbps | PCIe — many GB/s |
| Reliability | Good | Higher (dual-port, better error handling) | High |
| Cost | Low | Higher | Higher |
| Linux device | `/dev/sdX` | `/dev/sdX` | `/dev/nvmeXn1` |

---

## 🔑 The key distinctions

- **SATA** — cheaper, single-port, great for capacity and desktops. It's the mainstream consumer interface.
- **SAS** — uses the **SCSI** command set, supports **dual porting** (two independent paths to the drive for redundancy), higher queue depths, and better error recovery. That's why it's the enterprise/server choice. **A SAS backplane can also run SATA drives, but not the reverse.**
- **NVMe** — skips the SATA/SAS controller entirely and talks over **PCIe** for the lowest latency. This is what most **cloud SSD volumes** use today.

---

## 🧪 Hands-on — inspect interfaces in Linux

```bash
# TRAN column shows the transport (sata / sas / nvme)
# ROTA: 1 = spinning disk (HDD), 0 = SSD
lsblk -o NAME,SIZE,ROTA,TYPE,TRAN,MODEL
```
```text
NAME   SIZE ROTA TYPE TRAN  MODEL
sda   500G    1 disk sata  ST500DM002
sdb   1.8T    1 disk sas   SEAGATE ST2000
nvme0n1 400G  0 disk nvme  Amazon EC2 NVMe
```

```bash
# Detailed hardware view
sudo lshw -class disk -short

# Drive health (SMART data) — needs smartmontools
sudo smartctl -a /dev/sda
```

---

## ✅ Key takeaways

- SATA / SAS / NVMe are **disk interfaces**, the layer beneath partitions and filesystems.
- **SATA** = consumer, cheap, single-port. **SAS** = enterprise, SCSI, dual-port, more reliable. **NVMe** = PCIe, fastest, common in the cloud.
- Inspect with `lsblk -o NAME,TRAN,ROTA` (transport + HDD/SSD) and `smartctl` for health.

## 💬 Interview questions

1. *SATA vs SAS?* → SATA is consumer/single-port; SAS is enterprise/dual-port with the SCSI command set and better reliability.
2. *What is NVMe and why is it faster?* → it talks directly over PCIe, bypassing the SATA/SAS controller.
3. *How do you tell if a disk is SSD or HDD in Linux?* → `lsblk -o NAME,ROTA` (ROTA 0 = SSD).

---

[⬅ Previous: NFS](14-nfs.md) · [Back to index](../README.md) · [Next: MySQL / MariaDB ➡](16-mysql-mariadb.md)
