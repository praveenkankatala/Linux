# 4. Users & Permissions

[← Previous: Executing Multiple Commands](03-executing-multiple-commands.md) | [Home](README.md) | [Next: Viewing Permissions →](05-viewing-permissions-ls-l.md)

---

Linux is a **multi-user** system. It decides what you can and cannot do based on **who you
are** and the **permissions** set on each file and directory. This section explains the
model that everything else builds on.

---

## Who am I? — the `whoami` command

The `whoami` command prints the **current logged-in user's username**.

```bash
$ whoami
devops
```

Knowing who you are matters, because Linux grants or denies access based on your user
identity.

---

## Why permissions exist

Every file and directory in your account can be **protected from** — or **made accessible
to** — other users by changing its access permissions. **Every user is responsible for
controlling access to their own files.**

Permissions for a file or directory may be restricted by **type**.

---

## The 3 types of permission (`r`, `w`, `x`)

| Symbol | Permission | Meaning on a **file** | Meaning on a **directory** |
|--------|------------|-----------------------|-----------------------------|
| `r` | **Read** | View the file's contents | List the files inside |
| `w` | **Write** | Modify or delete the file | Add or remove files inside |
| `x` | **Execute** | Run the file as a program | Enter (`cd` into) the directory |

> **Important:** `x` means two different things! On a **file** it means "run this program."
> On a **directory** it means "you're allowed to enter it." A directory you can read but not
> execute is almost unusable.

---

## The 3 levels — who the permission applies to (`u`, `g`, `o`)

Each permission can be controlled separately for three groups of people.

| Symbol | Level | Who it means |
|--------|-------|--------------|
| `u` | **User** | Yourself — the owner of the file |
| `g` | **Group** | People in the same group / project |
| `o` | **Other** | Everyone else on the system |
| `a` | **All** | Shortcut for user + group + other |

---

## Putting the model together

There are **3 permission types** (`r w x`) and each can be set for **3 levels**
(`u g o`). That gives the familiar **9-character permission string** you'll read in the
next topic.

```
        USER        GROUP       OTHER
      ┌───────┐   ┌───────┐   ┌───────┐
      │ r w x │   │ r w x │   │ r w x │
      └───────┘   └───────┘   └───────┘
       yourself    your team   everyone
                                else
```

---

## Related identity commands (good to know)

| Command | What it shows |
|---------|---------------|
| `whoami` | Your current username |
| `id` | Your user ID, group ID, and all groups you belong to |
| `groups` | The groups your account is a member of |

```bash
$ id
uid=1000(devops) gid=1000(devops) groups=1000(devops),27(sudo)
```

---

## Try It Yourself

```bash
# Who am I?
whoami

# Full identity details
id

# Which groups am I in?
groups

# Create a file and see who owns it (details explained in the next topic)
touch myfile.txt
ls -l myfile.txt
```

**Question:** A teammate is in the same **group** as you. Which of `u`, `g`, `o`
controls what *they* can do to your file?

<details>
<summary>Show answer</summary>

`g` (group). Your teammate isn't you (`u`) and isn't a total stranger (`o`) — they share
your group, so the **group** permissions apply to them.

</details>

---

## Key Takeaways

- `whoami` tells you your current username.
- Permissions come in **3 types**: read (`r`), write (`w`), execute (`x`).
- Permissions apply at **3 levels**: user (`u`), group (`g`), other (`o`).
- `x` means "run" on a file but "enter" on a directory.
- `3 types × 3 levels` = the 9-character permission string.

---

[← Previous: Executing Multiple Commands](03-executing-multiple-commands.md) | [Home](README.md) | [Next: Viewing Permissions →](05-viewing-permissions-ls-l.md)
