---
name: brain
description: Second-brain integration. Captures session context (work done, decisions, bugs fixed, patterns, TILs, snippets, open questions) and flushes to a PARA + Zettelkasten Obsidian vault. Use when user types `/brain` subcommands (on/off/save/status/note/init/adr/meeting/interview/1on1) or mentions logging the session to their vault.
---

# Brain — Second-Brain Vault Integration

Capture session context as you work, then flush to a PARA + Zettelkasten Obsidian vault on demand.

## Vault Location

Resolve the vault path in this order:

1. `$BRAIN_VAULT` environment variable
2. `~/.brainrc.json` → `{ "vault": "<absolute-path>" }` (written by the init wizard)
3. `~/brain` (default fallback)

**Never** write outside the resolved vault path. **Never** run `git commit`, `git push`, `git add`, or destructive git operations inside the vault — staging via `Edit`/`Write` only. The user owns the vault's git history.

## First-Run Gate (mandatory init)

**Every `/brain` subcommand except `init` must check vault readiness first.**

A vault is considered **ready** when:
- The resolved vault path exists as a directory, AND
- `<vault>/99-meta/system-rules.md` exists (signals scaffold complete)

If the vault is **not ready**, do this before executing the requested subcommand:

1. Print: *"🧠 Vault not initialized. Running `/brain init` first."*
2. Resolve candidate path (env → config file → default `~/brain`).
3. Ask the user, one prompt:
   > *"Create vault at `<path>`? (yes / customize / no)"*
4. Branch:
   - **yes** → run the **Init Procedure** below. After scaffold, save the path to `~/.brainrc.json` (only if not already set via `$BRAIN_VAULT`). Then continue with the original subcommand.
   - **customize** → ask for absolute path. Validate it. Scaffold there. Save to `~/.brainrc.json`. Continue.
   - **no** → abort the original subcommand. Print: *"Aborted. Run `/brain init` when ready."*

**Special case:** if the directory exists and has PARA folders but no `system-rules.md`, treat as a partial install — offer to top up missing folders + seed `system-rules.md`, do not overwrite existing files.

**`/brain init` itself** skips this gate and runs directly (it *is* the init).

## Session-Start Protocol

At the first user message in a session, if the current working directory is **not** inside the vault:

1. Announce verbatim, one line:

   > 🧠 **Brain detected at `<vault-path>`. Log this session? (yes / no / later)**

2. Wait for the user's reply. Interpret:
   - `yes` / `y` / `log it` / `log` → set internal flag `BRAIN=on`. Read `<vault>/99-meta/system-rules.md` and the project's `02-areas/career/<basename(cwd)>/` folder if it exists.
   - `no` / `n` / `skip` → set `BRAIN=off`. Don't mention the brain again unless the user types `/brain on`.
   - `later` / anything else → set `BRAIN=pending`. Continue normal work; ask once more if the user types `/brain` later.

If CWD is inside the vault, skip the announcement — the user is editing the brain directly.

## Vault-First Lookup

**Before answering any technical question**, search the vault for prior knowledge. The user already paid the token cost to capture it — reuse instead of regenerate.

### When to trigger

Trigger lookup when the user asks a question that involves:
- A programming language (`rust`, `typescript`, `python`, `go`, `sql`, etc.)
- A framework (`axum`, `react`, `nextjs`, `django`, etc.)
- A tool / domain (`oracle query`, `git rebase`, `docker compose`, `auth`, `payments`, etc.)
- A debugging symptom (error message, stack trace, "why is X happening")
- A "how do I" / "what's the best way to" / "remind me how" phrasing

Skip lookup for: pure conversational replies, meta questions about the brain plugin itself, or when the user explicitly says "don't check the vault."

### Lookup procedure

1. **Extract keywords** from the question: language, framework, domain, error tokens.
2. **Grep these paths** (in order, stop early if strong match found):
   - `<vault>/05-notes/permanent/*.md` — atomic claims, evergreen
   - `<vault>/10-snippets/<lang>/*.md` — reusable code blocks
   - `<vault>/11-debugging/*.md` — past bugs with root cause + fix
   - `<vault>/02-areas/*/*/<area>-patterns.md` — area-specific techniques
   - `<vault>/12-adr/*.md` — past architectural decisions (if topic feels architectural)
3. Use `Grep` with case-insensitive search, multiple keyword passes if needed.

### How to answer

- **If strong vault match found:**
  - Lead the answer with vault content. Quote or paraphrase faithfully.
  - **Always cite the source path**, e.g. *"From your `10-snippets/rust/parse-csv.md`:"*
  - Add model knowledge only to fill gaps the vault doesn't cover. Mark clearly: *"(beyond your notes)"*
- **If partial match found:**
  - Combine: vault content first (cited), then model knowledge to complete.
  - End with: *"Worth updating `<file>`? Run `/brain note <addition>` to capture."*
