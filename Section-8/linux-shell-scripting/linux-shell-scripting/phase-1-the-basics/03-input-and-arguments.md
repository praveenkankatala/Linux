[🏠 Home](../README.md)  ·  [Phase 1 — The Basics](README.md)  ·  **Topic 3**

# Topic 3 — User Input and Command Arguments

## 3.1  Reading interactive input with read

`read` pauses the script and stores whatever the user types into a variable. `-p` shows a prompt on the same line.

*greet.sh*

```bash
#!/usr/bin/env bash

read -p "What is your name? " name
echo "Welcome, $name!"
```

```console
$ ./greet.sh
What is your name? Devops
Welcome, Devops!
```

### Useful read flags

| Flag | Effect |
| --- | --- |
| `-p "text"` | Print a prompt before reading |
| `-s` | Silent — don't echo input (for passwords) |
| `-r` | Raw — do not treat backslash as an escape (almost always use this) |
| `-t 5` | Time out after 5 seconds |
| `-n 1` | Return after a single character (no Enter needed) |

*read-flags.sh*

```bash
read -rsp "Enter password: " pass; echo
read -rp  "Continue? [y/N] " -n 1 answer; echo
```

> [!TIP]
> **Default to read -r**
>
> Without `-r`, a backslash in the input is swallowed as an escape character, mangling things like Windows paths. Make `read -r` your default.

## 3.2  Command-line arguments: $1, $2, $@, $#

Arguments passed when launching the script are available as numbered variables. This is how real tools work — no interactive prompt, just `./deploy.sh prod v2.1`.

| Variable | Meaning |
| --- | --- |
| `$0` | The script's own name |
| `$1`, `$2`, … | First, second, … positional argument |
| `$#` | The number of arguments |
| `$@` | All arguments as separate quoted words (use `"$@"`) |
| `$*` | All arguments as one string (rarely what you want) |

*args.sh*

```bash
#!/usr/bin/env bash
# args.sh

echo "Script name: $0"
echo "First arg:   $1"
echo "Second arg:  $2"
echo "Arg count:   $#"
echo "All args:    $@"
```

```console
$ ./args.sh prod v2.1
Script name: ./args.sh
First arg:   prod
Second arg:  v2.1
Arg count:   2
All args:    prod v2.1
```

> [!WARNING]
> **Always quote "$1" and "$@"**
>
> Same golden rule as before. `"$@"` (quoted) preserves each argument exactly, even if one contains spaces — essential when passing arguments on to another command. Unquoted `$@` re-splits them and breaks.

## 3.3  A guard clause: checking argument count

Professional scripts validate their input before doing work. We will cover `if` formally in Phase 2, but here is the pattern you will use constantly.

*rename-safe.sh*

```bash
#!/usr/bin/env bash
# rename-safe.sh <source> <destination>

if [[ $# -ne 2 ]]; then
  echo "Usage: $0 <source> <destination>" >&2
  exit 1
fi

echo "Would move '$1' to '$2'"
```

```console
$ ./rename-safe.sh onefile
Usage: ./rename-safe.sh <source> <destination>
$ ./rename-safe.sh a.txt b.txt
Would move 'a.txt' to 'b.txt'
```

> [!IMPORTANT]
> **🎯 Try It Yourself — Exercise 3**
>
> Write `greet-anyone.sh` that takes a name as `$1`. If no argument is given, fall back to asking interactively with `read`. Then print `Hello, <name>!`. Quote everything correctly.

### Solution

*greet-anyone.sh*

```bash
#!/usr/bin/env bash
name="$1"

if [[ -z "$name" ]]; then
  read -rp "Enter a name: " name
fi

echo "Hello, $name!"
```

> [!NOTE]
> `-z "$name"` tests whether the string is empty. We will meet the full family of test operators in Topic 6.

---

⬅️ [Topic 2: Variables and the Golden Rule of Quoting](02-variables-and-quoting.md)  |  ⬆️ [Phase 1 index](README.md)  |  [Topic 4: Arithmetic and Command Substitution](04-arithmetic-and-command-substitution.md) ➡️
