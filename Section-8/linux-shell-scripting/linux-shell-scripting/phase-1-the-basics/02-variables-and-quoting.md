[🏠 Home](../README.md)  ·  [Phase 1 — The Basics](README.md)  ·  **Topic 2**

# Topic 2 — Variables and the Golden Rule of Quoting

## 2.1  Defining and reading variables

A variable is a named box holding text. Assign with `name=value` and read it back with a `$` in front. The single most common beginner error is putting spaces around the `=`.

*vars.sh*

```bash
#!/usr/bin/env bash

name="Ada"          # correct: NO spaces around =
greeting='Hello'    # single or double quotes both work here

echo "$greeting, $name!"
```

```console
Hello, Ada!
```

> [!WARNING]
> **No spaces around =**
>
> `name = "Ada"` does **not** assign a variable. The shell reads it as: run the command `name` with arguments `=` and `Ada`. You'll get `name: command not found`. Write `name="Ada"` with no spaces.

## 2.2  Single quotes vs double quotes vs no quotes

This distinction trips up everyone. The rule: **double quotes expand variables; single quotes are literal.**

*quoting.sh*

```bash
fruit="mango"

echo "I like $fruit"    # double: variable expands
echo 'I like $fruit'    # single: printed literally
echo I like $fruit      # no quotes: expands, but risky
```

```console
I like mango
I like $fruit
I like mango
```

## 2.3  The golden rule: always quote your variables

When you use an unquoted `$var`, the shell splits it on spaces and expands wildcards like `*`. This causes bugs that only appear when the value contains a space — for example, a filename like `My Report.txt`.

*why-quote.sh*

```bash
file="My Report.txt"

# WRONG — the shell sees two arguments: 'My' and 'Report.txt'
rm $file

# RIGHT — one argument, exactly as intended
rm "$file"
```

```console
$ rm $file
rm: cannot remove 'My': No such file or directory
rm: cannot remove 'Report.txt': No such file or directory
```

> [!TIP]
> **The habit that prevents a hundred bugs**
>
> Quote every variable expansion by default: `"$var"`, `"$1"`, `"$(command)"`. Only drop the quotes when you *deliberately* want word-splitting (rare). If you install one habit from this whole course, make it this one.

## 2.4  Curly braces disambiguate names

*braces.sh*

```bash
name="file"

echo "$name_backup"     # looks for a variable called name_backup (empty!)
echo "${name}_backup"   # correct: expands name, then appends _backup
```

```console

file_backup
```

## 2.5  Environment variables and command scope

A plain variable exists only inside your script. `export` sends it into the environment so child processes inherit it. Useful ones already exist: `$HOME`, `$USER`, `$PATH`, `$PWD`.

*env.sh*

```bash
greeting="hi"          # shell variable (local to this shell)
export API_ENV="prod"  # environment variable (inherited by children)

echo "$HOME belongs to $USER"
```

```console
/home/devops belongs to devops
```

> [!IMPORTANT]
> **🎯 Try It Yourself — Exercise 2**
>
> Write `backup-name.sh` that sets `base="database"` and `date=$(date +%F)`, then prints a filename of the form `database-2026-07-07.sql.gz` using `${...}` braces where needed. Test that it still works if you change `base` to `web app` (with a space) — quote correctly so nothing breaks.

### Solution

*backup-name.sh*

```bash
#!/usr/bin/env bash
base="database"
date="$(date +%F)"

filename="${base}-${date}.sql.gz"
echo "Backup file: $filename"
```

```console
Backup file: database-2026-07-07.sql.gz
```

---

⬅️ [Topic 1: Environment, Your First Script, and chmod +x](01-environment-and-first-script.md)  |  ⬆️ [Phase 1 index](README.md)  |  [Topic 3: User Input and Command Arguments](03-input-and-arguments.md) ➡️
