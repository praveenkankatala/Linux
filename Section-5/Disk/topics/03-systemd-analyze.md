# 03 · systemd-analyze — Profiling Boot Time

[⬅ Previous: Boot Process](02-boot-process.md) · [Back to index](../README.md) · [Next: MOTD ➡](04-motd.md)

---

## 🎯 What is it?

`systemd-analyze` is a **built-in stopwatch and profiler for boot**. It tells you *how long* the machine took to boot and *which service* was slow.

> ⏱️ **Analogy:** It's the lap-timer for your boot "relay race" (from the previous topic). It shows the total time and the time each runner (service) took — so you know exactly where to speed things up.

**When you'll use it:** "This AMI / instance takes too long to become ready" — this is your first stop.

---

## 🧰 The core commands

| Command | What it tells you |
|---------|-------------------|
| `systemd-analyze time` | Total boot time (firmware + loader + kernel + userspace) |
| `systemd-analyze blame` | Every service, sorted by how long it took |
| `systemd-analyze critical-chain` | The *time-critical path* that actually delayed boot |
| `systemd-analyze plot > boot.svg` | A visual timeline of the whole boot |
| `systemd-analyze dump` | Full internal state (verbose) |
| `systemd-analyze verify <unit>` | Check a unit file for errors |

---

## 🧪 Hands-on

### 1. How long did we boot?

```bash
systemd-analyze time
```
```text
Startup finished in 1.153s (kernel) + 3.482s (initrd) + 12.918s (userspace) = 17.554s
multi-user.target reached after 12.874s in userspace
```
Read it as: kernel was fast, initramfs OK, and **userspace (services) took 12.9s** — that's where to look.

### 2. Which services were slow?

```bash
systemd-analyze blame
```
```text
6.431s  cloud-init.service
3.882s  NetworkManager-wait-online.service
1.204s  tuned.service
0.913s  sshd.service
0.402s  amazon-ssm-agent.service
```

### 3. What was *actually* on the critical path?

```bash
systemd-analyze critical-chain
```
```text
multi-user.target @12.874s
└─cloud-init.service @6.44s +6.43s
  └─network-online.target @6.43s
    └─NetworkManager-wait-online.service @2.55s +3.88s
```

> [!IMPORTANT]
> **`blame` vs `critical-chain` — this trips people up:**
> - `blame` lists slow services **even if they ran in parallel** and didn't hold anything up.
> - `critical-chain` shows only the **serial path** that decided the finish time.
>
> Always optimise the **critical-chain**, not just the biggest number in `blame`.

### 4. Generate a visual timeline

```bash
systemd-analyze plot > boot.svg
# Copy boot.svg to your laptop and open in a browser —
# each service is a bar on a timeline. Great for spotting bottlenecks.
```

---

## 🔧 A real-world fix

A frequent culprit on cloud VMs is **`NetworkManager-wait-online.service`**, which blocks boot until the network is fully up. If nothing early needs the network, you can stop waiting for it:

```bash
sudo systemctl disable NetworkManager-wait-online.service
```

> [!WARNING]
> Only disable this if you're sure no early-boot service depends on the network being ready. Otherwise those services may fail intermittently.

---

## ✅ Key takeaways

- `systemd-analyze time` = total boot time, split by phase.
- `blame` = slowest services (may be parallel).
- `critical-chain` = the path that *actually* set the boot time — optimise this.
- `plot > boot.svg` = a visual timeline.

## 💬 Interview questions

1. *How do you find why a server boots slowly?* → `systemd-analyze time`, then `blame` and `critical-chain`.
2. *Difference between blame and critical-chain?* → blame lists all slow units; critical-chain is the serialized path that determined finish time.

---

[⬅ Previous: Boot Process](02-boot-process.md) · [Back to index](../README.md) · [Next: MOTD ➡](04-motd.md)
