---
description: Connect THIS Claude session to your Slack self-DM — binds this terminal to a new Slack thread you can drive from your phone. (You can also just say "connect to slack".)
argument-hint: "[LABEL] [interval e.g. 1m|5m] [in <path>]"
allowed-tools: Bash, Read, Write, Edit, mcp__slack__slack_read_user_profile, mcp__slack__slack_send_message, mcp__slack__slack_read_thread
---

Connect **this** session to Slack. Args = `$ARGUMENTS`: optional `LABEL`, optional poll `interval`
(default `1m`), optional `in <path>` for the working dir (default: current dir).

0. **FIRST, load the skill (mandatory):** use the **Skill tool** to load the `slackline-session`
   skill, and do everything below by following *that* skill's instructions. Do NOT connect or poll
   from memory or from this command text — the SKILL.md is the only source of truth (message envelope,
   heartbeat, interrupts, approvals all live there, and improvising skips them).
0b. **Then run the skill's Connect GATE (step 0):** the bypass-mode banner is the **FIRST** thing you
   print — before any env check, config read, anchor, or cron (skill step 0a). Then do the best-effort
   mode check: only if you can positively confirm this session is NOT in
   `--permission-mode bypassPermissions`, STOP after the banner instead of connecting. Otherwise proceed.
   Mode often can't be detected, so the banner always leads — never connect silently. A non-bypass
   session hangs the moment you're away.
1. Ensure `~/.claude/slack-bridge/sessions/` exists.
2. **Pick LABEL:** the arg if given; else the basename of `cwd`; else `session-1`. If a
   `sessions/<LABEL>.json` already exists with a LIVE pid (`kill -0`), pick a `-2`, `-3`… suffix so
   two live sessions never share a label.
3. **Run the connect path of the now-loaded `slackline-session` skill** for this LABEL: resolve the
   self-DM, post the `🤖 Slackline connected …` anchor, write `config.json` + `sessions/<LABEL>.json`,
   record this pid, and `CronCreate` the poll (per `${CLAUDE_PLUGIN_ROOT}/lib/registry.md`). Note: the
   cron prompt itself must instruct each cycle to load the skill first (see the skill's Connect step).
   Then run the first poll cycle.
4. Print a summary: label, thread bound, poll interval, cwd, and how to drive it (reply in that
   thread from your phone; you can interrupt long work with `cancel`/`hold` or a new instruction;
   `stop` in-thread asks for confirmation before disconnecting, or run `/slack-disconnect` here).
   Note: the bypass-mode banner is printed **first**, before any connect work (skill Connect step 0a).

**Keep this terminal open** — it only polls while this session is running and idle. Because no one
is at the keyboard, run this terminal in an autonomous mode (e.g. start it with
`claude --permission-mode bypassPermissions`, or accept-edits / pre-allowed tools); Slackline will ask
you **in Slack** before anything destructive.
