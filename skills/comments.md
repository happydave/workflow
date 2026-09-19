---
name: comments
description: Use when writing or reviewing code comments, documentation prose, or a reply in an MR discussion — deciding how much to write, what to cut, and how to match a project's existing voice.
---

# Comments and Documentation

## Overview

Agent-written comments read as machine-generated because of **what** they say, not how many there
are. Too little beats too much.

## The recipe

1. **Read 2–3 reference files** in the same layer you would be happy to have written; they set the
   length, the phrasing, and what the neighbours bother to document. Do not pick them by commit
   author or date: AI-assisted work lands under human names.
2. **Cut by category** (below), sentence by sentence: each survives under Keep or falls under a
   row. Judge a touched comment whole, even if you added one line of it.
3. **Match the neighbours' form** — phrasing, length, dividers. Form never rescues a sentence a Cut
   row removes: neighbours full of citations do not keep yours.

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
| **Restates the code** | Adds nothing the body, the signature, or a comment at the use site already says | Delete — but keep an identifier-opening line where the language's doc convention expects one, and judge what follows |
| **Rots** | "ten of the seventeen", "today", fleet state | Drop the number and the date; keep the rule or reason they stand for |
| **Cites what the reader cannot open** | "per AC3", "the plan's table" | Keep the reason, drop the citation |

## Keep these

Keep a comment when it stops a reader **re-deriving a defect already paid for**, or states a
constraint the code cannot express — a unit, an invariant, a contract.

A test catches a wrong edit too, so: **a test proves the behaviour, a comment prevents the edit.**
Keep it when the wrong edit would look obviously correct; drop it when it would look wrong. The
same test breaks a tie with a Cut row. Above a test, "the edit" is weakening or deleting that test,
and cut substance can move into the assertion message rather than out of the file.

Verify a mechanism before keeping it — a confident, wrong or half-true explanation is worse than
none. A claim the repo cannot verify stays as a bare fact or goes; any inference drawn from it
goes.

State a kept reason as fact; do not build the case. If a sentence needs a comparison to stand up, it
is an argument, not a constraint.

## Form

Doc comments 1–3 lines, inline comments one, unless the file's own run longer. Name the
mechanism — "uses `$push` to avoid read-modify-write", not "avoids concurrency issues".

## Replies in an MR discussion

The same cuts apply to what you write back to a reviewer.

- Answer the question asked; say what changed and where (file and line, or the sha once pushed —
  say so if it is not yet); stop.
- Size to their message: a one-line comment gets a one-line reply. The size bounds how you say each
  thing below, not whether.
- Do not restate their point, argue against objections nobody raised, or narrate how the fix was
  found.
- Declining their suggestion: the one constraint that rules it out, stated as fact — the one the
  reader can check.
- A correction they need (wrong version, wrong line), or a decision they cannot see in the diff
  (a sibling left alone, a side effect of the change), is one sentence of fact: disclosed, not
  defended.
- Ask a clarifying question before acting on a guess. If you already acted, say so and how to undo
  it, then ask; do not dress the action as a question.

## Common mistakes

| Mistake | Correct approach |
|---|---|
| Reference files chosen by commit author or date | Nominate them by reading them |
| Treating a density number as the finding | It picks the file; the category decides the edit |
| Deleting a comment when only its citation was wrong | Keep the substance, drop the citation |
| Rewriting a file's comments during an unrelated change | Scope the pass to the comments you touched |
| "Agreed, and done", followed by a defence of the change | Stop at "done" and where |

## Source

Derived from a comment pass over a service repository, the reviewer feedback behind it, and the
replies drafted to that review, 2026-09.
