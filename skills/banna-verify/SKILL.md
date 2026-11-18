---
name: banna-verify
description: Bring a project's app up on this machine and exercise the change on the real stack — start the servers (ports and data checked first, nothing else disturbed), sign in, click through what changed or smoke-check the API, report what passed and what was fixed, then clean up. Use when asked to run, start, try, screenshot or demo the app, or when /banna-build needs the real stack driven at step 9 before a change is called done. Works for any stack; it learns how the project runs from the project itself.
when_to_use: Invoked by /banna-build (step 9), or run directly with /banna-verify to see a change working before calling it done.
argument-hint: [what changed: an area, a feature, or a plan id to verify; empty = the latest plan]
category: Workflow
tags: [workflow, verification]
---

# Verify on the real stack

A green test suite is not "done" for a change someone will look at or call: tests assert
values, and rendering, wiring, and configuration fail where assertions are blind. This
skill gets the real thing running, uses it, and reports what it found. It assumes nothing
about the stack: it reads how **this** project runs and leaves everything it did not
start alone.

Target: $ARGUMENTS — what changed. If empty, verify the affected workflow of the latest
plan under `.banna/plans/`.

## 1. Learn how this project runs

Look, in this order, and use the first that answers:

1. `.agents/launch.json` — named server configurations, with ports. It is
   agent-neutral: any agent running these skills reads and writes this one file.
2. The project's instruction files: `AGENTS.md` / `CLAUDE.md`, the area's `docs/README.md`,
   the root `README.md` — a "run it" section.
3. The manifests: `package.json` scripts, `Makefile`, `docker-compose*.yaml`,
   `manage.py`, `pyproject.toml`, `Cargo.toml`, `go.mod`.

Write down: which processes make up the app (API, web, admin, worker, database), the
command and port of each, which environment files they read, whether a database needs
creating, migrating or seeding, and how a person signs in. If the config declares several
areas, run every one the change touches or depends on. If nothing documents it, say so,
work it out from the manifests, and offer to add a `.agents/launch.json` entry once it
works.

## 2. Check ports and data before starting anything

```bash
${SKILL_DIR}/scripts/port-owner.sh 8000 3000   # free, or who holds it
```

`$SKILL_DIR` is this skill's folder, however the running agent names it:
`CLAUDE_SKILL_DIR` in Claude Code, `ZCODE_SKILL_DIR` in ZCode; an agent that exports no
such variable resolves it to the directory containing this SKILL.md.

- **A port is taken by something you did not start:** it is someone's other project.
  Never kill it and never bind over it (a server on `127.0.0.1:P` shadows one on
  `*:P` for local traffic). Pick another free port and pass it through the project's
  own mechanism: a flag, or a **gitignored** override file (`.env.local`). Keep
  everything that names the port consistent: base URLs, CORS and CSRF origins, proxy
  targets, OAuth callbacks. Say which ports you used.
- **Data:** never point a seeding or destructive run at the person's real database.
  Use a throwaway one (a temporary SQLite file, or a scratch database you create and
  drop) unless they asked to use their own data. Migrations on their local dev database
  are fine when the app cannot start without them; say so.
- **Environment files:** create a missing one from `.env.example` into the gitignored
  file; never overwrite or print an existing one.

## 3. Start it

Prefer starting the processes `.agents/launch.json` names (create the file with an entry
if missing: a name, the command, the port). Otherwise start each process in the background with its
output going to a log file in the session scratch directory, never a bare `&` in the
foreground shell. Then wait until each one answers:

```bash
${SKILL_DIR}/scripts/wait-for.sh http://localhost:8010/health 60
```

Check the logs for tracebacks even when the port answers. A dev server that "started"
and a request that works are different things. Missing a runtime or tool you do not
have (a database server, a browser binary)? Say exactly which, and stop; do not install
system software unasked.

## 4. Verify what changed

Sign in with the project's test or seed accounts — its seed command, fixtures, or
example config. A local-dev account you create is fine; its password is recorded in a
gitignored file, not repeated in chat.

**Something a person looks at** (a page, a form, an editor): open it in the agent's
built-in browser if it has one; otherwise screenshot to a file and read the image back,
or drive Playwright when the project already uses it. Click through **the screens the
change touched** — the empty, error, and success states, plus the languages, themes,
and widths the project supports. Read the browser console and the failed network
requests.

**No screen** (an API, a library, a job): drive it with real requests against the live
process — the sign-in handshake the project uses, a happy path through each touched
endpoint, and the refusals (unauthenticated, forbidden, invalid input, not found). A
smoke script in the scratch directory is fine; it is not a test to commit unless asked.

What to look for, because tests miss it: a layout that breaks at one width, dates or
numbers formatted for the wrong locale, text in the wrong language or direction, a
missing loading or empty state, a request the browser blocks (CORS, cookies, mixed
content), a stale generated client, a migration or seed that never ran.

## 5. Fix, report, clean up

- Fix what the run finds before reporting, each fix as its own change; a finding that
  changes the design is a divergence to raise, not to improvise.
- Report in this shape — /banna-build logs it at step 9:
  - **Ran:** processes, ports, accounts (by name)
  - **Covered:** screens, requests, states, locales
  - **Found and fixed:** each issue with its fix
  - **Skipped:** what you could not run and why — a skipped check is said plainly,
    never omitted
  - **Verdict:** pass or fail
- Leave servers running only if the person asked to try the app themselves; then give
  the URLs and how to stop them. Otherwise stop every process you started, drop scratch
  databases, and delete temporary files and uploaded media you created. Never stop or
  remove anything that was already there.
