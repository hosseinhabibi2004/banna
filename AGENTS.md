# AGENTS.md

This repo hosts the banna workflow skills. One folder per skill under `skills/`, each in
the [Agent Skills](https://agentskills.io) format: `SKILL.md` with frontmatter (`name`,
`description`, `when_to_use`, `argument-hint`, `category`, `tags`, `allowed-tools`)
plus any templates and scripts the skill needs. `agents/openai.yaml` carries the
OpenAI-interface metadata per skill.

Conventions for changes here:

- Every skill is prefixed `banna-`; cross-references between skills use the
  `/banna-<name>` invocation form.
- Skills are agent-neutral. A skill never names one vendor's env var, file, or models
  as a hard dependency: use `${SKILL_DIR}` with the resolution rule at first use, treat
  `AGENTS.md` as the instruction-file default (`CLAUDE.md` is the Claude Code import
  path), keep the project launch config at `.agents/launch.json` (never a per-agent
  path), and source every model id from the project's `catalog`.
- Skill prose is direct and imperative, numbered steps, no marketing voice. State the
  rule, then the exception.
- Skills never guess defaults: anything a skill needs from the project comes from
  `.banna/config.yaml`, and a missing or invalid config stops the skill.
- The config template lives inside `skills/banna-init/config-template.yaml` — skills
  install as folders, so anything a target project needs must ship inside a skill.
