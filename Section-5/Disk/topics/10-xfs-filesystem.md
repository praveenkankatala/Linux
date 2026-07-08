# 10 · XFS & the `xfs_info` Command

[⬅ Previous: Swap Space](09-swap-space.md) · [Back to index](../README.md) · [Next: RAID ➡](11-raid.md)

---

## 🎯 What is XFS?

**XFS** is the **default filesystem on the RHEL family** (RHEL, Rocky, Alma, Amazon Linux). It's a high-performance **journaling** filesystem built for large files and large volumes.

> 📓 **"Journaling" analogy:** Before making a change, XFS writes a quick note in a journal ("about to update file X"). If power is lost mid-write, on reboot it reads the journal and finishes or discards the half-done change — so the filesystem stays consistent. No lengthy full scan needed.

**Why you'll meet it constantly:** almost every RHEL/Amazon Linux volume you create with `mkfs.xfs` is XFS. It grows **online** and scales to petabytes.

---

## 🔍 `xfs_info` — read a filesystem's geometry

`xfs_info` shows the internal layout of an XFS filesystem. Run it against a **mount point** (or the device):

```bash
sudo xfs_info /app
```
```text
meta-data=/dev/mapper/datavg-lv01 isize=512  agcount=4, agsize=...
data     =                        bsize=4096  blocks=3932160, imaxpct=25
naming   =version 2               bsize=4096
log      =internal log            bsize=4096  blocks=2560
realtime =none                    extsz=4096  blocks=0
```

**The fields that matter:**

| Field | Meaning |
|-------|---------|
| `bsize=4096` | Block size = 4 KB |
| `blocks=3932160` | Total number of blocks (× bsize = total size) |
| `agcount=4` | Allocation groups — XFS splits the FS into groups it can write to **in parallel** (performance) |
| `log=internal` | The journal is stored inside the filesystem |

You'll use `xfs_info` to **confirm the size** before and after growing a volume.

---

## 🧰 Core XFS operations

| Task | Command |
|------|---------|
| Create XFS | `mkfs.xfs /dev/xvdf1` |
| Show geometry | `xfs_info /mountpoint` |
| **Grow** (online, by mount point) | `xfs_growfs /mountpoint` |
| **Repair** (must be **unmounted**) | `xfs_repair /dev/xvdf1` |
| Label | `xfs_admin -L mylabel /dev/xvdf1` |
| Defragment | `xfs_fsr /mountpoint` |

---

## 🧪 Hands-on — grow an XFS filesystem

This is the payoff from the [LVM topic](08-lvm.md): after enlarging the LV, grow XFS to fill it, **online**:

```bash
# After: sudo lvextend -L +5G /dev/datavg/lv01
sudo xfs_info /app            # note current 'blocks='
sudo xfs_growfs /app          # grow to fill the LV — no unmount, no downtime
sudo xfs_info /app            # 'blocks=' is now larger
df -h /app
```

> [!IMPORTANT]
> **XFS grows by MOUNT POINT, not device.** Use `xfs_growfs /app`, not `xfs_growfs /dev/...`. And remember from the LVM topic: **XFS can only grow, never shrink.**

---

## 🚑 Repairing XFS

```bash
# Must be UNMOUNTED first
sudo umount /dev/xvdf1
sudo xfs_repair /dev/xvdf1
#   -n  = dry run (report problems, change nothing)
#   -L  = last resort: zero the log if it's corrupt (can lose recent data)
```

> [!NOTE]
> **XFS has no `fsck`.** The command `fsck.xfs` exists but does **nothing** (returns success) — that's by design. The real repair tool is **`xfs_repair`**, and the device must be unmounted. See the [fsck topic](12-filesystem-check-fsck.md) for the full picture.

---

## ✅ Key takeaways

- XFS is the **default RHEL/Amazon Linux filesystem** — journaling, fast, scales huge.
- `xfs_info /mount` shows geometry (block size, total blocks, allocation groups).
- Grow it **online** with `xfs_growfs /mount` — by mount point, not device.
- Repair with `xfs_repair` (unmounted). **XFS never shrinks and has no fsck.**

## 💬 Interview questions

1. *Default filesystem on RHEL?* → XFS.
2. *How do you grow an XFS filesystem?* → `xfs_growfs /mountpoint`, online.
3. *How do you check/repair XFS?* → `xfs_repair` (unmounted); XFS has no working `fsck`.
4. *Can XFS shrink?* → No.

---

[⬅ Previous: Swap Space](09-swap-space.md) · [Back to index](../README.md) · [Next: RAID ➡](11-raid.md)
