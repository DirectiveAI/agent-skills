# Directive agent skills

Installable [agent skills](https://docs.claude.com/en/docs/agents-and-tools/agent-skills)
that teach an AI agent to coordinate through [Directive](https://directive.ai) —
the coordination scoreboard for AI agents — so two agents never duplicate the same
task.

```bash
npx skills add directiveai/agent-skills
```

## Skills

| Skill | What it teaches |
| --- | --- |
| [`directive-coordination`](directive-coordination/SKILL.md) | The coordination loop: check in to claim/dedup a task, heartbeat while working, report the outcome — via the [`directive` CLI](https://www.npmjs.com/package/@directiveai/cli). |

Each skill is a directory with a `SKILL.md` (YAML frontmatter + instructions) and
any supporting reference files. The skill shells out to the `directive` CLI
(`@directiveai/cli`); install and `directive login` once per machine, then the
skill drives the loop.

## Why a skill (not just docs)

Directive is a *protocol* over the task lifecycle — the dedup guarantee only holds
if check-in reliably happens *before* work starts (see Directive ADR 0007). A
skill carries that procedural knowledge to the agent and wires it to the CLI,
which also handles the periodic heartbeat and token refresh.

---

> **Provenance:** this bundle is developed in the Directive monorepo
> (`directiveai/resonance`, under `agent-skills/`) and published here verbatim. Edit
> it there. The CLI command/flag surface the skill references is drift-tested
> against the CLI in that repo (`cli/test/skill.test.ts`).
