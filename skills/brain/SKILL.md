---
name: brain
description: Second-brain integration. Captures session context (work, decisions, bugs, patterns, TILs, snippets, questions) and flushes to a PARA + Zettelkasten Obsidian vault. Use when user types `/brain` subcommands or mentions logging the session.
---

# Brain — Second-Brain Vault Integration

Capture in memory while you work; flush to vault on `/brain save`. Vault-first lookup reuses your notes before model knowledge to save tokens.

## Vault path

Resolve in order: `$BRAIN_VAULT` → `~/.brainrc.json` (`{"vault": "<abs-path>", "projects": {"<cwd>": "on"|"off"}}`) → default `~/brain`.

**Hard rule:** never write outside the vault path. Never run `git add`/`commit`/`push` in the vault — staging via `Edit`/`Write` only.

## First-run gate

Every subcommand except `init` requires: (1) path defined (env or rc file), AND (2) `<vault>/99-meta/system-rules.md` exists. If either missing, prompt: *"🧠 Run `/brain init` now? (y/n)"* — `y` runs init then resumes; `n` aborts. Never silently scaffold.

## Session-start protocol

If `cwd` is inside the vault, skip. Else check `~/.brainrc.json` `projects[cwd]`:

- `"on"` → set `BRAIN=on` silently. No prompt.
- `"off"` → respect, stay off.
- missing → ask once: *"🧠 Brain detected. Log this project? (yes / no / later)"*. Persist `yes`/`no` to `projects[cwd]`. `later` doesn't persist.

This kills the per-session prompt token after first consent.

## Buckets (when `BRAIN=on`)

In-memory only. Do not write until `/brain save`.

| Bucket | Captures |
|---|---|
| `project` | basename(cwd) + 1-line goal |
| `work_done` | files touched, features, refactors, PRs |
| `decisions` | architectural/technical choices (ADR candidates) |
| `bugs_fixed` | symptom → root cause → fix → prevention |
| `patterns` | reusable techniques |
| `tils` | one-line learnings |
| `snippets` | reusable code blocks |
| `open_questions` | unresolved items |

## Subcommands

| Cmd | Action |
|---|---|
| `init` | Scaffold PARA + copy templates + seed `system-rules.md`. Persist path to `~/.brainrc.json`. |
| `on` / `off` / `skip` | Toggle BRAIN flag. `off`/`skip` discards session log. |
| `status` | Print buckets as markdown preview. No writes. |
| `save` | Run flush procedure (see `references/flush.md`). |
| `note <text>` | Classify + append to bucket. |
| `adr` | Create `12-adr/adr-NNN-<slug>.md` from latest load-bearing decision. |
| `meeting <topic>` / `interview <stk>` / `1on1 <name>` | File note (see `references/filing.md`). |
| `search <q>` | Lazy index search — top 5 cited matches in `05-notes/permanent/` only. |
| `dig <q>` | Deep search — all index entries (snippets, debugging, ADRs, area patterns). Use sparingly. |
| `promote <src>` | Daily-line or phrase → permanent note + backlink (see `references/promote.md`). |
| `stats` | Vault health dashboard (see `references/stats.md`). |
| `reindex` | Rebuild `99-meta/vault-index.json` (see `references/index.md`). |
| `quiet` | Suppress vault-first lookup for the session. |

## Vault-first lookup (default = lazy)

Before answering technical questions (language, framework, debugging, "how do I X"):

1. Read `<vault>/99-meta/vault-index.json` once.
2. Token-match query against `title` + `tags` + `first_line`.
3. **Default scope:** `05-notes/permanent/` only. One pass.
4. If matches → lead answer with vault content, **cite path**, fill gaps from model knowledge marked `(beyond your notes)`.
5. If zero matches → answer normally; if substantive, end with *"Worth saving? `/brain note <topic>`."*

For deeper search (snippets, debugging, ADRs, patterns), user runs `/brain dig <q>` explicitly. Don't auto-grep all paths — wastes tokens on misses.

Skip lookup entirely on: meta questions about brain, conversational replies, `/brain quiet` set.

## Flush procedure (summary)

`/brain save` runs in order — full spec in `references/flush.md`:

1. Append `## Session — <project> @ HH:MM` to `06-daily/<YYYY-MM-DD>.md`.
2. New project? → ask `work` or `side`, scaffold from template.
3. Known area? (scan `02-areas/*/*/` dynamic) → append to changelog/patterns/incidents.
4. Ask once: promote any `patterns`/`tils` → `05-notes/permanent/`?
5. Each `bugs_fixed` → `11-debugging/<symptom>.md`.
6. Load-bearing `decisions` → suggest `/brain adr` (don't auto-create).
7. Each snippet → `10-snippets/<lang>/<verb>-<thing>.md`.
8. Refresh index entries for files written this session.
9. Print files staged. End with: *"✅ Brain staged. Review with `cd <vault> && git diff --stat HEAD`."*
10. Never `git add`/`commit`/`push`.

## Init procedure

1. Resolve path: `$BRAIN_VAULT` → `~/.brainrc.json` → ask user (default `~/brain`).
2. Confirm. On accept, persist to `~/.brainrc.json` if not env-set.
3. Create folders: `00-inbox`, `01-projects/{work,side}`, `02-areas/{career,learning}`, `03-resources`, `04-archive`, `05-notes/permanent`, `06-daily`, `07-maps`, `08-templates`, `10-snippets`, `11-debugging`, `12-adr`, `99-meta`.
4. Copy `${CLAUDE_PLUGIN_ROOT}/skills/brain/templates/*.md` → `<vault>/08-templates/`. Skip existing.
5. Copy `${CLAUDE_PLUGIN_ROOT}/skills/brain/defaults/system-rules.md` → `<vault>/99-meta/system-rules.md`. Skip existing.
6. Suggest `cd <vault> && git init` if no `.git`.

## Tag taxonomy

Namespaced only, max ~5 per note: `#status/`, `#type/`, `#lang/`, `#domain/`, `#difficulty/`, `#area/<slug>`, `#fw/<framework>`. Convert bare tags before saving (`#rust` → `#lang/rust`). Full taxonomy in `<vault>/99-meta/system-rules.md`.

## Hard rules

1. Never write to vault unless `BRAIN=on` AND user typed `/brain save` (or confirmed at session end).
2. Never `git add`/`commit`/`push` in vault.
3. Never delete vault files without per-file confirmation.
4. Never modify `99-meta/system-rules.md`, `99-meta/migration-*.md`, vault `README.md` unless explicitly asked.
5. Never echo unseen brain content unless asked.
6. Project-local `CLAUDE.md` wins on conflict.

## Reminders

- 30+ msgs `BRAIN=on` no save → one nudge: *"🧠 Flush now?"*. No nag.
- Session end (`bye`/`done`) + non-empty log → ask once.

## Reference files (load on demand)

- `references/flush.md` — full flush spec with edge cases
- `references/filing.md` — meeting/interview/1on1 routing + mom-test discipline
- `references/promote.md` — promote workflow detail
- `references/stats.md` — stats dashboard format
- `references/index.md` — vault index schema + reindex algorithm
- `references/lookup.md` — vault-first lookup ranking + scope rules
