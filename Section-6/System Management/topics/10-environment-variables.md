# 10. Environment Variables

[← Back to index](../README.md)

Environment variables are named values that every program inherits — the shell's "short-term memory" for configuration. By convention they're written in **UPPERCASE**.

## Common variables

| Variable | Description | Example |
|----------|-------------|---------|
| `$PATH` | Directories the shell searches for commands | `/usr/bin:/bin:/usr/local/bin` |
| `$HOME` | Your home directory | `/home/testuser` |
| `$USER` | Your username | `ec2-user` |
| `$SHELL` | Path to your shell | `/bin/bash` |
| `$PWD` | Present working directory | `/var/log` |
| `$LANG` | Language and character encoding | `en_US.UTF-8` |

## Working with variables

```bash
echo $PATH                       # view one variable
printenv                         # list ALL exported variables
MY_NAME="Girish"                 # set a variable (this session only)
export API_KEY="12345"           # make it available to child programs
export PATH=$PATH:/opt/app/bin   # add a folder to PATH
```

## The `$PATH` "command not found" trap

`$PATH` is the most important variable. When the shell sees a command, it looks through each folder in `$PATH` to find it. If a program is installed but you get **"command not found"**, its folder is probably missing from `$PATH`.

```bash
export PATH=$PATH:/opt/new_app/bin   # add the missing folder
```

> 💡 **Temporary vs permanent.** Anything you `export` in the terminal lasts only for that session. To make it permanent, add the `export` line to `~/.bashrc` (or `~/.bash_profile`) so it loads every time you open a shell.

## 🧪 Practice

1. View your PATH and home directory:
   ```bash
   echo $PATH
   echo $HOME
   ```
2. Create and export your own variable, then confirm a child process sees it:
   ```bash
   export GREETING="hello"
   bash -c 'echo $GREETING'      # prints: hello
   ```
3. Set one *without* export and see the difference:
   ```bash
   SECRET="hidden"
   bash -c 'echo $SECRET'        # prints nothing — not exported
   ```

**✅ Success check:** The exported variable is visible to the child shell; the non-exported one is not.

---
[← Previous: System Utilities](09-system-utilities.md) | [Next: Terminal & Session Management →](11-terminal-session-management.md)
