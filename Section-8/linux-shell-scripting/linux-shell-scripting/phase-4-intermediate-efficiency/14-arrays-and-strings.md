[🏠 Home](../README.md)  ·  [Phase 4 — Intermediate Efficiency](README.md)  ·  **Topic 14**

# Topic 14 — Arrays and String Manipulation

## 14.1  Indexed arrays

*arrays.sh*

```bash
#!/usr/bin/env bash
servers=("web01" "web02" "db01")

echo "First:  ${servers[0]}"     # index from 0
echo "Count:  ${#servers[@]}"    # number of elements
echo "All:    ${servers[@]}"     # every element

servers+=("cache01")             # append
echo "After append: ${servers[@]}"
```

```console
First:  web01
Count:  3
All:    web01 web02 db01
After append: web01 web02 db01 cache01
```

| Expression | Gives you |
| --- | --- |
| `${arr[i]}` | Element at index i |
| `${arr[@]}` | All elements (each quoted separately) |
| `${#arr[@]}` | Number of elements |
| `${!arr[@]}` | All the indices |
| `arr+=(x)` | Append element x |
| `${arr[@]:1:2}` | A slice: 2 elements starting at index 1 |

## 14.2  Looping over an array (the safe way)

*array-loop.sh*

```bash
for s in "${servers[@]}"; do      # always quote the expansion
  echo "Checking $s"
done

# When you need the index too:
for i in "${!servers[@]}"; do
  echo "$i -> ${servers[$i]}"
done
```

> [!WARNING]
> **Quote "${arr[@]}", not ${arr[*]}**
>
> `"${arr[@]}"` expands to each element as its own word — correct even when elements contain spaces. `"${arr[*]}"` joins everything into one string. When looping, you almost always want `[@]` with quotes.

## 14.3  Building an array from command output

*mapfile.sh*

```bash
# mapfile (aka readarray) reads lines into an array — bash 4+
mapfile -t files < <(find . -maxdepth 1 -name '*.sh')
echo "Found ${#files[@]} scripts"
printf '  %s\n' "${files[@]}"
```

> [!NOTE]
> `-t` strips the trailing newline from each line. `< <(...)` is process substitution — feeding a command's output as if it were a file. Full details in Phase 5, Topic 20.

## 14.4  String manipulation with ${...}

Bash has built-in string surgery — no need to shell out to `sed` or `cut` for simple cases. It's faster and clearer.

| Expression | Purpose | Example → result |
| --- | --- | --- |
| `${#s}` | Length | `s=hello` → 5 |
| `${s:2:3}` | Substring (offset, length) | → `llo` |
| `${s^^}` / `${s,,}` | Upper / lower case | → `HELLO` / `hello` |
| `${s#pre}` | Remove shortest prefix | `${f#*/}` |
| `${s##*/}` | Remove longest prefix (basename) | `/a/b/c` → `c` |
| `${s%.txt}` | Remove shortest suffix | `a.txt` → `a` |
| `${s%/*}` | Remove to last slash (dirname) | `/a/b/c` → `/a/b` |
| `${s/old/new}` | Replace first match | `aa` → `Xa` |
| `${s//old/new}` | Replace all matches | `aa` → `XX` |
| `${s:-default}` | Use default if unset/empty | → `default` |

*string-ops.sh*

```bash
path="/var/log/nginx/access.log"

echo "file:  ${path##*/}"     # access.log   (basename)
echo "dir:   ${path%/*}"      # /var/log/nginx (dirname)
echo "stem:  ${path##*/}" ; f=${path##*/}
echo "noext: ${f%.log}"       # access
echo "upper: ${f^^}"          # ACCESS.LOG
```

```console
file:  access.log
dir:   /var/log/nginx
stem:  access.log
noext: access
upper: ACCESS.LOG
```

> [!IMPORTANT]
> **🎯 Try It Yourself — Exercise 14**
>
> Given `files=("report.pdf" "data.csv" "notes.txt" "budget.csv")`, loop over the array and print only the `.csv` files, each with its extension stripped. Use array iteration and `${...}` string ops — no external commands.

### Solution

*csv-filter.sh*

```bash
#!/usr/bin/env bash
files=("report.pdf" "data.csv" "notes.txt" "budget.csv")

for f in "${files[@]}"; do
  [[ "$f" == *.csv ]] || continue
  echo "CSV: ${f%.csv}"
done
```

```console
CSV: data
CSV: budget
```

---

⬅️ [Topic 13: Functions](13-functions.md)  |  ⬆️ [Phase 4 index](README.md)  |  [Topic 15: Parsing Flags with getopts](15-getopts.md) ➡️
