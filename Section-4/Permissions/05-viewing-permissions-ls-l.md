# 5. Viewing Permissions (`ls -l`)

[← Previous: Users & Permissions](04-users-and-permissions.md) | [Home](README.md) | [Next: chmod Symbolic Mode →](06-chmod-symbolic-mode.md)

---

File or directory permissions can be displayed by running the **`ls -l`** command. The
output begins with a **10-character permission string** that tells you exactly who can do
what.

```bash
$ ls -l report.txt
-rwxrw-r--  1  devops  staff  2048  Jun 24 10:30  report.txt
```

---

## Reading the permission string

The **first character** tells you the *type*. The remaining **nine** are three groups of
`rwx` — one group each for **user**, **group**, and **other**.

```
   -      rwx      rwx      rwx
  ┌─┐    ┌───┐    ┌───┐    ┌───┐
  │ │    │   │    │   │    │   │
  └─┘    └───┘    └───┘    └───┘
  file    user    group    other
  type
```

| Block | Position | Controls |
|-------|----------|----------|
| `-` (or `d`, `l`) | 1st char | **File type:** `-` = file, `d` = directory, `l` = link |
| `rwx` | chars 2–4 | Permissions for the **User** (owner) |
| `rwx` | chars 5–7 | Permissions for the **Group** |
| `rwx` | chars 8–10 | Permissions for **Other** (everyone else) |

> A dash `-` in any slot means "that permission is **not** granted." So `r--` means read
> only; `rw-` means read and write but not execute.

---

## Worked reading

Let's decode `-rwxrw-r--`:

| Chunk | Applies to | Meaning |
|-------|-----------|---------|
| `-` | type | It is a regular **file** |
| `rwx` | user (owner) | Can **read, write, and execute** |
| `rw-` | group | Can **read and write**, but not execute |
| `r--` | other | Can **only read** |

---

## The full `ls -l` line explained

```
-rwxrw-r--  1  devops  staff  2048  Jun 24 10:30  report.txt
│           │  │       │      │     │             │
│           │  │       │      │     │             └─ name
│           │  │       │      │     └─ last modified date/time
│           │  │       │      └─ size in bytes
│           │  │       └─ group that owns it
│           │  └─ user that owns it
│           └─ number of hard links
└─ type + permissions (the 10 characters)
```

---

## Spotting file types quickly

```bash
$ ls -l
drwxr-xr-x  2 devops staff 4096 Jun 24 09:00 projects     # d = directory
-rw-r--r--  1 devops staff  128 Jun 24 09:01 notes.txt    # - = file
lrwxrwxrwx  1 devops staff    8 Jun 24 09:02 shortcut -> notes.txt   # l = link
```

---

## Try It Yourself

```bash
# 1. Create a file and view its permissions
touch demo.txt
ls -l demo.txt

# 2. Create a directory and compare — notice the leading 'd'
mkdir demofolder
ls -l

# 3. List a system directory with long format
ls -l /etc | head

# 4. Read the string out loud for each line:
#    "type - user rwx - group rwx - other rwx"
```

**Decode these strings:**
1. `drwxr-x---`
2. `-r--r--r--`

<details>
<summary>Show answers</summary>

1. A **directory**. Owner: full access (`rwx`). Group: read + enter (`r-x`). Other: nothing (`---`).
2. A **file** that everyone can **read**, but nobody can write or execute (`r--` for all three).

</details>

---

## Key Takeaways

- `ls -l` shows a 10-character permission string.
- 1st character = type (`-` file, `d` directory, `l` link).
- Next 9 = three `rwx` blocks for **user / group / other**.
- A `-` in any slot means that permission is missing.

---

[← Previous: Users & Permissions](04-users-and-permissions.md) | [Home](README.md) | [Next: chmod Symbolic Mode →](06-chmod-symbolic-mode.md)
