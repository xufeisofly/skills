---
name: rpt-workflow
description: Uses the RPT (Research -> Plan -> Task) workflow to develop medium-to-large requirements in an existing codebase, externalizing research, design, task execution, and verification into resumable state files. After the user confirms the plan, all tasks execute autonomously via subagents with a final verification at the end. Use when the user asks to implement a new requirement, feature, multi-file change, architecture or data-flow change, or ambiguous work that needs controlled review; skip for copy tweaks, small single-file edits, obvious bugs, or pure renames.
---

# RPT Workflow

RPT splits development into Research, Plan, and Task phases. Research and Plan each end in a user checkpoint; once the plan is confirmed, the Task phase runs to completion without further user review. State lives in files, not only in chat context. Templates are in [REFERENCE.md](REFERENCE.md).

## Trigger Check

- Use: new requirements/features, multi-file changes, architecture or data-flow changes, ambiguous requirements, high-cost mistakes.
- Skip: copy changes, small single-file edits, obvious bugs, pure renames.
- If uncertain: ask the user whether to use the full RPT workflow.

## State Files

If `docs/agents/rpt-workflow.md` exists (written by `/setup-xufeisofly-skills`), use its state-file location and verification commands instead of asking. Otherwise, confirm the requirement directory with the user.

Then create:

- `overview.md`: the single source of progress truth; records phase, active task, file links, and task status.
- `research.md`: read-only research, data flow, conventions, constraints, and open questions.
- `plan.md`: chosen design, alternatives, file changes, acceptance criteria, task breakdown, task handoffs, review notes.
- `tasks/task-0X-<slug>.md`: one task's Plan / Implementation / Verify sections.

All RPT documents must be written in English.

## Workflow

1. **Research (user checkpoint).** Read code, tests, config, and similar implementations. Only write `overview.md` and `research.md`. End with open questions / key assumptions, then wait for user answers before Plan.

2. **Plan (user checkpoint — the last one).** Pressure-test the design with the `grill-me` skill, then write `plan.md`: chosen design, alternatives and rejection reasons, file-change order, task breakdown with dependencies and handoffs, and overall acceptance criteria. Because no user review happens after this point, the plan must be executable without further input: every task needs concrete scope and executable acceptance criteria. Wait for user confirmation before entering Task.

3. **Task loop (autonomous — no user review).** Execute tasks one at a time, in dependency order, without stopping for user confirmation between steps or between tasks. For each task:
   - **Plan**: write the task file's Plan section — approach, affected files, and executable acceptance criteria derived from `plan.md`.
   - **Implement**: dispatch a subagent to execute the task plan. The main agent never writes implementation code itself — it owns orchestration, boundary control, review of subagent output, and integration. Give the subagent the task file plus the relevant slices of `plan.md` and `research.md`. Record substantive changes and changed files in the Implementation section.
   - **Verify**: run exactly the checks promised in the task Plan; record commands, key output, and pass/fail. On failure, fix via a subagent and re-verify — a task is complete only when verification passes.
   - Update task status in `overview.md`, then continue directly to the next task.

4. **Final verify (after all tasks).** Run the overall acceptance criteria from `plan.md` plus the project's standard build/test checks. Record the results in `overview.md`, then report to the user: task-by-task summary, final verification output, and any deviations from the plan.

## Escalation

The task loop is autonomous, but real problems still surface to the user:

- A task reveals a design problem: stop the loop, record the issue in `plan.md`, return to Plan, and re-confirm with the user — a confirmed plan that turned out wrong needs a new confirmation.
- Verification keeps failing (about 3 attempts) or a subagent is blocked: record the state in the task file and `overview.md`, then escalate to the user.
- A required capability is missing and cannot be installed: record the blocker and ask the user whether downgraded execution is allowed.
- Tasks are added, split, or removed during execution without changing the design: update handoffs in `plan.md` and status in `overview.md`, reassess downstream dependencies, and continue — no user review needed.
