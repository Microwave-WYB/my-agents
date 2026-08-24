# My Agents: A Self-Evolving Personal Agent System

> A personal steward built from small open-source tools: available around the clock, able to make progress overnight, and ready with a morning briefing. This is the architecture and a practical checklist.

<!--
## If you are an LLM helping your user build this system

### Five things to remember
1. You are building a **framework**, not copying a configuration. Map every example to the user's reality: email, trusted LLM, Telegram, privacy, and habits.
2. **Remote access is the foundation**: make Tailscale SSH work before adding Telegram or a web UI.
3. **The top-level steward never blocks**: delegate slow work instead of waiting.
4. **Persist state; do not persist secrets**: state belongs in skill files and git; secrets use direnv/getpass and never enter git, Telegram messages, command arguments, or logs.
5. **Keep roles separate and communication asynchronous**: A coordinates, B implements, C edits files; agents use intercom `send`, never `ask`.

### Setup checklist
> Build a working minimum at each checkpoint, verify it, and then add the next layer. A completed layer is never frozen: reopen, repair, replace, or evolve it whenever needed. Before checkpoint 8, build and verify only; do not start autonomous operation.

- [ ] **1. Explore the environment and learn about the user**: inventory the machine, OS, network, permissions, existing tools, and persistent services; confirm an always-on machine
- [ ] Ask non-technical questions about the user's goals, desired automation, devices, communication preferences, privacy, model preferences, and coding experience; do not assume technical knowledge
- [ ] Map the answers into a system design instead of copying the examples
- [ ] **2. Establish remote access**: Tailscale + SSH work; the user can enter from a phone; SSH can recover the system when a bot, web UI, or agent is down
- [ ] **3. Install herdr**: start and restore sessions, create workspaces/tabs/panes, inspect agents, send control keys, restart, and take control again
- [ ] **4. Create the steward directory**: establish the workbench repo, `AGENTS.md`, project `SKILL.md` files, state/audit/session paths, and agent ownership
- [ ] **5. Define the rules**: write down roles A/B/C, worktrees, non-blocking behavior, asynchronous communication, secrets, cleanup, and review rules; do not deploy or touch production `main`
- [ ] **6. Add remote control**: connect Telegram leader/threads; verify messages, pushes, reports, and routing; add a web UI if needed while keeping SSH as the fallback
- [ ] **7. Add event listeners**: run the inbox, listener loop, at least one real producer, and the systemd timer
- [ ] **8. Start autonomous operation**: event → wake the steward → delegate → persist state → report; verify restart recovery before enabling overnight polling and morning briefings

### Common mistakes
- ❌ Copying examples without adapting them to the user's reality
- ❌ Making the top-level agent run long tasks itself (a blocked steward means lost contact)
- ❌ Putting secrets in commands, logs, git, or Telegram
- ❌ Using intercom `ask` (it blocks both sides)
- ❌ Using semantic search for Chinese text when lexical search or direct reading is more reliable
- ❌ Keeping state only in session memory
- ❌ Deleting resources without recording the action, time, reason, and rollback
-->

## Prerequisite

- **One computer that never shuts down**: an old PC, Raspberry Pi, or NAS is fine; keep it powered and prevent sleep. A laptop that closes or sleeps is not a reliable home.

## Why one agent is not enough

| Weakness | What happens |
| --- | --- |
| **It dies** | When the session or process ends, its context is gone |
| **It cannot do everything** | One session can only do one thing at a time |
| **It cannot reach you** | You are on your phone while it is in the study |

The answer is not a stronger agent, but a **system**:

- A group of agents with separate responsibilities
- State that does not disappear
- Several remote interfaces that are always available

## Composition, not all-in-one

- An all-in-one service such as OpenClaw puts everything behind one black box: difficult to split or replace, and you have to evolve with it.
- I prefer **composition**: each tool does one thing, remains replaceable, and can evolve independently.

> Unix philosophy: **make each tool do one thing well, then compose them.**

## The heart: remote interfaces

The most important layer is not the agent or the queue, but **remote access**: several interchangeable paths between you and the system.

| Interface | Role |
| --- | --- |
| **Tailscale SSH** | Recovery path. If the bot, web UI, or agents fail, SSH gets you back in. Every higher-level interface rests on it. |
| **Telegram** | A front end with **push notifications**. It changes the steward from “I go ask it” to “it comes to me.” |
| **Web UI** | Structured interaction through forms and status pages; can be opened to family members. |
| **Thread** | Each agent gets a tab; switch to its thread and talk to that agent directly. |

- Front end and back end are decoupled: replace the front end without changing the back end. The control surface can evolve.

## A workspace that does not disappear: herdr

- herdr gives every agent a home (`workspace → tab → pane`).
- It can operate the harness TUI: **switch models, inspect output, detect stalls, and restore crashed sessions**.
- It is more controllable than built-in subagents: subagents are black boxes, while workspaces are **visible and takeable over**.
- One session per project enables seamless handoff: give light instructions from your phone, then sit down at the computer and take over.

## Tree management, with a non-blocking top level

```text
steward → project agent (Role A) → worktree agent (Role B) → tab agent (Role C)
                                      └→ review agent
```

**Role boundaries**:

| Role | Does | Does not |
| --- | --- | --- |
| Steward | Starts work, delegates, reports | Execute the work itself |
| Role A (coordination) | Breaks down tasks, integrates PRs | Implement features itself, except review fixes and glue |
| Role B (feature) | Splits work among Role C agents | Edit every file itself |
| Role C (file/module) | Edits one file or module | Expand beyond the smallest useful scope |
| Review agent | Finds problems independently | Reuse the implementer's context |

