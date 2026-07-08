# 6. Changing Permissions — Symbolic Mode

[← Previous: Viewing Permissions](05-viewing-permissions-ls-l.md) | [Home](README.md) | [Next: chmod Numeric Mode →](07-chmod-numeric-mode.md)

---

The command to change permissions is **`chmod`** (short for **change mode**). *Symbolic
mode* uses **letters** and the `+` / `-` / `=` signs to add, remove, or set permissions —
it reads almost like plain English.

> Want the full list of every `chmod` option? Run `man chmod` to open its manual page.

---

## The building blocks

A symbolic `chmod` command combines three pieces: **who**, **what operation**, and
**which permission**.

```
chmod  [who][operator][permission]  filename
```

| Levels (who) | Operators (what) | Permissions (which) |
|--------------|------------------|---------------------|
| `u` = user | `+` add permission | `r` = read |
| `g` = group | `-` remove permission | `w` = write |
| `o` = other | `=` set exactly (overwrite) | `x` = execute |
| `a` = all | | |

---

## Common commands

| Command | What it does |
|---------|--------------|
| `chmod g-w filename` | Removes **write** from the **group** (`g` = group, `w` = write) |
| `chmod g+w filename` | Same idea, but **adds** write to the group |
| `chmod a-r filename` | Removes **read** from **all** (user + group + other) |
| `chmod a-r-w-x filename` | Removes **all** permissions from **all** at once |
| `chmod a+r+w+x filename` | **Adds all** permissions to **all** at once |

> `a` is a handy shortcut. `chmod a+x file` is the same as `chmod ugo+x file`.

---

## `+`, `-`, and `=` — what's the difference?

- `+` **adds** a permission, leaving the others untouched.
- `-` **removes** a permission, leaving the others untouched.
- `=` **sets exactly** what you list and **clears the rest**.

```bash
chmod u+x  file    # add execute for the owner
chmod g-w  file    # remove write from the group
chmod o=r  file    # other gets read ONLY (write/execute cleared)
```

---

## Example in action

```bash
$ ls -l notes.txt
-rw-rw-r--  1 devops staff  120  notes.txt

$ chmod g-w notes.txt        # take write away from the group

$ ls -l notes.txt
-rw-r--r--  1 devops staff  120  notes.txt
```

Notice how the **group block** changed from `rw-` to `r--` — write was removed, everything
else stayed the same.

---

## Combining changes in one command

You can separate multiple changes with commas:

```bash
chmod u+x,g-w,o-r script.sh
#     |    |    |
#     |    |    └─ remove read from other
#     |    └─ remove write from group
#     └─ add execute for user
```

---

## Making a script runnable (the classic use case)

```bash
$ ls -l backup.sh
-rw-r--r--  1 devops staff  310 backup.sh   # not executable yet

$ chmod u+x backup.sh        # give the owner execute permission

$ ls -l backup.sh
-rwxr--r--  1 devops staff  310 backup.sh   # now the owner can run it

$ ./backup.sh                # run it
```

---

## Try It Yourself

```bash
# Setup
touch practice.txt
ls -l practice.txt

# 1. Add execute for the owner
chmod u+x practice.txt
ls -l practice.txt

# 2. Remove read from others
chmod o-r practice.txt
ls -l practice.txt

# 3. Give everyone read + write in one go
chmod a+rw practice.txt
ls -l practice.txt

# 4. Set 'other' to read-only exactly (clears the rest for other)
chmod o=r practice.txt
ls -l practice.txt
```

**Challenge:** Starting from `-rw-rw-rw-`, write a single `chmod` command that leaves it as
`-rwxr--r--` (owner gains execute, group and other lose write).

<details>
<summary>Show answer</summary>

```bash
chmod u+x,go-w file
```
`u+x` adds execute for the owner; `go-w` removes write from both group and other.

</details>

---

## Key Takeaways

- `chmod` = **change mode** (change permissions).
- Format: **who** (`u/g/o/a`) + **operator** (`+`/`-`/`=`) + **permission** (`r/w/x`).
- `+` adds, `-` removes, `=` sets exactly (and clears the rest).
- Combine changes with commas: `chmod u+x,g-w file`.
- `man chmod` lists every option.

---

[← Previous: Viewing Permissions](05-viewing-permissions-ls-l.md) | [Home](README.md) | [Next: chmod Numeric Mode →](07-chmod-numeric-mode.md)
