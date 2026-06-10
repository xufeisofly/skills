# RPT Workflow — Per-repo Config

This file is read by the `rpt-workflow` skill. Edit it directly to change conventions; re-run `/setup-xufeisofly-skills` only to restart from scratch.

## State-file location

RPT state for each requirement lives under:

```
<FILL IN — e.g. .rpt/<requirement-slug>/>
```

Each requirement directory contains:

- `overview.md` — status / progress source of truth
- `research.md` — research, data flow, constraints, open questions
- `plan.md` — chosen design, alternatives, file changes, task breakdown, acceptance criteria
- `tasks/task-0X-<slug>.md` — per-task Plan / Implementation / Verify

## Verification commands

A task is done only when these pass. The final verification runs them after all tasks.

| Check | Command |
|-------|---------|
| Build | `<FILL IN or leave blank>` |
| Test  | `<FILL IN or leave blank>` |
| Lint / format | `<FILL IN or leave blank>` |

Wrapper (prefix every command with this, if any): `<FILL IN or "none">`

## Doc language

RPT documents are written in: `<FILL IN — default English>`
