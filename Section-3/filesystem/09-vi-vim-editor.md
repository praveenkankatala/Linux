# 09 — The `vi` / `vim` Editor

On a bare production server there's often no GUI, no VS Code, and sometimes not even `nano`. But `vi` is **always** there — it's mandated by the POSIX standard, so every Linux and Unix box ships it. When an instance is broken and SSH is your only way in, `vi` is the editor you'll actually use.

On modern RHEL / Amazon Linux, `vi` is usually `vim` ("vi improved"): same core, plus syntax highlighting, undo history, and split windows. Everything here works in both.

> **⭐ The one rule for beginners:** `vi` starts in **Normal mode**, where letters are *commands*, not text. If typing produces chaos, press **`Esc`**, then type **`:q!`** and Enter to quit without saving. You can always escape.

- [The mental model: modes](#the-mental-model-modes)
- [Opening files](#opening-files)
- [Insert mode — entering text](#insert-mode--entering-text)
- [Moving around](#moving-around-normal-mode)
- [Editing text](#editing-text-the-real-power)
- [Search & replace](#search--replace)
- [Saving & quitting](#saving--quitting)
- [Beyond the basics](#beyond-the-basics)
- [Guided hands-on walkthrough](#guided-hands-on-walkthrough)

---

## The mental model: modes

`vi` is **modal**. The same keystroke does different things depending on the mode you're in. This is the one idea that confuses newcomers — and the one idea that makes `vi` fast once it clicks.

| Mode | You are... | Enter it by | Leave it by |
|------|-----------|-------------|-------------|
| **Normal** (Command) | Navigating & running edit commands | This is the default; press `Esc` | Press `i`, `:`, etc. |
| **Insert** | Typing text like a normal editor | `i` `a` `o` (and friends) | Press `Esc` |
| **Visual** | Selecting a block of text | `v` `V` `Ctrl-v` | Press `Esc` |
| **Command-line** (ex) | Running `:` commands (save, quit, replace) | `:` `/` `?` | `Enter` or `Esc` |

> **⭐ Esc is home.** Whenever you're unsure what mode you're in, press `Esc` to return to Normal mode. Most "vi is impossible" moments are just being stuck in the wrong mode.

The flow you'll repeat all day:

```
Open file  ->  [Normal]  move to the spot
           ->  press i    [Insert]  type your text
           ->  press Esc  [Normal]  back to safety
           ->  type :wq   save and quit
```

---

## Opening files

```bash
$ vi app.conf          # open existing, or create if it doesn't exist
$ vi +45 app.conf      # open and jump to line 45
$ vi + app.conf        # open at the LAST line
$ vi +/error app.log   # open at the first line matching 'error'
$ vi file1 file2       # open several files (use :n to switch)
$ view app.conf        # open READ-ONLY (safe inspection)
```

---

## Insert mode — entering text

Several keys enter Insert mode; they differ only in **where** the cursor lands. Picking the right one saves a movement.

| Key | Enters Insert mode and... |
|-----|---------------------------|
| `i` | **i**nserts *before* the cursor |
| `a` | **a**ppends *after* the cursor |
| `I` | inserts at the first non-blank of the line |
| `A` | appends at the **end** of the line |
| `o` | **o**pens a new line **below** and starts typing |
| `O` | **O**pens a new line **above** |
| `s` | deletes the character under the cursor, then inserts |
| `cc` | **c**hanges (clears) the whole line, then inserts |

> **Tip:** `A` (append at end of line) and `o` (open line below) are the two you'll reach for most.

**Hands-on:**

```bash
$ vi hello.txt
# vi opens in Normal mode. Then press:
#   i            -> now in INSERT mode (bottom shows -- INSERT --)
#   type: Hello, this is my first vi file.
#   Esc          -> back to NORMAL mode
#   :wq          -> write and quit
```

---

## Moving around (Normal mode)

Keep your hands on the home row — `hjkl` instead of arrow keys. All movement below happens in **Normal mode** (press `Esc` first).

### Character & line

| Key | Moves the cursor |
|-----|------------------|
| `h` `j` `k` `l` | left, down, up, right |
| `0` | to the **start** of the line |
| `^` | to the first **non-blank** character |
| `$` | to the **end** of the line |
| `w` / `b` | forward a **w**ord / **b**ack a word |
| `e` | to the **e**nd of the current/next word |

### Screen & file

| Key | Jumps to |
|-----|----------|
| `gg` | the **first** line of the file |
| `G` | the **last** line of the file |
| `45G` or `:45` | line number **45** |
| `Ctrl-f` / `Ctrl-b` | **f**orward / **b**ack one full screen |
| `Ctrl-d` / `Ctrl-u` | **d**own / **u**p half a screen |
| `{` / `}` | previous / next paragraph |
| `%` | jump to the **matching bracket** `()` `{}` `[]` |

> **Tip:** Movement accepts a **count**: `5j` moves down 5 lines, `3w` jumps 3 words, `10G` goes to line 10. This "number + motion" idea powers everything in vi.

---

## Editing text (the real power)

`vi`'s editing follows a grammar: **operator + motion**. You press a verb (delete, change, yank) then a movement, and the verb applies over that movement.

### Delete

| Key | Deletes |
|-----|---------|
| `x` | the single character under the cursor |
| `dd` | the whole current line |
| `dw` | from cursor to the start of the next word |
| `d$` (or `D`) | from cursor to the end of the line |
| `dG` | from the current line to the end of the file |
| `3dd` | three lines (count + `dd`) |

### Change (delete + enter Insert in one move)

| Key | Changes |
|-----|---------|
| `cw` | the word (deletes it, drops you into Insert) |
| `cc` (or `S`) | the entire line |
| `c$` (or `C`) | from cursor to end of line |
| `r` | replace exactly **one** character (stays in Normal) |
| `R` | overtype mode — replace as you type until `Esc` |
| `~` | toggle the case of the character under the cursor |
| `J` | **J**oin the next line onto the current one |

### Copy (yank) & paste

`vi` calls copying **yanking**. Deleted text also goes to the same register, so `dd` then `p` *moves* a line.

| Key | Action |
|-----|--------|
| `yy` | yank (copy) the current line |
| `3yy` | yank three lines |
| `yw` | yank a word |
| `p` | paste **after** the cursor / below the line |
| `P` | paste **before** the cursor / above the line |
| `ddp` | swap the current line with the one below |

### Undo & redo — the safety net

| Key | Action |
|-----|--------|
| `u` | undo the last change (repeat to go further back) |
| `Ctrl-r` | redo |
| `.` | **repeat the last change** (the "dot command") |
| `U` | undo all changes on the current line |

> **⭐ The dot command `.`** repeats your last edit. Change one word with `cwNEW`+`Esc`, jump to the next occurrence with `n`, press `.` to repeat. It's the fastest way to make the same edit in many places.

### The grammar in action

Once you see the pattern, you can invent commands you were never taught:

| You want to... | Command | How it reads |
|----------------|---------|--------------|
| Delete 4 lines | `4dd` | delete + 4 lines |
| Change to end of line | `c$` | change + to-end-of-line |
| Delete up to next `/` | `dt/` | delete + till the char `/` |
| Yank 3 words | `3yw` | 3 + yank + word |
| Delete inside quotes | `di"` | delete + inside + quotes (vim) |

---

## Search & replace

### Searching

| Key | Action |
|-----|--------|
| `/text` | search **forward** for `text`, then Enter |
| `?text` | search **backward** |
| `n` | jump to the **next** match |
| `N` | jump to the **previous** match |
| `*` | search for the word under the cursor |

### Substitution — the `:s` command

`vi`'s find-and-replace, run from command-line mode. General form:

```
:[range]s/old/new/[flags]
```

| Command | Effect |
|---------|--------|
| `:s/foo/bar/` | replace the **first** `foo` on the current line |
| `:s/foo/bar/g` | replace **all** `foo` on the current line |
| `:%s/foo/bar/g` | replace all `foo` in the **whole file** (`%` = all lines) |
| `:%s/foo/bar/gc` | same, but **confirm** each change |
| `:%s/foo/bar/gi` | case-**i**nsensitive |
| `:10,20s/foo/bar/g` | only within lines 10–20 |

When you use `/gc`, vi asks at each match: `y` = yes, `n` = skip, `a` = all remaining, `q` = quit.

### The `:g` global command

Run a command on every line matching a pattern.

```
:g/^#/d       delete every line that starts with # (all comments)
:g/DEBUG/d    delete every line containing DEBUG
```

> **Tip:** Add `:set hlsearch` and `:set incsearch` to highlight matches and jump as you type. Clear the highlight with `:noh`.

---

## Saving & quitting

Press `Esc` first, then type these (they start with `:` and end with Enter).

| Command | Action |
|---------|--------|
| `:w` | **w**rite (save), stay in the file |
| `:q` | **q**uit (only if no unsaved changes) |
| `:wq` | write **and** quit |
| `:x` or `ZZ` | write if changed, then quit |
| `:q!` | quit **discarding** all changes (the escape hatch) |
| `:w!` | force write (e.g. a read-only file you own) |
| `:w newfile` | save a copy as `newfile` (Save As) |
| `:e!` | reload from disk, discarding edits |

> **⚠️ Watch out:** If `:q` refuses with *"No write since last change"*, vi is protecting unsaved work. Either save with `:wq`, or deliberately discard with `:q!`.

> **Note:** Opened a root-owned file but forgot `sudo`? Don't lose your work — in vim run `:w !sudo tee %`, then `:e!` to reload.

---

## Beyond the basics

### Visual mode — select, then act

| Key | Selection |
|-----|-----------|
| `v` | character-wise (move to extend) |
| `V` | line-wise (whole lines) |
| `Ctrl-v` | block/column (rectangular) |

```
V j j d          select 3 lines, delete them
v e c NEW Esc    select a word, change it
Ctrl-v (rows) I # Esc   prepend # to each selected line (comment a block)
```

### Multiple files & splits

| Command | Action |
|---------|--------|
| `:n` / `:N` | next / previous file |
| `:e file` | open another file in the window |
| `:sp file` | horizontal **sp**lit |
| `:vsp file` | **v**ertical split (side by side) |
| `Ctrl-w w` | move between split windows |
| `:ls` then `:b N` | list buffers, switch to buffer N |

### Handy `:set` options

| Command | Effect |
|---------|--------|
| `:set number` | show line numbers |
| `:set paste` | stop auto-indent mangling pasted blocks |
| `:syntax on` | enable syntax highlighting |
| `:set ignorecase` | case-insensitive searching |
| `:set list` | reveal tabs and line endings |

### Make it permanent: `~/.vimrc`

Options set with `:set` last only for the session. Put them in `~/.vimrc` and they load every time.

```vim
" ~/.vimrc — a sensible starter for server work
syntax on
set number
set hlsearch
set incsearch
set ignorecase
set smartcase
set autoindent
set tabstop=4
set expandtab
set ruler
```

> **Tip:** `set expandtab` + `set tabstop=4` inserts spaces instead of tabs — essential when editing **YAML** (Ansible playbooks, Kubernetes manifests), where a literal tab is a syntax error.

---

## Guided hands-on walkthrough

Do this once, start to finish. It exercises every core skill in ten minutes.

1. **Create a practice file** from the shell:
   ```bash
   $ cat > server.conf <<'EOF'
   # server config
   host = localhost
   port = 8080
   debug = true
   workers = 2
   # end
   EOF
   ```
2. **Open it:** `vi server.conf`
3. **Navigate:** press `G` (last line), then `gg` (top), then `3G` (line 3).
4. **Change a value:** search with `/8080` + Enter, then `cw`, type `9090`, press `Esc`.
5. **Add a line:** press `G`, put the cursor on `workers = 2`, press `o`, type `timeout = 30`, press `Esc`.
6. **Delete comments:** run `:g/^#/d`.
7. **Global replace:** `:%s/true/false/g`.
8. **Undo / redo:** press `u` a few times, then `Ctrl-r`.
9. **Save & quit:** press `Esc`, then `:wq`.
10. **Verify:** `cat server.conf`.

> **⭐ Bail-out at any point:** press `Esc` then `:q!` — you exit with the file untouched on disk and can start over.

---

### ✅ The 90% you'll use daily

`i` to type · `Esc` to stop · `:wq` to save-quit · `:q!` to bail · `dd` to delete a line · `/text` to find · `:%s/a/b/g` to replace · `u` to undo.

---

⬅️ [Prev: Editing Files](08-file-editing.md) | 🏠 [Index](README.md) | ➡️ [Cheat Sheet](CHEATSHEET.md)
