# 🗒️ Linux Sysadmin Cheat Sheet

[← Back to index](README.md)

One-line lookups for every command in this guide.

## Services
| Task | Command |
|------|---------|
| Start / enable a service | `systemctl start nginx` · `systemctl enable nginx` |
| Reload config, no downtime | `systemctl reload nginx` |
| Check if running | `systemctl is-active nginx` |
| List failed services | `systemctl --failed` |

## Processes
| Task | Command |
|------|---------|
| All processes (detailed) | `ps aux` · `ps -ef` |
| Find a process | `ps -ef \| grep [n]ame` |
| Live dashboard | `top` (P=CPU, M=mem, k=kill, q=quit) |
| Graceful / force kill | `kill -15 PID` · `kill -9 PID` |
| Kill by name | `pkill name` |

## Scheduling
| Task | Command |
|------|---------|
| Edit / list cron jobs | `crontab -e` · `crontab -l` |
| Daily at 2 AM | `0 2 * * * /path/script.sh` |
| One-off job | `at now + 30 minutes` |
| List / remove `at` jobs | `atq` · `atrm <id>` |

## Text (sed)
| Task | Command |
|------|---------|
| Replace in file (save) | `sed -i 's/old/new/g' file` |
| Delete line range | `sed '2,3d' file` |
| Delete matching lines | `sed '/pattern/d' file` |

## Packages
| Task | RHEL/Amazon | Ubuntu |
|------|-------------|--------|
| Install | `sudo dnf install <pkg>` | `sudo apt install <pkg>` |
| Remove | `sudo dnf remove <pkg>` | `sudo apt remove <pkg>` |
| Update all | `sudo dnf upgrade` | `sudo apt update && sudo apt upgrade` |
| Search | `dnf search <term>` | `apt search <term>` |

## Users
| Task | Command |
|------|---------|
| Create user + home | `sudo useradd -m user` |
| Set password | `sudo passwd user` |
| Add to sudo group | `sudo usermod -aG wheel user` |
| Delete user + home | `sudo userdel -r user` |
| Password max age | `sudo chage -M 90 user` |
| Who's logged in / history | `who` · `w` · `last` |
| My identity / groups | `id` · `whoami` |

## System info
| Task | Command |
|------|---------|
| RAM | `free -h` |
| Disk | `df -h` |
| CPU cores | `lscpu` |
| Kernel / OS | `uname -a` · `cat /etc/os-release` |
| Quick math | `echo "10/3" \| bc -l` |

## Logs
| Task | Command |
|------|---------|
| Service logs | `journalctl -u sshd` |
| Live tail | `journalctl -f` · `tail -f /var/log/secure` |
| Errors only | `journalctl -p err` |
| Support bundle | `sudo sosreport` |

## Networking & monitoring
| Task | Command |
|------|---------|
| IP addresses | `ip addr` |
| Routing table | `ip route` |
| Listening ports | `ss -tulpn` |
| Disk I/O | `iostat` |
| Kernel messages | `dmesg` |

## Maintenance
| Task | Command |
|------|---------|
| Shutdown / reboot | `sudo shutdown now` · `sudo reboot` |
| Set hostname | `sudo hostnamectl set-hostname name` |
| Persistent session | `screen -S name` (Ctrl+A then D to detach) |
| Record session | `script file.log` |

---
[← Back to index](README.md)
