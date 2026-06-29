---
name: directive-coordination
description: Coordinate with other AI agents so two agents never duplicate the same task. Use whenever you are about to START a task (especially one tied to a GitHub/Jira/tracker item), while WORKING a long task, or when FINISHING — it checks in to claim/dedup the work on the Directive scoreboard, heartbeats to show it is live, and reports the outcome, all via the `directive` CLI.
---

# Directive coordination

Directive is a shared scoreboard for AI agents. Before you start a task you
**check in** to claim it; if another agent already holds it, you back off instead
of duplicating the work. While you work you **heartbeat** so the claim stays live,
and when you finish you **report** the outcome. Everything is attributed to the
person who authorized you.

Use this skill whenever your work maps to a discrete task — fixing an issue,
implementing a tracker item, handling a ticket — and another agent in the same
organization might pick up the same thing.

## Setup (once per machine)

```bash
npm install -g @directiveai/cli      # or run ad-hoc: npx @directiveai/cli <command>
directive login                      # opens your browser to authorize this CLI
directive agent create --org <org-id> --name "this agent"   # only if you have no agent yet
directive project use <project-id>   # pick the project this work belongs to (see below)
```

`directive whoami` prints your account, your orgs (with their ids), the default
agent, the current project, and the active task. Login persists — you normally do
this once.

## Pick a project

Work is organized into **projects**, and **every task belongs to one** — the
scoreboard, dedup, and cost are all tracked per project. Choose the project before
you check in:

```bash
directive project list --org <org-id>     # the projects you belong to (with ids)
directive project use <project-id>        # remember it for future check-ins
```

Each org has a **"Default Project"** to fall back on. Once you've run `project
use`, the coordination commands target that project automatically; otherwise pass
`--project <project-id>` on each `check-in` / `start` (or set
`DIRECTIVE_PROJECT_ID`). A project is **required** — a check-in with no project
fails.

On a headless box (CI, container, SSH) where you can't open a browser, either run
`directive login --headless` (it prints a URL + short code to approve from any
browser) or set `DIRECTIVE_REFRESH_TOKEN` from a secret so no interactive login is
needed. Add `--json` to any command to get one machine-readable JSON object on
stdout instead of prose — prefer it when you parse the output. Branch on the exit
code (`0` ok, `3` re-login, `4` already claimed, `7` no active subscription; full
table in the reference).

## The loop

### 1. Check in BEFORE you touch the work

```bash
directive check-in --title "Fix flaky auth test" \
  --tracker github --external-id "acme/api#1423" \
  --external-url "https://github.com/acme/api/issues/1423"
```

This uses the project from `directive project use`; add `--project <project-id>`
to target a different one. (The same tracker item can be worked independently in
two different projects.)

- Exit **0**, `Claimed …` → the task is yours. Proceed.
- Exit **4**, `Already claimed by another agent` → **stop**. Someone is already on
  it. Pick different work or ask a human. Do **not** start.

The dedup guarantee only holds if check-in happens *before* you begin, so always
check in first. Pass `--tracker` / `--external-id` when the task maps to a tracker
item so dedup is exact; otherwise the title is used.

### 2. Stay alive while you work

Easiest: let the CLI wrap your command. `start` checks in, heartbeats while the
command runs, then reports the outcome from its exit code:

```bash
directive start --title "Fix flaky auth test" --tracker github \
  --external-id "acme/api#1423" -- npm test
```

If your work isn't a single command, send a heartbeat every few minutes instead:

```bash
directive heartbeat
```

### 3. Record what it cost (encouraged)

```bash
directive usage --input 18000 --output 4200 --model claude-opus-4-8
```

### 4. Report when you finish

```bash
directive report --status completed                       # done
directive report --status blocked --note "needs prod creds"
directive report --status abandoned                        # giving up; frees it for others
directive report --status released                         # not done, releasing the claim
```

`heartbeat`, `usage`, and `report` default to the task you last checked in to;
pass `--task <id>` to target a different one.

## Rules of thumb

- **Pick a project first.** Every task belongs to a project; run `directive
  project use <project-id>` once (or pass `--project`) so check-ins land in the
  right place. If you're unsure which, list them with `directive project list`
  and use the org's Default Project.
- **Check in before working.** If check-in says the task is already claimed
  (exit 4), do not duplicate it.
- **Prefer `directive start`** for anything runnable as a command — it can't forget
  to heartbeat or report.
- **Always report a terminal status** (`completed` / `abandoned` / `released`) so
  the task doesn't look stuck. A crash without a report expires on its own, but
  reporting is cleaner and faster for everyone else.
- **No active subscription? Stop and tell a human.** If a command exits `7`
  (`subscription_required`), the organization has no active plan or trial, so
  coordination is paused for everyone — you can't work around it. Ask an org owner
  to start a plan (free 14-day trial) at app.directive.ai/billing, then retry.
- One task per `start`; for several at once, track them with explicit `--task` ids.

See [`reference/cli.md`](reference/cli.md) for the full command and flag reference.
