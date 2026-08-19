---
name: slackline-session
description: Connect THIS Claude session to a Slack self-DM thread and serve it CLI-style — poll the thread, run commands with full tools in the working dir, reply in-thread, and ask in Slack (not the terminal) before anything destructive. Used by /slack-connect and by its own poll cron; also the target when the user says "connect to slack".
---

# Slackline session — connect & poll cycle

You bind **this** Claude session to one Slack self-DM thread and act as a remote CLI:
the user replies in that thread from their phone, you run the work here and reply in-thread.
No launcher, no spawning — the human opened this terminal; you just connect it.

Read `${CLAUDE_PLUGIN_ROOT}/lib/registry.md` for file schemas / the interval→cron mapping /
the approval convention if unsure. Base dir: `~/.claude/slack-bridge/`. Marker = `🤖`.
Your identity file: `~/.claude/slack-bridge/sessions/<LABEL>.json`.

## Connect (first invocation — when no cron is scheduled yet for this LABEL)
0. **GATE — this session MUST be in `bypassPermissions` mode.** Once you're away from the keyboard, any
   non-bypass session hangs forever on the first terminal permission prompt (see the Approval-gate
   rationale). The bypass reminder is **unconditional and comes FIRST**, because the mode often can't be
   detected reliably and a silent/late reminder leaves the user thinking it's fine.

   **(a) SHOW THE BANNER FIRST — before any other action.** The VERY FIRST thing you do on connect —
   *before* the env-var check, *before* `mkdir`, *before* reading `config.json`, *before* anchoring the
   thread or creating the cron, before any tool call at all — is print the banner below to the terminal.
   Rules:
   - First resolve `<cwd>` (the `in <path>` arg, else current dir), `<LABEL>` (arg, else `basename cwd`),
     and `<interval>` (arg, else `1m`) — these need no tool call, so the banner can lead.
   - Print it **VERBATIM inside the box below** (a real fenced code block, so it renders as a highlighted,
     monospaced banner the operator cannot miss) — do NOT paraphrase it, do NOT fold it into a prose
     sentence, do NOT drop the box, do NOT move it below the summary.
   - If Slackline was loaded via `--plugin-dir`, append that same `--plugin-dir <path>` to the launch
     command in step 2 of the box.

   **(b) Best-effort mode check** (AFTER the banner is printed): try to determine the current mode —
   `Bash: echo "${CLAUDE_PERMISSION_MODE:-}${CLAUDE_CODE_PERMISSION_MODE:-}"` (set on some versions,
   empty on others); if empty, fall back to what you know about this session's launch. **Only if you can
   POSITIVELY confirm it is NOT bypass** → STOP: do not anchor, write state, or create the cron; the
   banner you already printed tells the user how to relaunch. Otherwise proceed with the connect.

   Terminal banner (print exactly this, filled in — FIRST, before anything else):
   ```
   ┌────────────────────────────────────────────────────────────────────────────┐
   │  ℹ️  INFO — JUST A HEADS-UP, NO ACTION NEEDED IF ALREADY IN BYPASS MODE       │
   ├────────────────────────────────────────────────────────────────────────────┤
   │                                                                            │
   │   ✅  ALREADY in bypass? (launched with --permission-mode                  │
   │       bypassPermissions, OR set it as your default in settings/your IDE)   │
   │       You're all set — ignore everything below. This is just an FYI.       │
   │                                                                            │
   ├──────────────────────  only if you are NOT  ───────────────────────────────┤
   │                                                                            │
   │  Slackline only works in bypassPermissions mode. If this session isn't in  │
   │  it, I'll FREEZE on the first approval prompt once you walk away — and you  │
   │  won't know why. I can't reliably detect the mode, so relaunch to be sure: │
   │                                                                            │
   │    1) Exit this session:  Ctrl-C twice  (or type /exit)                    │
   │    2) Run in the project you want to drive:                                │
   │                                                                            │
   │         cd "<cwd>" && claude --permission-mode bypassPermissions           │
   │                                                                            │
   │    3) Reconnect:  /slack-connect <LABEL> <interval> in <cwd>               │
   │                                                                            │
   │  VS Code / Cursor: set bypass as default instead — see the README          │
   │  ("Make bypass the default"). No need to relaunch from a terminal.         │
   │                                                                            │
   └────────────────────────────────────────────────────────────────────────────┘
   ```
   After the banner and the connect steps, also add a short one-line bypass note to the **Slack anchor**
   message (the box is terminal-only). Never connect silently without the reminder.
