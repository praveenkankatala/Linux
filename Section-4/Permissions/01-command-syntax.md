# 1. Command Syntax

[← Back to Home](README.md) | [Next: Help Commands →](02-help-commands.md)

---

Almost every Linux command follows the **same predictable pattern**. Once you understand
this structure, you can read and write any command with confidence.

## The pattern

```
command [options] [arguments]
```

| Part | What it is |
|------|------------|
| **command** | The program or utility you want to run. |
| **[options]** | Modify *how* the command behaves. Usually start with `-` (short) or `--` (long). |
| **[arguments]** | The target(s) the command acts on — files, directories, users, etc. |

> The square brackets `[ ]` just mean "optional." Many commands run fine with no options
> or arguments at all (for example, `ls` on its own).

---

## Worked example

```bash
ls -la /var/log
```

This lists **all** files (`-a`) in `/var/log` with **full details** (`-l`).

Breaking it down:

| Part | Role | Meaning |
|------|------|---------|
| `ls` | Command | The program that lists directory contents |
| `-la` | Options | `-l` = long/detailed view, `-a` = include hidden files |
| `/var/log` | Argument | The directory the command acts on |

---

## Anatomy at a glance

```
   ls          -la            /var/log
   ^            ^                ^
   |            |                |
 COMMAND      OPTIONS         ARGUMENT
 (what to    (how to         (what to
   run)       run it)         act on)
```

**In short:** `Command → Option(s) → Argument(s)`

---

## Short vs long options

Many options come in two forms that do the same thing:

| Short form | Long form | Meaning |
|------------|-----------|---------|
| `-a` | `--all` | Include hidden files |
| `-l` | (no long form here) | Long listing format |
| `-h` | `--human-readable` | Show sizes like `4K`, `2M` |

```bash
ls -a          # short
ls --all       # long — identical result
```

You can also **combine short options**: `-l` + `-a` + `-h` becomes `-lah`.

```bash
ls -lah /var/log
```

---

## Try It Yourself

Open a terminal and run each command. Predict the output before you press Enter.

```bash
# 1. Command with no options or arguments
ls

# 2. Command + option
ls -l

# 3. Command + combined options
ls -la

# 4. Command + options + argument
ls -lah /etc

# 5. The same option, short and long form
ls -a
ls --all
```

**Questions to check your understanding:**
1. In `cp -r folder1 folder2`, which part is the command, option, and arguments?
2. What is the difference between `ls -l -a` and `ls -la`?

<details>
<summary>Show answers</summary>

1. `cp` = command, `-r` = option (recursive), `folder1 folder2` = arguments.
2. Nothing — they are identical. Separate short options can be combined behind one `-`.

</details>

---

## Key Takeaways

- Every command follows: **command → options → arguments**.
- Options change behaviour and usually start with `-` or `--`.
- Arguments are what the command acts on.
- Short options can be combined: `-lah` = `-l -a -h`.

---

[← Back to Home](README.md) | [Next: Help Commands →](02-help-commands.md)
