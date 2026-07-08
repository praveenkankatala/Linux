# 6. Package Management (`apt` & `dnf`/`yum`)

[← Back to index](../README.md)

Instead of downloading `.exe` files from websites, Linux installs software from **repositories** — trusted servers run by your distribution that host thousands of verified programs. The package manager also handles **dependencies** automatically: if a program needs another library to run, it downloads that too.

Being fluent in **both** families is what separates a beginner from a professional:

- **Debian / Ubuntu** use **`apt`**
- **RHEL / CentOS / Amazon Linux** use **`dnf`** (or older `yum`)

## Command comparison

| Action | Debian / Ubuntu (`apt`) | RHEL / Amazon (`dnf` or `yum`) |
|--------|--------------------------|-------------------------------|
| Refresh package index | `sudo apt update` | *(done automatically on install)* |
| Upgrade all packages | `sudo apt upgrade` | `sudo dnf upgrade` |
| Install a package | `sudo apt install <pkg>` | `sudo dnf install <pkg>` |
| Remove a package | `sudo apt remove <pkg>` | `sudo dnf remove <pkg>` |
| Search repositories | `apt search <term>` | `dnf search <term>` |
| Show package info | `apt show <pkg>` | `dnf info <pkg>` |

## Key ideas

- **`update` vs `upgrade` (apt):** `update` refreshes the *list* of what's available; `upgrade` actually downloads and installs the newer versions. Common to chain them: `sudo apt update && sudo apt upgrade`.
- **`remove` vs `purge` (apt):** `remove` uninstalls but keeps config files; `purge` deletes the config files too, for a clean slate.
- **Clean up leftovers:** `sudo apt autoremove` sweeps away unused dependencies left behind by removed packages.

> ⚠️ **Watch the family!** Amazon Linux and RHEL do **not** understand `apt`. To install the `tree` utility on an Amazon Linux EC2 instance, the answer is `sudo dnf install tree` — **not** `sudo apt install tree`.

## 🧪 Practice

1. Search for a small utility:
   ```bash
   # RHEL/Amazon:
   dnf search tree
   # Ubuntu:
   apt search tree
   ```
2. Install it, confirm it works, then remove it:
   ```bash
   sudo dnf install -y tree      # (or: sudo apt install -y tree)
   tree --version
   sudo dnf remove -y tree       # (or: sudo apt remove -y tree)
   ```

**✅ Success check:** `tree --version` prints a version number while installed, and "command not found" after removal.

---
[← Previous: Text Processing with sed](05-text-processing-sed.md) | [Next: User & Group Management →](07-user-group-management.md)
