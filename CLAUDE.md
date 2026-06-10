Skills live under `skills/`, one directory per skill, each with a `SKILL.md`.

Every shippable skill must:

- have a directory under `skills/<skill-name>/` containing `SKILL.md`
- be listed in `.claude-plugin/plugin.json` (this is what the `npx skills` installer reads)
- be referenced in the top-level `README.md`, with the skill name linked to its `SKILL.md`

Bundled reference material for a skill (templates, extra docs) lives alongside its `SKILL.md` in the same directory and is linked from `SKILL.md`.