- **If no match found:**
  - Answer normally from model knowledge.
  - If the answer was substantive (not trivial), end with: *"Worth saving? `/brain note <topic>`, or I can draft a snippet for `10-snippets/<lang>/`."*

### Token discipline

- Read only files that match. Don't slurp the whole vault.
- Prefer `Grep` (returns matching lines) over `Read` (whole file) for the initial scan.
- Read full file only when the snippet header / first match looks promising.

### Opt-out

If the user types `/brain quiet` or "skip vault" in a message, suppress vault lookup for the rest of the session. Re-enable on `/brain on` or explicit "check the vault."

## Buckets to Track (when `BRAIN=on`)

Maintain an internal session log. **Do not write to the vault** until the user types `/brain save` or confirms at session end.

| Bucket | Examples |
|---|---|
| `project` | basename of cwd, brief one-line goal of this session |
| `work_done` | files touched, features added, refactors, PRs opened |
| `decisions` | architectural / technical choices worth recording (ADR candidates) |
| `bugs_fixed` | symptom → root cause → fix → prevention (debugging-log candidates) |
| `patterns` | reusable techniques (permanent-note or area `*-patterns.md` candidates) |
| `tils` | one-line "today I learned" items |
| `snippets` | small reusable code blocks |
| `open_questions` | unresolved items, fleeting-note candidates |

Keep entries terse. The user edits on commit.

## Brain Subcommands

Recognize these in user messages (typed via `/brain <subcommand>` or in plain text):

| Command | Action |
|---|---|
| `on` | Set `BRAIN=on` |
| `off` / `skip` | Set `BRAIN=off`. Discard session log. |
| `status` | Print in-memory bucket contents as markdown preview. Do not write. |
| `save` | Run the **Flush Procedure** below |
| `note <text>` | Classify and append `<text>` to the appropriate bucket immediately |
| `init` | Scaffold the PARA folder structure + copy bundled templates into the vault |
| `adr` | Create a new ADR from the latest load-bearing decision (uses `12-adr/adr-template.md`) |
| `meeting <topic>` | File a meeting note (see **Filing Rules** below) |
| `interview <stakeholder>` | File a requirements interview note (mom-test discipline) |
| `1on1 <name>` | File a 1:1 note |

## Flush Procedure (`/brain save`)

Run in order. Use `Edit`/`Write` on vault paths. **Stage only.**

### 1. Daily log entry

- Path: `<vault>/06-daily/<YYYY-MM-DD>.md`
- If file doesn't exist: create from `<vault>/08-templates/daily-note-template.md`, fill the date, then append.
- Append section: `## Session — <project> @ HH:MM` (24h time) with concise bullets from `work_done`, `decisions`, `bugs_fixed`, `tils`.

### 2. New project detection

If `basename(cwd)` does not match any directory under `01-projects/work/`, `01-projects/side/`, or `02-areas/*/`:

- Ask: *"Looks like a new project — `01-projects/work/` or `01-projects/side/`?"*
- On answer, copy the matching template (`work-project-template.md` or `side-project-template.md`) to `01-projects/<bucket>/<basename>.md` and pre-fill goal + tech stack from session context.

### 3. Known area detection (dynamic — no hardcoded list)

Scan `<vault>/02-areas/career/*/` and `<vault>/02-areas/learning/*/` at runtime. If `basename(cwd)` matches an area folder name:

- Append a date-stamped entry to `02-areas/<category>/<area>/<area>-changelog.md` summarizing `work_done` + `decisions`. Create the changelog file if missing.
- If `bugs_fixed` is non-empty, append to `11-debugging/<area>-incidents.md` (or new `11-debugging/<symptom>.md` if the bug warrants its own log).
- If `patterns` is non-empty, append to `02-areas/<category>/<area>/<area>-patterns.md` under a new dated section.

### 4. Durable insights

For each `patterns` or `tils` entry, ask once:

> *"Promote any of these to permanent notes? List numbers or 'no'."*

For each marked: create `05-notes/permanent/<slug>.md` from `permanent-note-template.md`, pre-filled.

### 5. Debugging logs

For each `bugs_fixed` entry (always log, no prompt): create `11-debugging/<short-symptom>.md` from `debugging-log-template.md`, pre-filled.

### 6. ADRs

If any `decisions` entry looks load-bearing, suggest but do **not** auto-create:

> *"Decision X looks ADR-worthy — run `/brain adr` to create one."*

### 7. Snippets

For each `snippets` entry: create `10-snippets/<lang>/<verb>-<thing>.md` from `code-snippet-template.md`.

### 8. Print summary

List every file written/edited. End with the literal line:

> ✅ Brain staged. Review with `cd <vault> && git diff --stat HEAD` then commit when ready.

### 9. **Do not** `git add`, `git commit`, or `git push` in the vault.

## Init Procedure (`/brain init`)

Resolve vault path: `$BRAIN_VAULT` → `~/.brainrc.json` → ask user (default `~/brain`). If vault dir doesn't exist, ask the user to confirm creation at that path. After confirmation, persist the path to `~/.brainrc.json` if not already set via env var.

