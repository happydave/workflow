---
name: comments
description: Use when writing or reviewing code comments and documentation prose — deciding how much to write, what to cut, and how to match a project's existing voice.
---

# Comments and Documentation

## Overview

Agent-written comments read as machine-generated because of **what** they say, not how many there
are. Too little beats too much.

## The recipe

1. **Read 2–3 reference files** in the same layer you would be happy to have written. This is the
   step that pays — it shows the length, the phrasing, and what the neighbours bother to document.
   Do not pick them by commit author or date: AI-assisted work lands under human names.
2. **Cut by category** (below). Category decides every call.
3. **Match the neighbours' form** — phrasing, length, dividers.

Comments per 100 code lines is triage for *which file to open*, never an input to a single decision;
a struct file scores high on one doc line per field.

```sh
for f in "$@"; do awk '/^[[:space:]]*(\/\/|#)/{c++} /./{t++} END{if(t>c) printf "%5.1f  %s\n", 100*c/(t-c), FILENAME}' "$f"; done
```

## Cut these

| Category | Looks like | Do |
|---|---|---|
| **Argues a case** | Defends a choice against alternatives nobody proposed | Delete; it belongs in the commit message |
| **Narrates history** | "previously asserted the opposite", "this used to…" | Delete; never converse in a comment |
| **Restates the code** | Adds nothing the body or signature already says | Delete — but keep an identifier-opening line where the language's doc convention expects one, and judge what follows |
| **Cites what the reader cannot open** | "per AC3", "the plan's table" | Keep the reason, drop the citation. A citation survives only if it resolves for every reader of the repo |

## Keep these

Keep a comment when it stops a reader **re-deriving a defect already paid for**, or states a
constraint the code cannot express.

A test catches a wrong edit too, so: **a test proves the behaviour, a comment prevents the edit.**
Keep it when the wrong edit would look obviously correct; drop it when it would look wrong. Above a
test, "the edit" is weakening or deleting that test, and cut substance can move into the assertion
message rather than out of the file.

Verify a mechanism before keeping it — a confident, wrong explanation is worse than none.

State a kept reason as fact; do not build the case. If a sentence needs a comparison to stand up, it
is an argument, not a constraint.

## Form

- Doc comments 1–3 lines, naming the mechanism — "uses `$push` to avoid read-modify-write", not
  "avoids concurrency issues".
- Inline comments one line, describing the next statement.
- A whole package can be uniformly over-commented; neighbours alone do not settle it.

## Common mistakes

| Mistake | Correct approach |
|---|---|
| Reference files chosen by commit author or date | Nominate them by reading them |
| Treating a density number as the finding | It picks the file to open; the category decides the edit |
| Deleting a comment when only its citation was wrong | Keep the substance, drop the citation |
| Rewriting a file's comments during an unrelated change | Scope the pass to what you touched |

## Source

Derived from a comment pass over a service repository and the reviewer feedback behind it, 2026-09.
