[🏠 Home](../README.md)  ·  [Phase 1 — The Basics](README.md)  ·  **Topic 4**

# Topic 4 — Arithmetic and Command Substitution

## 4.1  Integer arithmetic with (( )) and $(( ))

The shell treats everything as text by default, so `2 + 2` is just a string. For real integer math, use the arithmetic contexts `$(( ))` (to get a value) and `(( ))` (to perform an operation).

*math.sh*

```bash
a=7
b=3

sum=$(( a + b ))       # note: no $ needed on names inside (( ))
echo "sum = $sum"

(( count = a * b ))    # (( )) performs and assigns directly
echo "count = $count"

(( count++ ))          # increment in place
echo "after ++ = $count"
```

```console
sum = 10
count = 21
after ++ = 22
```

| Operator | Meaning | Example → result |
| --- | --- | --- |
| `+ - * /` | Add, subtract, multiply, divide | `$(( 9 / 2 ))` → 4 |
| `%` | Modulo (remainder) | `$(( 9 % 2 ))` → 1 |
| `**` | Exponent | `$(( 2 ** 8 ))` → 256 |
| `++ --` | Increment / decrement | `(( i++ ))` |
| `+= -= *=` | Compound assignment | `(( total += 5 ))` |

> [!WARNING]
> **The shell does integer math only**
>
> `$(( 9 / 2 ))` is `4`, not `4.5` — the fractional part is discarded. For decimals you need an external tool such as `bc` or `awk`: `echo "scale=2; 9/2" | bc` gives `4.50`. We use `awk` for math in Phase 3.

## 4.2  Command substitution: $(command)

Command substitution runs a command and *substitutes its output* into your script. This is one of the most powerful ideas in shell — it lets you capture the result of anything into a variable.

*subst.sh*

```bash
today=$(date +%F)
files=$(ls | wc -l)
kernel=$(uname -r)

echo "On $today there are $files items here, kernel $kernel"
```

```console
On 2026-07-07 there are 14 items here, kernel 6.1.0-aws
```

> [!NOTE]
> **$(...) vs backticks**
>
> You will see the old backtick form `` `date` `` in legacy scripts. Prefer `$(date)` — it nests cleanly, e.g. `$(dirname "$(readlink -f "$0")")`, whereas nested backticks are painful and error-prone.

## 4.3  Putting it together

Command substitution plus arithmetic is a common combination — capture a number from a command, then compute with it.

*diskcheck.sh*

```bash
#!/usr/bin/env bash
# diskcheck.sh — rough free-space report for the root filesystem

# Pull the 'used%' column for / and strip the % sign
used=$(df --output=pcent / | tail -1 | tr -dc '0-9')
free=$(( 100 - used ))

echo "Root filesystem: ${used}% used, ${free}% free"
```

```console
Root filesystem: 38% used, 62% free
```

> [!IMPORTANT]
> **🎯 Try It Yourself — Exercise 4**
>
> Write `circle.sh` that reads a radius with `read`, then prints the (integer) area using `$(( ))` and the approximation area ≈ 3 × r × r. Then print a more precise value using `awk` for real decimals.

### Solution

*circle.sh*

```bash
#!/usr/bin/env bash
read -rp "Radius: " r

rough=$(( 3 * r * r ))
echo "Rough area (integer): $rough"

precise=$(awk -v r="$r" 'BEGIN { printf "%.2f", 3.14159 * r * r }')
echo "Precise area: $precise"
```

```console
$ ./circle.sh
Radius: 5
Rough area (integer): 75
Precise area: 78.54
```

---

⬅️ [Topic 3: User Input and Command Arguments](03-input-and-arguments.md)  |  ⬆️ [Phase 1 index](README.md)  |  [Topic 5: Exit Status and Conditionals](../phase-2-logic-and-automation/05-exit-status-and-conditionals.md) ➡️
