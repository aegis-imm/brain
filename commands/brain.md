---
description: Second-brain vault dispatcher. Subcommands - on / off / skip / status / save / note <text> / init / adr / meeting <topic> / interview <stakeholder> / 1on1 <name>
---

# /brain — Second-Brain Dispatcher

User invoked: `/brain $ARGUMENTS`

Parse `$ARGUMENTS` as `<subcommand> [rest...]`. Dispatch per the table below by following the **brain skill** (loaded from `skills/brain/SKILL.md`).

| Subcommand | Action |
|---|---|
| (empty) | Print usage: list all subcommands and current `BRAIN` flag state |
| `on` | Set `BRAIN=on`. If first time this session, read `<vault>/99-meta/system-rules.md` |
| `off` / `skip` | Set `BRAIN=off`. Discard session log without writing |
| `status` | Print in-memory buckets as a markdown preview. **Do not write to the vault.** |
| `save` | Run the **Flush Procedure** from the skill (steps 1–9) |
| `note <text>` | Classify `<text>` into the appropriate bucket and append immediately. Confirm classification briefly. |
| `init` | Run the **Init Procedure** from the skill — scaffold PARA folders + copy templates + default `system-rules.md` |
| `adr` | Create `12-adr/adr-NNN-<slug>.md` from `adr-template.md` for the most recent load-bearing decision |
| `meeting <topic>` | File a meeting note per **Meeting + Interview Filing Rules** in the skill |
| `interview <stakeholder>` | File a requirements interview note (mom-test discipline) |
| `1on1 <name>` | File a 1:1 note under the current area's `1on1/` folder |
| `quiet` | Suppress vault-first lookup for the rest of the session (advisor + retrieval go silent) |

**Vault path resolution:** `$BRAIN_VAULT` env var first, else `~/brain`. If the vault doesn't exist and the subcommand requires it, suggest `/brain init`.

**Hard rule:** Never run `git add`, `git commit`, or `git push` in the vault. Staging via `Edit`/`Write` only.

After any write action, end with the literal line:

> ✅ Brain staged. Review with `cd <vault> && git diff --stat HEAD` then commit when ready.
