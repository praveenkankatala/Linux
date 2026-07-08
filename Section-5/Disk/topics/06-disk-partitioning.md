# 06 · Disk Partitioning

[⬅ Previous: Linux Storage](05-linux-storage.md) · [Back to index](../README.md) · [Next: The df Command ➡](07-df-command.md)

---

## 🎯 What is a partition?

A **partition** is a *logical slice* of a physical disk. It sets the boundaries inside which a filesystem (or an LVM/RAID member) lives.

> 🍕 **Analogy:** A disk is a whole pizza. Partitioning is slicing it — each slice can have different toppings (filesystems) and be given to different people (mount points). You can have one big slice or several smaller ones.

---

## 🆚 MBR vs GPT — two partition schemes

| | **MBR** (msdos) | **GPT** |
|---|---|---|
| Max disk size | **2 TB** | 9.4 ZB (basically unlimited) |
| Max partitions | 4 primary (or 3 + extended) | 128 by default |
| Firmware | BIOS | UEFI (also BIOS-compatible) |
| Tooling | `fdisk` | `gdisk` / `parted` |

> [!TIP]
> Use **GPT** for anything new — especially disks **larger than 2 TB**. MBR is legacy but still common on small boot disks.

---

## 🧪 Hands-on

We'll partition a fresh 10 GB disk `/dev/xvdf`. **Confirm it's the right disk with `lsblk` first!**

### Option A — `fdisk` (MBR, interactive)

```bash
sudo fdisk /dev/xvdf
```
Then, inside fdisk:
```text
Command (m for help): n       ← new partition
Partition type: p             ← primary
Partition number (1-4): 1
First sector: <Enter>          ← accept default
Last sector: <Enter>           ← use whole disk (or +5G for 5 GB)
Command (m for help): p        ← print to review
Command (m for help): w        ← WRITE and exit
```

Tell the kernel to re-read the new table:
```bash
sudo partprobe /dev/xvdf
lsblk /dev/xvdf
#   xvdf
#   └─xvdf1
```

### Option B — `parted` (GPT, scriptable, for disks > 2 TB)

```bash
sudo parted /dev/xvdf --script mklabel gpt
sudo parted /dev/xvdf --script mkpart primary xfs 0% 100%
sudo parted /dev/xvdf --script print
```

---

## 🗂️ Put a filesystem on it and mount it

```bash
# Create an XFS filesystem (the RHEL default)
sudo mkfs.xfs /dev/xvdf1
#   ext4 alternative: sudo mkfs.ext4 /dev/xvdf1

# Mount it now (temporary — gone after reboot)
sudo mkdir -p /data
sudo mount /dev/xvdf1 /data
df -h /data
```

### Make the mount **permanent** with `/etc/fstab`

Always use the **UUID**, because device names can change:

```bash
# 1. Get the UUID
sudo blkid /dev/xvdf1
#   /dev/xvdf1: UUID="a1b2c3d4-..." TYPE="xfs"

# 2. Add a line to /etc/fstab:
#    <UUID>              <mount>  <fs>  <options>       <dump> <pass>
UUID=a1b2c3d4-...   /data    xfs    defaults        0      0

# 3. TEST it before trusting a reboot:
sudo mount -a          # mounts everything in fstab; errors show up here
```

> [!WARNING]
> **A typo in `/etc/fstab` can stop the machine from booting.** Always run `sudo mount -a` after editing and fix any error *before* you reboot. On cloud servers, add the `nofail` option to non-root mounts so a missing volume doesn't block boot:
> ```
> UUID=... /data xfs defaults,nofail 0 0
> ```

---

## 📖 Understanding the fstab fields

```
UUID=a1b2c3d4-...   /data   xfs   defaults   0   0
     └─ device       └─ mount └─fs └─options  │   └─ fsck order (0=skip, 1=root, 2=others)
                                              └─ dump backup flag (usually 0)
```

---

## ✅ Key takeaways

- A partition is a slice of a disk; **MBR** (≤2 TB, 4 primary) vs **GPT** (huge, 128 parts).
- `fdisk` for MBR, `parted`/`gdisk` for GPT; run `partprobe` after changing the table.
- `mkfs.xfs` / `mkfs.ext4` creates the filesystem; `mount` attaches it.
- Persist mounts in `/etc/fstab` **by UUID**, add `nofail` on cloud, and **always `mount -a` to test**.

## 💬 Interview questions

1. *MBR vs GPT?* → MBR limited to 2 TB / 4 primary partitions; GPT supports huge disks and 128 partitions.
2. *Why use UUID in fstab?* → device names can change; UUIDs are stable.
3. *A bad fstab entry — what happens and how do you recover?* → boot may fail; boot to rescue, fix the line, `mount -a`.

---

[⬅ Previous: Linux Storage](05-linux-storage.md) · [Back to index](../README.md) · [Next: The df Command ➡](07-df-command.md)
