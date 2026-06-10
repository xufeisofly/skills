# RPT Workflow Reference

## Document Responsibilities

- `overview.md` = status. Only records where the workflow is: current phase, active task, overall progress, and links to the other files. Update it after each task is completed.
- `research.md` = understanding. Only records code/information research, data flow, existing conventions, constraints, open questions, and key assumptions.
- `plan.md` = definition. Only records the design, why the work is split that way, how tasks hand off to each other, and how the whole requirement will be accepted. Update it when tasks are added, removed, or changed.
- `tasks/*.md` = individual task details. Each task has Plan / Implementation / Verify sections.

Task lists appear in both `plan.md` and `overview.md`, but for different reasons: `plan.md` contains the full description, rationale, and dependencies; `overview.md` only contains number / name / status / link.

## Checkpoint Summary

| Moment | Default behavior |
|---|---|
| Research complete | Present open questions and wait for user answers on key issues |
| Plan complete | Deliver `plan.md`, accept review, and revise until the user confirms — this is the final user checkpoint |
| Task loop | Autonomous: per task, write Plan → dispatch subagent to Implement → Verify; update `overview.md`; no user review between tasks |
| All tasks complete | Run final verification (overall acceptance criteria + standard build/test), record in `overview.md`, report to user |

## overview.md Template

```markdown
# <Requirement Name> — Overview

## Requirement
<One-sentence goal + background/motivation>

## Current Status
- Phase: Research / Plan / Task / Done
- Active: task-0X <name>

## Files
- Research: ./research.md
- Plan: ./plan.md
- Tasks: ./tasks/

## Task Progress
| # | Name | Status | File |
|---|------|--------|------|
| 01 | ... | Done / In Progress / Todo / Blocked | ./tasks/task-01-xxx.md |

## Final Verification
<Filled after all tasks: overall acceptance checks run, output summary, pass/fail>
```

## research.md Template

```markdown
# <Requirement Name> — Research

## Relevant Code and Data Flow
<Key files, call chain, existing conventions>

## Reusable Existing Implementations
<Similar logic that can be referenced or reused>

## Constraints
<Available libraries / architecture style / naming and directory conventions / boundaries that must not move>

## Open Questions / Key Assumptions
1. <Ambiguous requirement point that needs user approval>
2. <Key assumption that would affect the plan if wrong>
```

## plan.md Template

```markdown
# <Requirement Name> — Plan

## Chosen Design
<Final design>

## Alternatives Considered
<Alternative + why it was rejected>

## File Changes
<Which files will be created/modified, and in what order>

## Overall Acceptance Criteria
<Observable result when the whole requirement is complete; used for final verification>

## Task Breakdown
### task-01 <name>
- Scope: ...
- Depends on: <which prior task output, or "none">
- Produces: <what later task consumes>

## Review Notes
<User review feedback + changes made in response>
```

## tasks/task-0X-<slug>.md Template

```markdown
# task-0X <name>

## Plan (written before dispatching the subagent)
- Approach: ...
- Affected files: ...
- Acceptance criteria / passing conditions: <specific executable checks>

## Implementation (filled from subagent output)
<What changed, what logic was added, and which files changed>

## Verify (run the precommitted checks)
- Command run: ...
- Actual output: ...
- Conclusion: pass / fail (+ reason)
```
