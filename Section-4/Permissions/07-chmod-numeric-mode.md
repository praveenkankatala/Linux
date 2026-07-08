# 7. Changing Permissions — Numeric (Octal) Mode

[← Previous: chmod Symbolic Mode](06-chmod-symbolic-mode.md) | [Home](README.md) | [Next: Access Control Lists →](08-access-control-lists-acl.md)

---

Permissions for a file or directory can also be assigned **numerically**. Each permission
is given a number, and you **add them up** for each level. This is the fastest way to set
all permissions at once.

```
chmod  4 4 4   filename
```

---

## Which digit controls which level

The three digits are always in the order **User → Group → Other**.

```
   chmod   7     5     5     filename
           │     │     │
           │     │     └──►  Other
           │     └────────►  Group
           └──────────────►  User
```

> **Order matters!** `chmod 750` and `chmod 057` are completely different. Always read the
> digits left to right as **user, group, other**.

---

## How the numbers are built

Each digit is the **sum** of the permissions you want:

| Permission | Value |
|------------|-------|
| Read (`r`) | **4** |
| Write (`w`) | **2** |
| Execute (`x`) | **1** |
| None (`-`) | 0 |

Add them to get a single digit from 0 to 7:

| Number | Permission Type | Symbol (`rwx`) | Meaning |
|:------:|-----------------|:--------------:|---------|
| **0** | No permission | `---` | No read, no write, no execute |
| **1** | Execute only | `--x` | Can run / execute the file |
| **2** | Write only | `-w-` | Can modify / write the file |
| **3** | Write + Execute | `-wx` | Can write and execute |
| **4** | Read only | `r--` | Can read the file |
| **5** | Read + Execute | `r-x` | Can read and execute |
| **6** | Read + Write | `rw-` | Can read and write |
| **7** | Read + Write + Execute | `rwx` | Full permission |

**Quick math:** `7 = 4+2+1`, `6 = 4+2`, `5 = 4+1`, `4 = 4`.

---

## Putting it together — `chmod 755`

`chmod 755 script.sh` is one of the most common settings:

```
   7        5        5
  rwx      r-x      r-x
 (user)  (group)  (other)

  7 = 4+2+1 = read + write + execute
  5 = 4+0+1 = read +         execute
```

So the **owner can do everything**, while the **group and others can read and run** the
script but **not change** it.

---

## Handy everyday examples

| Command | Result (u / g / o) | Typical use |
|---------|:------------------:|-------------|
| `chmod 644 file` | `rw- r-- r--` | Normal document: owner edits, others read |
| `chmod 600 file` | `rw- --- ---` | Private file: only the owner can read/write |
| `chmod 755 dir` | `rwx r-x r-x` | Programs and directories everyone can use |
| `chmod 700 dir` | `rwx --- ---` | Private folder: only you can enter it |
| `chmod 777 file` | `rwx rwx rwx` | Everyone full access — **use with care!** |

> **Security note:** `777` gives *everyone* full control, including the ability to modify or
> delete the file. Avoid it on real systems — prefer the least access that still works.

---

## Symbolic vs numeric — same result, two styles

These two commands do exactly the same thing:

```bash
chmod u=rwx,g=rx,o=rx script.sh    # symbolic
chmod 755 script.sh                # numeric — shorter
```

- **Numeric** is fastest when setting **all** permissions at once.
- **Symbolic** is best for **tweaking one thing** without touching the rest.

---

## Try It Yourself

```bash
# Setup
touch app.sh
ls -l app.sh

# 1. Owner full, group+other read/execute
chmod 755 app.sh
ls -l app.sh          # -rwxr-xr-x

# 2. Standard document permissions
chmod 644 app.sh
ls -l app.sh          # -rw-r--r--

# 3. Private to you only
chmod 600 app.sh
ls -l app.sh          # -rw-------

# 4. Directory only you can enter
mkdir secret
chmod 700 secret
ls -ld secret         # drwx------
```

**Convert these to numbers:**
1. `rwxr-xr--`
2. `rw-rw----`

<details>
<summary>Show answers</summary>

1. `rwx`=7, `r-x`=5, `r--`=4 → **`754`**
2. `rw-`=6, `rw-`=6, `---`=0 → **`660`**

</details>

---

## Key Takeaways

- Numeric mode uses one digit per level: **user, group, other**.
- Values: **read = 4, write = 2, execute = 1** — add them up.
- `7` = full (`rwx`), `6` = `rw-`, `5` = `r-x`, `4` = `r--`, `0` = none.
- `chmod 755` and `chmod 644` are the two you'll use most.
- Use numeric to set everything at once; symbolic to change one thing.

---

[← Previous: chmod Symbolic Mode](06-chmod-symbolic-mode.md) | [Home](README.md) | [Next: Access Control Lists →](08-access-control-lists-acl.md)
