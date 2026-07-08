# 8. Access Control Lists (ACL)

[← Previous: chmod Numeric Mode](07-chmod-numeric-mode.md) | [Home](README.md) | [Next: Quick Reference →](09-quick-reference.md)

---

## What is ACL in Linux?

**ACL (Access Control List)** in Linux is an advanced permission mechanism that allows you
to set **fine-grained permissions** on files and directories.

Unlike traditional Linux permissions (`rwx` for **owner**, **group**, and **others**),
ACLs let you:

- Give **different permissions to multiple users**.
- Assign permissions to **multiple groups**.
- Control access in a far more **flexible** way.

---

## Why use ACL?

Traditional permissions are **limited** — there are only **three sets**: owner, group, and
others. That works until two different people each need a *different* level of access to the
**same** file.

With ACL, you can:

- Allow user **alice** to **read** a file,
- Allow user **bob** to **write** the same file,
- **While keeping the default group/others permissions intact.**

> The classic problem ACL solves: "I need to give *one specific person* access to *one
> specific file* — without adding them to a group or opening it up to everyone."

---

## Basic ACL commands

| Goal | Command |
|------|---------|
| Check if ACL is supported | `mount \| grep acl` |
| View the ACL of a file/directory | `getfacl filename` |
| Set ACL for a **user** | `setfacl -m u:alice:r file.txt` |
| Set ACL for a **group** | `setfacl -m g:developers:rw file.txt` |
| Remove a specific ACL entry | `setfacl -x u:alice file.txt` |
| Set a **default** ACL on a directory | `setfacl -d -m u:alice:rw project/` |

---

## Understanding the `setfacl` flags

| Flag / part | Meaning |
|-------------|---------|
| `-m` | **Modify** — add or change an ACL entry |
| `-x` | **Remove** a specific ACL entry |
| `-d` | **Default** — applies to *new* files created inside a directory |
| `u:name:perm` | ACL entry for a **user** (e.g. `u:alice:r`) |
| `g:name:perm` | ACL entry for a **group** (e.g. `g:devs:rw`) |

The `perm` part uses the same letters you already know: `r`, `w`, `x`.

---

## Full example — walk through it

```bash
# Give alice read, and bob read+write, on the same file
$ setfacl -m u:alice:r  report.txt
$ setfacl -m u:bob:rw   report.txt

# Check the result
$ getfacl report.txt
# file: report.txt
# owner: devops
user::rw-
user:alice:r--        <-- extra ACL entry just for alice
user:bob:rw-          <-- extra ACL entry just for bob
group::r--
mask::rw-
other::r--
```

Both alice and bob now have their **own** permissions on the file, and the group/other
permissions are untouched — something traditional `chmod` simply cannot do.

---

## How to spot a file that has ACLs

When a file has ACLs set, `ls -l` shows a **`+`** at the end of the permission string:

```bash
$ ls -l report.txt
-rw-r--r--+  1 devops staff  512 Jun 24 report.txt
#         ^
#         the + means "this file has extra ACL rules — run getfacl to see them"
```

---

## Default ACLs (for directories)

A **default ACL** on a directory automatically applies to **new files** created inside it —
great for shared project folders.

```bash
# Every new file alice creates (or that's created) in project/
# will automatically give alice read+write
$ setfacl -d -m u:alice:rw project/
```

---

## Removing ACLs

```bash
# Remove one entry (alice's access)
$ setfacl -x u:alice report.txt

# Remove ALL ACL entries from a file (back to normal permissions)
$ setfacl -b report.txt
```

---

## Try It Yourself

> These commands need users named `alice`/`bob` to exist, or you can substitute your own
> username. On some systems you may need `sudo` to add users.

```bash
# Create a test file
touch shared.txt

# View its (empty) ACL baseline
getfacl shared.txt

# Give your own user explicit read+write via ACL, then confirm
setfacl -m u:$(whoami):rw shared.txt
getfacl shared.txt

# Notice the + appear in ls -l
ls -l shared.txt

# Clean up — remove all ACLs
setfacl -b shared.txt
ls -l shared.txt        # the + is gone
```

**Question:** Your file is `-rw-r-----` and you need *just* one external user, `carol`, to
read it — without changing group or other. Which command?

<details>
<summary>Show answer</summary>

```bash
setfacl -m u:carol:r file
```
This adds a read entry only for `carol`, leaving group and other exactly as they were.

</details>

---

## Key Takeaways

- ACLs add **fine-grained**, per-user and per-group permissions beyond `rwx`.
- `getfacl` **views** ACLs; `setfacl` **sets** them.
- `-m` modifies, `-x` removes one entry, `-b` removes all, `-d` sets a default for new files.
- A `+` after the permission string in `ls -l` means the file has ACLs.
- Use ACLs when the basic owner/group/other model isn't flexible enough.

---

[← Previous: chmod Numeric Mode](07-chmod-numeric-mode.md) | [Home](README.md) | [Next: Quick Reference →](09-quick-reference.md)
