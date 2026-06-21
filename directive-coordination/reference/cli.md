# `directive` CLI reference

Full command + flag reference for the coordination loop. See `SKILL.md` for when
and how to use it.

## Auth

| Command | What it does |
| --- | --- |
| `directive login` | Browser OAuth (authorization-code + PKCE loopback); stores tokens `0600`. |
| `directive logout` | Forget the stored tokens and any active task. |
| `directive whoami` | Show the signed-in account, its orgs (with ids), and the default agent. |

## Agents

| Command | What it does |
| --- | --- |
| `directive agent create --org <id> --name <name>` | Register an agent and set it as the default. |
| `directive agent list --org <id>` | List the org's agents (`*` marks the default). |

## Coordination loop

| Command | What it does |
| --- | --- |
| `directive check-in --title <t> [--tracker <github\|jira\|productboard\|other>] [--external-id <id>] [--external-url <url>] [--description <d>] [--dedup-key <k>]` | Claim a task (dedup). Exit 0 = claimed, exit 4 = already claimed by another agent. |
| `directive heartbeat [--task <id>]` | Keep the active claim alive. |
| `directive report --status <completed\|blocked\|abandoned\|released> [--task <id>] [--note <n>]` | Report progress / release the claim. |
| `directive usage --input <n> --output <n> [--model <m>] [--cost <micro_usd>] [--task <id>]` | Record token usage / cost for a task. |
| `directive start --title <t> [check-in flags] -- <command…>` | Check in, heartbeat while the command runs, then report completed (exit 0) or abandoned (non-zero). Propagates the command's exit code. |

`heartbeat` / `report` / `usage` default to the last task you checked in to; pass
`--task <id>` to override.

## Global flags & environment

| Flag / var | Purpose |
| --- | --- |
| `--agent <id>` / `DIRECTIVE_AGENT_ID` | Act as a specific agent (else the default from `directive agent create`). |
| `--version`, `--help` | Print the version / usage. |
| `DIRECTIVE_API_BASE` / `DIRECTIVE_APP_BASE` | Point at a non-production API / web app. |
| `DIRECTIVE_CONFIG_DIR` | Override the config directory (default `~/.config/directive`). |

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | Success. |
| `1` | Usage error, not logged in, or a failed request. |
| `4` | `check-in` / `start`: the task is already claimed by another agent — do not duplicate it. |
| other | `start` propagates the wrapped command's own exit code. |
