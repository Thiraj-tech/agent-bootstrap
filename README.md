# agent-bootstrap

A Claude Code plugin that bootstraps a tailored subagent group for any project and bundles a handful of generally useful skills, so you don't have to rebuild this setup by hand in every repo.

## What's in it

**Original to this plugin:**
- **`agent-group-generator`** — interviews you about your project's work areas, then writes a `.claude/agents/*.md` subagent for each one, built from reusable patterns: an optional "never comment code" rule, a read-only-database-via-MCP contract for a data specialist agent, a delegation convention between sibling agents, and a self-maintenance footer so each agent tells future-you when its facts need re-checking.
- **`commit-message`** — writes a commit message from your currently staged changes (ticket key from the branch if there is one, Title Case subject, bullets per logical change). Never runs `git commit` itself.
- **`updatemem`** — a token-efficient, per-project implementation log under `.claude/memory/`: a small index (reference number + title) plus one file per entry, so recalling or updating a past feature only ever costs reading one small file, not a whole history. Supports `/updatemem REF-003 ...` to jump straight to a known entry, and falls back to fuzzy title matching (with a confirmation step when it's ambiguous) otherwise.

**Vendored, unmodified, from upstream (see [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)):**
- **`ponytail`** and its `ponytail-audit` / `ponytail-debt` / `ponytail-gain` / `ponytail-help` / `ponytail-review` companions — lazy/YAGNI-mode code review and simplification, from [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) (MIT).
- **`code-review`** command — a single `/code-review` command for reviewing a diff or PR. This is the simpler community-marketplace version, **not** Claude Code's built-in multi-agent `/code-review` (levels, `--comment`, `--fix`, cloud review) — if you already have that available, prefer it; this one is here for environments that don't.
- **`frontend-design`** and **`skill-creator`** skills, from Anthropic's official plugin marketplace (Apache-2.0).

## Install

```
/plugin marketplace add Thiraj-tech/agent-bootstrap
/plugin install agent-bootstrap
```

## Usage

- `/agent-group-generator` (or just ask to "set up subagents for this project") in any repo — answer its short interview and it writes the agent files directly.
- `/commit-message` with changes staged.
- `/updatemem` after building or fixing something worth remembering, or `/updatemem REF-003 ...` to update a known entry. The log lives under the installing project's `.claude/memory/` — most projects gitignore `.claude/`, so treat this as a local, single-machine log unless you deliberately carve out that path in your `.gitignore`.
- `ponytail`, `ponytail-review`, etc. and `/code-review` work as their upstream docs describe.

## A note on `skill-creator`

This session may already have `skill-creator` available from the official Anthropic marketplace. Installing `agent-bootstrap` alongside it gives you two copies under different plugin namespaces (`agent-bootstrap:skill-creator` vs `skill-creator:skill-creator`) — harmless, since Claude Code namespaces skills by plugin, just don't be surprised to see it listed twice.

## Maintaining this plugin

- **Local test loop:** `/plugin marketplace add <local path>` once, then `/plugin marketplace update agent-bootstrap` after each edit, and re-test in a scratch project before pushing.
- **Versioning:** bump `version` in `.claude-plugin/plugin.json` (semver) and tag the matching commit on each release.
- **Refining `agent-group-generator`:** its real test is running it against a different real project each time — note where the interview was unclear or the output needed hand-editing, and fold that back into `skills/agent-group-generator/SKILL.md`.
- **Keeping vendored copies current:** these are copies, not references, so they will drift from upstream. Periodically diff them against the live upstream sources and pull in changes; record the last-synced upstream version in `THIRD_PARTY_NOTICES.md`.

## License

This repo's own content (agent-group-generator, commit-message, updatemem, and the packaging around them) is MIT — see [LICENSE](LICENSE). Vendored third-party content keeps its own upstream license — see [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
