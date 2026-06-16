---
name: rpt-workflow
description: Uses the RPT (Research -> Plan -> Task) workflow to develop medium-to-large requirements in an existing codebase, externalizing research, design, task execution, and verification into resumable state files. After the user confirms the plan and chooses an execution mode — run all tasks autonomously, or pause for review after each task — tasks execute via subagents, ending with a final verification and a behavior-preserving code-simplification pass. Use when the user asks to implement a new requirement, feature, multi-file change, architecture or data-flow change, or ambiguous work that needs controlled review; skip for copy tweaks, small single-file edits, obvious bugs, or pure renames.
---

# RPT Workflow

RPT splits development into Research, Plan, and Task phases. Research and Plan each end in a user checkpoint. After the plan is confirmed, the user chooses how the Task phase runs — fully autonomous, or pausing for review after each task. State lives in files, not only in chat context. Templates are in [REFERENCE.md](REFERENCE.md).

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

2. **Plan (user checkpoint).** Pressure-test the design with the `grill-me` skill, then write `plan.md`: chosen design, alternatives and rejection reasons, file-change order, task breakdown with dependencies and handoffs, and overall acceptance criteria. Because the user may choose to run the whole Task phase autonomously, the plan must be executable without further input: every task needs concrete scope and executable acceptance criteria. Wait for user confirmation before entering Task.

3. **Choose execution mode, then run the Task loop.** Right after the plan is confirmed and before executing any task, ask the user how the Task phase should run, and record the choice in `overview.md` so it survives a resume:
   - **Autonomous**: execute all tasks straight through, without stopping between tasks. Best when the plan is solid and the user wants to step away.
   - **Step-by-step**: after each task's verification passes, stop and present what changed plus the verification result, then wait for the user's go-ahead before starting the next task. Best when the user wants to inspect progress or suspects the plan may need adjusting along the way.

   Ask once, up front — don't re-ask before every task. Then execute tasks one at a time, in dependency order. For each task:
   - **Plan**: write the task file's Plan section — approach, affected files, and executable acceptance criteria derived from `plan.md`.
   - **Implement**: dispatch a subagent to execute the task plan. The main agent never writes implementation code itself — it owns orchestration, boundary control, review of subagent output, and integration. Give the subagent the task file plus the relevant slices of `plan.md` and `research.md`. Record substantive changes and changed files in the Implementation section.
   - **Verify**: run exactly the checks promised in the task Plan; record commands, key output, and pass/fail. On failure, fix via a subagent and re-verify — a task is complete only when verification passes.
   - Update task status in `overview.md`. In **autonomous** mode, continue directly to the next task. In **step-by-step** mode, present the completed task and wait for the user's go-ahead before the next one.

4. **Final verify (after all tasks).** Run the overall acceptance criteria from `plan.md` plus the project's standard build/test checks. Record the results in `overview.md`.

5. **Simplify (after final verify passes).** Dispatch the `code-simplifier` subagent over the code changed in this workflow (the union of changed files recorded in the task files' Implementation sections). It must preserve behavior — clarity, consistency, and dead-code cleanups only, no functional changes. If it changed anything, re-run the final verification from step 4; simplification is complete only when verification still passes — if re-verification fails, revert the simplification edits (the verified pre-simplification state wins) and note this in `overview.md`. If it changed nothing, record "no changes". Record the simplification summary and re-verification result in `overview.md`, then report to the user: task-by-task summary, final verification output, simplification summary, and any deviations from the plan.

## Escalation

Whichever execution mode the user picked, real problems always surface to the user:

- A task reveals a design problem: stop the loop, record the issue in `plan.md`, return to Plan, and re-confirm with the user — a confirmed plan that turned out wrong needs a new confirmation.
- Verification keeps failing (about 3 attempts) or a subagent is blocked: record the state in the task file and `overview.md`, then escalate to the user.
- A required capability is missing and cannot be installed: record the blocker and ask the user whether downgraded execution is allowed.
- Tasks are added, split, or removed during execution without changing the design: update handoffs in `plan.md` and status in `overview.md`, reassess downstream dependencies, and continue — no user review needed.
