# Vault-First Lookup — Detail

## Trigger conditions

User question involves: language (`rust`, `ts`, `python`...), framework (`axum`, `react`...), tool/domain (`oracle query`, `git rebase`, `auth`...), debugging symptom (error message, stack trace), "how do I" / "what's the best way" / "remind me how" phrasing.

**Skip:** meta questions about brain plugin, conversational replies, when `/brain quiet` set, when user says "skip vault" / "don't check vault."

## Default scope (lazy)

- `/brain search <q>` and implicit lookup → **only** `05-notes/permanent/` index entries.
- One pass. Top 5 results.

## Deep scope (explicit)

- `/brain dig <q>` → all index entries: `permanent`, `snippets`, `debugging`, `adr`, area `*-patterns.md`.
- Use only when user asks deeper or first pass returned zero.

## Ranking

Tokenize query (lowercase, drop stopwords `the`/`a`/`is`/`how`/`what`). Score per index entry:

- +3 for token in `title`
- +2 for token in `tags` (exact tag match)
- +1 for token in `first_line`
- +1 for tag-namespace match (`#lang/rust` when query has `rust`)

Top 5 by score. Tie-break by `mtime` desc.

## Answer composition

- **Strong match** → lead with vault content. Cite path: *"From your `10-snippets/rust/parse-csv.md`:"*. Add model knowledge only for gaps, marked `(beyond your notes)`.
- **Partial match** → vault first (cited), then model. End with: *"Worth updating `<file>`? `/brain note <addition>`."*
- **No match** → answer normally. If substantive, end with: *"Worth saving? `/brain note <topic>`."*

## Token discipline

- Read only matched files. Never slurp full vault.
- Use `Grep` (matching lines) over `Read` (whole file) for first scan.
- `Read` full file only when index header looks promising.
