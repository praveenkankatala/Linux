[🏠 Home](../README.md)  ·  [Phase 2 — Logic & Automation](README.md)  ·  **Topic 8**

# Topic 8 — while Loops

## 8.1  Looping while a condition holds

*countdown.sh*

```bash
#!/usr/bin/env bash
count=5
while (( count > 0 )); do
  echo "T-minus $count"
  (( count-- ))
  sleep 1
done
echo "Liftoff!"
```

## 8.2  The essential pattern: read a file line by line

This is the single most important `while` idiom in shell. Memorise it exactly — the flags matter.

*read-lines.sh*

```bash
#!/usr/bin/env bash
# Print each line of a file with a line number
n=0
while IFS= read -r line; do
  (( n++ ))
  echo "$n: $line"
done < "input.txt"
```

| Piece | Why it's there |
| --- | --- |
| `IFS=` | Stops leading/trailing whitespace from being trimmed |
| `read -r` | Stops backslashes from being interpreted |
| `line` | The variable each line is stored in |
| `< "input.txt"` | Feeds the file into the loop's input |

> [!WARNING]
> **Feed the file with < , not a pipe**
>
> `cat file | while read ...` also works, but the `while` runs in a *subshell*, so variables you set inside (like `n`) are lost when the loop ends. Redirecting with `done < file` keeps the loop in your current shell. We revisit subshells in Phase 5.

## 8.3  Reading columns from each line

`read` can split each line into multiple variables. Give it a delimiter with `IFS`.

*read-csv.sh*

```bash
#!/usr/bin/env bash
# users.csv holds:  alice,admin,active
while IFS=, read -r user role status; do
  echo "$user is a $role ($status)"
done < "users.csv"
```

```console
alice is a admin (active)
bob is a developer (active)
carol is a auditor (disabled)
```

## 8.4  Reading from a command with process substitution

To loop over a command's output *without* the subshell trap, feed it in with `< <(...)`. This keeps variables alive (full treatment in Phase 5, Topic 20).

*count-procs.sh*

```bash
#!/usr/bin/env bash
total=0
while IFS= read -r pid; do
  (( total++ ))
done < <(pgrep -u "$USER")
echo "You are running $total processes"
```

> [!IMPORTANT]
> **🎯 Try It Yourself — Exercise 8**
>
> You have a file `servers.txt` with one hostname per line. Write `ping-all.sh` that reads it line by line and, for each host, prints `UP` or `DOWN` based on `ping -c1 -W1`. Count how many are up and print a summary at the end (this is why we avoid the subshell pipe!).

### Solution

*ping-all.sh*

```bash
#!/usr/bin/env bash
up=0 total=0
while IFS= read -r host; do
  [[ -z "$host" ]] && continue     # skip blank lines
  (( total++ ))
  if ping -c1 -W1 "$host" &>/dev/null; then
    echo "UP    $host"; (( up++ ))
  else
    echo "DOWN  $host"
  fi
done < "servers.txt"

echo "---"
echo "$up of $total hosts are up"
```

---

⬅️ [Topic 7: for Loops](07-for-loops.md)  |  ⬆️ [Phase 2 index](README.md)  |  [Topic 9: stdout, stderr, and Redirection](../phase-3-text-handling-and-io/09-redirection.md) ➡️
