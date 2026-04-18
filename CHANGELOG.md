# Changelog

All notable changes to this plugin are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.5.0] — 2026-04-18

### Added
- `/brain dig <query>` — deep search across snippets, debugging logs, ADRs, and area patterns. Use when default `/brain search` (permanent notes only) doesn't find what you need.
- `references/` directory — `flush.md`, `filing.md`, `promote.md`, `stats.md`, `index.md`, `lookup.md`. Verbose specs that load only when the relevant subcommand runs.
- Per-project consent map in `~/.brainrc.json` — `projects: {<cwd>: "on" | "off"}`. Session-start protocol asks once per project, then remembers.

### Changed
- **SKILL.md compressed 411 → 126 lines (-70%).** Stripped rationale paragraphs, deduped vault path resolution, tightened tables, removed inline JSON example. Verbose detail moved to `references/`.
- Vault-first lookup is now **lazy by default** — only `05-notes/permanent/` on first pass, top 5 results. Deep search is opt-in via `/brain dig`.
- README leads with the climate / token-efficiency angle.

### Performance
- Estimated ~1500 tokens saved per session from SKILL.md alone.
- Per-project consent kills ~30 tokens per repeat session.
- 1K users × 100 sessions/month ≈ 150M tokens/month not generated.

## [0.4.0] — 2026-04-18

### Added
- `99-meta/vault-index.json` cache — flat JSON of every note's path, title, tags, first line, mtime. Vault-first lookup reads this once instead of recursive grep.
- `/brain reindex` — rebuild the index from scratch.
- `/brain search <query>` — direct token-scored search via the index, top 5 cited matches.
- `/brain promote <source>` — daily-note line or phrase → permanent note with backlink, optionally replacing the original line with `[[wikilink]]`.
- `/brain stats` — vault health dashboard (volume, pipeline, areas, growth) using `Glob` + `mtime` only, no file reads.

### Changed
- Flush procedure step 10 — refresh index entries for files written/modified that session.

## [0.3.1] — 2026-04-18

### Changed
- First-run gate is now **explicit refuse + one-tap recovery** instead of silent auto-wizard.
- If vault isn't configured or scaffolded, the gate prints *"🧠 Run `/brain init` now? (y/n)"*. Answer `y` runs init then resumes the original subcommand; `n` aborts cleanly.
- README's First Run section reworked as 4 explicit steps.

## [0.3.0] — 2026-04-18

### Added
- First-run gate — every subcommand except `init` checks vault readiness before running.
- Vault path resolution now reads `~/.brainrc.json` (`{ "vault": "<path>" }`) as a fallback between `$BRAIN_VAULT` and the `~/brain` default. Init wizard persists the chosen path here.

## [0.2.0] — 2026-04-18

### Added
- Vault-first lookup — before answering technical questions, the skill greps `05-notes/permanent/`, `10-snippets/<lang>/`, `11-debugging/`, `02-areas/*/<area>-patterns.md`, and `12-adr/`. If matches found, the answer leads with vault content and cites the file path.
- `/brain quiet` — suppress vault-first lookup for the rest of the session.

## [0.1.0] — 2026-04-18

### Added
- Initial release.
- One skill (`brain`), one slash command (`/brain`), 9 templates, default `99-meta/system-rules.md`.
- Subcommands: `init`, `on`, `off`, `skip`, `status`, `save`, `note`, `adr`, `meeting`, `interview`, `1on1`.
- PARA + Zettelkasten folder scaffold via `/brain init` (15 folders).
- Vault path resolution: `$BRAIN_VAULT` → `~/brain` default.
- Hard rule: plugin never runs `git add`, `git commit`, or `git push` in the vault.
