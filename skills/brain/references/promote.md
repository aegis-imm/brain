# Promote Procedure

`/brain promote <source>` — turn a daily-note line or phrase into a permanent note.

## Source forms

- File path with line ref: `06-daily/2026-04-18.md#L42`
- Bare phrase: `idempotent retries are necessary in distributed queues`

## Steps

1. If path+line: `Read` that line and ~5 lines of surrounding context.
2. Suggest slug: lowercase kebab-case from first 4–6 content words.
3. Create `<vault>/05-notes/permanent/<slug>.md` from `08-templates/permanent-note-template.md`. Pre-fill:
   - `title` = humanized slug
   - **Claim** = source line/phrase
   - **Source** = backlink to daily note path
4. If source was daily note, ask: *"Replace the original line with `[[<slug>]]`? (y/n)"*. On `y`, edit daily.
5. Append new file to `vault-index.json`.
6. Print path + suggest 2 related notes (quick search using slug words).
