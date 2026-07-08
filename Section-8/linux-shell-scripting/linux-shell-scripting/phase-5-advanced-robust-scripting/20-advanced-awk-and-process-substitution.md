[🏠 Home](../README.md)  ·  [Phase 5 — Advanced & Robust Scripting](README.md)  ·  **Topic 20**

# Topic 20 — Advanced awk and Process Substitution

## 20.1  awk with variables, conditions, and multiple actions

*awk-vars.sh*

```bash
# Pass a shell variable INTO awk with -v
threshold=80
df -h | awk -v t="$threshold" 'NR>1 {
  gsub(/%/, "", $5)             # strip the % from the use column
  if ($5 +0 > t) print "HIGH:", $6, $5"%"
}'
```

```console
HIGH: / 88%
HIGH: /var 91%
```

## 20.2  Grouping and aggregating (awk as mini-SQL)

awk's associative arrays let it group and sum in a single pass — the kind of task you'd otherwise reach for a database or pandas to do.

*awk-groupby.sh*

```bash
# Sum bytes transferred (field 10) per client IP (field 1) in an access log
awk '{ bytes[$1] += $10 }
     END { for (ip in bytes) printf "%-15s %d\n", ip, bytes[ip] }' access.log \
  | sort -k2 -rn | head
```

```console
203.0.113.12    48213945
198.51.100.7    12097331
192.0.2.44       8830112
```

| awk feature | Use |
| --- | --- |
| `-v name=val` | Inject a shell value into awk |
| `arr[key] += n` | Group and accumulate |
| `for (k in arr)` | Iterate an awk associative array |
| `gsub(/re/, s, $f)` | Regex substitute within a field |
| `BEGIN{FS=",";OFS="\t"}` | Set input / output field separators |
| `length($0)`, `substr(s,i,n)` | String helpers |

## 20.3  Process substitution: <(command)

`<(command)` makes a command's output look like a *file* (a temporary named pipe). This unlocks two big patterns: feeding command output to tools that expect filenames, and comparing two live command outputs.

*procsub.sh*

```bash
# 1) diff the output of two commands directly — no temp files
diff <(sort file1.txt) <(sort file2.txt)

# 2) compare users on two hosts
diff <(ssh host1 'cut -d: -f1 /etc/passwd | sort') \
     <(ssh host2 'cut -d: -f1 /etc/passwd | sort')

# 3) feed a pipeline into a while loop WITHOUT a subshell
count=0
while IFS= read -r line; do (( count++ )); done < <(grep ERROR app.log)
echo "$count errors"      # count survives — not lost in a subshell"
```

> [!NOTE]
> **Why this beats a pipe here**
>
> `grep ERROR app.log | while read ...` would run the loop in a subshell, so `count` resets to 0 afterward. `< <(grep ...)` keeps the loop in the current shell, so the variable persists. This is the definitive fix for the subshell trap you met in Topics 8 and 19.

## 20.4  Output process substitution: >(command)

*out-procsub.sh*

```bash
# Send one stream to two different consumers at once
./run-backup.sh \
  > >(gzip > backup.log.gz) \
  2> >(mail -s 'backup errors' admin@example.com)
```

## 20.5  Capstone: a real diagnostic one-liner

Everything from this course, combined — parse a log, filter, group, sum, sort, format:

*slowest-endpoints.sh*

```bash
# Top 10 slowest endpoints by average response time (ms) from an app log
# log format: ... METHOD /path ... rt=NNN
awk '{
  match($0, /rt=([0-9]+)/, m)          # capture response time
  match($0, /(GET|POST) ([^ ]+)/, p)   # capture method + path
  key = p[1] " " p[2]
  sum[key] += m[1]; n[key]++
}
END {
  for (k in sum) printf "%8.1f ms  %6d hits  %s\n", sum[k]/n[k], n[k], k
}' app.log | sort -rn | head -10
```

```console
   842.3 ms    1204 hits  POST /api/report/generate
   511.7 ms    8890 hits  GET /api/search
   208.4 ms   40122 hits  GET /api/user/profile
```

> [!IMPORTANT]
> **🎯 Try It Yourself — Exercise 20**
>
> Using process substitution, write a one-liner that shows which packages are installed on the current host but NOT listed in a reference file `baseline.txt` (one package per line). Then write an awk command that reads a CSV of `region,sales` and prints total sales per region, sorted highest first.

### Solution

*exercise20.sh*

```bash
# Packages present but not in the baseline:
comm -23 <(dpkg --get-selections | awk '{print $1}' | sort) \
         <(sort baseline.txt)

# Sales per region:
awk -F, '{ s[$1] += $2 } END { for (r in s) print s[r], r }' sales.csv \
  | sort -rn
```

---

⬅️ [Topic 19: Concurrency: Background Jobs, wait, Subshells](19-concurrency.md)  |  ⬆️ [Phase 5 index](README.md)  |  _End of course_ 🎉
