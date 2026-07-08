[🏠 Home](../README.md)  ·  [Phase 5 — Advanced & Robust Scripting](README.md)  ·  **Topic 18**

# Topic 18 — Associative Arrays (Hash Maps)

Indexed arrays use numbers as keys. **Associative arrays** use *strings* — they're the dictionaries / hash maps of bash (4.0+). Perfect for lookups, counters, and config maps.

## 18.1  Declare, set, and read

*assoc-basic.sh*

```bash
#!/usr/bin/env bash
declare -A env_url            # -A is REQUIRED for associative arrays

env_url[dev]="https://dev.example.com"
env_url[prod]="https://example.com"
env_url[staging]="https://stg.example.com"

echo "prod -> ${env_url[prod]}"
echo "there are ${#env_url[@]} environments"
```

```console
prod -> https://example.com
there are 3 environments
```

> [!WARNING]
> **declare -A is mandatory**
>
> Without `declare -A`, bash treats the name as an ordinary indexed array and every string key silently collapses to index 0 — a baffling bug where all your values overwrite each other. Always declare associative arrays explicitly. (They also require bash 4.0+; the macOS system bash is 3.2 and does NOT support them.)

## 18.2  Iterating over keys and values

*assoc-iter.sh*

```bash
for key in "${!env_url[@]}"; do        # ! gives the KEYS
  echo "$key = ${env_url[$key]}"
done

# membership test
if [[ -v env_url[prod] ]]; then echo "prod is defined"; fi
```

| Expression | Gives |
| --- | --- |
| `${map[key]}` | Value for a key |
| `${!map[@]}` | All keys |
| `${map[@]}` | All values |
| `${#map[@]}` | Number of entries |
| `[[ -v map[key] ]]` | True if that key exists |
| `unset 'map[key]'` | Delete an entry |

## 18.3  The killer use case: counting

Associative arrays make frequency counting trivial — a common task when analysing logs.

*status-tally.sh*

```bash
#!/usr/bin/env bash
declare -A count

# Tally HTTP status codes from an access log (field 9 in common format)
while read -r code; do
  (( count[$code]++ ))
done < <(awk '{ print $9 }' access.log)

for code in "${!count[@]}"; do
  printf '%s : %d\n' "$code" "${count[$code]}"
done | sort
```

```console
200 : 1843
301 : 52
404 : 96
500 : 7
```

> [!IMPORTANT]
> **🎯 Try It Yourself — Exercise 18**
>
> Build a small 'inventory' associative array mapping fruit names to stock counts. Write functions `add <fruit> <qty>` (increment) and `report` (print each fruit and its total, sorted). Add several quantities of overlapping fruits and confirm the totals accumulate.

### Solution

*inventory.sh*

```bash
#!/usr/bin/env bash
declare -A stock

add()    { stock[$1]=$(( ${stock[$1]:-0} + $2 )); }
report() { for f in "${!stock[@]}"; do echo "$f: ${stock[$f]}"; done | sort; }

add apple 3; add banana 5; add apple 2; add cherry 10
report
```

```console
apple: 5
banana: 5
cherry: 10
```

---

⬅️ [Topic 17: Traps, Signals, and Cleanup](17-traps-and-signals.md)  |  ⬆️ [Phase 5 index](README.md)  |  [Topic 19: Concurrency: Background Jobs, wait, Subshells](19-concurrency.md) ➡️
