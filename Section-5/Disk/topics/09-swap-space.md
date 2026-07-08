# 09 · Swap Space

[⬅ Previous: LVM](08-lvm.md) · [Back to index](../README.md) · [Next: XFS Filesystem ➡](10-xfs-filesystem.md)

---

## 🎯 What is swap?

**Swap** is disk space the kernel uses as **overflow when RAM is full**. When physical memory runs low, inactive memory pages are moved ("swapped") to disk to free up RAM.

> 🎒 **Analogy:** RAM is your desk — fast, but small. Swap is the drawer under the desk. When the desk is cluttered, you move papers you're not using right now into the drawer. It's slower to reach, but it stops the desk from overflowing.

**Two reasons swap matters:**
1. **Safety cushion** — prevents the "Out Of Memory killer" from killing your processes under a memory spike.
2. **Hibernation** — the machine saves RAM to swap when it sleeps.

> [!NOTE]
> Swap is **much slower than RAM**. It's a cushion, not a substitute for having enough memory. If a server swaps constantly, the fix is *more RAM*, not more swap.

---

## 🧪 Hands-on — inspect current swap

```bash
free -h              # shows Mem and Swap totals
swapon --show        # active swap devices/files
cat /proc/swaps
```

### `swappiness` — how eagerly the kernel swaps

A value from **0–100**. Lower = keep pages in RAM longer (good for databases).

```bash
cat /proc/sys/vm/swappiness      # often 30–60 by default

# Change it now:
sudo sysctl vm.swappiness=10

# Make it permanent:
echo 'vm.swappiness=10' | sudo tee /etc/sysctl.d/99-swap.conf
```

---

## 🧪 Hands-on — add swap as a FILE (most common on cloud)

A swap **file** is the easiest way to add swap — no spare partition needed.

```bash
# 1. Create a 2G file (fallocate is instant)
sudo fallocate -l 2G /swapfile
#    fallback if fallocate is unavailable:
#    sudo dd if=/dev/zero of=/swapfile bs=1M count=2048

# 2. Lock down permissions (root-only, or swapon refuses it)
sudo chmod 600 /swapfile

# 3. Format it as swap and turn it on
sudo mkswap /swapfile
sudo swapon /swapfile
swapon --show          # confirm it's active

# 4. Make it permanent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

> [!WARNING]
> Step 2 is not optional. If `/swapfile` is world-readable, `swapon` will **refuse** it (anyone could read memory contents from disk). Permissions **must** be `600`.

### Turn swap off (e.g. to remove it)

```bash
sudo swapoff /swapfile
sudo rm /swapfile               # then remove the fstab line too
```

---

## 🧪 Alternative — swap as a partition or LVM volume

```bash
sudo mkswap /dev/datavg/swaplv
sudo swapon /dev/datavg/swaplv
```

---

## 📏 How much swap?

A rough modern guideline (there's no single rule):

| RAM | Suggested swap |
|-----|----------------|
| ≤ 2 GB | 2× RAM |
| 2–8 GB | = RAM |
| 8–64 GB | 4–8 GB (or half RAM) |
| > 64 GB | Often minimal; depends on workload |

> Many cloud/container hosts run with **little or no swap** and rely on right-sized RAM instead. Databases often set low `swappiness` to avoid latency spikes.

---

## ✅ Key takeaways

- Swap = disk-based overflow for RAM; a safety cushion, not a RAM replacement.
- Add a swap file: `fallocate` → `chmod 600` → `mkswap` → `swapon` → add to `/etc/fstab`.
- `swappiness` (0–100) controls how eagerly the kernel swaps; lower it for DB servers.
- `free -h` and `swapon --show` inspect swap.

## 💬 Interview questions

1. *What is swap and when is it used?* → disk overflow when RAM is exhausted; also hibernation.
2. *How do you add swap without a spare partition?* → create a swap file (`fallocate`/`mkswap`/`swapon`).
3. *What does swappiness do?* → tunes how aggressively the kernel moves pages to swap.

---

[⬅ Previous: LVM](08-lvm.md) · [Back to index](../README.md) · [Next: XFS Filesystem ➡](10-xfs-filesystem.md)
