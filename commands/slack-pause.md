---
description: Pause or resume a Slackline session's polling without disconnecting it. Defaults to the session bound to this terminal.
argument-hint: "pause|resume [LABEL]"
allowed-tools: Bash, Read, Write, Edit, mcp__slack__slack_send_message
---

Pause or resume polling. Args = `$ARGUMENTS`: `pause` or `resume`, and an optional `LABEL`
(default = the session whose `pid` == this terminal's `$$`). Pausing keeps the registration,
thread binding, and cursor; it only stops the cron.

Target `sessions/<LABEL>.json` (missing / dead pid → print status, stop).

**pause:**
1. `CronDelete(cron_id)`; clear `cron_id`; set `paused:true`, `status:"paused"`.
2. Post `🤖 [<LABEL>] paused — no polling until resumed.` to its `thread_ts`.

**resume:**
1. `CronCreate(cron_expr_for(poll_interval), recurring=true,
   prompt="Invoke the slackline-session skill to run ONE poll cycle for session <LABEL>.")`;
   store `cron_id`.
2. Set `paused:false`, `status:"ready"`.
3. Post `🤖 [<LABEL>] resumed — polling every <poll_interval>.` to its `thread_ts`.

Print a one-line confirmation. (Cron mapping: `${CLAUDE_PLUGIN_ROOT}/lib/registry.md`.)
