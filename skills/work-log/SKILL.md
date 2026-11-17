---
name: work-log
description: Build a Jira timesheet for one or more workdays from the user's git commits across the repos in ~/code, weighted by commit complexity and shaped like their past Jira worklogs, then log it to Jira only after the user approves the sheet. Use whenever the user types /work-log, or asks to log time, fill a timesheet, or book hours for given days.
---

# work-log

Draft a timesheet from real evidence, show it, and write worklogs to Jira only after the user says the sheet is approved. Logging time is outward-facing and hard to undo, so never call `addWorklogToJiraIssue` before an explicit approval of the exact sheet shown.

## Working hours

- Monday to Friday, 08:00 to 17:00 local time, 8 hours logged per day. Skip weekends unless the user asks.
- One hour of lunch per day. Pick it at random per day from 13:00-14:00, 14:00-15:00 and 14:30-15:30, and vary it between days. Nothing is logged inside lunch.
- Time is in 15-minute steps. An entry that runs into lunch is split into two worklogs, one before and one after.
- Use the timezone offset found in the user's previous worklogs for `started` (for example `+0530`).

## Steps

1. **Dates.** Resolve the days asked for (for example "16 and 17" means those dates in the current month). Drop weekend days and say so.
2. **Identity.** `git config user.email` for the commit author, and `atlassianUserInfo` for the Jira account id. Get the cloud id from the site in recent ticket links (for example `yoursite.atlassian.net`).
3. **Past worklogs.** Run JQL `worklogAuthor = currentUser() AND worklogDate >= "<about 3 weeks back>"` with `fields: ["summary","worklog"]`. The result is large and is saved to a file: parse it with `python3 -I`, keep only the user's worklogs, and print `started | offset | timeSpent | key | comment`. From this learn the start time, offset, typical entry sizes and comment style. Also check that the target days are not already logged. If a day already has time, show it and log only the gap.
4. **Commits.** For every git repo directly under `~/code`, run `git log --all --no-merges --author=<email> --since=<day 00:00> --until=<next day 05:00>` with `--date=format:'%Y-%m-%d %H:%M %z' --pretty='%ad|%h|%s' --shortstat`. Commits made between 00:00 and 05:00 belong to the previous workday. Take the Jira key from the subject (`[A-Z][A-Z0-9]+-[0-9]+`). Commits without a key go to the user to place or drop.
5. **Supporting evidence.** JQL `(assignee = currentUser() OR assignee was currentUser()) AND updated >= <first day> AND updated < <day after last>` with `fields: ["summary","status","created","updated"]`. Use created and updated times to order work within a day and to place commits that landed after midnight.
6. **Complexity.** Size each key's commits from the shortstat and the diff (`git show --stat`, and the diff itself when the size is unclear):
   - Small (about 1 file, under 40 changed lines, copy or config): 15m to 1h.
   - Medium (2 to 4 files or 40 to 150 lines, one module): 1h to 3h.
   - Large (5 or more files, over 150 lines, schema change, several modules or agents): 3h to 6h.
   Planning, review and testing go with the ticket they served. Scale the estimates so each day totals exactly 8h, and round to 15 minutes.
7. **Schedule.** Lay the entries out from 08:00 in the order the evidence suggests (commit time, Jira created and updated times). Skip the lunch hour and split any entry that crosses it. One ticket can appear more than once in a day.
8. **Comments.** One line per entry in the user's style from step 3: sentence case, says what was done, no ticket key, no trailing full stop. Two entries for the same ticket get different comments that describe each part.
9. **Gaps.** If a day has no commits or too little evidence to fill 8 hours, say so plainly. Propose moving adjacent work onto it or ask what to log. Never invent tickets.

## Output

Show one table per day, then the per-ticket totals and the grand total:

| Start | End | Time | Ticket | Comment |
|---|---|---|---|---|

Under the tables, list the evidence used (commits with their sizes) and every assumption made, such as work moved between days or lunch slots chosen. Then ask for approval and stop.

## Logging (after approval only)

- Call `addWorklogToJiraIssue` once per row with `issueIdOrKey`, `timeSpent` (for example `1h 45m`), `started` as `YYYY-MM-DDTHH:MM:00.000<offset>`, `commentBody` set to the comment, and `contentFormat: "markdown"`.
- If the user edits the sheet, show the changed sheet again before logging.
- Afterwards, list every worklog created with its id. If any call fails, say which rows failed and don't retry them blindly.
