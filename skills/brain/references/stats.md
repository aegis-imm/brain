# Stats Dashboard

`/brain stats` walks vault and prints markdown. Use `Glob` + `mtime` only — no file-content reads.

## Output format

```markdown
## Vault stats — <date>

**Volume**
- Total notes: <n>
- Permanent notes: <n>
- Daily notes: <n> (streak: <days>)
- Snippets: <n>
- Debugging logs: <n>
- ADRs: <n>

**Pipeline**
- Inbox unsorted: <n> ⚠ if >10
- Active projects (work): <n>
- Active projects (side): <n>
- Stale projects (>30d no edit): <n> — list paths

**Areas**
| Area | Notes | Last update |
|---|---|---|
| <area> | <n> | <YYYY-MM-DD> |

**Recent**
- Last `/brain save`: <date>
- Notes added this week: <n>
- Permanent-note growth (30d): +<n>
```

## Cost discipline

- Never read note bodies — only `Glob` paths and `stat` mtimes.
- Cap stale-project list at 10 entries. Truncate with "+N more" if exceeded.
