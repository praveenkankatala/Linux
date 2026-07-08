[🏠 Home](../README.md)  ·  [Phase 4 — Intermediate Efficiency](README.md)  ·  **Topic 13**

# Topic 13 — Functions

## 13.1  Defining and calling

A function is a named block you can call repeatedly. Define it before you call it. Note there are no parentheses at the call site — you pass arguments like a command.

*functions.sh*

```bash
#!/usr/bin/env bash

greet() {
  echo "Hello, $1!"
}

greet "Ada"       # call with an argument
greet "Devops"
```

```console
Hello, Ada!
Hello, Devops!
```

## 13.2  Arguments inside a function

Inside a function, `$1`, `$2`, `$@`, and `$#` refer to the function's *own* arguments — not the script's. This is exactly the mechanism you learned in Phase 1, scoped to the function.

*func-args.sh*

```bash
backup() {
  local src="$1"
  local dest="$2"
  echo "Backing up $src -> $dest ($# args)"
  cp -a "$src" "$dest"
}

backup /etc/hosts /tmp/hosts.bak
```

## 13.3  local vs global — the most important habit

By default, every variable in bash is **global** — a variable set inside a function leaks out and can clobber one elsewhere. Declaring variables `local` confines them to the function.

*local-scope.sh*

```bash
counter="OUTER"

bad() {  counter="changed"; }      # no local -> overwrites the global
good() { local counter="changed"; } # local -> global is safe

bad;  echo "after bad():  $counter"
counter="OUTER"
good; echo "after good(): $counter"
```

```console
after bad():  changed
after good(): OUTER
```

> [!TIP]
> **Declare every function variable local**
>
> Make `local x` your reflex for any variable a function uses internally. It prevents a whole category of 'why did that value change?' bugs, especially in larger scripts and when functions call other functions.

## 13.4  Returning values

This surprises newcomers: `return` in bash sets an **exit status** (0–255), not a value. To return *data*, print it to stdout and capture it with command substitution.

*returns.sh*

```bash
# Return a status (success/failure) with 'return':
is_even() { (( $1 % 2 == 0 )); }   # last command's status IS the return

if is_even 8; then echo "8 is even"; fi

# Return DATA by printing it and capturing with $(...):
timestamp() { date '+%Y-%m-%d_%H-%M-%S'; }

now=$(timestamp)
echo "Snapshot taken at $now"
```

> [!WARNING]
> **return is not a value**
>
> `return 42` sets `$?` to 42, and codes only go up to 255. `result=$(myfunc)` captures what the function *printed*, which is how you return strings or numbers larger than 255. Don't `echo` debug text inside a function whose stdout you plan to capture — it becomes part of the 'returned' value.

## 13.5  A small library pattern

*mini-lib.sh*

```bash
#!/usr/bin/env bash
# A reusable logging + guard pattern seen in production scripts

log()  { echo "[$(date '+%T')] $*"; }
die()  { echo "[FATAL] $*" >&2; exit 1; }

require_cmd() {
  local cmd="$1"
  command -v "$cmd" >/dev/null || die "$cmd is not installed"
}

require_cmd jq
log "jq is present, continuing"
```

> [!IMPORTANT]
> **🎯 Try It Yourself — Exercise 13**
>
> Write a function `max` that takes two numbers and prints the larger one. Then write `clamp <value> <lo> <hi>` that prints the value forced into the range [lo, hi]. Use `local` throughout and capture results with `$(...)`.

### Solution

*clamp.sh*

```bash
#!/usr/bin/env bash
max() { (( $1 > $2 )) && echo "$1" || echo "$2"; }
min() { (( $1 < $2 )) && echo "$1" || echo "$2"; }

clamp() {
  local v="$1" lo="$2" hi="$3"
  v=$(max "$v" "$lo")
  v=$(min "$v" "$hi")
  echo "$v"
}

echo "max(3,9)      = $(max 3 9)"
echo "clamp(15,0,10)= $(clamp 15 0 10)"
```

```console
max(3,9)      = 9
clamp(15,0,10)= 10
```

---

⬅️ [Topic 12: Data Extraction with awk](../phase-3-text-handling-and-io/12-awk.md)  |  ⬆️ [Phase 4 index](README.md)  |  [Topic 14: Arrays and String Manipulation](14-arrays-and-strings.md) ➡️
