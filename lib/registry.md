# Slackline registry & shared conventions

Single source of truth for the on-disk state that Slackline sessions share. The
`slackline-session` skill and the `/slack-*` commands all read/write these files.

## Base directory

`~/.claude/slack-bridge/` (expand `~` to the real home). Create it if missing.

```
~/.claude/slack-bridge/
├── config.json                 # global config, written once on first connect
└── sessions/
    └── <LABEL>.json            # one per connected Claude session/terminal
```

There is **no launcher and no message relay** — each connected session polls only its OWN Slack
thread. Parallelism = open N terminals and `/slack-connect` each; N independent threads, no races.

## config.json

```json
{
  "channel_id": "D0XXXXXXX",   // resolved self-DM channel id (a "D..." id)
  "user_id": "U0XXXXXXX",      // the logged-in Slack user id
  "marker": "🤖"               // prefix on every message WE post; never act on these
}
```

`channel_id` is discovered on first connect: post the "connected" anchor message to `user_id`,
and read `channel_id` from the send response (it comes back as a `D…` id). Use that `D…` id for
all later `slack_read_thread` / `slack_send_message` calls.

## sessions/<LABEL>.json

```json
{
  "label": "DEV",
  "thread_ts": "1786990147.236469",   // the self-DM anchor message that binds this session
  "cwd": "/Users/<you>/project",
  "pid": 23456,                        // pid of THIS session's claude process ($$)
  "last_reply_ts": "1786990179.984819",// CURSOR: highest ts SEEN in this thread, incl. your own 🤖 posts
  "cron_id": "ef56gh78",               // CronCreate job id for this session's poll
  "poll_interval": "1m",
  "paused": false,
  "status": "ready",                   // connecting | ready | paused | closed
  "heartbeat_ts": "2026-08-18T09:01:00Z",
  "auto_approve": []                   // verbs pre-approved this session, e.g. ["git","npm"]
}
```

## The `🤖` marker rule (loop-safety)

- Every message a session posts MUST start with the `marker` (`🤖`).
- When reading, IGNORE any message whose text starts with the marker — that is our own output.
  Match the marker robustly: leading `🤖` (unicode) OR the shortcode `:robot_face:`, since
  `slack_read_thread` may return either form.
- **The MCP posts as the logged-in user**, so a message's author id does NOT distinguish the
  session's own posts from the human's replies — the `🤖` marker is the ONLY discriminator. Get it
  wrong and the session either loops on itself or ignores real commands.
- Cursor discipline: `last_reply_ts` is the highest ts *seen* (any author, incl. own posts).
  Advance it every cycle to the max ts returned plus the ts of any reply just posted; compare ts
  **numerically** (decimal), never as strings. This is what prevents re-processing and prevents
  "already replied, so skip" mistakes. There is no in-thread trigger word.

## Approval convention (approvals go to Slack, not the terminal)

Because no one is at the connected terminal, native permission prompts must never be relied on.
Before any **destructive / irreversible / ambiguous / outward-facing / outside-cwd** action, the
session posts `🤖 [LABEL] [approve?] <description> — `\``<exact command>`\`` — reply yes/no.` and
waits inline (poll the thread every ~12s, up to ~5 min):

- `yes` / `y` / `approve` / `ok` / `do it` → proceed.
- `no` / `n` / `deny` / `cancel` / `stop`, or timeout → skip; reply `🤖 [LABEL] skipped — no approval.`
- `yes always <verb>` → append `<verb>` to `auto_approve`; commands starting with that verb skip
  the ask for the rest of the session.

Read-only actions (Read/Grep/Glob/`ls`/`cat`/`git status`, etc.) never ask.

**Load the skill, don't improvise.** These notes are a *reference schema*, NOT a substitute for the
`slackline-session` skill. Every connect and every poll cycle must first **load `slackline-session` via
the Skill tool** and follow it — running a cycle from these notes or the cron prompt alone skips the
message envelope, heartbeat, and interrupt handling. The cron prompt created on connect enforces this.

**Related session behaviors** (defined in the `slackline-session` skill, noted here so this file
doesn't contradict it): `stop`/`quit`/`shutdown`/`exit` is **confirmation-gated** — the session asks
`[approve?] close this session … — yes/no` and only tears down (CronDelete + `status:"closed"`) on a
`yes`. Long-running work is **acknowledged up front**, posts progress per step, and sends a **status
heartbeat at least every ~2 min** while an op runs past ~2 min; `heartbeat_ts` is refreshed each cycle.
A new non-`🤖` message mid-task **interrupts** (abort words stop and stay live; any other instruction
pre-empts and switches).

## Interval → cron expression

Poll cadence is stored as a short string (`1m`, `5m`, `10m`, `1h`). Convert to a 5-field cron for
`CronCreate`. Cron's floor is 1 minute.

| Interval | Cron          | Notes                              |
|----------|---------------|------------------------------------|
| `1m`     | `*/1 * * * *` | default (fastest cron allows)      |
| `Nm` N≤59| `*/N * * * *` | N should divide evenly; else round |
| `1h`     | `0 * * * *`   | hourly                             |
| `Nh` N≤23| `0 */N * * *` | every N hours                      |

Changing an interval = `CronDelete(old cron_id)` then `CronCreate(new expr, recurring:true)`, and
store the new `cron_id` + `poll_interval` in the session file.

## PID liveness

A session is "live" if `kill -0 <pid>` succeeds. Use this to default `/slack-*` commands to the
current terminal's session (`pid == $$`), to avoid label collisions on connect, and to flag stale
sessions in `/slack-status`.
