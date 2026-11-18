---
name: banna-init
description: Scaffold banna into the current project — write .banna/config.yaml with this repo's real areas and test commands, create the plans/ and handoffs/ folders with their archive subfolders, and print what the next step (/banna-draft) will do. Run once per project before any other banna skill.
when_to_use: Run directly with /banna-init in a fresh project. Every other banna skill reads .banna/config.yaml and stops if it is missing — this skill creates it.
argument-hint: [optional: one line describing the project, used to pre-fill areas]
category: Workflow
tags: [workflow, setup]
allowed-tools: Read, Grep, Glob, Write, Bash(mkdir:*), Bash(ls:*), Bash(find:*), Bash(date:*)
---

You are scaffolding `.banna/` into the current project. You write exactly two kinds of
things — the config file and the folder skeleton — and nothing else. You never touch
source code. You ask before writing anything: the whole point is that the config
describes **this** repo, not a guess about it.

The config template lives next to this skill at `${SKILL_DIR}/config-template.yaml` —
read it first (`$SKILL_DIR` is this skill's folder, however the running agent names it:
`CLAUDE_SKILL_DIR` in Claude Code, `ZCODE_SKILL_DIR` in ZCode; an agent that exports no
such variable resolves it to the directory containing this SKILL.md). It carries the
neutral defaults (difficulty, stages, selection, freeze and archive rules). Your job is
to replace its placeholder areas, test commands, and model catalog with this repo's
reality.

**1. Look before asking.**
Walk the repo: top-level folders, manifests (`package.json`, `pyproject.toml`, `go.mod`,
`Cargo.toml`), existing test setups, existing `docs/` or `AGENTS.md`. From
that, form a draft picture of the areas — one area per deployable or independently
testable slice (`api/`, `web/`, `admin/`, a package, the root) — and one test command per
area. If a slice has no tests yet, its command is still required; propose the natural one
for its stack.

**2. Ask what the walk could not settle.**
One question at a time, each with your proposed answer, until every area has a name,
its `code`/`docs` paths, and its test command. Also settle the model catalog: which agent
runs this repo (Claude Code, Codex, OpenCode, ZCode, …), and the two or three models it
offers, strongest to cheapest — the template's catalog ids are Claude's by default and
must match what the agent actually serves. Do not ask about anything else the template
already defaults sensibly (stages, freeze, archive) — those stay as scaffolded
and can be edited later. If the target project is a single-package repo, one `root` area
is a complete answer.

**3. Write.**
- `.banna/config.yaml` from the template, with the real `areas:` and `tests.commands:`
  filled in and the placeholder comment lines removed where they were replaced.
- `.banna/plans/` and `.banna/handoffs/`, each with an `archive/` subfolder holding a
  `.gitkeep`.

Report the config path, the areas table (name → code → docs → test command), and the two
folders created. Then print the next step verbatim:

> Run `/banna-draft <what you want to build>` to write your first plan.
> banna-draft will interview you, draft the plan, and send it to a cold review
> before you approve it for banna-build.
