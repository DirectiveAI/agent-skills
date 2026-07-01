# `directive` CLI reference

Full command + flag reference for the coordination loop. See `SKILL.md` for when
and how to use it.

## Auth

| Command | What it does |
| --- | --- |
| `directive login` | Browser OAuth (authorization-code + PKCE loopback); stores tokens `0600`. |
| `directive login --headless` | Device-code flow (RFC 8628) for CI / no-browser hosts: prints a URL + short code to approve from any browser, then polls. |
| `directive logout` | Forget the stored tokens and any active task. |
| `directive whoami` | Show the signed-in account, its orgs (with ids), the default agent, and the current selections (org + project) and active task. |

**Headless / CI without an interactive login:** set `DIRECTIVE_REFRESH_TOKEN`
(recommended — the CLI mints a fresh access token per run) or `DIRECTIVE_TOKEN`
(a short-lived access token) from a secret store. Env credentials take precedence
over the on-disk token file and are never written to disk.

## Organizations

| Command | What it does |
| --- | --- |
| `directive org list` | List the orgs you belong to (`*` marks the current one). |
| `directive org use <id>` | Set the current org for the org-scoped commands below. |

The org-scoped commands (`agent`, `project`) default to the current org: `--org
<id>`, else `DIRECTIVE_ORG_ID`, else `directive org use`.

## Agents

| Command | What it does |
| --- | --- |
| `directive agent create [--org <id>] --name <name>` | Register an agent and set it as the default. |
| `directive agent list [--org <id>]` | List the org's agents (`*` marks the default). |

## Projects

Work is organized into projects, and every task belongs to one. Pick a project
before checking in.

| Command | What it does |
| --- | --- |
| `directive project list [--org <id>]` | List the projects you belong to (`*` marks the current one). |
| `directive project create [--org <id>] --name <name> [--slug <s>] [--description <d>]` | Create a project and set it as the current one. |
| `directive project use <id>` | Set the current project for future check-ins. |

## Coordination loop

| Command | What it does |
| --- | --- |
| `directive check-in --title <t> [--project <id>] [--tracker <github\|jira\|productboard\|other>] [--external-id <id>] [--external-url <url>] [--description <d>] [--dedup-key <k>]` | Claim a task (dedup) in a project. Exit 0 = claimed, exit 4 = already claimed by another agent. |
| `directive heartbeat [--task <id>]` | Keep the active claim alive. |
| `directive report --status <completed\|blocked\|abandoned\|released> [--task <id>] [--note <n>]` | Report progress / release the claim. |
| `directive usage --input <n> --output <n> [--model <m>] [--cost <micro_usd>] [--task <id>]` | Record token usage / cost for a task. |
| `directive start --title <t> [--project <id>] [check-in flags] -- <command…>` | Check in, heartbeat while the command runs, then report completed (exit 0) or abandoned (non-zero). Propagates the command's exit code. |

A project is **required** for `check-in` / `start`: it comes from `--project
<id>`, else `DIRECTIVE_PROJECT_ID`, else `directive project use`. `heartbeat` /
`report` / `usage` default to the last task you checked in to; pass `--task <id>`
to override.

## Global flags & environment

| Flag / var | Purpose |
| --- | --- |
| `--agent <id>` / `DIRECTIVE_AGENT_ID` | Act as a specific agent (else the default from `directive agent create`). |
| `--org <id>` / `DIRECTIVE_ORG_ID` | Target a specific org for org-scoped commands (else the current one from `directive org use`). |
| `--project <id>` / `DIRECTIVE_PROJECT_ID` | Check in to a specific project (else the current one from `directive project use`). |
| `--json` | Emit one machine-readable JSON object on stdout (the result, or `{ "error": code, "message": … }`). Human/progress lines go to stderr; the exit code still signals success. Prefer this when parsing output. |
| `--version`, `--help` | Print the version / usage. |
| `DIRECTIVE_TOKEN` / `DIRECTIVE_REFRESH_TOKEN` | Headless credentials (see Auth). Take precedence over the on-disk token file. |
| `DIRECTIVE_API_BASE` / `DIRECTIVE_APP_BASE` | Point at a non-production API / web app. |
| `DIRECTIVE_CONFIG_DIR` | Override the config directory (default `~/.config/directive`). |

## Exit codes

Stable contract, so scripts and agents can branch without parsing prose.

| Code | Meaning |
| --- | --- |
| `0` | Success. |
| `1` | Unexpected / runtime error (network, unknown server error). |
| `2` | Usage error (missing or invalid flags, unknown command). |
| `3` | Auth required (not logged in, token invalid/expired, account not provisioned) — run `directive login`. |
| `4` | `check-in` / `start`: already claimed by another agent — do **not** duplicate it. |
| `5` | Not found (task, agent, or active claim). |
| `6` | Plan limit reached (a meter cap on an active plan) — raise it at app.directive.ai/billing. |
| `7` | No active subscription — the org has no plan or trial started, so coordination is paused. An org owner must start one (free 14-day trial) at app.directive.ai/billing; the agent can't proceed. |
| other | `start` / `run` propagate the wrapped command's own exit code (and `127` if it isn't executable). |
