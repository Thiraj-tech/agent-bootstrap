---
name: agent-group-generator
description: Interview the user about their project and generate a tailored group of Claude Code subagents (.claude/agents/*.md) for it, using reusable patterns (comment policy, read-only-DB-via-MCP contract, delegation/routing between sibling agents, a self-maintenance footer) instead of writing each agent from scratch. Use whenever the user types /agent-group-generator, or asks to set up subagents, split work across specialist agents, or build an "agent team" for a codebase.
---

# agent-group-generator

Generate a small group of project-specific subagents by interviewing the user, not by writing generic boilerplate. The output should read like it was hand-written for that one codebase.

## Step 1 — Interview

Ask only what you need; skip a question if the answer is already obvious from the repo (check for a README, package manifest, or existing `.claude/` setup first).

1. **Work areas.** What are this project's natural areas of work — the boundaries a team would already think in (e.g. "frontend", "billing service", "mobile app", "infra")? Aim for 1-4 areas; more than that usually means the areas are too finely sliced. Each area becomes one agent.
2. **Data/DB agent.** Does the project need a database specialist agent? If yes:
   - What DB engine, and how is it normally accessed from code (ORM, raw SQL, migrations)?
   - Is there a read-only MCP server or tool already set up for querying it? Get the exact tool names — do not guess them.
   - Should writes always be handed to the user as SQL to run themselves (recommended default), or does this project have a safe automated migration path?
3. **Comment policy.** Should generated agents carry a hard "never add code comments unless explicitly asked" rule? (Common preference, but confirm — don't assume.)
4. **Conventions.** Any branch/commit naming convention, vendored-code directories to exclude from searches, or other repo-wide rules worth baking into every agent?
5. **Output location.** Default `.claude/agents/`, confirm or change. Ask whether to also append an "Agents and routing" section to the project's CLAUDE.md (only if one exists or the user wants one created).

If the user answers all of this in their initial request, don't re-ask — just confirm your understanding in one line before writing files.

## Step 2 — Assemble each agent from these building blocks

Every generated agent's frontmatter needs `name`, a `description` that states its area and says to use it proactively for that area, and `tools:` scoped to what it actually needs (a code-focused agent typically wants `Read, Grep, Glob, Edit, Write, Bash`; add MCP tool names only where relevant).

**1. Comment policy clause** (only if the user opted in) — insert near the top, verbatim:
> Never add comments (in any language this project uses) unless the user explicitly asks.

**2. Read-only-DB-via-MCP contract** (only for the data/DB agent) — adapt this structure with the interview's answers filled in:
- No `Bash` in its `tools:` list — state explicitly that this is on purpose, so a future edit doesn't casually add it back.
- It may only run read operations (SELECT/SHOW/DESCRIBE/EXPLAIN or the read-only equivalents for the project's engine) through the named MCP tool(s). It must never run a write statement through that tool even if the tool would technically accept one, and never reach the DB through any other path (a CLI client, an app script, an ORM console).
- For every data-changing statement, give the user three things in order: (a) a read query that shows exactly what will be affected, (b) the change itself, scoped tightly and wrapped so the user can inspect before committing (a transaction they commit/roll back themselves, or the closest equivalent for the engine), (c) a verification read query for afterward. Never suggest an unscoped UPDATE/DELETE.
- If the MCP tool is missing or errors, don't work around it — hand the query to the user instead.

**3. Delegation/routing convention** (once there's more than one agent) — each agent's `description` names its area; add one line to each agent body naming the sibling agent(s) to defer to for the other side of a cross-area workflow, rather than reaching into files outside its own area. Add a short note (to the project's CLAUDE.md if it has one, otherwise to each agent) that the orchestrating session is responsible for splitting and sequencing cross-area work.

**4. Self-maintenance footer** — every agent ends with a section titled `## Keeping this agent current`:
- State what commit/state its factual claims (file paths, function names, schema, statuses) were verified against.
- State when to update it: whenever the user gives new facts, applies a change the agent describes, or drift is suspected.
- Instruct it to delete facts that are no longer true rather than leave them stale, and to re-verify against the current code/schema before relying on an old claim.

## Step 3 — Write the files

Use the Write tool to create each `.claude/agents/<name>.md` directly — don't just print them for the user to copy by hand. If a CLAUDE.md routing section was requested, append it (or create a minimal CLAUDE.md if none exists) rather than duplicating the routing rule inside every agent file.

## Step 4 — Report back

List the files written, one line per agent naming its area, and remind the user this project's own `.gitignore` may need to include `.claude/` (or just `.claude/agents/`) if they don't want these committed — check `.gitignore` first and only mention it if it doesn't already cover the path.
