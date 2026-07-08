# Part 3 — Package & Patch Management

*Topics 29–42.*

**In this part:** the RPM → YUM → DNF stack and when to drop to each layer, disciplined patch management (assess → snapshot → apply → reboot → verify), a complete production-ready automated patching script, building an air-gapped local repository, and the full `apt-get`/`dpkg` mapping for Ubuntu.

[⬅ Part 2](./02-remote-access-and-file-transfer.md) · [Index](./README.md)

---

## 29. System updates & repos — RPM

The RHEL package stack is three layers.

- **RPM** is the bottom layer: the low-level package format (`.rpm` files) and the `rpm` tool that installs a single file and records it in a local database. RPM does **not** resolve dependencies or fetch from the internet — if package A needs library B, `rpm` just fails and tells you.
- **YUM/DNF** sit on top and add dependency resolution + repositories. You use DNF daily; you drop to `rpm` to inspect or when you have a lone `.rpm` file.

| Layer | Tool | Does | Doesn't |
|---|---|---|---|
| Low | `rpm` | Install/query/verify a single package + local DB | Resolve deps, fetch from repos |
| High | `yum`/`dnf` | Resolve deps, download from repos, update all | Anything `rpm` can't already do |

### Anatomy of a package name

```
httpd-2.4.57-11.el9.x86_64.rpm
└─┬─┘ └──┬──┘ └┬┘ └┬┘ └──┬──┘
 name  version rel dist  arch
```

---

## 30. `rpm` — hands on

```bash
# QUERY (safe, read-only) — the -q family
rpm -qa                              # list ALL installed packages
rpm -qa | grep httpd                  # is httpd installed, which version?
rpm -qi httpd                          # detailed info (vendor, size, license, build)
rpm -ql httpd                           # list every FILE the package installed
rpm -qf /etc/httpd/conf/httpd.conf       # which package OWNS this file?
rpm -q --changelog httpd | head           # what changed / which CVEs fixed
```

```bash
# INSTALL / VERIFY / REMOVE
rpm -ivh package.rpm    # install (i), verbose (v), progress hashes (h)
rpm -Uvh package.rpm     # upgrade (or install if absent)
rpm -V httpd               # VERIFY: has any installed file been tampered with?
rpm -e httpd                 # erase (remove) — fails if deps depend on it
```

> [!TIP]
> **`rpm -qf` solves "where did this come from?"** Found a mystery binary or config file? `rpm -qf <path>` names the package that put it there — or tells you it's unmanaged (installed by hand). Essential during audits and OS-hardening reviews.

---

## 31. System updates & repos — YUM

**YUM** (Yellowdog Updater Modified) was the classic dependency-resolving package manager for RHEL 7 and earlier. It reads **repositories** (repos) — remote package catalogues defined in `/etc/yum.repos.d/*.repo` — downloads what you ask for **plus every dependency**, and installs it all. On RHEL 8+ `yum` is now just a symlink to `dnf`, so the commands are identical; you'll still see `yum` everywhere in scripts and on RHEL 7 / Amazon Linux 2.

```ini
# /etc/yum.repos.d/myrepo.repo
[baseos]                                            # repo ID (unique)
name=Base OS Packages                                 # human label
baseurl=https://repo.example.com/baseos/               # where packages live
enabled=1                                                # 1=use it, 0=defined but off
gpgcheck=1                                                # verify package signatures (keep = 1)
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY
```

---

## 32. `yum` — hands on

```bash
yum repolist                # which repos are active
yum search nginx             # find a package by keyword
yum info nginx                 # details before installing
yum install -y nginx            # install + all deps (-y = assume yes)
yum update                       # update EVERYTHING to latest
yum update httpd                  # update just one package
yum remove httpd                   # uninstall
yum history                          # audit log of every transaction
yum history undo 42                   # ROLL BACK transaction #42 — a lifesaver
```

> [!TIP]
> **`yum history undo` is your safety net:** Every install/update/remove is a numbered transaction. `yum history` lists them; `yum history undo <id>` reverses one. If an update breaks a service, you can roll the exact change back instead of guessing.

---

## 33. System updates & repos — DNF

**DNF** (Dandified YUM) is the modern default on RHEL 8+, Fedora, Rocky, Alma and **Amazon Linux 2023**. It's a rewrite of YUM with a better dependency solver, cleaner API, lower memory use and **modules/streams** for choosing package versions. The command syntax is deliberately identical to `yum` so muscle memory transfers directly.

