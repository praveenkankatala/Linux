# 13. System Maintenance & Recovery

[← Back to index](../README.md)

How to safely change a machine's power state, understand runlevels, rename a host, and rescue a server when you're locked out.

## Powering down & rebooting

`shutdown` is the safest method — it warns logged-in users and lets processes save before power is cut.

```bash
sudo shutdown now                 # shut down immediately
sudo shutdown +10 "Maintenance"   # in 10 minutes, with a message to users
sudo shutdown -c                  # cancel a pending shutdown
sudo reboot                       # safe restart (= shutdown -r now)
sudo halt                         # stop all processing (power may stay on)
```

## Runlevels (`init`) & systemd targets

**Runlevels** describe what state the system runs in. Older systems used numeric `init` levels; systemd uses named **targets**, but the numbers still work as shortcuts.

| Command | systemd target | Resulting state |
|---------|----------------|-----------------|
| `sudo init 0` | `poweroff.target` | Shut down and power off |
| `sudo init 1` | `rescue.target` | **Single-user mode**: no network, root shell (maintenance) |
| `sudo init 3` | `multi-user.target` | Full server mode, command line only |
| `sudo init 5` | `graphical.target` | Desktop / GUI mode |
| `sudo init 6` | `reboot.target` | Clean restart |

Modern equivalents: `sudo systemctl poweroff` and `sudo systemctl reboot`.

## Changing the hostname

```bash
hostnamectl status                          # view current names
sudo hostnamectl set-hostname web-server-01 # set a persistent hostname
```

After changing it, add the new name to the `127.0.0.1` line in `/etc/hosts` so tools like `sudo` can resolve the host.

> 💡 On cloud images the hostname can revert on reboot. To make it stick, set `preserve_hostname: true` in `/etc/cloud/cloud.cfg`.

## Recovery: reset a forgotten root password

On a systemd host you intercept the **GRUB** bootloader instead of typing `init 1`:

1. Reboot; at the GRUB menu press **`e`** to edit the boot entry.
2. Find the line starting with `linux` and append a space then: `rd.break`
3. Press **Ctrl + X** to boot.
4. Remount the filesystem as writable:
   ```bash
   mount -o remount,rw /sysroot
   ```
5. Enter the system root:
   ```bash
   chroot /sysroot
   ```
6. Set the new password:
   ```bash
   passwd
   ```
7. Flag an SELinux relabel (required on RHEL/Amazon):
   ```bash
   touch /.autorelabel
   ```
8. Type `exit` twice — the system relabels and reboots with the new password.

> ⚠️ **Security warning:** anyone with console or serial access can use this to take over a machine. Enterprises set a **GRUB bootloader password** to block it.

## 🧪 Practice

> ⚠️ Do these on a **throwaway VM** you can afford to reboot — not a production server.

1. Check your current hostname, then change it:
   ```bash
   hostnamectl status
   sudo hostnamectl set-hostname lab-node-01
   # log out and back in — your prompt shows the new name
   ```
2. Schedule and then cancel a shutdown (safe — nothing actually happens):
   ```bash
   sudo shutdown +5 "test"
   sudo shutdown -c
   ```

**✅ Success check:** The prompt reflects the new hostname, and the scheduled shutdown is cancelled.

---
[← Previous: Logging & Diagnostics](12-logging-and-diagnostics.md) | [Next: System Monitoring Toolkit →](14-system-monitoring-toolkit.md)
