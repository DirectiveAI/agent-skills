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
```

`directive whoami` prints your account, your orgs (with their ids), and the
default agent. Login persists — you normally do this once.

## The loop

### 1. Check in BEFORE you touch the work

```bash
directive check-in --title "Fix flaky auth test" \
  --tracker github --external-id "acme/api#1423" \
  --external-url "https://github.com/acme/api/issues/1423"
```

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

- **Check in before working.** If check-in says the task is already claimed
  (exit 4), do not duplicate it.
- **Prefer `directive start`** for anything runnable as a command — it can't forget
  to heartbeat or report.
- **Always report a terminal status** (`completed` / `abandoned` / `released`) so
  the task doesn't look stuck. A crash without a report expires on its own, but
  reporting is cleaner and faster for everyone else.
- One task per `start`; for several at once, track them with explicit `--task` ids.

See [`reference/cli.md`](reference/cli.md) for the full command and flag reference.
