[🏠 Home](../README.md)  ·  [Phase 4 — Intermediate Efficiency](README.md)  ·  **Topic 15**

# Topic 15 — Parsing Flags with getopts

Positional arguments (`$1`, `$2`) are fine for one or two inputs, but real tools take *flags*: `-f file`, `-v`, `-h`. `getopts` parses them the standard, professional way — handling bundling, missing arguments, and unknown flags for you.

## 15.1  The getopts skeleton

*backup.sh*

```bash
#!/usr/bin/env bash
# usage: backup.sh -f <file> [-v] [-h]

verbose=0
file=""

usage() {
  echo "Usage: $0 -f <file> [-v] [-h]"
  echo "  -f FILE   file to back up (required)"
  echo "  -v        verbose output"
  echo "  -h        show this help"
}

while getopts ":f:vh" opt; do
  case "$opt" in
    f) file="$OPTARG" ;;
    v) verbose=1 ;;
    h) usage; exit 0 ;;
    :) echo "Option -$OPTARG needs an argument" >&2; exit 1 ;;
    \?) echo "Unknown option: -$OPTARG" >&2; usage; exit 1 ;;
  esac
done
shift $(( OPTIND - 1 ))   # drop parsed options; leftovers remain in $@

[[ -n "$file" ]] || { echo "-f is required" >&2; usage; exit 1; }
(( verbose )) && echo "Backing up $file (verbose on)"
```

## 15.2  Decoding the option string

| Part | Meaning |
| --- | --- |
| `":f:vh"` | The options this tool accepts |
| Leading `:` | Silent error mode — YOU handle errors via the `:` and `?` cases |
| `f:` | `-f` takes an argument (the colon after it) |
| `v`, `h` | Flags with no argument |
| `$OPTARG` | The argument value for the current option |
| `$OPTIND` | Index of the next argument — used by the final `shift` |

## 15.3  Running it

```console
$ ./backup.sh -h
Usage: ./backup.sh -f <file> [-v] [-h]
  -f FILE   file to back up (required)
  -v        verbose output
  -h        show this help

$ ./backup.sh -v -f data.sql
Backing up data.sql (verbose on)

$ ./backup.sh -x
Unknown option: -x
Usage: ./backup.sh -f <file> [-v] [-h]
```

> [!TIP]
> **Why the final shift matters**
>
> `shift $(( OPTIND - 1 ))` removes all the parsed `-flags` so that any *positional* arguments (say, a list of files after the options) start cleanly at `$1`. Without it, `"$@"` would still contain the flags.

> [!NOTE]
> **getopts vs getopt**
>
> `getopts` (built into bash, no dash) is what you want — portable and simple, but short options only. The external `getopt` command supports `--long-options` but is fiddlier and less portable. For most scripts, stick with `getopts`.

> [!IMPORTANT]
> **🎯 Try It Yourself — Exercise 15**
>
> Write `greet.sh` that accepts `-n <name>` (required), `-c <count>` (optional, default 1) and `-u` (uppercase the greeting). It should print `Hello, <name>!` `count` times, uppercased if `-u` is set. Validate that `-n` was supplied.

### Solution

*greet.sh*

```bash
#!/usr/bin/env bash
name="" count=1 upper=0

while getopts ":n:c:u" opt; do
  case "$opt" in
    n) name="$OPTARG" ;;
    c) count="$OPTARG" ;;
    u) upper=1 ;;
    :) echo "-$OPTARG needs a value" >&2; exit 1 ;;
    \?) echo "bad option -$OPTARG" >&2; exit 1 ;;
  esac
done

[[ -n "$name" ]] || { echo "-n <name> is required" >&2; exit 1; }

msg="Hello, $name!"
(( upper )) && msg="${msg^^}"
for (( i = 0; i < count; i++ )); do echo "$msg"; done
```

```console
$ ./greet.sh -n devops -c 2 -u
HELLO, DEVOPS!
HELLO, DEVOPS!
```

---

⬅️ [Topic 14: Arrays and String Manipulation](14-arrays-and-strings.md)  |  ⬆️ [Phase 4 index](README.md)  |  [Topic 16: Defensive Flags and Error Logging](../phase-5-advanced-robust-scripting/16-strict-mode-and-logging.md) ➡️
