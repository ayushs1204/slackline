---
description: Change how often a Slackline session polls its Slack thread (deletes and re-creates its cron). Defaults to the session bound to this terminal.
argument-hint: "<interval e.g. 1m|5m|10m|1h> [LABEL]"
allowed-tools: Bash, Read, Write, Edit, mcp__slack__slack_send_message
---

Change a session's poll cadence. Args = `$ARGUMENTS`: an interval, and an optional `LABEL`
(default = the session whose `pid` == this terminal's `$$`).

1. Parse the interval → cron expression via the table in `${CLAUDE_PLUGIN_ROOT}/lib/registry.md`
   (`1m`→`*/1 * * * *`, `5m`→`*/5 * * * *`, `1h`→`0 * * * *`, `Nh`→`0 */N * * *`). Cron floor is
   1 minute — reject anything faster.
2. Target `sessions/<LABEL>.json` (missing / dead pid → print status, stop).
3. If `paused`, just record the new `poll_interval` (no live cron) and say so. Otherwise
   `CronDelete(old cron_id)` then `CronCreate(new_expr, recurring=true,
   prompt="Invoke the slackline-session skill to run ONE poll cycle for session <LABEL>.")`.
   Store the new `cron_id` + `poll_interval`.
4. Post `🤖 [<LABEL>] polling every <interval>.` to its `thread_ts`. Print a one-line confirmation.
