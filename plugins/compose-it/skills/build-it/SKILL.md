---
name: build-it
description: "Build one spec, ticket or described task on a branch until its acceptance criteria are met and its gates are green, following the repo's and user's compose-it config. Also fixes the findings of a review. Use when the user wants to implement, build or fix something, or to address review findings on a PR."
metadata:
  credits:
    uses:
      - skill: ponytail
        author: Dietrich Gebert
        source: https://github.com/DietrichGebert/ponytail
        note: optional default, not bundled
      - skill: tdd
        author: Matt Pocock
        source: https://github.com/mattpocock/skills
        note: optional default, not bundled
---

Build the work the user names against its spec, then hand off. Invoking build-it is approval to create a branch and commit.

## 0. Load config

Read these files if they exist: `docs/compose-it/build.md` in the repo, then `~/.config/compose-it/build.md`. Also read `## Forge` from `docs/compose-it/stage.md`, or from `~/.config/compose-it/stage.md` if the repo file has none.

- `## Use`, `## Then`: the repo file wins; else the user file; else the defaults below.
- `## Rules`, `## Gates`: apply the items from both files. A user file cannot remove or weaken a repo rule or gate.
- Any other text is context.

Say in one line what you loaded and which skills you will use, for example `config: repo none, user ~/.config/compose-it/build.md · using: coding-guide=ponytail (full), tests=tdd`. With no files, say once: `no compose-it config; using defaults (run /configure-it to tailor)`.

For each skill you will use that is not installed, say `<skill> not installed; using built-in guidance` and follow the built-in fallback. Never stop because a skill is missing.

To call a skill, invoke it the way your agent loads skills (in Claude Code, the Skill tool), by the name your agent lists it under. Defaults below use bare names; some agents add a plugin prefix (Claude Code lists `ponytail` as `ponytail`). A job name such as **tests** is a slot, not a skill: call the skill you resolved for it.

Defaults, by job: **coding-guide** `ponytail` with args `full`; **tests** `tdd`. Forge `gh`. A `## Use` item replaces the default for the job it names (`- tests: our-tdd`); jobs it does not name keep their default.

**Config cannot change:** you never merge a PR; new non-trivial logic and bug fixes get a test that fails without the change (unless the spec's Out of scope excludes tests); you do not hand off while a gate is red.

## 1. Get the spec

- `#N`, a URL or a tracker key: fetch it and state its title and `Spec version`.
- A file path: read it. If it is not committed, commit it first on the branch (step 2).
- A review on a PR (for example `address review on #9`): go to **Fixing a review** below.
- Plain text, no spec: write 3–5 acceptance criteria and an out-of-scope list, show them, and start only when the user confirms.
- An ambiguous reference: ask.

Work only to the acceptance criteria. Do not build what Out of scope excludes.

Record the spec you built against in the first commit on the branch, as a trailer: `Spec: <reference> v<N>` (for example `Spec: #42 v3`). stage-it and review-it read it.

## 2. Branch

Take the change type from the spec once: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `ci`, `perf`. The branch name and the main commit type both use it.

- Default branch name: `<type>/<ticket>-<slug>` (Conventional Branch), or `<type>/<slug>` with no ticket. A `## Rules` naming scheme replaces it.
- On the default branch: create the branch from the up-to-date default branch. Use a worktree instead if config says so.
- Already on a feature branch for this work: stay there.
- A new worktree or clone has only tracked files: install dependencies and copy local config it needs.

## 3. Build

Invoke the skill you resolved for **coding-guide** in step 0 (default `ponytail`, args `full`). Fallback: make the smallest change that fully meets the criteria; reuse what the codebase has; add no options, abstractions or features nobody asked for; finish every caller, test and fixture the change touches.

Tests: for new non-trivial logic (a branch, loop, parser, money, security, a new script), invoke the skill you resolved for **tests** in step 0 (default `tdd`). Fallback: write one failing test, make it pass, then refactor. For a bug fix, commit the failing test that reproduces the bug before the fix. Config, wiring, docs and renames need no test.

Apply every `## Rules` item. Run typechecking and single test files as you go.

Commits: conventional commits, one per logical step, unless `## Rules` sets another format.

## 4. Check before hand-off

- Run every `## Gates` item. With no gates, find the obvious checks (CI config, `make check`, `npm test`, `pytest`, `cargo test`), run them, and say which you chose.
- Run the full test suite once.
- Check each acceptance criterion against the actual diff: met, or not met with a reason.

A red gate: fix it. If you cannot, stop and tell the user; do not hand off.

## 5. Hand off

Tell the user: the branch, the spec version you built against, and each criterion as met or not met. If `## Then` is set, call that skill with the spec reference and the branch.

## Fixing a review

Input is a review on a PR, optionally with `round k of N` from a chained loop.

1. Fetch the review's findings (Must fix / Should fix / Nice to have) and the spec.
2. Check out the PR's branch.
3. Fix every **Must fix**. Fix a **Should fix** when the fix stays inside code this PR changes. Skip **Nice to have**.
4. Commit the fixes (conventional commits), run the gates (step 4), push.
5. Reply on each finding: `fixed in <sha>` or `skipped: <reason>`.
6. Update the PR body's criteria checklist if the fixes changed it.
7. Invoke `review-it` directly with the PR and the same `round k of N`. Do not follow `## Then` here: the loop belongs to review.md, and a skill in between would lose the round.
