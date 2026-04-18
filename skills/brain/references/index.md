# Vault Index

`<vault>/99-meta/vault-index.json` — flat JSON cache of every note's metadata. Lets vault-first lookup find candidates via one file read instead of recursive grep.

## Schema

Array of objects with keys: `path` (relative), `title` (from frontmatter), `type` (frontmatter), `tags` (frontmatter array), `first_line` (first non-blank, non-frontmatter line, max 200 chars), `mtime` (ISO string).

## Reindex algorithm (`/brain reindex`)

1. Walk `<vault>/**/*.md` excluding `00-inbox/`, `04-archive/`, `08-templates/`, `99-meta/`.
2. For each file: parse YAML frontmatter (between leading `---` markers) for `title`/`type`/`tags`. Skip if no frontmatter.
3. Extract first non-blank, non-frontmatter line as `first_line` (truncate 200 chars).
4. Stat for `mtime`.
5. Write JSON array to `<vault>/99-meta/vault-index.json`.
6. Print: *"Indexed N files in M ms. Skipped K (no frontmatter)."*

## Refresh on save

`/brain save` step 8 — for files written/modified this session, append new entries or replace existing by `path`. No full rebuild.

## Stale check

Before vault-first lookup: compare index `mtime` to vault directory `mtime`. If dir newer, print one-line nudge: *"🧠 Index stale — run `/brain reindex`."*. Don't block lookup.