1. Ensure `~/.claude/slack-bridge/sessions/` exists.
2. If `config.json` is missing: call `slack_read_user_profile` → `user_id`. (Reuse it if present.)
3. **Anchor the thread:** `slack_send_message(channel_id=<user_id or config.channel_id>,
   message="🤖 Slackline connected — reply in this thread to drive me. (session <LABEL>, cwd <cwd>)")`.
   The response returns the real `D…` `channel_id` and the message `ts`. This `ts` is your
   `thread_ts` and your initial `last_reply_ts`.
4. Write `config.json` if missing: `{channel_id, user_id, marker:"🤖"}`.
5. Record pid (`echo $$`). Write `sessions/<LABEL>.json` (see registry schema):
   `status:"ready"`, `last_reply_ts`=anchor ts, `poll_interval` (arg or `1m`), `paused:false`,
   `auto_approve:[]`, `cwd`.
6. `CronCreate(cron_expr_for(poll_interval),
   prompt="Slackline poll for session <LABEL>. STEP 1 (MANDATORY, do this FIRST, before anything
   else): use the Skill tool to load the 'slackline-session' skill. Do NOT run the poll cycle from
   memory, from this prompt, or from the registry notes — the SKILL.md you load is the ONLY source of
   truth (it defines the message envelope, the ~2-min heartbeat, interrupt handling, and approvals,
   and improvising skips all of them). STEP 2: follow that skill's Poll-cycle section to run EXACTLY
   one cycle for session <LABEL>, then stop.",
   recurring=true)`; store the returned id as `cron_id`.
7. Fall through and run one poll cycle now.

## Poll cycle (every invocation)
Read `sessions/<LABEL>.json` and `config.json`. If `paused` → end quietly.

> **There is NO trigger keyword inside a thread.** Every reply that is not yours is a command —
> even a bare one like `date`, `ls`, or `yes`. Do NOT wait for `claude!` or any prefix; do NOT skip
> a message just because you already replied earlier in this thread. The `claude!` word does not
> apply here.

`last_reply_ts` is your **cursor**: the highest message `ts` you have already seen in this thread
(any author, *including your own* `🤖` posts). It starts at the anchor ts.

1. `slack_read_thread(channel_id=CH, message_ts=<thread_ts>, oldest=<last_reply_ts>)`. This may
   re-return the message at the cursor and your own past replies — that's expected.
2. **Identify which are yours.** A message is YOURS if its text starts with the marker — match it
   robustly: leading `🤖` (unicode) OR the shortcode `:robot_face:` (Slack may return either).
   Remember: MCP posts as you, so author id does NOT distinguish you from the user — only the marker
   does.
3. **New commands** = messages whose `ts` is **numerically greater** than the cursor (compare as
   decimal numbers, e.g. `awk`/`bc` or float compare — not string compare) AND that are NOT yours.
   Process oldest → newest; run each (see **Run a command**), replying in `<thread_ts>`.
4. **Advance the cursor** to the numeric max `ts` of *everything* the read returned **plus** the ts
   of every reply you posted this cycle — including any acknowledgment/progress lines (see
   "Long-running work") and file uploads (`slack_send_message` returns each ts — use them). So after a
   cycle the cursor sits at your own latest reply, and the user's next message is the only thing newer.
5. Persist `sessions/<LABEL>.json` (last_reply_ts, cwd, status, heartbeat_ts = `date -u +%Y-%m-%dT%H:%M:%SZ`).
   Post NOTHING if there were no new commands. One invocation = one cycle (no internal loop, except
   the approval wait below).

## Message format — bracket EVERY message so it's visibly yours
MCP posts your messages as the user (same name, same avatar), so in the thread your output and their
commands look identical. **Every single message you post — no exceptions — MUST use this envelope**,
so each one is unmistakably yours and its start/end are obvious:

```
🤖 *[LABEL]* ━━━━━━━━━━━━
<your content — one line or many, result, code block, etc.>
━━━━━━━━━━━━━━ ⌁ end
```

- The message **must start with `🤖`** (the header line) — marker detection depends on the leading
  `🤖`. Put the label in bold right after it, then the top rule (~12 `━`).
