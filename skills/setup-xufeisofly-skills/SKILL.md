---
name: setup-xufeisofly-skills
description: Sets up an `## Agent skills` block in CLAUDE.md/AGENTS.md and `docs/agents/rpt-workflow.md` so the RPT workflow skill knows where this repo stores its state files and which build/test/verify commands count as "passing". Run once per repo before first use of `rpt-workflow`, or if that skill appears to be missing context about where to write state files or how to verify a task.
disable-model-invocation: true
---

# Setup xufeisofly's Skills

Scaffold the per-repo configuration that the `rpt-workflow` skill assumes:

- **State-file location** — where RPT writes `overview.md`, `research.md`, `plan.md`, and `tasks/` for each requirement
- **Verification commands** — the build / test / lint commands a task must pass before it is considered done, used in every task's Verify step and in the final verification
- **Doc language** — the language RPT documents are written in (the skill defaults to English)

This is a prompt-driven skill, not a deterministic script. Explore, present what you found, confirm with the user, then write.

## Process

### 1. Explore

Look at the current repo to understand its starting state. Read whatever exists; don't assume:

- `git remote -v` — what repo is this?
- `CLAUDE.md` and `AGENTS.md` at the repo root — does either exist? Is there already an `## Agent skills` section?
- `docs/agents/` — does this skill's prior output (`rpt-workflow.md`) already exist?
- Build config — `package.json` scripts, `Makefile`/`justfile`/`Taskfile`, `Cargo.toml`, `pyproject.toml`, `go.mod`, etc. — to propose the real build/test/lint commands instead of guessing.
- Any existing RPT state directories (e.g. `.rpt/`, `docs/rpt/`) that hint at an established convention.

### 2. Present findings and ask

Summarise what's present and what's missing. Then walk the user through the three decisions **one at a time** — present a section, get the user's answer, then move to the next. Don't dump all three at once.

Assume the user may not know what these terms mean. Each section starts with a short explainer, then shows the choices and the default.

**Section A — State-file location.**

> Explainer: RPT externalizes its progress into files instead of keeping everything in chat. For each requirement it creates a directory holding `overview.md` (status), `research.md` (understanding), `plan.md` (design), and a `tasks/` folder. Pick where those requirement directories should live so the skill stops asking every time.

- **`.rpt/<requirement-slug>/`** (default) — a dedicated top-level folder, gitignored or committed as you prefer
- **`docs/rpt/<requirement-slug>/`** — keep RPT state alongside other project docs
- **Other** — the user names a path convention; record it verbatim

If a state directory convention already exists in the repo, propose that.

**Section B — Verification commands.**

> Explainer: RPT's Task loop only marks a task done when verification passes, and runs a final verification after all tasks. The skill needs to know the exact commands that mean "this repo builds, tests, and lints cleanly" so subagents run the right checks instead of inventing them.

Propose concrete commands discovered in step 1. Confirm or let the user override:

- **Build** — e.g. `npm run build`, `cargo build`, `make build`
- **Test** — e.g. `npm test`, `cargo test`, `pytest`
- **Lint / format** — e.g. `npm run lint`, `cargo clippy`, `ruff check`
- **Any wrapper** — if commands must run through a wrapper (e.g. `nix develop -c <cmd>`, `docker compose run`), record it so every command is quoted with the wrapper.

It's fine to leave a slot empty if the repo has no such command.

**Section C — Doc language.**

> Explainer: RPT documents (`overview.md`, `research.md`, `plan.md`, task files) are written in one language for consistency. The skill defaults to English.

- **English** (default)
- **Other** — the user names the language

### 3. Confirm and edit

Show the user a draft of:

- The `## Agent skills` block to add to whichever of `CLAUDE.md` / `AGENTS.md` is being edited (see step 4 for selection rules)
- The contents of `docs/agents/rpt-workflow.md`

Let them edit before writing.

### 4. Write

**Pick the file to edit:**

- If `CLAUDE.md` exists, edit it.
- Else if `AGENTS.md` exists, edit it.
- If neither exists, ask the user which one to create — don't pick for them.

Never create `AGENTS.md` when `CLAUDE.md` already exists (or vice versa) — always edit the one that's already there.

If an `## Agent skills` block already exists in the chosen file, update its contents in-place rather than appending a duplicate. Don't overwrite user edits to the surrounding sections.

The block:

```markdown
## Agent skills

### RPT workflow

State files live under [one-line summary of the chosen path]. Verification = [one-line summary of the build/test/lint commands]. Docs in [language]. See `docs/agents/rpt-workflow.md`.
```

Then write `docs/agents/rpt-workflow.md` using [rpt-workflow-config.md](./rpt-workflow-config.md) in this skill folder as the seed template, filling in the three decisions.

### 5. Done

Tell the user the setup is complete and that `rpt-workflow` will now read `docs/agents/rpt-workflow.md` for state-file location and verification commands. Mention they can edit that file directly later — re-running this skill is only necessary to restart from scratch.
