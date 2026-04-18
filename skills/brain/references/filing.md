# Meeting / Interview / 1on1 Filing Rules

When user asks to file a meeting, 1:1, or interview note. If `BRAIN=on`, file directly; else prompt for `/brain on` first.

## Routing table

| Type | Path | Template |
|---|---|---|
| Generic meeting (project-scoped) | `02-areas/career/<area>/meetings/<YYYY-MM-DD>-<short-topic>.md` | `meeting-note-template.md` |
| 1:1 | `02-areas/career/<area>/1on1/<YYYY-MM-DD>-<name>.md` | `1on1-template.md` |
| Requirements interview | `02-areas/learning/research-methodology/interviews/<YYYY-MM-DD>-<stakeholder>-<topic>.md` | `meeting-note-template.md` (mom-test framing) |
| Standalone (no area) | `06-daily/<today>.md` as `## Meeting — <topic> @ HH:MM` | `meeting-note-template.md` body only |
| Cross-team | `06-daily/<today>.md`, then ask if it should also live in `02-areas/career/general/meetings/` | `meeting-note-template.md` |

## Filename rules

- `<short-topic>` = lowercase kebab-case, ≤5 words
- `<stakeholder>` = person/org being interviewed, kebab-case (initials OK if privacy-sensitive)
- Date always padded `YYYY-MM-DD`

## After filing

- Append 1-line entry to `<area>-changelog.md`: `<YYYY-MM-DD> · meeting · <topic> · <decisions count>`
- Cross-link from `07-maps/<area>-index.md` under "Active meetings / interviews" (create section if missing)

## Mom-test discipline (interviews only)

- Lead with **concrete past behavior** — "last time you needed X, what did you do?"
- Capture **current workarounds** — that's the real demand signal
- Mark **painful** vs **nice-to-have** explicitly
- End with **commitment ask** — next step, intro, follow-up. Did they commit?
