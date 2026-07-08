# 9. System Utility Commands

[← Back to index](../README.md)

The everyday helper commands that don't fit anywhere else — checking the time, doing quick math, and finding where a program lives.

| Command | Use | Handy example |
|---------|-----|---------------|
| `date` | Show or set the system date/time | `date +%F` → `2026-06-18` (great for backup filenames) |
| `cal` | Display a text calendar | `cal 2026`, or `cal -3` (prev/this/next month) |
| `uptime` | How long the system has run + load average | Load > core count = overloaded |
| `hostname` | Show the system name | `hostname -I` shows the machine's IP address(es) |
| `uname` | Print system information | `uname -r` = kernel release; `uname -a` = everything |
| `which` | Find where an executable lives | `which python3` → `/usr/bin/python3` |
| `bc` | Command-line calculator (high precision) | `echo "10 / 3" \| bc -l` → `3.333…` |

## Examples

```bash
$ date +%F
2026-06-18

$ uptime
 11:32:04 up 1 day,  5:32,  2 users,  load average: 0.05, 0.02, 0.01

$ echo "10 / 3" | bc -l
3.33333333333333333333
```

> 💡 **`bc` needs `-l` for decimals.** Without it, `echo "10 / 3" | bc` returns `3` (integer division only).

> 💡 If `cal` is missing on a minimal server, it lives in the `util-linux` (or `ncal`) package.

## 🧪 Practice

1. Print today's date in `YYYY-MM-DD` format:
   ```bash
   date +%F
   ```
2. Show this year's calendar:
   ```bash
   cal 2026
   ```
3. Do some math:
   ```bash
   echo "scale=2; 22 / 7" | bc      # → 3.14
   ```
4. Find where `bash` lives:
   ```bash
   which bash
   ```

**✅ Success check:** All four commands return sensible output.

---
[← Previous: System Information](08-system-information.md) | [Next: Environment Variables →](10-environment-variables.md)
