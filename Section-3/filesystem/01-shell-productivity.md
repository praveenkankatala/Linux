# 01 — Shell Productivity

These habits save more keystrokes than anything else you'll learn. They cost nothing and pay back every session.

- [Tab Completion](#tab-completion)
- [Command History & Arrow Keys](#command-history--arrow-keys)
- [The Pipe `|`](#the-pipe-)
- [Redirection](#redirection)

---

## Tab Completion

Press **Tab** and Bash finishes what it can unambiguously complete — commands, file names, paths, and (with completion scripts) even sub-commands and flags.

```bash
# type 'cat acc' then press Tab -> completes to access.log
$ cat acc<Tab>
$ cat access.log

# ambiguous: Tab once = beep, Tab twice = list all candidates
$ cd /et<Tab><Tab>
etc/

# command names complete too: 'systemc' + Tab -> systemctl
$ systemc<Tab>
```

- **One Tab** completes when there is exactly one match.
- **Two Tabs** list every match when the completion is ambiguous.

> **Tip:** Install extended completions once with `sudo yum install -y bash-completion` (RHEL / Amazon Linux) or `sudo apt install bash-completion` (Ubuntu). After that, `git che<Tab>` → `git checkout`, `systemctl res<Tab>` → `restart`, and more.

---

## Command History & Arrow Keys

Bash records the commands you run. The **Up arrow** walks backward through that history; **Down arrow** walks forward. You rarely need to retype anything.

```bash
$ history        # show the numbered history list
$ !!             # run the previous command again
$ !125           # run command number 125 from history
$ !cat           # run the most recent command starting with 'cat'
$ !$             # reuse the LAST argument of the previous command
```

A common pattern: run a command, then reuse its last argument.

```bash
$ mkdir /opt/myapp/config
$ cd !$          # expands to: cd /opt/myapp/config
```

### Keyboard shortcuts worth memorizing

| Keystroke | What it does |
|-----------|--------------|
| `Ctrl + R` | **Reverse-search** history as you type (press again to cycle older matches) |
| `Ctrl + A` / `Ctrl + E` | Jump to **start** / **end** of the line |
| `Ctrl + U` / `Ctrl + K` | Delete to **start** / **end** of the line |
| `Ctrl + W` | Delete the **word** before the cursor |
| `Ctrl + L` | Clear the screen (same as `clear`) |

> **Tip:** `Ctrl+R` then typing `ssh` instantly finds the last `ssh` command you ran — a lifesaver when the host string is long. Press **Enter** to run it, or arrow keys to edit first.

---

## The Pipe `|`

The pipe is the single most important idea in Linux text processing. It connects the **standard output** of one command to the **standard input** of the next, so small tools combine into powerful pipelines — with **no temporary files**.

```bash
command1 | command2 | command3
#   stdout ->  stdin  stdout -> stdin
```

Read it left to right: the output of `command1` becomes the input of `command2`, and so on.

### Why it matters

Each Linux tool does one thing well. `grep` filters, `sort` orders, `uniq` de-duplicates, `wc` counts. Alone they're limited; piped together they answer real questions.

```bash
# How many unique client IPs hit the server?
$ cut -d' ' -f1 access.log | sort | uniq | wc -l
4

# Top requested URLs, most frequent first
$ cut -d' ' -f3 access.log | sort | uniq -c | sort -rn
      3 /index.html
      1 /login
      1 /assets/app.js
      1 /api/order
```

Pipe into a **pager** to scroll long output:

```bash
$ dmesg | less                    # scroll kernel messages
$ ps aux | grep java              # find a running process
$ journalctl -u nginx | tail -50  # last 50 log lines for a service
```

---

## Redirection

Redirection sends output to (or reads input from) **files** rather than another command. Don't confuse it with the pipe.

| Symbol | Meaning | Example |
|--------|---------|---------|
| `\|` | Send output to another **command** | `ls \| grep log` |
| `>` | Send output to a **file** (overwrite) | `ls > files.txt` |
| `>>` | Send output to a **file** (append) | `date >> run.log` |
| `<` | Read a command's input **from a file** | `sort < users.csv` |
| `2>` | Redirect **errors** (stderr) to a file | `cmd 2> err.log` |
| `&>` | Redirect **both** output and errors | `cmd &> all.log` |

```bash
$ echo "hello" > note.txt      # creates/overwrites note.txt
$ echo "world" >> note.txt     # appends a second line
$ ls /nope 2> errors.txt       # send the error message to a file
$ command > out.txt 2>&1       # send output AND errors to the same file
```

> **⚠️ Watch out:** `>` empties the target file **before** the command runs. Something like `sort data.txt > data.txt` will destroy `data.txt`. Use a temporary file (`sort data.txt > tmp && mv tmp data.txt`) for in-place work.

---

### ✅ Quick recap

- **Tab** completes; **Tab Tab** lists options.
- **Up arrow** / `!!` recall commands; **Ctrl+R** searches history.
- **`|`** connects commands; **`>` `>>`** write to files.

---

⬅️ [Back to Index](README.md) | ➡️ [Next: Compression & Archiving](02-compression-and-archiving.md)
