# 9. Quick Reference Cheat Sheet

[← Previous: Access Control Lists](08-access-control-lists-acl.md) | [Home](README.md)

---

Everything from this guide on one page. Bookmark it.

## Command structure

```
command [options] [arguments]
```

## Getting help

| Task | Command |
|------|---------|
| One-line description | `whatis ls` |
| Quick option list | `ls --help` |
| Full manual (quit with `q`) | `man ls` |

## Identity

| Task | Command |
|------|---------|
| Current username | `whoami` |
| User/group IDs and groups | `id` |
| Your groups | `groups` |

## Running multiple commands

| Operator | Meaning | Example |
|:--------:|---------|---------|
| `;` | Run all, pass or fail | `ls ; pwd ; whoami` |
| `&&` | Run next only if previous succeeds | `mkdir x && cd x` |
| `\|\|` | Run next only if previous fails | `ping -c1 host \|\| echo down` |
| `\|` | Pipe output into next command | `ls -l \| grep .txt` |

## Viewing permissions

```bash
ls -l           # long listing with permission strings
ls -ld folder   # show a directory's own permissions, not its contents
```

Permission string layout:

```
-  rwx  rwx  rwx
│  │    │    └─ other
│  │    └────── group
│  └─────────── user (owner)
└────────────── type: - file, d directory, l link
```

## The permission letters

| Letter | On a file | On a directory |
|:------:|-----------|----------------|
| `r` | read contents | list contents |
| `w` | modify/delete | add/remove files |
| `x` | run as program | enter (`cd`) |

## Levels

| `u` | `g` | `o` | `a` |
|-----|-----|-----|-----|
| user | group | other | all |

## chmod — symbolic mode

| Command | Effect |
|---------|--------|
| `chmod u+x file` | Add execute for owner |
| `chmod g-w file` | Remove write from group |
| `chmod o=r file` | Set other to read-only exactly |
| `chmod a+rwx file` | Add all permissions to all |
| `chmod u+x,g-w file` | Multiple changes at once |

Operators: `+` add · `-` remove · `=` set exactly

## chmod — numeric (octal) mode

Values: **read = 4 · write = 2 · execute = 1** (add them up)

| Digit | rwx | Digit | rwx |
|:-----:|:---:|:-----:|:---:|
| 0 | `---` | 4 | `r--` |
| 1 | `--x` | 5 | `r-x` |
| 2 | `-w-` | 6 | `rw-` |
| 3 | `-wx` | 7 | `rwx` |

Order is always **user · group · other**:

| Command | Result | Use |
|---------|:------:|-----|
| `chmod 644 file` | `rw- r-- r--` | Normal document |
| `chmod 600 file` | `rw- --- ---` | Private file |
| `chmod 755 file` | `rwx r-x r-x` | Scripts / directories |
| `chmod 700 dir` | `rwx --- ---` | Private folder |
| `chmod 777 file` | `rwx rwx rwx` | Everyone full (avoid!) |

## ACL — fine-grained permissions

| Task | Command |
|------|---------|
| Check ACL support | `mount \| grep acl` |
| View ACLs | `getfacl file` |
| Give a user access | `setfacl -m u:alice:r file` |
| Give a group access | `setfacl -m g:devs:rw file` |
| Default ACL on a dir | `setfacl -d -m u:alice:rw dir/` |
| Remove one entry | `setfacl -x u:alice file` |
| Remove all ACLs | `setfacl -b file` |

A `+` after the permissions in `ls -l` means the file has ACLs:
`-rw-r--r--+`

---

## 60-second mental model

```
WHO am I?            ->  whoami
WHAT can I do?       ->  ls -l   (read the rwx string)
CHANGE it (letters)  ->  chmod g+w file
CHANGE it (numbers)  ->  chmod 755 file
ONE person only      ->  setfacl -m u:name:rw file
NEED help?           ->  man command
```

---

[← Previous: Access Control Lists](08-access-control-lists-acl.md) | [Home](README.md)
