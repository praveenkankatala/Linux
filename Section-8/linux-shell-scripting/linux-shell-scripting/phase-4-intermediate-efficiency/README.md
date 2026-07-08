[🏠 Home](../README.md)

# Phase 4 — Intermediate Efficiency
> _Writing clean code_

So far your scripts have been flat lists of commands. Real tools are organised into **functions**, hold collections in **arrays**, and accept proper command-line **flags**. This phase is the jump from 'a script that works' to 'a script a teammate can read, extend, and rely on.'

## 📚 Topics in this phase

| # | Topic | What you'll learn |
| --- | --- | --- |
| 13 | [Functions](13-functions.md) | Functions — reusability, arguments, local vs global |
| 14 | [Arrays and String Manipulation](14-arrays-and-strings.md) | Arrays and string manipulation (slicing, length) |
| 15 | [Parsing Flags with getopts](15-getopts.md) | Parsing flags with getopts (-f, -h) |

## ✅ Phase 4 Checklist

- Define functions, pass them arguments (`$1` inside), and reuse them.
- Declare internal variables `local`; return status with `return`, data via stdout + `$(...)`.
- Create, append to, slice, and loop over indexed arrays with `"${arr[@]}"`.
- Do string surgery with `${#s}`, `${s:o:l}`, `${s##*/}`, `${s%.ext}`, `${s//a/b}`.
- Parse `-f value`, `-v`, `-h` flags with a `getopts` + `case` block and a usage function.

> [!NOTE]
> **Next up — Phase 5**
>
> Your code is now clean and modular. The final phase makes it *robust and production-grade*: defensive flags, traps and cleanup, associative arrays, concurrency, and advanced awk with process substitution.

---
⬅️ [Phase 3 — Text Handling & I/O](../phase-3-text-handling-and-io/README.md)  |  🏠 [Home](../README.md)  |  [Phase 5 — Advanced & Robust Scripting](../phase-5-advanced-robust-scripting/README.md) ➡️
