---
name: commit-message
description: Generate a commit message from the currently staged changes - a ticket key if the branch has one, a Title Case subject, then short bullets summarising each logical change. Use whenever the user types /commit-message or asks for a commit message, commit comment, or commit summary for what they have staged. Only writes the message, never commits.
allowed-tools: Bash(git branch:*), Bash(git diff:*), Bash(git status:*), Bash(git log:*)
---

# commit-message

Write a commit message for the staged changes and hand it to the user. The user commits it themselves, so never run `git commit`, `git add`, or anything else that changes the repo.

## Steps

1. Get the branch with `git branch --show-current` and look for a ticket key: try `[A-Z]+-[0-9]+` (Jira-style, e.g. `feature/PROJ-123` gives `PROJ-123`) first, then `#[0-9]+` (GitHub issue style, e.g. `fix-123` or `123-short-desc` giving `#123`). If neither matches, check the last few `git log` subjects on this branch for a convention (a key repeated in prior commits, or a Conventional Commits type like `feat:`/`fix:`). If nothing gives you a clear prefix, ask the user, or write the subject without one — don't invent a key.
2. Read what is staged: `git diff --cached --stat`, then `git diff --cached` for the actual changes. Use only the staged diff. If nothing is staged, say so and stop. Do not fall back to unstaged changes, since the user staged deliberately and an unstaged edit would make the message wrong.
3. Work out the logical changes. Group by what the change does, not by file: three files edited for one fix is one bullet, and one file with two unrelated fixes is two bullets.
4. Write the message in the format below.

## Format

```
PROJ-123 - Short Title Case Summary Of The Whole Change

- First Change In Title Case
- Second Change In Title Case
```

(Omit the `PROJ-123 - ` prefix entirely if no ticket key/convention applies — don't leave a placeholder.)

- Subject: `<key> - <Title Case Summary>` (or just `<Title Case Summary>` with no key), imperative mood, roughly 70 characters or fewer. It should cover the whole set of changes, so if the bullets span several things, name the common thread.
- Bullets: one per logical change, starting with an imperative verb (Add, Fix, Hide, Lock, Remove, Update), Title Case like the subject, no trailing full stop, one line each. Aim for 2 to 6. Fewer is fine for a small change, and a one-line change can skip the bullets.
- Describe behaviour, not files. "Hide Checklist Tab For Terminal Statuses" tells a reviewer something; "Edit view.php" does not.
- No comments, code blocks, or emoji inside the message, and no attribution trailers unless the user asks or the project's own conventions require them.

## Output

Show the message in a single fenced block so it is easy to copy, then add one line saying what was staged (for example "4 files staged") and, if `git status` shows unstaged changes that were left out, mention that so the user can decide. Do not commit.
