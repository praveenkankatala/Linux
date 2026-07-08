# 3. Executing Multiple Commands

[← Previous: Help Commands](02-help-commands.md) | [Home](README.md) | [Next: Users & Permissions →](04-users-and-permissions.md)

---

You can run **several commands on a single line**. The separator you choose decides whether
the next command runs no matter what, or only when the previous one succeeds. This is the
foundation of **scripting, automation, and everyday tasks**.

---

## Run commands one after another — `;`

The semicolon (`;`) runs each command **in order**. Every command runs **independently** —
even if one fails, the next one still runs.

```bash
ls ; pwd ; whoami
```

- Each command works independently.
- If one command fails, the next one **will still run**.

```bash
# Even though the first command fails (no such folder),
# pwd and whoami still run:
cd /does-not-exist ; pwd ; whoami
```

---

## Command chaining operators

Beyond simply listing commands, Linux lets you **chain** commands so their behaviour
depends on each other.

| Operator | Name | Behaviour |
|----------|------|-----------|
| `;` | Semicolon | Run all commands in sequence, regardless of success or failure |
| `&&` | AND | Run the next command **only if** the previous one **succeeded** |
| `\|\|` | OR | Run the next command **only if** the previous one **failed** |
| `\|` | Pipe | Send the **output** of one command as **input** to the next |

> **How "success" is decided:** every command returns an exit code. `0` means success;
> anything else means failure. `&&` and `||` react to that code.

---

### `&&` — run next only if the previous succeeded

```bash
mkdir project && cd project
```

`cd project` runs **only if** `mkdir project` succeeded. Perfect for "do this, then that"
steps that must not run out of order.

### `||` — run next only if the previous failed

```bash
ping -c1 example.com || echo "Site is unreachable"
```

The `echo` runs **only if** the `ping` failed — a simple fallback / error message.

### `|` — the pipe: pass output into the next command

```bash
ls -l | grep ".txt"
```

`ls -l` produces a list, and `grep` filters it to show only lines containing `.txt`.
Pipes let you build powerful one-liners by combining small commands.

```bash
# Count how many .txt files are in the current folder
ls -l | grep ".txt" | wc -l
```

---

## Comparing them side by side

```
A ; B     ->  run A, then run B     (always)
A && B    ->  run A, then run B      only if A worked
A || B    ->  run A, then run B      only if A failed
A | B     ->  feed A's output into B as its input
```

---

## Why it matters

Chaining commands powers three everyday uses:

- **Scripting** — building reusable sequences of commands.
- **Automation** — letting tasks run without manual steps.
- **Daily tasks** — saving time on repetitive work.

---

## Try It Yourself

```bash
# 1. Sequence with ; (all run no matter what)
echo "one" ; echo "two" ; echo "three"

# 2. && stops the chain if a step fails
mkdir demo && cd demo && echo "Now inside: $(pwd)"

# 3. || provides a fallback
cat missing-file.txt || echo "That file does not exist"

# 4. Pipe filters output
ls /etc | grep "conf"

# 5. Combine them
mkdir logs && ls -l | grep "logs"
```

**Predict the output:**
1. `false && echo "hi"` — does `hi` print?
2. `false || echo "hi"` — does `hi` print?

<details>
<summary>Show answers</summary>

`false` is a command that always fails.
1. **No** — `&&` only continues on success, and `false` failed.
2. **Yes** — `||` runs the next command on failure, and `false` failed.

</details>

---

## Key Takeaways

- `;` runs everything in order, pass or fail.
- `&&` runs the next command only on success.
- `||` runs the next command only on failure.
- `|` (pipe) feeds one command's output into the next.
- These operators are the building blocks of scripts and automation.

---

[← Previous: Help Commands](02-help-commands.md) | [Home](README.md) | [Next: Users & Permissions →](04-users-and-permissions.md)