- Close with the bottom rule ending in `⌁ end`. The top+bottom rules bracket the whole message; the
  user's own replies have neither, so yours stand out at a glance.
- **This applies to short messages too** — acks, progress lines, and control-word confirmations
  (`paused`, `resumed`, `cwd → …`, `disconnected`, `skipped — no approval`, `[approve?] …`). A
  one-line body still gets the header rule and the `⌁ end` footer. Consistency is the whole point:
  if it starts with `🤖 *[LABEL]* ━━━`, the user knows instantly it's you.
- **Code/shell output** goes in a ```code block``` *between* the two rules (rules stay outside the
  fence). **File/image uploads:** the `initial_comment` caption uses the same envelope —
  `🤖 *[LABEL]* ━━━` / caption / `━━━ ⌁ end` — and the attached file appears with it.

## Run a command
- **Control words** (case-insensitive, whole message):
  - `stop`/`quit`/`shutdown`/`exit` → **ask before closing** (do NOT tear down immediately). Post the
    approval prompt in the envelope: `[approve?] close this session and stop polling? — reply yes/no.`
    — note its ts as `CLOSE_TS`. Then wait inline exactly like the approval gate (`Bash: sleep 12` →
    `slack_read_thread(oldest=CLOSE_TS)` for a non-🤖 reply with ts `> CLOSE_TS`, up to ~5 min / ~25
    loops):
    - `yes`/`y`/`approve`/`ok`/`do it` → NOW tear down: reply
      `🤖 [LABEL] disconnected — you can close this window.`, `CronDelete(cron_id)`, set
      `status:"closed"`, persist, END (process nothing after).
    - `no`/`cancel`/`n` or timeout → reply `🤖 [LABEL] still connected — polling continues.`, advance
      the cursor past the confirmation exchange, and keep polling (do NOT close).
  - `pause` → `paused=true`, `CronDelete(cron_id)`, reply `🤖 [LABEL] paused.` (reversible via
    `resume`, so no confirmation needed — unlike `stop`).
  - `resume` → recreate cron, `paused=false`, reply `🤖 [LABEL] resumed — polling every <interval>.`
  - `interval <DUR>` → `CronDelete` old + `CronCreate` new; store `cron_id`+`poll_interval`;
    reply `🤖 [LABEL] polling every <DUR>.`
- `cd <path>` → update `cwd` (persists), reply `🤖 [LABEL] cwd → <path>`.
- **Anything else** → **ALWAYS post an immediate acknowledgment FIRST — before you run a single tool
  call** — so the user instantly sees you received the request and started (no dead air). This ack is
  mandatory for every work-doing command, even ones you expect to be quick; the user is on their phone
  and needs the "on it" signal. Post (envelope): `🤖 *[LABEL]* ━━━` / `on it — <what you're about to
  do>…` / `⌁ end`. **Then** do the work with your full tools (Bash/Read/Edit/…) in `cwd`, and reply
  with the result in the envelope. Use a ```code block``` for shell output; truncate to ~3000 chars and
  say if truncated. For anything beyond a few seconds, also post progress + the ~2-min heartbeat — see
  below. (Only pure control words above — `pause`/`resume`/`cd`/`interval`/`stop` — skip the "on it"
  ack, since they get their own one-line confirmation.)

## Long-running work — acknowledge first, then keep the user posted
The user is on their phone and can't see the terminal — silence reads as "stuck". So for every
work-doing request: **acknowledge immediately** (the mandatory "on it" ack above, before any tool
call), then post progress as steps land, and — for anything running past ~2 min — send a status
heartbeat at least **every ~2 min** so the thread never goes quiet for long. A quick task gets its
ack + result; a slow one (org deploys, `sf apex run test`, Playwright/browser runs, installs, large
builds, multi-step work) gets ack → progress → heartbeats → result. Concretely:

1. **Acknowledge immediately, before starting**, so the reply isn't dead air. Each ack/progress
   post uses the envelope (see "Message format") — even though the body is one line:
   ```
   🤖 *[LABEL]* ━━━━━━━━━━━━
   on it — <what you're about to do>. Give me a minute…
   ━━━━━━━━━━━━━━ ⌁ end
   ```
