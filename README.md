# brain — Second-Brain Plugin for Claude Code

> PARA + Zettelkasten second-brain integration. Capture session context as you work and flush it to a structured Obsidian vault — without ever letting Claude touch your git history.

`brain` turns Claude Code into a capture-first knowledge worker. While you code, it tracks decisions, bugs fixed, patterns discovered, TILs, snippets, and open questions in memory. When you're ready, `/brain save` flushes everything to your vault as daily notes, debugging logs, ADR candidates, area changelogs, and permanent notes — already-classified and pre-formatted.

The vault is yours. The plugin only stages writes. You commit on your own schedule.

## Install

```text
/plugin marketplace add aegis-imm/brain
/plugin install brain@brain-marketplace
```

Then restart Claude Code (or run `/reload-plugins`).

## First run

1. **Pick a vault location.** Set `BRAIN_VAULT` in your shell (recommended), otherwise `~/brain` is used:

   ```bash
   export BRAIN_VAULT="$HOME/Documents/brain"   # or wherever
   ```

2. **Scaffold the vault** with the PARA folder structure and bundled templates:

   ```text
   /brain:brain init
   ```

   This creates 15 folders, copies 9 templates into `08-templates/`, and seeds a default `99-meta/system-rules.md` (tag taxonomy, naming conventions, weekly review checklist).

3. **(Optional) Initialize git in the vault** so you have history:

   ```bash
   cd "$BRAIN_VAULT" && git init
   ```

4. **Start a session.** On the first message of any project, the plugin asks:

   > 🧠 Brain detected at `<vault-path>`. Log this session? (yes / no / later)

   Reply `yes` to start tracking. Work normally. When done:

   ```text
   /brain:brain save
   ```

## Command reference

> Plugin commands are namespaced. The slash command is `/brain:brain <subcommand>`.

| Subcommand | What it does |
|---|---|
| `init` | Scaffold PARA folders + copy templates + seed `system-rules.md` |
| `on` | Start tracking the current session |
| `off` / `skip` | Stop tracking and discard the in-memory session log |
| `status` | Print the in-memory buckets as a markdown preview (no writes) |
| `save` | Flush session log to the vault: daily note + changelogs + debugging logs + snippets |
| `note <text>` | Append `<text>` to the appropriate bucket (auto-classified) |
| `adr` | Create `12-adr/adr-NNN-<slug>.md` from the latest load-bearing decision |
| `meeting <topic>` | File a meeting note under the current area |
| `interview <stakeholder>` | File a requirements interview note (mom-test discipline) |
| `1on1 <name>` | File a 1:1 note under the current area's `1on1/` folder |
| `quiet` | Suppress vault-first lookup for the rest of the session |

## Vault-first lookup (token-saver)

When you ask a technical question (rust, axum, oracle, debugging, etc.), the plugin searches your vault **before** answering from model knowledge:

1. Greps `05-notes/permanent/`, `10-snippets/<lang>/`, `11-debugging/`, `02-areas/*/<area>-patterns.md`, `12-adr/`
2. If matches → leads the answer with your own notes, cites the file path
3. If no matches → answers normally and offers to capture the answer with `/brain:brain note`

**Why:** vault notes are already compressed by you. Pulling 200 tokens of your own notes beats 2000 tokens of model regen. Over time, your vault becomes a personal knowledge cache.

To suppress lookup for a session: `/brain:brain quiet`.

## What gets captured

While `BRAIN=on`, the plugin tracks (in memory only — nothing is written until `/brain:brain save`):

| Bucket | Examples |
|---|---|
| `project` | basename of cwd, brief one-line goal of this session |
| `work_done` | files touched, features added, refactors, PRs opened |
| `decisions` | architectural / technical choices worth recording |
| `bugs_fixed` | symptom → root cause → fix → prevention |
| `patterns` | reusable techniques |
| `tils` | one-line "today I learned" items |
| `snippets` | small reusable code blocks |
| `open_questions` | unresolved items |

## Vault layout

```
<vault>/
├── 00-inbox/                # capture-first; sort later
├── 01-projects/
│   ├── work/                # active work projects with deadlines
│   └── side/                # personal projects with goals
├── 02-areas/
│   ├── career/              # ongoing professional responsibilities
│   └── learning/            # ongoing study tracks
├── 03-resources/            # reference material, no active work
├── 04-archive/              # done, dormant, or abandoned
├── 05-notes/
│   └── permanent/           # Zettelkasten — atomic, evergreen claims
├── 06-daily/                # daily logs, one file per YYYY-MM-DD
├── 07-maps/                 # MOCs (maps of content), index notes
├── 08-templates/            # template files (seeded by /brain init)
├── 10-snippets/             # reusable code blocks, by language
├── 11-debugging/            # incident / bug logs with root cause + prevention
├── 12-adr/                  # architectural decision records
└── 99-meta/                 # vault meta: rules, migrations, system docs
    └── system-rules.md      # canonical contract — read on every session
```

## Tag taxonomy

All tags are namespaced. No bare tags (`#rust` → `#lang/rust`). Max ~5 tags per note.

| Namespace | Examples | Purpose |
|---|---|---|
| `#status/` | `draft`, `active`, `done`, `archived` | Lifecycle |
| `#type/` | `note`, `permanent`, `meeting`, `1on1`, `adr`, `debugging`, `snippet`, `daily`, `project` | Note kind |
| `#lang/` | `rust`, `typescript`, `python`, `go`, `sql` | Programming language |
| `#domain/` | `auth`, `payments`, `infra`, `frontend` | Subject domain |
| `#difficulty/` | `easy`, `medium`, `hard` | Effort required |
| `#area/` | `<area-slug>` | Cross-link to a `02-areas/` folder |
| `#fw/` | `react`, `axum`, `nextjs`, `django` | Framework |

Edit `<vault>/99-meta/system-rules.md` to extend or change the taxonomy.

## Hard rules (the plugin will never break these)

1. **Never writes to the vault** unless `BRAIN=on` AND you've typed `/brain:brain save` (or confirmed at session end).
2. **Never runs `git add`, `git commit`, or `git push`** in the vault. Staging via `Edit`/`Write` only — you commit manually.
3. **Never deletes vault files** without explicit per-file confirmation.
4. **Never modifies** `99-meta/system-rules.md`, `99-meta/migration-*.md`, or vault `README.md` unless explicitly asked.
5. **Project-local `CLAUDE.md` always wins** if it conflicts with the brain skill.

## Configuration

| Env var | Default | Purpose |
|---|---|---|
| `BRAIN_VAULT` | `~/brain` | Absolute path to your vault root |

## Customizing templates

The templates copied by `/brain:brain init` live in your vault at `08-templates/`. Edit them freely — the plugin reads the vault copies, not the bundled originals, when filing new notes.

To re-seed templates from the plugin (e.g. after updating the plugin), delete the file in `08-templates/` and run `/brain:brain init` again — it skips files that already exist.

## License

[MIT](LICENSE) © 2026 Kiattipat Sukonthapong
