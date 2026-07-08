[🏠 Home](../README.md)  ·  [Phase 3 — Text Handling & I/O](README.md)  ·  **Topic 12**

# Topic 12 — Data Extraction with awk

`awk` is a small programming language for column-oriented text. Where `cut` struggles, `awk` shines: it splits on any whitespace by default, can do math, filter rows, and format output. You'll use maybe 10% of it constantly.

## 12.1  Fields: $1, $2, ... $NF

`awk` splits every line into fields. `$1` is the first field, `$NF` is the last, and `$0` is the whole line.

*awk-fields.sh*

```bash
# Print user (col 1) and home dir (col 6) from /etc/passwd
awk -F: '{ print $1, $6 }' /etc/passwd | head -3
```

```console
root /root
daemon /usr/sbin
bin /bin
```

| Variable | Meaning |
| --- | --- |
| `$1, $2, …` | Field by position |
| `$NF` | The last field |
| `$0` | The entire line |
| `NF` | Number of fields on this line |
| `NR` | Current line (record) number |
| `-F:` | Set the field separator to `:` |

## 12.2  Filtering rows with patterns

An `awk` program is `pattern { action }`. Lines matching the pattern get the action; omit the pattern to act on every line.

*awk-filter.sh*

```bash
# ps output — show only processes using > 50% CPU
ps aux | awk '$3 > 50 { print $1, $3, $11 }'

# Print lines where field 3 equals "active"
awk -F, '$3 == "active" { print $1 }' users.csv

# Print the 1st field of lines matching a regex
awk '/ERROR/ { print $1 }' app.log
```

## 12.3  Formatted output with printf

*awk-printf.sh*

```bash
df -h | awk 'NR>1 { printf "%-12s %s used of %s\n", $1, $3, $2 }'
```

```console
/dev/root    3.1G used of 7.6G
tmpfs        0 used of 483M
/dev/nvme1n1 12G used of 50G
```

> [!NOTE]
> `NR>1` skips the header row. `%-12s` left-aligns a string in a 12-char column. `printf` in awk needs an explicit `\n`, unlike `print`.

## 12.4  Summing and counting: BEGIN and END

`BEGIN` runs before any input, `END` runs after all of it — perfect for totals and averages.

*awk-sum.sh*

```bash
# Total the sizes in column 5 of an ls -l listing
ls -l | awk 'NR>1 { total += $5 } END { print total, "bytes" }'

# Average of a column of numbers
awk '{ sum += $1; n++ } END { printf "avg = %.2f\n", sum/n }' nums.txt
```

```console
48213 bytes
avg = 42.75
```

> [!IMPORTANT]
> **🎯 Try It Yourself — Exercise 12**
>
> From the output of `ps aux`, print the top 5 processes by memory usage (column 4 is %MEM), showing the user, %MEM, and command, nicely aligned. Combine awk with sort and head.

### Solution

*top-mem.sh*

```bash
#!/usr/bin/env bash
ps aux \
  | awk 'NR>1 { printf "%-10s %5s%%  %s\n", $1, $4, $11 }' \
  | sort -k2 -rn \
  | head -5
```

```console
mysql       8.4%  /usr/sbin/mysqld
devops      3.1%  /usr/lib/firefox/firefox
root        2.7%  /usr/bin/dockerd
www-data    1.9%  nginx
devops      1.2%  bash
```

---

⬅️ [Topic 11: Stream Editing with sed](11-sed.md)  |  ⬆️ [Phase 3 index](README.md)  |  [Topic 13: Functions](../phase-4-intermediate-efficiency/13-functions.md) ➡️
