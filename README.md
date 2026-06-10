# xufeisofly's Skills

A small collection of agent skills (slash commands and behaviors) for Claude Code,
Codex, and other coding agents. Install them into any repo with one command.

## Quickstart

1. Run the skills.sh installer:

   ```bash
   npx skills@latest add xufeisofly/skills
   ```

2. Pick the skills you want, and which coding agents you want to install them on.
   **Make sure you select `/setup-xufeisofly-skills`.**

3. Run `/setup-xufeisofly-skills` in your agent. It will:
   - Ask where this repo should store RPT state files (`overview.md`, `research.md`, `plan.md`, `tasks/`)
   - Ask which build / test / lint commands count as "passing" (used by `/rpt-workflow`'s verification)
   - Ask which language to write RPT documents in (default English)

4. You're ready to go.

## Skills

| Skill | Description |
|-------|-------------|
| [`/rpt-workflow`](./skills/rpt-workflow/SKILL.md) | Research → Plan → Task workflow for medium-to-large requirements in an existing codebase. Externalizes research, design, task execution, and verification into resumable state files. After you confirm the plan, tasks execute autonomously via subagents with a final verification. |
| [`/setup-xufeisofly-skills`](./skills/setup-xufeisofly-skills/SKILL.md) | One-time per-repo setup for the skills above. Records the RPT state-file location and verification commands into `docs/agents/rpt-workflow.md` and a `## Agent skills` block in `CLAUDE.md`/`AGENTS.md`. |

## Local development

Symlink every skill in this repo into `~/.claude/skills` so the local Claude CLI picks them up:

```bash
./scripts/link-skills.sh
```

List all skills in the repo:

```bash
./scripts/list-skills.sh
```

## Layout

```
skills/
  rpt-workflow/             # the RPT workflow skill
    SKILL.md
    REFERENCE.md            # state-file templates
  setup-xufeisofly-skills/  # per-repo setup skill
    SKILL.md
    rpt-workflow-config.md  # seed template written into docs/agents/
.claude-plugin/plugin.json  # manifest the npx installer reads
```

## License

MIT — see [LICENSE](./LICENSE).
