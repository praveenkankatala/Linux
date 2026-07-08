# 07 — `awk`: The Field-Aware Powerhouse

`awk` reads input line by line, **splits each line into fields**, and lets you act on those fields. Where `grep` filters whole lines, `awk` understands **columns** — perfect for logs, CSVs, and any tabular text.

### Built-in variables you'll use constantly

| Variable | Meaning |
|----------|---------|
| `$0` | The entire current line |
| `$1`, `$2`, `$3` … | The 1st, 2nd, 3rd field |
| `NF` | **N**umber of **F**ields on this line |
| `NR` | **N**umber of the current **R**ecord (line) |
| `-F','` | Set the input field separator (here: comma) |

---

## Printing columns

```bash
# print field 1 (default separator = any whitespace)
$ awk '{print $1}' access.log
10.0.0.5
10.0.0.7
...

# print the URL (field 3) and status (field 4)
$ awk '{print $3, $4}' access.log

# CSV: set the field separator with -F
$ awk -F',' '{print $2, $3}' users.csv
name role
ravi engineer
...
```

---

## Filtering with conditions

```bash
# only lines where the status field ($4) is 500
$ awk '$4 == 500 {print $0}' access.log

# only rows in the 'data' dept (field 4 of the CSV), skipping the header
$ awk -F',' 'NR>1 && $4=="data" {print $2}' users.csv
sam
leah

# lines with more than 3 fields
$ awk 'NF > 3' access.log

# pattern match inside awk (like grep, but field-aware)
$ awk '/GET/ {print $1, $3}' access.log
```

---

## Computing — sums, counts, averages

This is where `awk` leaves the other filters behind. `BEGIN` runs **before** input, `END` runs **after** the last line — perfect for totals.

```bash
# count requests per IP (a group-by, entirely inside awk)
$ awk '{count[$1]++} END {for (ip in count) print count[ip], ip}' access.log
3 10.0.0.5
2 10.0.0.7
1 10.0.0.9

# sum a numeric column (e.g. bytes in field 5)
$ awk '{total += $5} END {print "total bytes:", total}' access.log

# average of a column
$ awk '{sum+=$1; n++} END {print sum/n}' numbers.txt

# print a header, then formatted rows
$ awk -F',' 'NR==1{print "USER\tROLE"; next} {print $2"\t"$3}' users.csv
```

---

## Structure reference

| Construct | Meaning |
|-----------|---------|
| `pattern { action }` | Run `action` on every line matching `pattern` |
| `BEGIN { ... }` | Run **once** before reading input |
| `END { ... }` | Run **once** after all input (totals, reports) |
| `arr[key]++` | Associative array — the basis of group-by counts |
| `NR==1 { ... ; next }` | Handle the header row, then skip to the next line |

> **Tip:** `awk` alone can replace a `grep | cut | sort` chain in many cases. When you find yourself piping three filters just to reshape columns, ask whether **one** `awk` would be clearer.

---

### ✅ Quick recap

- Print a column → `awk '{print $3}' file`.
- Set a CSV separator → `awk -F','`.
- Filter by a field → `awk '$4 == 500'`.
- Group-by count → `awk '{c[$1]++} END{for(k in c) print c[k], k}'`.
- Totals → use `BEGIN` / `END`.

---

⬅️ [Prev: grep](06-grep.md) | 🏠 [Index](README.md) | ➡️ [Next: Editing Files](08-file-editing.md)
