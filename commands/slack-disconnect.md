---
description: Disconnect a Slackline session — cancel its poll cron and close its Slack binding. Defaults to the session bound to this terminal.
argument-hint: "[LABEL] | all"
allowed-tools: Bash, Read, Write, Edit, mcp__slack__slack_send_message
---

Disconnect Slackline. Target = `$ARGUMENTS` (a `LABEL`, `all`, or empty = the session bound to
**this** terminal — match `sessions/*.json` whose `pid` == `$$`).

For each targeted `sessions/<LABEL>.json`:
1. `CronDelete(cron_id)`.
2. Post `🤖 [<LABEL>] disconnected — you can close this window.` to its `thread_ts`
   (`channel_id` from `config.json`).
3. Set `status:"closed"` (leave the file for `/slack-status` history).

If no target matches a session, print what IS connected (same info as `/slack-status`) and ask
which to disconnect. Note: this only cancels polling; the Terminal window must be closed manually.
