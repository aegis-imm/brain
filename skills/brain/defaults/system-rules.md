---
title: "System Rules"
created: 2026-04-18
type: meta
tags: [#type/meta]
---

# System Rules

This file is the canonical contract for how the vault is organized. Edit it deliberately — the brain skill reads this on every session start.

## Folder layout (PARA + extensions)

```
00-inbox/                # capture-first; sort later
01-projects/
  work/                  # active work projects with deadlines
  side/                  # personal projects with goals
02-areas/
  career/                # ongoing professional responsibilities
  learning/              # ongoing study tracks
03-resources/            # reference material, no active work
04-archive/              # done, dormant, or abandoned
05-notes/
  permanent/             # Zettelkasten — atomic, evergreen claims
06-daily/                # daily logs, one file per YYYY-MM-DD
07-maps/                 # MOCs (maps of content), index notes
08-templates/            # template files used by /brain init
10-snippets/             # reusable code blocks, organized by language
11-debugging/            # incident / bug logs with root cause + prevention
12-adr/                  # architectural decision records
99-meta/                 # vault meta: rules, migrations, system docs
```

## Naming conventions

- **Slugs:** lowercase kebab-case, no spaces or underscores
- **Daily notes:** `06-daily/YYYY-MM-DD.md`
- **ADRs:** `12-adr/adr-NNN-<short-title>.md` (zero-padded, monotonically increasing)
- **Debugging logs:** `11-debugging/<symptom>.md`, e.g. `n-plus-one-query-in-orm.md`
- **Permanent notes:** `05-notes/permanent/<atomic-concept>.md`
- **Meeting notes:** `02-areas/<category>/<area>/meetings/YYYY-MM-DD-<short-topic>.md`
- **1:1 notes:** `02-areas/<category>/<area>/1on1/YYYY-MM-DD-<name>.md`
- **Snippets:** `10-snippets/<lang>/<verb>-<thing>.md`

## Tag taxonomy

Use only these namespaces. No bare tags. Max ~5 tags per note.

| Namespace | Examples | Purpose |
|---|---|---|
| `#status/` | `draft`, `active`, `done`, `archived` | Lifecycle |
| `#type/` | `note`, `permanent`, `meeting`, `1on1`, `interview`, `adr`, `debugging`, `snippet`, `daily`, `project`, `area`, `meta` | What kind of note |
| `#lang/` | `rust`, `typescript`, `python`, `go`, `sql` | Programming language (snippets, code-heavy notes) |
| `#domain/` | `auth`, `payments`, `infra`, `frontend`, `data` | Subject domain |
| `#difficulty/` | `easy`, `medium`, `hard` | Effort / understanding required |
| `#area/` | `<area-slug>` | Cross-link to a `02-areas/` folder |
| `#fw/` | `react`, `axum`, `nextjs`, `django` | Framework |

**Migration rule:** convert any bare tag to a namespaced form before saving. `#rust` → `#lang/rust`.

## Linking conventions

- Use `[[wikilinks]]` for cross-references between notes
- Use full relative paths only when linking to non-note files (e.g. images, code in repos outside the vault)
- Each `02-areas/<category>/<area>/` folder should have a `<area>-index.md` that serves as its MOC, also linked from `07-maps/`

## Per-area files

When a new area is created under `02-areas/`, the brain skill expects (or creates on demand):

- `<area>.md` — the area's main note
- `<area>-changelog.md` — append-only date-stamped log of work and meetings
- `<area>-patterns.md` — accumulated reusable techniques specific to this area
- `meetings/` — folder for meeting notes
- `1on1/` — folder for 1:1 notes (if applicable)

## Hard rules

1. **Capture first, organize later.** New, unsorted notes go to `00-inbox/` and get moved during weekly review.
2. **Atomic permanent notes.** One claim per note in `05-notes/permanent/`. If a note has multiple claims, split it.
3. **Daily notes are append-only during the day.** Edit historical daily notes only to fix typos or add cross-links.
4. **Archive, don't delete.** Move to `04-archive/` rather than removing files.
5. **The brain skill never `git commit` or `git push` here.** All vault git history is owned by the user.

## Weekly review checklist

- [ ] Process `00-inbox/` to zero
- [ ] Move done items from `01-projects/` to `04-archive/`
- [ ] Skim daily notes for the week, promote insights to `05-notes/permanent/`
- [ ] Update area `<area>-index.md` MOCs with any new notes
- [ ] Commit the week
