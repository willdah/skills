---
name: configure-it
description: "Set up compose-it for a repo or for yourself: read the repo's existing conventions (CI, test commands, PR templates, CONTRIBUTING, CODEOWNERS, agent instructions, installed skills) and draft the docs/compose-it phase files that plan-it, build-it, stage-it and review-it follow."
disable-model-invocation: true
metadata:
  credits:
    adapted:
      - from: setup-matt-pocock-skills
        author: Matt Pocock
        source: https://github.com/mattpocock/skills
        note: the explore, propose, confirm, write flow
---

Draft the compose-it config for this environment. You only read the repo and write config files: never commit, push, open PRs or merge. Explore first, propose one phase at a time, write only what the user confirms. The format is in this plugin's `docs/config.md`: four optional headings, `## Use`, `## Rules`, `## Gates`, `## Then`, plus `## Forge` in `stage.md`.

## 1. Explore

Read what exists; do not assume:

- Forge: `git remote -v`. GitHub, GitLab, Forgejo or Gitea, other. Is its CLI installed and logged in (`gh`, `glab`, `tea`)?
- Tracker: issues on the forge, or Jira or Linear named in `CONTRIBUTING.md`, the README or agent instructions.
- Gates: CI config (`.github/workflows/`, `.gitlab-ci.yml`, `.forgejo/workflows/`, `.gitea/workflows/`, `Jenkinsfile`), `Makefile`, `package.json` scripts, `pyproject.toml`, `Cargo.toml`, pre-commit hooks. What does CI run on a PR?
- House templates: PR and issue templates.
- Rules: `CONTRIBUTING.md`, `CODEOWNERS`, `SECURITY.md`, `CLAUDE.md`, `AGENTS.md`, branch and commit conventions in git history.
- Existing config: `docs/compose-it/*.md` and `~/.config/compose-it/*.md`.
- Installed skills: which of the defaults (`grilling`, `to-spec`, `to-tickets`, `ponytail`, `tdd`, `ponytail-review`) and which alternatives the user has. Propose an alternative as `- <job>: <skill>`, for example `- tests: our-tdd`.

## 2. Ask where to write

Ask once: **repo** (`docs/compose-it/`, shared with the team, committed) or **user** (`~/.config/compose-it/`, yours, every repo). Recommend repo when the rules are the team's, user for personal habits.

## 3. Propose each phase

Go in order: plan, build, stage, review. For each:

1. Show a draft file built from what you found, with one line per item saying where it came from, for example `- make check  (from .github/workflows/ci.yml)`.
2. Leave out anything that matches the defaults. If nothing differs, say `plan: defaults fit, no file` and move on.
3. Lead with your recommendation so the user can accept in a word. Ask only where the choice really branches, for example a missing default skill with an installed alternative.

Things to propose when you find them:

- `plan.md`: where specs go (forge issues, a tracker, or files); required spec fields.
- `build.md`: the CI commands as `## Gates`; branch and commit conventions that differ from Conventional Branch and conventional commits; rules from `CONTRIBUTING.md` and agent instructions; a worktree rule if the user works with parallel agents.
- `stage.md`: `## Forge` commands when the forge is not GitHub (map each `gh` command the skills use to the forge CLI, including how to mark a PR draft and ready). A house PR template is used automatically, so it needs no line.
- `review.md`: compliance and review rules from `CONTRIBUTING.md`, `SECURITY.md`, `CODEOWNERS`; whether review-it may mark PRs ready; chaining and loop rounds.

## 4. Write

Write only the files the user confirmed, to the chosen place. Do not commit; tell the user which files you wrote and that a repo-level file should go through their normal review.
