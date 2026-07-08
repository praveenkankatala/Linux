[🏠 Home](../README.md)  ·  [Phase 2 — Logic & Automation](README.md)  ·  **Topic 5**

# Topic 5 — Exit Status and Conditionals

## 5.1  The exit status: $?

After any command runs, `$?` holds its exit status. Check it immediately — it changes with every command.

*exit-status.sh*

```bash
ls /etc/hostname   # a file that exists
echo "exit status: $?"

ls /no/such/path   # a path that does not
echo "exit status: $?"
```

```console
/etc/hostname
exit status: 0
ls: cannot access '/no/such/path': No such file or directory
exit status: 2
```

| Exit code | Convention |
| --- | --- |
| `0` | Success |
| `1` | General error |
| `2` | Misuse of a builtin / bad arguments |
| `126` | Command found but not executable |
| `127` | Command not found |
| `130` | Terminated by Ctrl-C (SIGINT) |

> [!TIP]
> **Set your own exit status**
>
> End a script with `exit 0` for success or `exit 1` (or higher) for failure. Automation tools — Jenkins, Ansible, cron wrappers — read that number to decide whether a step passed or failed.

## 5.2  if / elif / else

`if` runs a command and branches on its exit status. The `[[ ... ]]` you'll see everywhere is itself a command (a *test*) that returns 0 (true) or 1 (false).

*grade.sh*

```bash
#!/usr/bin/env bash
read -rp "Enter a score (0-100): " score

if (( score >= 90 )); then
  echo "Grade: A"
elif (( score >= 70 )); then
  echo "Grade: B"
elif (( score >= 50 )); then
  echo "Grade: C"
else
  echo "Grade: F"
fi
```

```console
$ ./grade.sh
Enter a score (0-100): 82
Grade: B
```

> [!WARNING]
> **Syntax that must be exact**
>
> `if` requires `then`, and the block ends with `fi` (if backwards). The semicolon before `then` is needed when they share a line. Missing the space inside brackets — `[[$x` instead of `[[ $x` — is the number-one beginner error and produces a cryptic `command not found`.

## 5.3  Branching on a command's success

You don't need brackets at all when you're testing whether a command succeeded — `if` reads the exit status directly.

*ping-check.sh*

```bash
#!/usr/bin/env bash
host="8.8.8.8"

if ping -c1 -W1 "$host" &>/dev/null; then
  echo "$host is reachable"
else
  echo "$host is DOWN" >&2
  exit 1
fi
```

> [!NOTE]
> `&>/dev/null` throws away all output (we cover redirection in Phase 3). We only care about ping's *exit status* here, not its chatter.

## 5.4  && and || — quick one-liners

*and-or.sh*

```bash
# Run the second command ONLY IF the first succeeds
mkdir -p /tmp/build && cd /tmp/build

# Run the second command ONLY IF the first fails
cd /nonexistent || echo "could not enter directory"
```

```console
could not enter directory
```

> [!IMPORTANT]
> **🎯 Try It Yourself — Exercise 5**
>
> Write `checkuser.sh <username>` that uses `id "$1" &>/dev/null` to test whether a user exists on the system. Print `<user> exists` or `<user> not found` and exit with status 0 or 1 accordingly.

### Solution

*checkuser.sh*

```bash
#!/usr/bin/env bash
user="$1"

if id "$user" &>/dev/null; then
  echo "$user exists"
  exit 0
else
  echo "$user not found" >&2
  exit 1
fi
```

---

⬅️ [Topic 4: Arithmetic and Command Substitution](../phase-1-the-basics/04-arithmetic-and-command-substitution.md)  |  ⬆️ [Phase 2 index](README.md)  |  [Topic 6: String, Integer, and File Tests](06-test-operators.md) ➡️