2. **Post a short progress line as each meaningful step lands** (each in its own envelope), then the
   final result — e.g. `deployed ✓ — running the Apex tests now…` then `tests green (3/3) ✓`. Keep
   each to a line; don't narrate every micro-step. An ack + a final result is plenty for a simple
   job — reserve step-by-step updates for genuinely long, multi-stage tasks.
3. All of this happens **inside the one poll cycle** (the cycle runs to completion synchronously —
   that's fine). Every line you post starts with `🤖` and is *yours*.
4. **NEVER run a slow command as a blocking foreground call — always background it, then poll in a
   loop.** ⚠️ This is the single most important rule for staying responsive. **While any foreground
   tool call is running, this session is BLIND** — it cannot read the thread, so it cannot see your
   interrupts and cannot post a heartbeat until that call returns. A long blocking call is exactly the
   "keeps working, ignores me" failure. So: **if a command might take more than ~15–20 seconds**
   (org deploys, `sf apex run test`, Playwright/browser runs, installs, builds, big greps, anything
   whose duration you're unsure of), you MUST launch it with `Bash run_in_background` and drive it
   through the loop below — do NOT call it as a normal blocking Bash tool call.

   **The loop (run it the whole time the op is running):**
   `sleep 10` → `TaskOutput` (finished? grab the latest useful log line) **and** on **every** iteration
   `slack_read_thread(oldest=<cursor>)` (did the user send anything?). This one loop does three jobs:
   detect completion, **catch interrupts every ~10s**, and drive the heartbeat.
   - **Interrupts are checked EVERY iteration (~10s), not just at heartbeat time.** A **new non-🤖
     message** (ts > cursor) — in this loop, or between steps of a multi-step job — means the user is
     breaking in: **immediately** hand off to **Interrupting in-progress work** below (which
     `TaskStop`s the background op) before doing anything else. Do not finish the current op first.
   - **Heartbeat — status at least every ~2 min.** Count loops (~12 × 10s ≈ 2 min). Each time elapsed
     crosses a ~2-min mark **and the op is still running**, post a status line (envelope, one line) —
     `still on it — <what's running>, ~<N>m in… <latest log line if useful>` — then reset the timer
     so posts land ~every 2 min, not more often. Real step completions (`deploy ✓`, `tests green`)
     still post immediately; the heartbeat only fills the silent gaps. A sub-2-min task just gets its
     ack + result — no heartbeat.
   - **Multi-step foreground work** (a chain of individually-quick calls — reads, edits, small
     commands): between each step, do a quick `slack_read_thread(oldest=<cursor>)` so interrupts are
     still caught in the gaps. The blind-spot rule applies per call: keep each foreground call short,
     and background the moment any single step could run long.
5. **Cursor:** each `slack_send_message` returns a ts — track them and, at end of cycle, advance
   `last_reply_ts` to the numeric **max** ts across *all* your posts this cycle (ack + every progress
   line + every heartbeat + final + any file uploads), so none of them get re-read as a command
   next cycle.

## Interrupting in-progress work
The user is on their phone and may want to break in while you're mid-task — that's expected and
supported. You only see it because you poll the thread at each step (see "Long-running work" step 4);
there is no OS-level pre-emption. When a **new non-🤖 message** (ts > cursor) shows up while you're
working, stop and classify it:

- **Abort words** — `abort` / `cancel` / `hold` / `wait` / `nvm` / `stop that` → stop the current
  work (`TaskStop` any background op), reply `🤖 [LABEL] ⏹ stopped — <what was running>. What next?`,
  advance the cursor, and end the cycle. The session stays live and keeps polling.
- **Session-control word** — `stop` / `quit` / `shutdown` / `exit` / `pause` → stop the current work,
  then run the control-word flow (so `stop` still goes through the close-confirmation gate above).
- **Anything else = a new instruction** (interrupt-and-redirect) → stop the current work
  (`TaskStop` any background op), post
  `🤖 [LABEL] ⏸ interrupting — dropping <what was running>, switching to your new request…`, and add
  one line on what the abandoned work left behind (e.g. "deploy finished, tests not run"). Then
  **run the new instruction right away** — the user asked to break in, so don't add an "are you sure"
  gate for the switch itself. (Destructive-action approvals still apply to the new instruction.)

**Cursor:** the interrupting message plus every ack/abort/confirmation post has a ts — advance
`last_reply_ts` to the numeric max across all of them so none get re-read next cycle.

## Sharing a file or screenshot into the thread
When a reply is worth an image/file (a screenshot, a generated report, a log), upload it **into the
current thread** with the two-step Slack flow:
1. Exact byte size (portable — macOS BSD `stat` first, GNU/Windows-bash `stat` as fallback):
   `Bash: stat -f%z <path> 2>/dev/null || stat -c%s <path>`. Then
   `slack_get_file_upload_url(filename, content_length=<that size>, alt_txt=<short description>)`
   → returns a **File ID** and an **Upload URL**.
2. POST the bytes (assign the URL to a var so its signed token isn't mangled):
   `Bash: URL='<upload url>' && curl -sS -X POST "$URL" -H "Content-Type: image/png" --data-binary @<path> -w "\nHTTP %{http_code}\n"`
   — expect `HTTP 200`. Use the right Content-Type (`image/png`, `text/html`, …).
3. `slack_complete_file_upload(file_id, channel_id=CH, thread_ts=<thread_ts>,
   initial_comment="🤖 [LABEL] <caption>", title=<title>)`. The `initial_comment` is your message —
   it MUST start with `🤖`.
4. `complete_file_upload` does **not** return the new message ts, so **re-read the thread**
   (`slack_read_thread(oldest=<thread_ts>)`) and advance the cursor to the numeric max ts, so your
   own file post is never re-read as a user command.

## Approval gate — ASK IN SLACK, never in the terminal
Nobody is at this terminal, so **never** rely on a terminal prompt. Before any action that is
**destructive, irreversible, ambiguous, outward-facing, or outside `cwd`** — e.g. `rm` / delete /
truncate / overwrite an existing file's contents wholesale, `git push`/`--force`/reset --hard,
`sudo`, package installs, mass edits, deleting or mutating records/data, sending messages to
anyone else, network calls with side effects — you MUST get Slack approval first:

1. Post the approval prompt **in the envelope** (see "Message format"): header line, then
   `[approve?] <one-line plain description> — `\``<exact command>`\`` — reply yes/no.`, then the
   `⌁ end` footer. Note the ts this post returns — call it `APPROVE_TS`.
2. **Wait inline:** `Bash: sleep 12`, then `slack_read_thread(oldest=<APPROVE_TS>)` looking for a
   reply that is NOT yours (marker check as in the poll cycle) with ts numerically `> APPROVE_TS`.
   Repeat until such a reply arrives or ~5 min elapses (~25 loops).
3. Reply text `yes`/`y`/`approve`/`ok`/`do it` → proceed. `no`/`n`/`deny`/`cancel`/`stop` or
   timeout → do NOT act; reply `🤖 [LABEL] skipped — no approval.` Advance the cursor past the
   approval reply (and your own posts) either way.
4. If the reply is `yes always <verb>` (e.g. `yes always git`), append `<verb>` to `auto_approve`
   in the session file and proceed; future actions whose command starts with that verb skip the ask
   for the rest of this session. Read-only actions (Read/Grep/Glob/`ls`/`cat`/`git status`) never ask.

## Rules
- Never post a message that doesn't start with `🤖`; never act on one that starts with `🤖` /
  `:robot_face:` (that is your own output — MCP posts as the user, so the marker is the ONLY thing
  that tells your messages apart from theirs).
- No trigger keyword in a thread: every reply that isn't yours and is newer than the cursor is a
  command, however short.
- One invocation = one cycle (plus connect on the first). The approval wait and the long-running
  background loop are the only places the cycle loops internally.
- **A foreground tool call is a blind spot — you can't read Slack while one runs.** So NEVER run a
  command that might exceed ~15–20s as a blocking call: background it (`Bash run_in_background`) and
  drive the poll loop, checking the thread every ~10s (see "Long-running work" step 4). This is what
  makes interrupts and heartbeats actually work.
- During long work, poll the thread **every ~10s** so the user can interrupt at any time; a new
  message mid-task pre-empts the current work **immediately** — `TaskStop` the background op and
  handle it before continuing (see "Interrupting in-progress work"). Anything running past ~2 min
  gets a status heartbeat at least every ~2 min — never let the thread go silent for long.
- You can't close your own Terminal window; on `stop` you ask for approval first, then — only if
  approved — cancel your cron and tell the user.
