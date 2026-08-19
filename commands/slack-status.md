---
description: Show Slackline status — every connected session with pid liveness, poll interval, paused state, bound thread, and working dir.
allowed-tools: Bash, Read
---

Show **Slackline status**. Read-only.

1. If `~/.claude/slack-bridge/sessions/` is empty or missing → print "No Slackline sessions." Stop.
2. For each `~/.claude/slack-bridge/sessions/*.json`: read it, check `pid` with `kill -0`, and print
   a table row: `LABEL | <live|dead|closed> | every <interval> | <paused?> | thread <thread_ts> | cwd`.
3. If a session has a dead pid but `status != "closed"`, note it as "stale — its terminal is gone;
   run `/slack-disconnect <LABEL>` to clean up."

Do not modify anything.
