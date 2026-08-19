# 🛰️ Slackline

### Message your laptop. It gets to work.

Slackline lets you run [Claude Code](https://claude.com/claude-code) on your own computer and drive it
from a **Slack message to yourself** — from your phone, from a meeting, from the train.

- 🚀 Kick off a deploy before you leave your desk, then check on it from the coffee queue.
- 🧪 Ask *"what's failing in the tests?"* on your commute and land at the office with it already fixed.

Your agent keeps working; **you just steer it by text.**

> **No bot to install. No token to manage. No server to run.**
> It's just Claude Code on your machine, talking to your own Slack DM.

Your whole conversation lives in that DM — you send commands, and Claude's **🤖** replies come back into
the same thread. The work happens on your laptop.

```
       📱 Your phone — the Slack DM                    🖥️  Your laptop — Claude Code
   ┌────────────────────────────────────┐          (already running in your project)
   │  you ▸ run the test suite          │ ───────▶  picks it up, runs the suite
   │  🤖 on it — running tests…         │ ◀───────  …with your tools & logins
   │  🤖 still on it, ~2m in… 34/50     │ ◀───────  posts progress as it goes
   │  🤖 [approve?] git push — yes/no   │ ◀───────  pauses at the risky step
   │  you ▸ yes                         │ ───────▶  pushes
   │  🤖 done ✓ 50/50 green, pushed     │ ◀───────
   └────────────────────────────────────┘
```

**Another day, from a meeting — turn a doc into a deck and get it in your Drive:**

```
   ┌──────────────────────────────────────────────────┐
   │  you ▸ build a 5-slide deck on Project Aurora     │ ───────▶  reads the project docs,
   │        from the project docs, drop it in my Drive │           drafts + builds the deck
   │  🤖 on it — drafting 5 slides…                    │ ◀───────
   │  🤖 [approve?] write Aurora-Deck.pptx to ~/Drive  │ ◀───────  waits for your ok
   │  you ▸ yes                                        │ ───────▶  writes it → Drive syncs
   │  🤖 done ✓ it's in your Drive → drive.google.com/…│ ◀───────  link back in the thread
   └──────────────────────────────────────────────────┘
```

<br>

---

## Contents

> ⚡ **In a hurry? Jump to [🚀 Quick start](#-quick-start)** — install, one setup rule, connect.

- [✨ What you get](#-what-you-get)
- [🧠 Why it's built this way](#-why-its-built-this-way)
- [💡 When you'd reach for it](#-when-youd-reach-for-it)
- [❗ Read this first — the one setup rule](#-read-this-first--the-one-rule-that-makes-it-work)
- [🚀 Quick start](#-quick-start)
- [💬 Driving it from Slack](#-driving-it-from-slack)
- [⚙️ How it works](#️-how-it-works-under-the-hood)
- [🖥️ Platform support](#️-platform-support)
- [📦 Installing as a plugin](#-installing-as-a-plugin)
- [⚠️ Good to know](#️-good-to-know)

---

## ✨ What you get

- **Your agent works while you're away from the desk.** Long deploys, test runs, refactors, data
  loads — start it, walk off, steer and approve from Slack. Work doesn't stall because you stepped out.
- **Everything runs on your machine, as you.** Your logins, your files, your `sf` CLI, your browser
  sessions — Claude uses them the way you would. Nothing to re-authenticate, no integrations to wire.
- **Zero infrastructure.** No hosted bot, no server, no webhook, no token to provision or rotate. Your
  laptop simply checks your Slack DM on a schedule. Nothing to stand up, nothing to attack.
- **Full results come back to you.** Answers, code, screenshots, generated decks and reports — posted
  straight into the thread.
- **You stay in control.** Before anything risky (deleting, pushing, installing, touching data), it
  asks you in Slack and waits for a `yes`.

<br>

## 🧠 Why it's built this way

Most "control it remotely" tools mean standing up machinery — SSH, a hosted bot, a service account,
tokens to rotate. Slackline skips all of that with one idea: **your laptop already can do the work and
already has your access — so let it poll your own Slack DM and act on what you send.**

| | Slackline | The usual remote setup |
|---|---|---|
| **Hosting** | Nothing — runs on your laptop | A server / cloud bot to run and maintain |
| **Identity** | Runs *as you*, with your logins | A service account + "who owns the bot?" |
| **Secrets** | No token to paste or leak | Tokens to provision and rotate |
| **Attack surface** | Laptop polls *out*; nothing listens | An inbound endpoint someone can reach |
| **App access** | Uses the apps you already use | An API/connector per integration |

Because it's *your* computer, Claude works the way you do — for example:

- 🌐 **Browser** — opens a site, navigates, fills in details (via Playwright, using your logged-in state)
- 📁 **Google Drive** — just drops the file in your **local Drive folder** and it syncs. No Drive API, no connector, no admin approval
- ☁️ **Salesforce** — your `sf` CLI and browser are already authed to your orgs; no separate creds

> ⚠️ **These are examples of what's possible, not switches the plugin flips on.** Slackline doesn't
> bundle a browser, a Drive integration, or Salesforce creds. Claude decides *which* tools to use — and
> whether to use them at all — based on **your project's instructions** (e.g. a `CLAUDE.md`) and **what
> you ask it from Slack**, drawing on whatever is already set up on your machine. You point it at what
> you want; it plans from there.

**Slack becomes a remote control for Claude Code** — you steer the agent from your pocket, and it works
your machine and apps the way you would.

<br>

## 💡 When you'd reach for it

> *Your laptop stays on with the terminal open — you're away from the **desk**, not the machine.*

- 🚶 **Step out, work keeps going** — start a deploy or test suite, then leave for an errand; watch
  progress, answer the one approval, grab the result from Slack.
- 🚆 **Commute triage** — *"what's failing?"* → *"fix the null check, redeploy, re-run."* Arrive green.
- 🧑‍💼 **Caught in a meeting** — *"generate a one-pager from `architecture.md` and drop it in my Drive."*
  It's in the thread before the meeting ends.
- 🌙 **After-hours / on-call** — a job breaks at night: inspect logs, run a query, restart it — from
  your phone, without opening the laptop.
- 🔀 **Two things at once** — desk terminal deep in Project A; a second connected thread runs a
  Project-B report in parallel.
- ⏳ **Long job, zero babysitting** — walk away from a slow build; the heartbeat keeps you posted, and
  you can interrupt to redirect anytime.
- 🔎 **Pocket lookups** — *"OWD on Account in staging?"*, *"summarize today's git log"*, *"grep for TODOs."*

<br>

## ❗ Read this first — the one rule that makes it work

**Always launch the session that you'll drive from Slack with
`--permission-mode bypassPermissions`.** This isn't optional — without it the plugin can't do its job.
Here's why:

- Claude Code's normal *"allow this action? (yes/no)"* prompts appear **in the terminal**, and they are
  **never sent to Slack**. There is no setting that forwards them to your phone.
- So if you walk away from a normal session and Claude needs permission, that terminal prompt just
  **sits there unanswered and the whole session hangs** — your Slack command never completes.
- Launching with `--permission-mode bypassPermissions` removes those terminal prompts entirely.
  Slackline then becomes your safety net: **it** asks you **in Slack** before anything destructive
  (delete, push, install, touch data) and waits for your `yes`.

Start the session with permissions bypassed, in the project you want to drive:

```bash
cd ~/your-project
claude --permission-mode bypassPermissions
```

> **Note:** `--permission-mode bypassPermissions` and the older `--dangerously-skip-permissions` flag
> do the same thing (both bypass all terminal permission checks) — use whichever you prefer.
> The *softer* modes (`--permission-mode acceptEdits`, or pre-allowed tools) still stop to prompt in
> the terminal for *some* actions, so they can still hang while you're away — bypass is the reliable
> choice for truly unattended, drive-from-Slack use.

### Don't want to type the flag every time? Make bypass the default

If you launch Claude Code from **VS Code** or **Cursor**, or you just don't want to pass the flag on
every run, set bypass as the default once. The setting lives in a *different place per client*:

**Terminal CLI, JetBrains, and Cursor's CLI** — add this to `~/.claude/settings.json` (applies globally)
or a project's `.claude/settings.json`:

```json
{
  "permissions": {
    "defaultMode": "bypassPermissions"
  }
}
```

**VS Code extension** — it uses its **own** setting, not `.claude/settings.json`. In VS Code **Settings**
set:

```json
"claudeCode.initialPermissionMode": "bypassPermissions"
```

You may first need to turn on the extension's **"Allow dangerously skip permissions"** toggle for the
bypass option to become available.

**Cursor** — it runs the CLI under the hood, so the `~/.claude/settings.json` `defaultMode` above applies.
If Cursor's Claude Code panel exposes its own initial-permission-mode setting, that one takes precedence.

> ⚠️ Bypass is for a machine you trust. Claude Code **refuses to start in bypass when running as
> `root`/`sudo`**, and the **first** bypass session shows a one-time "accept responsibility" prompt you
> must accept. Slackline remains your guardrail either way — it asks you **in Slack** before anything
> destructive.

<br>

## 🚀 Quick start

**One-time setup:** the prerequisite + step 1. **Every time** you want to drive a project: step 2.

---

### ✅ Prerequisite — connect the Slack MCP

Slackline talks to Slack **through the Slack MCP**, so it must be connected in Claude Code **before you
install the plugin**.

Run `/mcp`:
- **slack** already connected? → skip to step 1.
- Not there? → connect it, then continue.

> 💡 The steps to add the Slack MCP **differ per person/org** (corporate vs. personal Slack; some orgs
> need their own OAuth client id / redirect), so follow your organization's process. Once `/mcp` shows
> **slack** connected, come back here.

---

### 1️⃣ Install the plugin — *one time*

Inside a running Claude Code session (at the **`claude` prompt**, not your shell), run these **one at a
time** — marketplace first, then install:

```
/plugin marketplace add ayushs1204/slackline
```
```
/plugin install slackline@slackline
```

At the install prompt, choose **Install for you**.

<details>
<summary>No <code>/plugin</code> command in your Claude Code version? Load it directly instead.</summary>

```bash
git clone https://github.com/ayushs1204/slackline.git
claude --permission-mode bypassPermissions --plugin-dir ./slackline
```
</details>

---

### 2️⃣ Start a fresh session & connect — *every time*

In your **shell/terminal**, open a **new** session in the project you want to drive, launched in
**bypass mode** (this session stays running and answers Slack):

**a)** go to the project you want to drive:

```bash
cd ~/your-project
```

**b)** launch a fresh Claude Code session in bypass mode:

```bash
claude --permission-mode bypassPermissions
```

Then, inside that new session (at the `claude` prompt), connect it to your Slack DM — either plain
English or the command works (the command defaults the label to the folder name and polls every 1m):

```
connect to slack
```
```
/slack-connect
```

It posts **🤖 Slackline connected** to your Slack DM — reply in that thread from anywhere.

> **Keep the terminal open** — it only works while this session is running.

<br>

## 💬 Driving it from Slack

Just reply in the thread — any message that isn't its own is a command. No trigger word needed.

| You send | What happens |
|---|---|
| `run the tests` · `what's failing?` · *any instruction* | Runs on your machine and replies with the result |
| `yes` / `no` | Approve or skip the action it's waiting on |
| `yes always git` | Stop asking for `git …` for the rest of this session |
| `cancel` · `hold` · `wait` · `abort` | Stop what's running now, stay connected |
| *a new instruction while it's busy* | Drops the current task and switches to the new one |
| `cd <path>` | Change the working directory |
| `pause` / `resume` | Stop / restart checking the thread (keeps the connection) |
| `interval 5m` | Check the thread more or less often |
| `stop` / `quit` | Asks you to confirm, then disconnects |

Every message it posts starts with **🤖** and is wrapped in a clear header/footer, so its replies never
look like your own messages.

<br>

## ⚙️ How it works (under the hood)

1. The session posts an anchor message to your Slack DM and remembers that thread.
2. On a schedule (default every minute) it reads the thread for anything new from you.
3. New message → it runs the work in your project folder with full tools, posting progress as it goes.
4. **Long tasks stay chatty** — it acknowledges up front, updates you per step, and sends a status
   **heartbeat at least every ~2 minutes** so the thread never goes silent.
5. **You can interrupt** — send `cancel` to stop, or a new instruction to redirect it mid-task.
6. **Risky actions ask first** — it posts `[approve?] … — yes/no` and waits for your reply before acting.

State lives in `~/.claude/slack-bridge/`. Open several terminals and `/slack-connect` each for
independent parallel threads.

<br>

## 🖥️ Platform support

Works wherever Claude Code runs — **macOS, Linux, and Windows** (native or WSL). Slackline's moving
parts (the scheduler, the Slack MCP, the shell commands) are platform-neutral. On Windows, run Claude
Code in a bash-capable shell (WSL or Git Bash) for the handful of shell helpers the skill uses.

<br>

## 📦 Installing as a plugin

Slackline is a **Claude Code plugin** — it bundles the connect-and-poll skill and the `/slack-*`
commands. The one thing it *doesn't* bundle is the Slack MCP itself (its setup varies by organization),
so connecting that MCP is a **prerequisite** you satisfy once *before* installing — see
[Quick start](#-quick-start).

The repository is also its own **plugin marketplace** (see
[`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json)), so anyone can install it from
inside Claude Code — run the two commands **one at a time** (see [Quick start](#-quick-start) above),
and pick **Install for you** at the install prompt:

```
/plugin marketplace add ayushs1204/slackline
```
```
/plugin install slackline@slackline
```

<details>
<summary><b>How to publish this to a marketplace (for maintainers)</b></summary>

A "marketplace" in Claude Code is just a git repo containing a `.claude-plugin/marketplace.json` that
lists one or more plugins. Slackline already has one, so:

- **This repo *is* the marketplace.** It's public — just share the two commands above. Teammates
  point `/plugin marketplace add` at `ayushs1204/slackline` and install `slackline@slackline`. Ship
  updates by pushing new commits / tags; users update with `/plugin marketplace update`.
- **To list it in a *shared/community* marketplace instead**, add an entry for this plugin to that
  marketplace repo's `marketplace.json` (name + source pointing at this repo), and open a PR to it.
  Nothing about this repo needs to change — a marketplace can reference plugins hosted anywhere.
- **For a team**, you can also pre-load it via Claude Code settings (`extraKnownMarketplaces` +
  `enabledPlugins`) so it's available to everyone without each person running the install commands.

</details>

<br>

## ⚠️ Good to know

- **The terminal has to stay open** — the connection lives with the running session and expires after
  7 days of the scheduler, so it isn't a true always-on daemon (yet).
- **Response time ≈ one poll interval** (as fast as ~1 minute). Approvals and interrupts are handled
  within the same cycle, not an extra round-trip.
- **Approvals are the agent's judgment, not a hard lock.** The guardrail is the skill choosing to ask
  before risky actions. Blast radius is your own machine and your own DM.

<br>

## 📄 License

MIT © Ayush Shukla