### What DNF adds over YUM

- **Faster, more reliable dependency solving** (libsolv engine) — fewer "no package available" dead ends.
- **Modules & streams** — e.g. install Node.js 18 vs 20 from the same repo: `dnf module install nodejs:20`.
- **Better rollback and cleaner Python API** for automation (`dnf` is scriptable via Ansible's `dnf` module).
- **`dnf-automatic`** — a built-in service for unattended security updates.

---

## 34. `dnf` — hands on

```bash
dnf check-update             # list available updates WITHOUT installing
dnf install -y nginx           # install
dnf upgrade                     # update all (dnf's word for "yum update")
dnf upgrade --security            # ONLY security-relevant updates
dnf autoremove                     # clean up orphaned dependencies
dnf list installed                   # everything installed
dnf provides */htpasswd               # which package provides this command/file?
dnf history                             # transaction log (same as yum)
dnf module list nodejs                   # available Node.js streams
```

> [!NOTE]
> **`dnf provides` beats guessing:** You know you need the `dig` command but not the package name? `dnf provides '*/dig'` tells you it's in `bind-utils`. Works for any file or binary path.

---

## 35. DNF vs YUM

| Aspect | YUM (legacy) | DNF (modern) |
|---|---|---|
| Default on | RHEL ≤7, Amazon Linux 2 | RHEL ≥8, AL2023, Fedora, Rocky/Alma |
| Dependency solver | Older, occasionally wrong | libsolv — fast & correct |
| Memory/performance | Heavier | Lighter, faster |
| Modules/streams | No | Yes |
| Command syntax | `yum install` | `dnf install` (identical) |
| On RHEL 8+ | `yum` → symlink to `dnf` | The real tool |

> [!TIP]
> **Practical takeaway:** Write **`dnf`** in new scripts. But since `yum` is a symlink to `dnf` on RHEL 8+, old `yum` scripts keep working unchanged — you don't have to rewrite anything. Know both because Amazon Linux 2 (still very common) uses `yum` while AL2023 uses `dnf`.

---

## 36. System upgrade & patch management

Patch management is applying updates in a **controlled, reversible, verifiable** way — the difference between `dnf update -y` on a lab box and running fleet-wide patching in production. The discipline is a lifecycle, not a command.

### The patch lifecycle

1. **Inventory** — know what's installed and what the current kernel is (`uname -r`, `rpm -qa`).
2. **Assess** — list available updates, especially security ones (`dnf check-update`, `dnf updateinfo list security`).
3. **Snapshot / backup** — EBS snapshot, VM snapshot, or at least record the transaction so you can roll back.
4. **Apply** — patch in a maintenance window, ideally security-only first.
5. **Reboot if needed** — kernel/glibc/systemd updates require a reboot to take effect.
6. **Verify** — services back up, versions correct, no failed units (`systemctl --failed`).

> [!NOTE]
> **`needs-restarting`:** `needs-restarting -r` (from `dnf-utils`) tells you whether a **reboot is required** (kernel/core-lib change). `needs-restarting -s` lists individual **services** to restart. This lets you avoid unnecessary reboots while never missing a required one.

```bash
# Security-only patching — patch the risk, not everything
dnf updateinfo list security                # what security fixes are pending
dnf updateinfo info security                 # CVE details for each
dnf upgrade --security -y                     # apply ONLY security updates
dnf upgrade --advisory=RHSA-2024:1234          # apply one specific advisory
```

---

## 37. Patch management — hands on (manual run)

```bash
# 1. Baseline BEFORE
uname -r                             # current kernel
rpm -qa > /root/pkgs-before.txt        # package snapshot

# 2. See what's coming
dnf check-update
dnf updateinfo summary                 # counts by severity

# 3. Apply (security first, then full in a window)
dnf upgrade --security -y

# 4. Do you need a reboot?
needs-restarting -r ; echo "exit=$?"     # exit 1 => reboot required

# 5. After reboot, verify
systemctl --failed                        # any units that didn't come back?
uname -r                                    # new kernel active?
```

> [!WARNING]
> **The kernel-reboot trap:** `dnf update` installs a new kernel but the **old one keeps running until you reboot**. `uname -r` will still show the old version. Teams get burned believing they're patched against a kernel CVE when they haven't rebooted. Always check `needs-restarting -r`.

---

## 38. Complete automated patching script

This is the whole lifecycle wrapped in one idempotent, logged, safe-by-default Bash script. It mirrors the professional **precheck → patch → kernel-detect → reboot → postcheck** pattern: it snapshots state, patches (security-only by default), decides whether a reboot is genuinely needed, and writes an audit log. Drop it into cron or trigger it from Ansible/SSM.

```bash
#!/usr/bin/env bash
# auto-patch.sh — safe, logged patch run for RHEL-family hosts
# Usage: ./auto-patch.sh [--full] [--reboot]
set -euo pipefail

MODE="--security"    # default: security updates only
DO_REBOOT=0

for a in "$@"; do
  [[ "$a" == "--full" ]] && MODE=""
  [[ "$a" == "--reboot" ]] && DO_REBOOT=1
done

LOG="/var/log/auto-patch-$(date +%F_%H%M%S).log"
exec > >(tee -a "$LOG") 2>&1    # everything to console AND log

echo "=== Patch run $(date) on $(hostname) mode=${MODE:-full} ==="

# Pick dnf or yum automatically (AL2 vs AL2023/RHEL8+)
PM=$(command -v dnf || command -v yum)
echo "Package manager: $PM"

# --- 1. PRECHECK: snapshot state ---
K_BEFORE=$(uname -r)
$PM -q list installed > "/root/pkgs-before-$(date +%F).txt" || true
echo "Kernel before: $K_BEFORE"

# --- 2. ASSESS ---
if ! $PM check-update >/dev/null 2>&1; then
  RC=$?
  [[ $RC -eq 100 ]] && echo "Updates are available." || { echo "check-update failed rc=$RC"; exit 1; }
else
  echo "System already up to date. Nothing to do."; exit 0
fi

# --- 3. PATCH ---
echo ">> Applying updates ($MODE)..."
$PM upgrade $MODE -y

# --- 4. KERNEL / REBOOT DETECT ---
REBOOT_NEEDED=0
if command -v needs-restarting >/dev/null 2>&1; then
  needs-restarting -r || REBOOT_NEEDED=1
else
  # Fallback: compare running kernel to newest installed kernel
  K_LATEST=$(rpm -q --last kernel | head -1 | awk '{print $1}' | sed 's/kernel-//')
  [[ "$K_BEFORE" != "$K_LATEST" ]] && REBOOT_NEEDED=1
fi
echo "Reboot needed: $REBOOT_NEEDED"

# --- 5. POSTCHECK ---
FAILED=$(systemctl --failed --no-legend | wc -l)
echo "Failed units after patch: $FAILED"
[[ $FAILED -gt 0 ]] && systemctl --failed --no-legend

# --- 6. REBOOT (only if needed AND authorised) ---
if [[ $REBOOT_NEEDED -eq 1 ]]; then
  if [[ $DO_REBOOT -eq 1 ]]; then
    echo "Rebooting in 1 minute..."; shutdown -r +1 "Post-patch reboot"
  else
    echo "REBOOT REQUIRED but --reboot not passed. Schedule a window."
    exit 2    # non-zero so orchestrator knows a reboot is pending
  fi
fi

echo "=== Patch run complete. Log: $LOG ==="
```

> [!TIP]
> **Wiring it into your stack:** Trigger this via **AWS SSM Run Command / Patch Manager**, Ansible's `dnf`/`command` modules, or a Jenkins job across a fleet. Return codes are the contract: `0` = clean, `2` = reboot pending (let the orchestrator schedule the window), `1` = failure (alert). This is the same precheck/patch/reboot/postcheck bundle pattern used in enterprise patch workflows.

---

## 39. Local repository

A **local repo** is your own package mirror — a directory or web server holding `.rpm` files with an index that `dnf`/`yum` can consume. You build one when servers **can't reach the internet** (air-gapped, PCI/pharma-regulated networks), to **guarantee identical versions** across a fleet, or simply to **patch faster** without hitting external mirrors every time.

### Why teams run local repos

- **Air-gapped / restricted networks** — no direct internet, common in regulated pharma/finance.
- **Version consistency** — every host installs the exact same, approved package set (no drift).
- **Speed & bandwidth** — one download from upstream, many installs from the LAN.
- **Change control** — you decide when new packages enter the mirror.

---

## 40. Creating a local repository — hands on

```bash
# 1. Tooling + a directory to hold packages
dnf install -y createrepo_c httpd
mkdir -p /var/www/html/localrepo

# 2. Put .rpm files there (copy from media, or mirror from upstream)
cp /mnt/dvd/BaseOS/Packages/*.rpm /var/www/html/localrepo/
# or mirror an online repo:
# reposync --repoid=baseos -p /var/www/html/localrepo/

# 3. Generate the repo metadata (the repodata/ index)
createrepo_c /var/www/html/localrepo/

# 4. Serve it over HTTP
systemctl enable --now httpd
firewall-cmd --permanent --add-service=http && firewall-cmd --reload
```

```bash
# 5. On EACH CLIENT, point dnf at it
cat >/etc/yum.repos.d/local.repo <<'EOF'
[local]
name=Local Mirror
baseurl=http://repo-server.lab/localrepo/
enabled=1
gpgcheck=0
EOF

dnf clean all && dnf repolist       # confirm 'local' appears
dnf install -y somepackage           # now served from your LAN
```

> [!WARNING]
> **Re-run `createrepo` after adding packages:** The metadata is a snapshot. Every time you add or remove `.rpm` files you must re-run `createrepo_c` (or `createrepo_c --update`) or clients won't see the change. Automate this in the same job that syncs packages.

---

## 41. Advanced package management — `apt-get` (Debian/Ubuntu)

Ubuntu and Debian use a **different** stack: **`.deb`** packages managed by **`dpkg`** (the low-level tool, like `rpm`) with **`apt`/`apt-get`** on top (the dependency-resolving tool, like `dnf`). You need this whenever you touch Ubuntu servers, Docker base images, or CI runners. The concepts map one-to-one onto what you already know from RHEL.

| Job | RHEL (`dnf`) | Debian/Ubuntu (`apt`) |
|---|---|---|
| Low-level single package | `rpm` | `dpkg` |
| Refresh repo metadata | (automatic) | `apt update` (**required first**) |
| Install | `dnf install nginx` | `apt install nginx` |
| Upgrade all | `dnf upgrade` | `apt upgrade` |
| Remove | `dnf remove nginx` | `apt remove nginx` |
| Remove + config | `dnf remove` | `apt purge nginx` |
| Search | `dnf search` | `apt search` |
| Which pkg owns a file | `rpm -qf` | `dpkg -S` |
| List files in a pkg | `rpm -ql` | `dpkg -L` |
| Repo files live in | `/etc/yum.repos.d/` | `/etc/apt/sources.list(.d)` |

> [!NOTE]
> **`apt` vs `apt-get`:** `apt` is the newer, friendlier front-end (progress bars, colour) meant for interactive use. `apt-get` is the older, stable interface with guaranteed script-safe behaviour — **use `apt-get` in scripts/Dockerfiles**, `apt` at the keyboard.

---

## 42. `apt-get` — hands on

```bash
# The golden rule: ALWAYS update the index before installing
sudo apt-get update              # refresh package lists from repos
sudo apt-get install -y nginx      # install
sudo apt-get upgrade -y              # upgrade all installed packages
sudo apt-get dist-upgrade              # upgrade + handle changed deps/kernels
sudo apt-get remove nginx                # remove binaries
sudo apt-get purge nginx                   # remove binaries + config files
sudo apt-get autoremove                      # drop orphaned dependencies
```

```bash
# dpkg for low-level work
dpkg -l | grep nginx           # is it installed?
dpkg -L nginx                    # list its files
dpkg -S /usr/sbin/nginx           # which package owns this file?
dpkg -i package.deb                # install a lone .deb (no dep resolution)
sudo apt-get install -f              # ...then fix any missing deps
```

> [!WARNING]
> **Forgetting `apt-get update`:** Unlike `dnf`, `apt` caches the package index locally and does **not** refresh it automatically. Skip `apt-get update` and you'll install stale versions or get "404 Not Found" on packages that moved. In Dockerfiles, always chain `apt-get update && apt-get install -y ...` in the **same `RUN` layer**.

---

[⬅ Part 2](./02-remote-access-and-file-transfer.md) · [Index](./README.md) · [Next: Part 4 — Network Services: DNS, NTP, Web & Proxy ➡](./04-network-services-dns-ntp-web-proxy.md)