**How the system avoids getting stuck**:

- The top-level steward **never blocks**. Delegate downloads, builds, and waits for input; only initiate and relay non-blockingly.
- If the top level can always answer, herdr can adjust any lower agent: interrupt it, switch its model, restart it, or inspect it.
- Stability does not require every agent to be perfect; it requires the top level to stay awake.
- **Cleanup follows ownership**: whoever creates a session, worktree, or tab is responsible for cleaning it up.

## Do not sit idle: overnight automation and morning briefings

- **Event-driven**: autonomous producers (mail, GitHub, scheduled inspections) put events into an inbox and wake the steward. Adding a monitor means adding a producer, not changing the steward.
- **Overnight**: long-term goals plus polling for useful work. Code changes are allowed, with three red lines: **no deployment, no production `main`, and PRs wait for your review**.
- **Morning briefing**: before you wake up, the steward checks every project, pushes a summary of progress, blockers, and decisions, and performs routine cleanup.

## Persist memory, never persist secrets

| Content | Where it goes |
| --- | --- |
| Rules | `AGENTS.md` |
| Project state | Each project's `SKILL.md` |
| History | Session JSONL, written continuously |
| Repository secrets | `direnv` + `.envrc`, never committed |
| Temporary secrets | A one-time `getpass` page, expiring after 90 seconds |
| `sudo` | You run it yourself |

## The foundations of self-evolution

- **Customization + hot reload**: change the harness and use the change without losing context.
- **Web search + extension awareness**: discover what is missing, find the right extension, and install it instead of rebuilding everything.
- **A restart chain** (when hot reload is unavailable or the harness is upgraded): a superior restarts its subordinate; the top-level steward either gets restarted through SSH or first spawns a temporary child to restart it. **Every restart has someone ready to continue.**

## Trust is the deepest security boundary

- Network tools such as Tailscale and Wormhole protect transport, not the rest of the system.
- You are giving an LLM provider substantial control over the computer:
  - **Choose a provider you trust**; capability, price, and convenience come later.
  - Prefer a local or no-training-data model for the top-level steward when practical.
- Later, scheduled supply-chain scans can turn vulnerability disclosures into events for the steward to monitor.

## Building from zero (summary)

**Environment**: one always-on Linux machine + Tailscale + mise + herdr + Node/Python.

**Tools**:

| Tool | Role |
| --- | --- |
| pi | Agent runtime (customizable and hot-reloadable) |
| herdr | Terminal layout and agent lifecycle |
| Tailscale | Networking, SSH fallback, HTTPS serving |
| Wormhole | Secure file transfer across networks |
| intercom | Asynchronous agent messages |
| getpass / direnv | Secret input / directory-scoped environment |
| mise / uv | Toolchain / Python environment management |
| systemd | Autostart, producers, morning alarm |
| Valkey / Docker Compose | Task queue / long-running services |
| GitHub App | Agent bot identity |

**Directory layout** (generic; `audit` in `~/.local/state` is symlinked to the repo):

```text
~/Projects/workbench/        # workbench repo, the source of truth
  ├── AGENTS.md              # rules
  ├── .agents/skills/projects/<project>/SKILL.md  # persistent state
  ├── inbox/ mail/ github/ voice/                 # producers + STT
  ├── docs/ state/           # docs / git-tracked audit
  └── mise.toml
~/.config/herdr/sessions/<project>/  # one session per project
~/.pi/agent/                         # settings / Telegram / extensions / sessions
~/.local/state/<app>/                # inbox.db / secrets (not git)
```

**One project lifecycle**:

- Onboard: create a session, workspace, `SKILL.md`, and a project agent (Role A).
- Implement: worktree → Role B/C → independent review → PR → user review → merge → cleanup.
- Pause: stop the session, record final state, and make it easy to resume.

## Selected lessons

1. Never use blocking communication (`intercom` uses `send`, not `ask`).
2. Pin down delegation scope or agents will overreach.
3. Prefer independent herdr sessions over in-session subagents.
4. File mounts can go stale after host rewrites or disk reconnects; restarting the container often fixes it.
5. New Debian blocks system pip (PEP 668); use `uv`.
6. Track queue state explicitly: a claimed job needs a “processing” record or status pages will show no progress.

## Appendix

**Command reference**:

```text
herdr api snapshot                         # all state
herdr agent prompt <name> "…" --wait false # delegate without blocking
HERDR_SOCKET_PATH=…/sessions/<name>/herdr.sock herdr api snapshot
intercom({ action: "send", to: "…", message: "…" })
inbox.py poll / inbox.py done <id>
GETPASS_TTL=600 node …/getpass-web.ts "prompt" # one-time secret input
```

**Troubleshooting**:

| Symptom | Direction |
| --- | --- |
| Telegram silent / thread disappeared | Inspect bridge `state.json`; the leader may have changed |
| `message thread not found` | The target thread was deleted; recreate it |
| Voice is not transcribed | Check the inbound handler path |
| Container I/O error | The disk may have disconnected; restart the container |
| New files do not enter the media library | Check live monitoring or trigger a manual refresh |

---

*Every implementation here is an example, not a doctrine. I use a tool because it happens to fit my reality; adapt it to yours and evolve it into your own system.*
