# 08 — Editing Files

Before diving into `vi`, know the quick ways to write and change files straight from the shell. For one-liners and scripts these are often faster than opening any editor.

- [Create / overwrite with redirection](#create--overwrite-with-redirection)
- [Here-documents](#here-documents--write-a-whole-block)
- [tee — write and see at once](#tee--write-and-see-at-the-same-time)
- [sed — edit in place, no interaction](#sed--edit-a-file-in-place-non-interactive)
- [nano — the friendly editor](#nano--the-beginner-friendly-editor)

---

## Create / overwrite with redirection

```bash
# write a single line (OVERWRITES the file)
$ echo "server_port=8080" > app.conf

# APPEND a line (keeps existing content)
$ echo "log_level=info" >> app.conf

# multi-line with printf (\n = newline)
$ printf "line1\nline2\nline3\n" > notes.txt
```

---

## Here-documents — write a whole block

A here-doc feeds everything up to the marker into a command. Quoting the marker (`'EOF'`) stops the shell from expanding `$variables` inside the block.

```bash
$ cat > nginx.conf <<'EOF'
server {
    listen 80;
    root /var/www/html;
}
EOF
# nginx.conf now contains those lines exactly (no variable expansion)
```

---

## tee — write and see at the same time

`tee` reads stdin, writes it to a file **and** passes it through to the screen. It's the correct way to write **root-owned** files.

```bash
# useful when you need sudo to write a protected file
$ echo "net.ipv4.ip_forward=1" | sudo tee /etc/sysctl.d/99-fwd.conf
$ echo "extra" | sudo tee -a /etc/hosts     # -a = append
```

> **Note:** `sudo echo x > /etc/file` **fails** — the redirection is done by *your* shell (running as you), not by sudo. Use `... | sudo tee /etc/file` instead.

---

## sed — edit a file in place, non-interactive

`sed` (stream editor) changes text without opening an editor — ideal in scripts and automation.

```bash
# replace the FIRST match on each line (prints to screen — safe preview)
$ sed 's/8080/9090/' app.conf

# replace ALL matches, editing the file in place
$ sed -i 's/8080/9090/g' app.conf        # -i = in place, g = all matches

# keep a backup while editing in place
$ sed -i.bak 's/info/debug/' app.conf    # also writes app.conf.bak

# delete lines
$ sed -i '/^#/d' app.conf                 # delete all comment lines
$ sed -i '3d' app.conf                    # delete line 3
```

| Flag / part | Meaning |
|-------------|---------|
| `s/old/new/` | Substitute first `old` with `new` on each line |
| `s/old/new/g` | Substitute **all** occurrences |
| `-i` | Edit the file **in place** |
| `-i.bak` | Edit in place, keep a `.bak` backup |
| `/pat/d` | Delete lines matching `pat` |

> **⚠️ Watch out:** Always preview a `sed` substitution **without** `-i` first. Once `-i` runs, the change is written. `-i.bak` gives you an undo file — cheap insurance in production.

---

## nano — the beginner-friendly editor

If it's installed, `nano` is the gentlest interactive editor: the shortcuts are shown at the bottom of the screen.

```bash
$ nano app.conf
```

| Shortcut | Action (`^` means Ctrl) |
|----------|-------------------------|
| `Ctrl+O` | Save (write **O**ut) |
| `Ctrl+X` | E**x**it |
| `Ctrl+W` | Search (**W**here is) |
| `Ctrl+K` | Cut the current line |
| `Ctrl+U` | Paste (**U**ncut) |

> **Note:** `nano` isn't guaranteed to be installed on a minimal server — which is exactly why the next topic, `vi`, matters so much.

---

### ✅ Quick recap

- Quick write → `echo "x" > file` (append: `>>`).
- Multi-line block → here-doc `cat > f <<'EOF' ... EOF`.
- Root-owned file → `... | sudo tee /etc/file`.
- Scripted find-and-replace → `sed -i.bak 's/old/new/g' file`.

---

⬅️ [Prev: awk](07-awk.md) | 🏠 [Index](README.md) | ➡️ [Next: vi / vim Editor](09-vi-vim-editor.md)