Create folders if missing:

```
00-inbox/
01-projects/work/
01-projects/side/
02-areas/career/
02-areas/learning/
03-resources/
04-archive/
05-notes/permanent/
06-daily/
07-maps/
08-templates/
10-snippets/
11-debugging/
12-adr/
99-meta/
```

Copy bundled templates from `${CLAUDE_PLUGIN_ROOT}/skills/brain/templates/*.md` into `<vault>/08-templates/` (skip files that already exist; ask before overwriting).

Copy default `${CLAUDE_PLUGIN_ROOT}/skills/brain/defaults/system-rules.md` into `<vault>/99-meta/system-rules.md` (skip if exists).

Print summary: dirs created, templates copied, system-rules location, config file path (if written). Suggest the user `cd <vault> && git init` if not already a git repo.

## Meeting + Interview Filing Rules

When the user asks to file a meeting, 1:1, or interview note, route as follows. If `BRAIN=on`, file directly; otherwise prompt for `/brain on` first.

| Note type | Filing path | Template |
|---|---|---|
| Generic meeting (project-scoped) | `02-areas/career/<area>/meetings/<YYYY-MM-DD>-<short-topic>.md` | `meeting-note-template.md` |
| 1:1 (manager / report) | `02-areas/career/<area>/1on1/<YYYY-MM-DD>-<name>.md` | `1on1-template.md` |
| Requirements interview / mom-test | `02-areas/learning/research-methodology/interviews/<YYYY-MM-DD>-<stakeholder>-<topic>.md` | `meeting-note-template.md` (mom-test framing) |
| Standalone (no area) | `06-daily/<today>.md` as `## Meeting — <topic> @ HH:MM` | `meeting-note-template.md` body only |
| Cross-team (no single area) | `06-daily/<today>.md`, then ask if it should also live in `02-areas/career/general/meetings/` | `meeting-note-template.md` |

**Filename rules:**
- `<short-topic>` = lowercase kebab-case, ≤ 5 words
- `<stakeholder>` = person/org being interviewed, kebab-case (initials OK if privacy-sensitive)
- Always pad date as `YYYY-MM-DD`

**After filing:**
- Append a 1-line entry to `02-areas/<category>/<area>/<area>-changelog.md`: `<YYYY-MM-DD> · meeting · <topic> · <decisions count>`
- Cross-link from `07-maps/<area>-index.md` under "Active meetings / interviews" (create section if missing).

**Mom-test discipline (interview notes only):**
- Lead with **concrete past behavior**, not opinions ("last time you needed X, what did you do?")
- Capture **current workarounds** — that's the real demand signal
- Mark **painful** vs **nice-to-have** explicitly
- End with **commitment ask** (next step, intro, follow-up call) and whether they committed

## Tag Discipline

When writing vault notes, use only the namespaced taxonomy from `<vault>/99-meta/system-rules.md`:

- `#status/` — `draft`, `active`, `done`, `archived`
- `#type/` — `note`, `permanent`, `meeting`, `1on1`, `interview`, `adr`, `debugging`, `snippet`, `daily`, `project`, `area`
- `#lang/` — `rust`, `typescript`, `python`, etc.
- `#domain/` — domain area, e.g. `auth`, `payments`, `infra`
- `#difficulty/` — `easy`, `medium`, `hard`
- Optional: `#area/<slug>`, `#fw/<framework>`

Max ~5 tags per note. No bare tags (`#rust` → `#lang/rust`).

## Hard Rules

1. **Never write to the vault unless `BRAIN=on` AND the user has typed `/brain save` (or confirmed at session end).**
2. **Never `git commit` or `git push` in the vault.** Staging via `Edit`/`Write` only.
3. **Never delete vault files** without explicit per-file user confirmation.
4. **Never modify** `99-meta/system-rules.md`, `99-meta/migration-*.md`, or `README.md` in the vault unless explicitly asked.
5. **Never echo brain content the user hasn't already seen** unless they ask — privacy hygiene.
6. **Project-local `CLAUDE.md` always wins** if it conflicts with this skill.

## Reminder Behavior

- If `BRAIN=on` and the user has worked for ~30+ messages without a `/brain save`, gently nudge once: *"🧠 Want me to flush the session log now?"* Do not nag again.
- At session end (user types `bye`, `done`, etc.) with `BRAIN=on` and a non-empty session log, ask once: *"Flush session log to brain before we end?"*

## Naming Conventions (vault writes)

- Slugs: lowercase kebab-case
- Daily section header: `## Session — <project> @ HH:MM` (24h)
- ADR filename: `12-adr/adr-NNN-<short-title>.md` (find next free `NNN`)
- Debugging log: `11-debugging/<symptom>.md`, e.g. `n-plus-one-query-in-orm.md`
- Permanent note: `05-notes/permanent/<atomic-concept>.md`
