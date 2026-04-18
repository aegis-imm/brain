# Flush Procedure — Full Spec

`/brain save` runs these in order. Use `Edit`/`Write` only. Never `git add`/`commit`/`push`.

## 1. Daily log entry

- Path: `<vault>/06-daily/<YYYY-MM-DD>.md`
- If missing: create from `<vault>/08-templates/daily-note-template.md`.
- Append: `## Session — <project> @ HH:MM` (24h time) with concise bullets from `work_done`, `decisions`, `bugs_fixed`, `tils`.

## 2. New project detection

If `basename(cwd)` not under `01-projects/work/`, `01-projects/side/`, or `02-areas/*/*/`:

- Ask: *"New project — `01-projects/work/` or `01-projects/side/`?"*
- Copy matching template, pre-fill goal + tech stack from session context.

## 3. Known area (dynamic)

Scan `<vault>/02-areas/career/*/` and `<vault>/02-areas/learning/*/` at runtime. If `basename(cwd)` matches:

- Append date-stamped entry to `<area>-changelog.md` (`work_done` + `decisions`). Create file if missing.
- `bugs_fixed` non-empty → append to `11-debugging/<area>-incidents.md` (or new `11-debugging/<symptom>.md` if bug warrants own log).
- `patterns` non-empty → append to `<area>-patterns.md` under new dated section.

## 4. Durable insights

Ask once: *"Promote any of these to permanent notes? List numbers or 'no'."*

For each marked: create `05-notes/permanent/<slug>.md` from `permanent-note-template.md`, pre-filled.

## 5. Debugging logs

Each `bugs_fixed` entry (always log, no prompt): create `11-debugging/<short-symptom>.md` from `debugging-log-template.md`, pre-filled with symptom/root-cause/fix/prevention.

## 6. ADRs

Load-bearing `decisions` → suggest, don't auto-create:

> *"Decision X looks ADR-worthy — run `/brain adr` to create one."*

## 7. Snippets

Each `snippets` entry → `10-snippets/<lang>/<verb>-<thing>.md` from `code-snippet-template.md`.

## 8. Refresh index

Append/update entries in `<vault>/99-meta/vault-index.json` for files written/modified this session. No full rebuild — that's `/brain reindex`.

## 9. Print summary

List every file written. End with literal:

> ✅ Brain staged. Review with `cd <vault> && git diff --stat HEAD` then commit when ready.

## 10. Hard rule

Never `git add`, `git commit`, or `git push` in the vault.

## Naming

- Slugs: lowercase kebab-case
- Daily section: `## Session — <project> @ HH:MM`
- ADR: `12-adr/adr-NNN-<title>.md` (find next free `NNN`)
- Debugging: `11-debugging/<symptom>.md`
- Permanent: `05-notes/permanent/<atomic-concept>.md`
