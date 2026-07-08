[🏠 Home](../README.md)  ·  [Phase 1 — The Basics](README.md)  ·  **Topic 1**

# Topic 1 — Environment, Your First Script, and chmod +x

## 1.1  Which shell am I running?

Before writing anything, confirm your environment. Open a terminal and run these one at a time.

*terminal commands*

```bash
# Which shell is my interactive session using?
echo "$SHELL"

# Which bash binary and version?
which bash
bash --version
```

```console
$ echo "$SHELL"
/bin/bash
$ which bash
/usr/bin/bash
$ bash --version
GNU bash, version 5.2.21(1)-release (x86_64-pc-linux-gnu)
```

> [!NOTE]
> On Amazon Linux 2023 and RHEL you will see a 5.x bash. On older systems you may see 4.x. Anything 4.0+ handles everything in this course.

## 1.2  The shebang line

A script's very first line tells the OS which interpreter to use. It starts with `#!` (called a *shebang*) followed by the path to the interpreter.

*the shebang*

```bash
#!/usr/bin/env bash
```

You will see two common styles. Both are acceptable; the `env` form is more portable because it finds bash wherever it lives in the user's `PATH`.

| Shebang | Meaning | When to use |
| --- | --- | --- |
| `#!/bin/bash` | Run with bash at this exact path | Fine on Linux where bash is always /bin/bash |
| `#!/usr/bin/env bash` | Find bash via PATH, then run | More portable (macOS, BSD, custom installs) |
| `#!/bin/sh` | Run with the system POSIX shell | Only when you deliberately avoid bash features |

## 1.3  Write and run your first script

Create a file called `hello.sh`. Use any editor — `nano hello.sh`, `vim hello.sh`, or VS Code.

*hello.sh*

```bash
#!/usr/bin/env bash
# hello.sh — my very first script

echo "Hello, shell!"
echo "Today is $(date +%A)"
```

Now try to run it directly. This will **fail**, and that failure is a teaching moment.

```console
$ ./hello.sh
bash: ./hello.sh: Permission denied
```

## 1.4  chmod +x — making it executable

The file exists but has no *execute* permission, so the OS refuses to run it. `chmod +x` adds that permission.

*terminal commands*

```bash
# Add the execute bit for owner, group, and others
chmod +x hello.sh

# Verify — note the x's in the permission string
ls -l hello.sh

# Now it runs
./hello.sh
```

```console
$ ls -l hello.sh
-rwxr-xr-x 1 devops devops 84 Jul  7 09:12 hello.sh
$ ./hello.sh
Hello, shell!
Today is Tuesday
```

> [!NOTE]
> **Reading the permission string**
>
> In `-rwxr-xr-x`, ignore the first character (file type). The next nine are three groups of three: **owner** (`rwx`), **group** (`r-x`), **others** (`r-x`). The `x` is what lets a file execute.

## 1.5  Three ways to run a script

| Command | Needs +x? | What it does |
| --- | --- | --- |
| `./hello.sh` | Yes | Runs via the shebang line in a new process |
| `bash hello.sh` | No | Explicitly hands the file to bash to read |
| `source hello.sh` or `. hello.sh` | No | Runs in your *current* shell (can change your session) |

> [!WARNING]
> **./ is required**
>
> Typing just `hello.sh` gives `command not found`. The shell only searches directories in `PATH` for commands, and the current directory `.` is (correctly) not on `PATH` for security reasons. The `./` prefix says "the script right here."

> [!IMPORTANT]
> **🎯 Try It Yourself — Exercise 1**
>
> Create a script `sysinfo.sh` that prints your username, the current directory, and the hostname, each on its own labelled line. Make it executable and run it with `./sysinfo.sh`.
>
> Hint: the commands `whoami`, `pwd`, and `hostname` exist.

### Solution

*sysinfo.sh*

```bash
#!/usr/bin/env bash
# sysinfo.sh

echo "User:      $(whoami)"
echo "Directory: $(pwd)"
echo "Host:      $(hostname)"
```

```console
$ chmod +x sysinfo.sh && ./sysinfo.sh
User:      devops
Directory: /home/devops
Host:      ip-10-0-1-42
```

---

⬅️ _Start of course_  |  ⬆️ [Phase 1 index](README.md)  |  [Topic 2: Variables and the Golden Rule of Quoting](02-variables-and-quoting.md) ➡️
