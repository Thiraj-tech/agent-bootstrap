---
name: updatemem
description: Log or recall a project's implementation history in a token-efficient, two-part local file structure under .claude/memory/ - a small index (reference number + short title) plus one file per reference holding the full description and a dated changelog of later updates. Use whenever the user types /updatemem, or asks to log what was just built, record an implementation decision, or look up how a past feature was implemented. Never commits or touches git.
allowed-tools: Read, Write, Edit, Grep, Glob, Bash(date +%F)
---

# updatemem

Keep a per-project implementation log that stays cheap to read even as it grows, by never loading more than one entry at a time.

## Files

- `.claude/memory/index.md` — one line per entry: reference number, short title, last-updated date. This is the only file ever read in full; it should stay small no matter how long the project runs. Create it (with a one-line header and an empty table) if it doesn't exist yet.
- `.claude/memory/details/<ref>.md` — one file per reference, e.g. `details/REF-003.md`. Holds a `# <ref> — <title>` heading, the original detailed description, and any later `### Update — YYYY-MM-DD` sub-sections appended underneath. Never rewritten in place — history only ever grows. Create the `details/` directory on first use.

Both live under the *installing project's* `.claude/`, not inside this plugin. Most projects that adopt this pattern gitignore `.claude/`, so treat this as a local, single-machine log unless the user says otherwise — don't assume it's shared with teammates.

## Behavior

1. **Explicit-ref fast path.** If the invocation names a reference directly (e.g. `/updatemem REF-003 fixed the retry bug`), skip matching entirely and go to step 4 for that ref. This is the expected common path once the project has any history — the user usually knows what they're updating.
2. Otherwise, work out a short title for what was just built or changed, from the recent conversation. Ask the user for one if it isn't obvious.
3. Read `index.md` (treat a missing file as empty). Compare the new title against existing titles by keyword overlap:
   - Confident single match → treat as an update to that reference, go to step 4.
   - No plausible match → treat as new, go to step 5.
   - Ambiguous (more than one plausible match, or only partial overlap) → ask the user which existing reference this belongs to, or confirm it's new. Do not guess.
4. **Existing entry:** `Read` `details/<ref>.md` directly and surface its contents back to the user/conversation as context — this also covers the "recall a past feature" use case, since fetching is the same step whether or not anything new gets appended. If there's new implementation detail, a fix, or a change worth logging, append a `### Update — <today's date>` section (get the date with `date +%F`) to that file, and update that entry's date in `index.md`. Never delete or rewrite the original description.
5. **New entry:** find the current highest reference number in `index.md` (zero-padded, `REF-001`, `REF-002`, ...) and increment it; if the file is empty, start at `REF-001`. Append a row to `index.md`, and create `details/<ref>.md` with a `# <ref> — <title>` heading and the detailed description supplied.
6. Report the reference number and a one-line summary of what was written or found. Never run any git command from this skill.
